---
layout: post
title:  "Data Structures in C and Allocating"
date:   2024-10-13
categories: data-structures  memory
---
I've been researching ways to make C more ergonomic. After years of trial and error, I've found that the most maintainable code is typically pure, portable C. However, one major pain point keeping me in C++ is simplifying user code when dealing with data structures. While C++ allows defining classes with custom semantics, C lacks that kind of flexibility—or so I thought.

Let's build a C API for dynamic buffers that feels natural to use, supports static allocation, and can grow into heap memory when needed. The goal is to improve ergonomics while maintaining performance. We’ll also introduce a way to map static buffers to unique call sites, reducing memory consumption to a minimum. With this approach, we can write high-performance C code without sacrificing developer flow.

Many readers may already be familiar with [stretchy buffers](https://github.com/nothings/stb/blob/master/deprecated/stretchy_buffer.txt). By hiding a small struct behind a user pointer and using macros, we can mimic some of the ergonomic benefits of C++'s polymorphism in C. This technique allows us to operate on pointers directly with a clean array API, minimizing distractions and keeping the focus on solving problems, not on memory management details.

```c
typedef struct v2 { float x, y; } v2;
v2 V2(float x, float y) { return (v2){ x, y }; }

v2* pts = NULL;
apush(pts, V2(0,0));
apush(pts, V2(1,0));
apush(pts, V2(0,2));
for (int i = 0; i < asize(pts); ++i) {
	printf("%f,%f\n", pts[i].x, pts[i].y);
}
afree(pts);
```

A remaining issue with this approach is the frequent `malloc` and `free` calls under `apush` and `afree`. In games, we often build dynamic lists for queries like collisions, process the list, and then `free` it. Doing this many times per frame can lead to performance bottlenecks, especially due to lock contention in `malloc` and potential cache thrashing. Modern `malloc` implementations mitigate cache issues to some extent, but you’re still dependent on the underlying implemenation.

To maintain flow and avoid low-level tedium, we need an allocation scheme that efficiently handles these temporary data structures while remaining ergonomic for the developer. Let’s take a look at a common use case: querying for collisions.

```c
int N = 0;
Entity* ids = query(box, &N);
for (int i = 0; i < N; ++i) {
	on_collide(my_id, ids[i]);
}
free(ids);
```

This code doesn't use stretchy buffers. It's highly annoying to create the variable `N`, take the address of it, and then call `free`. You know you're performing a `malloc` and `free` for every single query. This fact sits in the back of your mind, causing anxiety, eroding your flow-state until eventually you capitulate and try thinking of a way to pre-allocate memory for your collision queries. Maybe you even succeed, and come up with some hand-coded API that caches memory for queries and dynamically grows it as-needed. This works, you're happy. Until you implement some other feature somewhere else, performing the same pattern. You perform a different kind of query on something else, build a list, traverse the list, and free the list. This other query also happens many times per game-tick. Anxiety rises slowly, and flow-state is broken once again.

Instead, each call-site requiring an array can be uniquely identified. A global temp-space buffer can be allocated once upon game initialization. Each unique call-site can allocate a static buffer from the global scratch space. Each subsequent call will simply return you the same buffer of temp-space memory. At the end of the frame all of these temporary allocations can be cleared all at once using a stack-based allocator, or [arena allocator](https://github.com/pervognsen/bitwise/blob/master/ion/common.c#L172-L209). This link is a nice arena implementation by Per Vognsen on GitHub you can check out if you haven't seen the technique before. There isn't much to it -- just a stack based allocator.

Combining an arena, and the stretchy buffer technique for polymorphism, we can create an initial statically allocated array that grows onto the heap as-needed. Here's the API I've come up with:

```c
Entity* ids = atmp(Entity, 32);
ids = query(ids, box);
for (int i = 0; i < asize(ids); ++i) {
	on_collide(my_id, ids[i]);
}
afree(ids);
```

The macro `atmp` uniquely identifies itself at the call-site. This lets the backing implementation cache the static buffer allocated from a global arena. Each subsequent call, for this particular query code, will fetch the same underlying buffer (assuming it doesn't overflow and allocate itself onto the heap). `afree` only calls `free` if the array has grown onto the heap, otherwise it's just an if-check no-op.

```c
// Allocates a static array using the global tmp allocator `tmp_alloc` (reset at end of frame).
// ...Uniquely identifies the callsite and continuously returns the same buffer.
// ...Simply call `afree` when done, as the array may grow beyond the intial static storage.
// ...Not thread-safe for now, but easily refactored.
#define atmp(T, n) (T*)atmp_impl(__COUNTER__, sizeof(T), n)
FORCE_INLINE void* atmp_impl(uint64_t alloc_id, int item_size, int n)
{
	void* a = g_tmp_cache ? hget(g_tmp_cache, alloc_id) : NULL;
	if (!a) {
		int bytes = item_size * n;
		a = astatic(tmp_alloc(bytes), bytes, item_size);
		hset(g_tmp_cache, alloc_id, a);
	}
	return a;
}
```

In the above snippet `hget` simply does a hashtable lookup, while `hset` simply sets a key-value pair within a global hash table. You should be able to easily adapt this to your own codebase. Of course there are threading concerns hitting a global hashtable, but, that's easily refactorable, and generally not a concern for most kinds of games.

At the end of the game-tick the global table `g_tmp_cache` can be cleared, along with the arena `tmp_alloc` is allocating from. All the callsites will simply fetch new static buffers as-needed, keeping memory consumption down to mostly just what's needed on a frame-to-frame basis.

But why bother uniquely identifying the callsite at all? Why not just return a new statically allocated buffer upon each `atmp` call? The reason is ergonomics. The underlying design of dynamically allocated arrays don't quite map onto arena style allocators. If the array needs to grow itself larger it requires a `realloc` or similar, which doesn't map well onto linearly allocated arenas/stacks, unless you're willing to simply drop your old memory and allocate more. This will cause your game to allocate quite a lot more memory than necessary.

In practice, this reduces memory usage significantly. Imagine a scenario with 1,000 calls to a collision query. Normally, each call would require a new allocation, but with this approach, memory usage stays constant, and only one allocation occurs per unique call site. In the above example memory consumption drops from ~20kb to 256 bytes.

Another downside is the use of `__COUNTER__` to uniquely identify callsites. `__COUNTER__` is non-standard C, but supported by GCC, Clang, and MSVC (all the big ones). It's possible to use a different, more portable solution involving static variables, assigning unique ids at runtime. However, it would require `atmp` to be called on a different line than the initialized array, which is unfortunately just annoying an less ergonomic.

```c
v2* pts = NULL;
atmp(pts, 32);
```

The implementation of `atmp` could then hide a unique id in a static variable, something like so:

```c
#define atmp(a, n) do { static int id = g_id_gen++; a = atmp_impl(id, sizeof(a), n); } while(0)
```

The final downside here is the buffer is semantically static, a lot like the `static` keyword. You won't be getting unique buffers each time a function is called, which restricts it's usage. In the event you want to return an actual unique buffer that will persist, get handed around to other functions, then simply allocating directly from the arena without a caching mechanism is preferred. A different macro can provide this quite easily, directly calling into the equivalent of `tmp_alloc` in your own codebase.

```c
// Similar to `atmp` but does not uniquely identify the callsite.
// ...Simply returns a statically allocated buffer.
#define ascratch(T, n) (T*)ascratch_impl(sizeof(T), n)
FORCE_INLINE void* ascratch_impl(int item_size, int n)
{
	int bytes = item_size * n;
	void* a = astatic(tmp_alloc(bytes), bytes, item_size);
	return a;
}
```

And there you have it! It's nothing particularly new or special, just a mishmash of a few different techniques and careful API design. The vast majority of dynamic arrays are reduced to a few if-checks and a single hashtable hit. Only in worst-case scenarios is `malloc` and `free` ever touched, and memory consumption scales relative to the number of unique callsites, instead of by how many times individual functions are called.
