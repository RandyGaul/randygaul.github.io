---
layout: post
title:  "Welcome to Jekyll!"
date:   2019-02-05
categories: math api-design
---
Here are the major points I’ve learned after trying out a variety of math libraries, both in my own time and my professional time, and after implementing a few styles with a variety of performance tests. These conclusions are based on my own personal experience, and I don’t have any profiling data on-hand, so I’ll write out my experience here in a “take it or leave it” style :)

1. All the 64-bit builds I’ve looked into (mainly MSVC) have given up on the FPU and perform traditional floating point operations on wide registers, even if logically only a single component is used. You may as well just implement your vector math library with wide-types from the start, since it’s not actually much more effort!
2. Loads and stores don’t matter so much anymore. I think this is probably due to point #1, but the fact of the matter is old wisdom about avoiding loads/stores really just doesn’t matter that much nowadays. Many older vector math libraries take extra care to construct their API to fervently avoid casting to/from float types, but in my personal experience this is all a big waste of time. For example I would say this implementation of the len function is perfectly fine:
{% highlight cpp %}
float len(v4 v)
{
	return sqrt(dot(v, v));
}
{% endhighlight %}
Somewhere in there the _mm_cvtss_f32 is probably used to pull a float out of a wide register. In practice this will simply be moving a value from one wide register to another, which is negligible.
3. Templatizing your math types is bad practice, for pretty much any reason. Applications have the best performance with the float type, and all other types are very rarely used, and only in specific scenarios. To handle these edge-cases, hand-rolling static or local vector types works perfectly fine. The general case only cares about floats. The templates will be needlessly complicated, and bring down compile/link times unnecessarily.
4. On MSVC quite a bit of code gets auto-vectorized in an intelligent way. This is relevant when your vec4 struct is just four individual floats. In my experience, so long as there is not a lot of pointer indirection, and the vectors/matrices are in big arrays of many vectors and many matrices, auto-vectorization usually kicks in. I noticed a big performance difference between debug and release builds, since debug builds seemed to get vectorized less often. The difference was jarring, so I do recommend using a wide-type instead of four individual floats.
5. Member functions in your vector class is perfectly fine for MSVC. I don’t have esoteric console experience, so your mileage may vary. Some older wisdom says compilers have a hard time optimizing SIMD code unless the SIMD types are naked (like: typedef vec4 __m128). Personally I’ve never experienced this problem on PC or even iOS.
6. AoS vs SoA. For example, SoA:
{% highlight cpp %}
struct vec4_array
{
	__m128* x;
	__m128* y;
	__m128* z;
	__m128* w;
};
{% endhighlight %}
vs AoS
{% highlight cpp %}
struct vec4
{
	__m128 m;
};
{% endhighlight %}
In my experience the second option is more than good enough in the general case. Since the implementation of operations like dot, cross, and others, are not going to be a perfect 4x speedup (due to extra shuffles and operations when compared to the SoA version), some people say the second option is “bad”. I disagree, and think the second option is perfectly fine. In cases where extra performance is needed, intrinsics can always be used by hand.

My personal favorite article on math libraries is Mitton’s from 2016: http://www.codersnotes.com/notes/maths-lib-2016/

Instead of focusing too much on old-wisdom regarding performance peculiarities, I would recommend anyone writing a math library to focus on their API and make sure their math is A) really straightforward, B) easy to extend, C) does not contain templates, for the sanity of compile/link times.

The vector math header should probably just define your vector and matrix types, and only include function declarations for the most obvious and common operations, like dot, len, add/sub. All the less common operations (everything else) can go into .inl files, or simply just another utility header (perhaps called vec_math_utilities.h). This will encourage everyone to add their little helpers or additions into the utility file, and in all likelihood they will find something they can already use as they go to make an addition. This will also make the math library itself not-so-scary, since the main header will only have a few structs and a few functions, making for very little reading to get up-and-running.

The same strategy can be used for all math or shape types, including AABBs, circles, OBBs, or any other kind of commonly used shape.
