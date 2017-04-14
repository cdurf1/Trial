
# Versioning Object Store

The Versioning Object Store (VOS) is responsible for providing and maintaining a persistent object store that supports byte-granular access and versioning. It maintains its own metadata in persistent memory and may store data either in persistent memory or on block storage, depending on available storage and performance requirements. It must provide this functionality with minimum overhead so that performance can approach the theoretical performance of the underlying hardware as closely as possible, both with respect to latency and bandwidth. Its internal data structures, in both persistent and non-persistent memory, must also support the highest levels of concurrency so that throughput scales over the cores of modern processor architectures. Finally, and critically, it must validate the integrity of all persisted object data to eliminate the possibility of silent data corruption, both in normal operation and under all possible recoverable failures.

This section provides the details for achieving the aforementioned design goals in building a versioning object store for DAOS-M.

This document contains the following sections:

- <a href="#71">VOS Concepts</a>
    - <a href="#711">VOS Indexes</a>
    - <a href="#712">Object Listing</a>
-  <a href="#72">Key Value Stores</a>
    - <a href="#721">Operations Supported with Key Value Store</a>
    - <a href="#723">Key in VOS KV Stores</a>
    - <a href="#724">Internal Data Structures</a>
- <a href="#73">Byte Arrays</a>
- <a href="#74">Document Stores</a>
- <a href="#75">Epoch Based Operations</a>
    - <a href="#751">VOS Discard</a>
    - <a href="#752">VOS Aggregate</a>
    - <a href="#753">VOS Flush</a>
- <a href="#76">VOS over NVM Library</a>
    - <a href="#761">Root Object</a>
- <a href="#77">Layout for Index Tables</a>
- <a href="#78">Transactions and Recovery</a>
    - <a href="#781">Discussions on Transaction Model</a>
- <a href="#79">VOS Checksum Management</a>
- <a href="#80">Metadata Overhead</a>
   
<a id="71"></a>
## VOS Concepts

The versioning object store provides object storage local to a storage node by initializing a VOS pool (vpool) as one shard of a DAOS pool. A vpool can hold objects for multiple object address spaces called containers. Each vpool is given a unique UID on creation, which is different from the UID of the DAOS pool. The VOS also maintains and provides a way to extract statistics like total space, available space, and number of objects present in a vpool.

The primary purpose of the VOS is to capture and log object updates in arbitrary time order and integrate these into an ordered epoch history that can be traversed efficiently on demand. This provides a major scalability improvement for parallel I/O by correctly ordering conflicting updates without requiring them to be serialized in time. For example, if two application processes agree how to resolve a conflict on a given update, they may write their updates independently with the assurance that they will be resolved in correct order at the VOS.

The VOS also allows all object updates associated with a given epoch and process group to be discarded. This functionality ensures that when a DAOS transaction must be aborted, all associated updates are discarded before the epoch is committed for that process group and becomes immutable. This ensures that distributed updates are atomic – i.e. when a commit completes, either all updates have been applied or been discarded.

Finally, the VOS may aggregate the epoch history of objects in order to reclaim space used by inaccessible data and to speed access by simplifying indices. For example, when an array object is “punched” from 0 to infinity in a given epoch, all data updated after the latest snapshot before this epoch, becomes inaccessible once the container is closed.

Internally, the VOS maintains an index of container UUIDs that references each container stored in a particular pool. The container itself contains three indices. The first is an index of objects used to map object IDs to object metadata efficiently when servicing I/O requests. The second index enumerates all object updates by epoch for efficient discard on abort and epoch aggregation. The third index maps container handle cookies, which identify updates from a process group to object IDs within that container. The container handle cookie is used for identifying object IOs associated with a process group and primarily would be used for aborting a transaction associated with a process group from DAOS-M. 

The <a href="#7a">figure</a> below shows interactions between the different indexes used inside a VOS pool. VOS objects are not created explicitly, but are created on first write by creating the object metadata and inserting a reference to it in the owning container’s object index. All object updates log the data for each update, which may be a new key value, an array extent or a multilevel KV object. References to these updates are inserted into the object metadata index, the container’s epoch index and also the container cookie index. Note that “punch” of an extent of an array object and of a key in a key value object are also logged as zeroed extents and negative entries respectively, rather than causing relevant array extents or key values to be discarded. This ensures that the full version history of objects remain accessible.

<a id="7a"></a>
![HLD_Graphics/Fig_008.png](HLD_Graphics/Fig_008.png " Interactions between container index, object index, epoch index and cookie within one VOS pool (vpool)")

When performing lookup on a KV object, the object index is traversed to find the index node with the highest epoch number less than or equal to the requested epoch (near-epoch) that matches the key. If a value or negative entry is found, it is returned. Otherwise a “miss" is returned meaning that this key has never been updated in this VOS. This ensures that the most recent value in the epoch history of the KV is returned irrespective of the time-order in which they were integrated, and that all updates after the requested epoch are ignored.

Similarly, when reading an array object, its index is traversed to create a gather descriptor that collects all object extent fragments in the requested extent with the highest epoch number less than or equal to the requested epoch. Entries in the gather descriptor either reference an extent containing data, a punched extent that the requestor can interpret as all zeroes, or a “miss”, meaning that this VOS has received no updates in this extent.  Again, this ensures that the most recent data in the epoch history of the array is returned for all offsets in the requested extent, irrespective of the time-order in which they were written, and that all updates after the requested epoch are ignored.

A “miss” is distinct from a punched extent in an array object or a punched KV entry in a KV to support storage tiering at higher layers in the stack. This allows the VOS to be used as a cache since it indicates that the VOS has no history at this extent or key, and therefore data must be obtained from the cache’s backing storage tier.

<a id="711"></a>

### VOS Indexes

Design of the object index and epoch index tables are similar to the container index table with the object ID and epoch number as keys respectively. The value of the object index table points to the location of the respective object structure. The value in case of the epoch index points to the object ID updated in that epoch. 

Objects can be either a key-value index data structure, byte-array index data structure, or a document store. The type of object is identified from the object ID. A document store supports creation of a KV index with two-levels of keys, wherein the value can be either an atomic value or a byte-array value. Document stores is VOS is discussed in the previous <a href="#74">section</a>.

VOS also maintains an index for container handle cookies. These cookies are created by DAOS-M to uniquely identify I/O from different process groups accessing container objects. The cookie index is shown in the <a href="#6a">figure</a> above. The primary motivation to maintain this information in VOS, is to allow discarding of object IOs for each process group. The cookie index table, helps to search and locate objects modified in a certain epoch by a specific cookie. This index can be a simple hash table where the key to the hash table is the cookie and the value is an epoch index table. This allows to fetch a list of all objects which were updated in an epoch associated with a cookie, which can be used while discarding. A hash table is chosen as the data structure to provide fast (1) lookups. But if the number of cookies are huge, to scale better the same table can be constructed with a two-level b+ tree where the cookie will be the key in first level whose value points to a tree with {object ID, epoch} as the key and the location of the object as its value. The trees at both levels, in this case, would resemble a B+ tree used in KV, as discussed in the previous <a href="#72">section</a>. 

<a id="712"></a>

### Object Listing

<a id="72"></a>

##Key Value Stores

<a id="721"></a>

### Operations Supported with Key Value Store

<a id="723"></a>

### Key in VOS KV Stores

<a id="724"></a>

### Internal Data Structures

<a id="73"></a>

## Byte Arrays

<a id="74"></a>

## Document Stores

<a id="751"></a>

## Epoch Based Operations

<a id="751"></a>

### VOS Discard

<a id="752"></a>

### VOS Aggregate

<a id="753"></a>

### VOS Flush

<a id="76"></a>

## VOS over NVM Library

<a id="761"></a>

### Root Object

<a id="77"></a>

## Layout for Index Tables

<a id="78"></a>

## Transactions and Recovery

<a id="781"></a>

### Discussions on Transaction Model

<a id="79"></a>

## VOS Checksum Management

<a id="80"></a>

## Metadata Overhead