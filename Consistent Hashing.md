# Consistent Hashing

In a distributed cache, there needs to be a scheme to map a particular key to a
specific server. In a cluster with K keys and N servers, the simplest and naive
approach is to partition based on key % N. This works well as long as the number
of servers doesn't change or losing cache while changing configuration is not a
problem. Consistent hashing is a simple technique which allows changing the
number of servers without remapping much of the cache. It provides a hash table
in a way that the addition or removal of one slot does not significantly change
the mapping of keys to slots. Change of a server remaps only K/n keys; while a
naive approach based on modulo operator will cause almost all keys to be
remapped. For example, during a transition from 3 servers to 4, the former will
invalidate only 1/4 or 25% of the keys while the latter will lose 75% of the
keys.

The canonical paper is unnecessarily complicated but gives a formal definition
to the problem. Dynamo paper explains the concepts better. Several other
variations of the protocols exist, for example, Chord in which a node knows only
a subset of the other nodes.

Consistent hashing introduces the concept of a ring. All keys, as well as
servers, are mapped to a point on the ring with a hash function like SHA-1. The
successor of a key is the first server encountered going clockwise on the ring
from the point the key is mapped to. Whenever a key is to be inserted into the
cache, it is inserted into its successor.

A node is a responsible for an arc on the ring from its predecessor to itself.
The addition of a new node "steals" a fraction of the arc from its successor,
and can be done dynamically to reduce the load. Similarly, when a node
disappears, the arc it owned will be taken over by its successor. Dynamo
introduces the concept of virtual nodes to reduce the impact of a failure in the
classic model. Instead of a single server taking all the load of its
predecessor, it can be distributed among the cluster by placing several virtual
nodes spread randomly in the ring.

The number of virtual nodes each server contributes to the ring can be tuned to
support a cluster of heterogeneous servers, and this is a significant advantage
over the naive approach. A web server with maybe 512MB memory to spare can add 4
virtual nodes to the ring while a dedicated caching server can add several in
proportion, leading to better utilization of the resources.

There are also several practical engineering concerns that need to be addressed.
The number of virtual nodes and their placement on the ring, the particular hash
function etc needs to be tuned to match application needs. When a new node is
added, we can choose b/w dropping the cache on its successor for the region that
is being taken over or copy the cache to the new server to avoid sudden spikes
on the underlying database. A handover will have to be handled in a semi-atomic
fashion to prevent classic concurrency issues. The random placement of nodes
might lead to non-uniform load distribution, and measures to counter this might
be necessary.

## References
  - [Paper](https://www.akamai.com/es/es/multimedia/documents/technical-publication/consistent-hashing-and-random-trees-distributed-caching-protocols-for-relieving-hot-spots-on-the-world-wide-web-technical-publication.pdf)
