# IPNS Blockchain

[InterPlanetary File System (IPFS)](https://ipfs.io/) is a content-addressable distributed file system that guarantees the fixity of the contents of a file identified by its cryptographic hash.
Files are resolved lazily over a peer-to-peer network.
However, content-addressable references are immutable in nature, hence, not practical in every application.
For example, if an HTML web page embeds an image using its reference, the reference needs to be updated each time the image is updated, otherwise the web page will still reference the old version of the image.
If the same image is included in many web pages, all of them needs to be updated, hence their own hashes will be changed as well.
This has a cascading effect and kills the primary purposes of including an object by reference rather than by value to achieve the separation of concerns and reuse.

To solve this issue IPFS uses InterPlanetary Naming System (IPNS) that provides mapping from a human-readable URI to its corresponding current IPFS hash.
The owner of a domain name can update mappings of all the URIs under that domain by signing the request with his/her private key.
IPNS can be implemented in many ways, but its current implementation uses [Distributed Hash Table (DHT)](https://en.wikipedia.org/wiki/Distributed_hash_table).
As a consequence, only the most recent mapping of each URI to its corresponding hash is available for resolution, forgetting any historical mappings.
This is not good from the archival perspective as the previous versions of a file might still exist in the IPFS store, but but their corresponding URI mappings are lost.

Traditional web archives might still have some historical observations where an older version of a file can be retrieved using a given URI, but such records will be outside the IPFS system and the history will be potentially sparse rather than transactional.

We can solve these issues by using [Blockchain](https://en.wikipedia.org/wiki/Blockchain) for IPNS records.
By doing so, IPFS can behave like a transactional archival engine while keeping all the historical "URI -> Hash" mappings in a public Blockchain.
Resolving a URI using IPNS Blockchain should return the current mapping, while resolving a URI with a Datetime should return the mapping as it existed at that time.
The [Memento Framework](https://tools.ietf.org/html/rfc7089) can be used for time-based IPNS resolution.

The IPNS Blockchain system is expected to provide the following functionalities:

* A signed "URI -> Hash" mapping can be recorded as a transaction in the Blockchain along with the datetime only if it is requested by an authorized user
* Authorization can be verified if a mapping of the same URI or any URIs in the authorized scope of the user with the corresponding public key exists already (where the scope can be a whole domain name or specific paths under a domain name)
* New authorizations associated with a user can be added by using traditional DNS-based challenges
* Support for multi-sign transactions can be added to that will be useful for organizations
* A transaction by a newly verified authorization for a URI implicitly terminates existing transaction chain of the same URI under a different user (considering the transfer of the ownership)
* A transaction that maps a URI to a special null hash (such as all zeros) is considered as an explicit removal of the mapping (to reflect HTTP 404 or 410)
* Resolving a URI involves traversing the Blockchain in the reverse-chronological order until a block is found that has a transaction with the requested URI (it provides the same functionality as the current DHT-based implementation)
* Resolving a URI with a datetime involves traversing the Blockchain in the reverse-chronological order until a block is found that has a transaction with the requested URI at or before the given datetime
* A resolution is considered to be failed if the traversal reaches the genesis block or potential matched transaction maps to a special null hash
* The system also provides a list of all the mappings associated with a given URI along with their transaction times

While resolving references that are deeper in the Blockchain can take some time, they are considered inactive, hence, they can be cached for an extended period (as a function of the depth in the Blockchain).
On the other hand when resolving a mapping with a supplied datetime, starting from the most recent block is not necessary, instead, skip indexing of blocks (at periodic intervals) can be used for short-circuiting the lookup.
This system is going to be used to follow the chain of individual URIs more frequently, hence, it can be optimized for a faster lookup by adding the reference of the previous block in each transaction related to the corresponding URIs (unless it is the first entry or the previous entry is in the same block, in which case special values can be used).
