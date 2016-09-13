# tinydsm

## Problem

In global memory systems (such as distributed shared memory or global address 
approaches), the location of different global data is typically maintained 
locally by some software translation mechanism.  I.e., if some global data is 
referenced, the local host has to lookup the location of the data, even if the
data is stored locally.  This adds significant overhead to local data lookups.  


## Solution

An approach to reducing the overhead of locally accessed data would be to 
rewrite the instructions in the code that does the access to just do a load 
from memory if we can determine that the data is stored locally. If the data are
moved (with page granularity), the page can be invalidated and then the code 
rewritten in place again to do the software lookup of the data’s remote 
location. Binary instrumentation coupled with code JITing can be used for 
rewriting code at runtime.

## Measurables

a) Overhead comparsion between a lookup at each memory access vs a direct 
LOAD or STORE for locally available data.
b) Page migration strategy
 b.1) Lazy : Invalidate the page and let the accesses go through a segfault handler
 b.2) Eager : Directly patch the accesses to do a lookup 

## Minimal Viable Implementation

a) Two node setup with one node running application and with all the allocated 
memory in the other node. So for each initial memory access the application 
needs to take a segfault, cross the network to fetch data from the other node. 
  a.1) After fetching and mapping data the access with a lookup which will 
  delegate to the actual memory LOAD or STORE. 
  a.2) Leave memory LOAD or STORE as it is.

b) Three node setup with the memory is allocated in the second node with
application running in the first node. After all the memory has been mapped in
the first node then do a full migration of allocated memory to the third node.
Repopulate memory using
  b.1) Using lazy strategy by taking a segfault and then rewriting with either 
  a lookup or direct LOAD or STORE
  b.2) Using eager strategy by eagerly patching all the accesses to either a 
  lookup of direct LOAD or STORE

## Design

* Page granularity
* Single owner
* Single writer/ multiple readers
* Page meta data -> address, ip, readers, owner, lock
* Allocate 
   Local -> Remote -> Local allocate -> Local pointer
* Read
   Load -> seg fault -> fetch
* Write
   Store -> seg fault -> fetch + update + transfer ownership 
