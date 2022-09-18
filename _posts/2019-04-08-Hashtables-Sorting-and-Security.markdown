---
layout: post
title:  "Hash Tables, Sorting, and Security"
date:   2019-04-08 15:28:52 -0700
categories: data-structures api-design hash
---
I’d like to recommend Mattias Gustavsson’s hash table he has [open sourced on github](https://web.archive.org/web/20190409084702/https://github.com/mattiasgustavsson/libs). The implementation stores a few arrays, the most interesting one is a layer of indirection between mapping hashes to key/value entries.

Each key/value pair is stored in their own contiguous array of keys, or values, at matching indices. This means the table as an API that allows users to loop over key/value pairs with a trivial for loop:

{% highlight cpp %}
key_t* keys = (key_t*)get_keys(table);
value_t* values = (value_t*)get_values(table);
int count = get_count(table);

for (int i = 0; i < count; ++i)
{
	key_t* key = keys + i;
	value_t* value = values + i;
	do_things(key, value);
}
{% endhighlight %}

This opens up some possibilities.

The table entries can be processed with a tight for loop, without running into “empty entries” which can cause branch mis-prediction performance hits.
Key/value pairs can be deleted from the table and remapped internally in a trivial manner, while looping over elements contiguously.
Key/value pairs can be swapped trivially, so the ordering of the keys and values arrays can be sorted. For example, quicksort can trivially sort key/value pairs using a comparison predicate and the swap function of the table.

Suddenly the hash table now serves many interested purposes. Without additional memory the table can act as a priority queue, a sorted array, or other similar data structures. This utility comes at the cost of an extra layer of indirection: the key value pairs can be moved around, because the table is storing a separate array of data that tracks the index of a particular key/value pair. This means that if the hash table itself is “cold” (none of the data is in a memory cache) twice the number of cache misses can occur. However, if the table is hot, little to no performance hit will be seen in the common case.

Additionally, performance gains when iterating key/value pairs will “win-back” any loss from the extra layer of indirection.

---


Let us cover secure hash tables. If a hash table is used to cache or map data coming in from the network (or any other insecure data), if the attacker can learn or guess what kind of table and hash function is in use, they can craft input to try to degenerate table lookups to a time complexity of O(N) linear time. This is a type of DoS attack. Not too long ago this attack was popularized on applications using popular hash functions such as murmurhash3.

[libsodium](https://web.archive.org/web/20190409084702/https://libsodium.gitbook.io/doc/hashing/short-input_hashing) has some functions made specifically for hash tables, where the hash function takes a secret seed. This lets the table mitigate DoS effectiveness on the table itself. The webpage recommends ensuring table sizes are a prime number in order to ensure all bits of the hash function are utilized. In my own code I have modified Mattias’s table to use the libsodium function in the above link, in order to mitigate this style of DoS attack.

This is not necessary for the common use-case, where the code’s internal hash function is very fast, and preferable when data is known to not be malicious.

---


Finally, I have made some small modifications to Mattias’s table to allow arbitrarily sized keys. This modification is quite trivial, and simply changes keys from uint64_t to void* + key_size, where the key_size is constant for the table’s lifetime.

My use case here was to map incoming IP addresses and ports to some encryption state, to efficiently implement a connection handshake process for a protocol I am working on. In this case a 64-bit key is not large enough to store a 16 bit port, and a potentially 18 byte IPv6 address.
