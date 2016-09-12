# tinydsm

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
