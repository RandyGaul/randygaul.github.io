---
layout: post
title:  "Datastructure APIs in C++"
date:   2020-02-01 15:28:52 -0700
categories: algorithm architecture data-structures
---
Designing a great data structure API is mostly a grey area with lots of room for opinions. Why care about my opinion? Well, take a peek at [some of my headers on GitHub](https://web.archive.org/web/20200408025742/https://github.com/RandyGaul/cute_headers). If you think they might look useful or interesting upon first glance, then read on, as all those headers were written with the ideas here in mind. Here are my major priorities for a data structure API to tackle, listed with the most important first.

1. Anticipate common fundamental use-cases and minimize friction for these cases.
2. Expose as little as possible to the user.
3. Incur as little overhead as possible.

Note I placed “overhead” as third, not first! In this case overhead means a few things.

* Run-time overhead
* Compile-time overhead
* Abstraction cost

We all know what run-time overhead is, it’s the cost of running the code, as in how slow or fast it is.

Compile-time overhead is usually overlooked by C++ developers. It’s completely possible to time compilation and run performance analysis, especially over a long period of time as a project matures. It’s also possible with MSVC to [diagnose specifically what is incurring long compile times](https://web.archive.org/web/20200408025742/https://aras-p.info/blog/2017/10/23/Best-unknown-MSVC-flag-d2cgsummary/). There’s really no excuse for having awful compilation times, even in really large projects with millions of lines of code. The key is to minimize inter-dependencies between different sections of code, in order to minimize the following equation (just an approximation, but a good approximation).

compile + link time = number of [translation units](https://web.archive.org/web/20200408025742/https://en.wikipedia.org/wiki/Translation_unit_(programming)) * amount of code to process

The actual solution to slow compiling/link times is to put as little code as possible into each translation unit as possible. This means some real thought and planning has to go into each included file. All other strategies, such as inredibuild, precompiled headers, or unity builds are merely band-aids that can be applied to codebases that grow large without a strong underlying architecture. In other words, these alternative strategies can be the cheapest and only option for old codebases, but, with some skill and foresight these problems can simply be avoided in the first place.

Abstraction overhead is not often talked about, and is also not really a quantifiable metric, making it up a candidate for perpetual debate and mysticism. Put as simply as I can, the cost of abstraction for a particular piece of code is how difficult it is to read and understand, especially at-a-glance. The general pattern for any code base is to start out lean and mean. Over time more people are hired to work on the code base, and more code is added. As more code gets added it becomes more and more difficult understand how everything fits together, and abstractions are used ubiquitously as a compromise. Each time an abstraction is added the difficulty in mustering a competent grok of the project becomes greater and greater, and thus nobody has the ability to reason about compile-time overhead any longer. At this point the ship is sinking, and nothing can save it. Large old projects eventually get scrapped in favor of starting over.

Here is a quick list of some things I think incur the most abstraction cost in C++.

* Exceptions
* Constructors/destructors
* Move semantics
* Smart pointers (yes, including unique_ptr)
* RAII in general
* Traditional usage of the class keyword (putting too much code into the header)
* Lambdas
* Templates
* constexpr
* Iterators

This is my own personal list of costly abstractions. In my experience the code that everyone appreciates the most (in the general sense) is code that solves hard problems without any of the above features. My reasoning is that these features are much more complicated than the potential benefits they might bring to the table, so code that simply omits these features is more likely to produce a more favorable benefit to complexity ratio. Avoid all of these features, especially in you headers and data structure APIs. Your compile times will survive long enough to thank you, and your users will thank you for not [over-engineering](https://web.archive.org/web/20200408025742/https://en.wikipedia.org/wiki/Overengineering) your API into oblivion.

A data structure API can minimize all three types of overhead. The first thing is to only templatize the tiniest amount of code as necessary. The next thing is to have as little code in headers as possible. These two steps will encourage people to include your header, since it won’t really affect compile times in a relevant way. The last thing is to reduce your own compile times in the implementation file by simply not including anything unless absolutely necessary.

Take a hash table as an example. First, implement the header with a C API, preferably using the PIMPL idiom or something similar. If done intelligently this C API can be exposed through DLLs trivially, and also exhibit resiliency in terms of [ABI stability](https://web.archive.org/web/20200408025742/https://en.wikipedia.org/wiki/Application_binary_interface). Here is my personal hash table API as an example.

{% highlight cpp %}
struct hashtable_slot_t
{
    uint64_t key_hash;
    int item_index;
    int base_count;
};
 
struct hashtable_t
{
    int count;
    int slot_capacity;
    hashtable_slot_t* slots;
 
    int key_size;
    int item_size;
    int item_capacity;
    void* items_key;
    int* items_slot_index;
    void* items_data;
 
    void* temp_key;
    void* temp_item;
    void* mem_ctx;
};
 
extern CUTE_API int CUTE_CALL hashtable_init(hashtable_t* table, int key_size, int item_size, int capacity, void* mem_ctx);
extern CUTE_API void CUTE_CALL hashtable_cleanup(hashtable_t* table);
 
extern CUTE_API void* CUTE_CALL hashtable_insert(hashtable_t* table, const void* key, const void* item);
extern CUTE_API void CUTE_CALL hashtable_remove(hashtable_t* table, const void* key);
extern CUTE_API void CUTE_CALL hashtable_clear(hashtable_t* table);
extern CUTE_API void* CUTE_CALL hashtable_find(const hashtable_t* table, const void* key);
extern CUTE_API int CUTE_CALL hashtable_count(const hashtable_t* table);
extern CUTE_API void* CUTE_CALL hashtable_items(const hashtable_t* table);
extern CUTE_API void* CUTE_CALL hashtable_keys(const hashtable_t* table);
extern CUTE_API void CUTE_CALL hashtable_swap(hashtable_t* table, int index_a, int index_b);
{% endhighlight %}

It requires stdint.h, a couple of defines, and that’s it. Including this into a translation unit should not increase compile times in a relevant way.

The next step is to add as small of a templated wrapper as possible. Here’s an example API I came up with.

{% highlight cpp %}
template <typename K, typename T>
struct dictionary
{
	dictionary();
	dictionary(void* user_allocator_context);
	dictionary(int capacity, void* user_allocator_context);
	~dictionary();

	T* find(const K& key);
	const T* find(const K& key) const;
	error_t find(const K& key, T* val_out);
	error_t find(const K& key, T* val_out) const;

	T* insert(const K& key);
	T* insert(const K& key, const T& val);
	void remove(const K& key);

	void clear();

	int count() const;
	T* items();
	const T* items() const;
	K* keys();
	const K* keys() const;

	void swap(int index_a, int index_b);

private:
	hashtable_t table;
};
{% endhighlight %}

The major observation is this templated wrapper calls into the C api. Each function is just one or a few lines each, and requires no other additional headers or dependencies of any kind. The templates are here merely to provide types over the void pointers. Personally, I like to think of the dictionary as an [abstract data type](https://web.archive.org/web/20200408025742/https://en.wikipedia.org/wiki/Abstract_data_type), as opposed to a “class”. Class in C++ is a really loaded term. I actually use the keyword struct just to try and avoid all the pre-loaded context or baggage that class brings with it, which is sort of silly of me, but whatever :)

Notice how there are no iterators. Iterators are awful for a variety of reasons, but one major reason iterators are horrible is the overhead they incur to compile times when templated. They generate large amounts of code in every single translation unit they touch, and encourage people to write “generic” iterators and “generic” algorithms that operate on iterators. This all leads to slower and slower compile times.

If I compare my own templated dictionary to unordered_map things start to come into perspective. My own dictionary includes stdint.h, and contains about 200 lines of code. unordered_map in VS2017 has 1k lines of code, but also includes tuple and xhash. Delving into those headers reveals cstring, cwchar, list, vector, new, type_traits, xutility, and it keeps going and going and going. I stopped estimating once I hit 20k lines of code. It’s possible to run a test example and look at the preprocessed output, but I digress, and think my point is already proven.
