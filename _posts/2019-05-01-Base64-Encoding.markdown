---
layout: post
title:  "Welcome to Jekyll!"
date:   2022-09-17 15:28:52 -0700
categories: algorithm compression serialization api-design
---
Base64 encoding has this nice feature where the encoded information is safely copy-pasteable and stored as plaintext in files. It uses numbers represented in Base64 with a limited character set. Here's the table from [RFC 4648](https://www.ietf.org/rfc/rfc4648.txt).

```
                      Table 1: The Base 64 Alphabet

     Value Encoding  Value Encoding  Value Encoding  Value Encoding
         0 A            17 R            34 i            51 z
         1 B            18 S            35 j            52 0
         2 C            19 T            36 k            53 1
         3 D            20 U            37 l            54 2
         4 E            21 V            38 m            55 3
         5 F            22 W            39 n            56 4
         6 G            23 X            40 o            57 5
         7 H            24 Y            41 p            58 6
         8 I            25 Z            42 q            59 7
         9 J            26 a            43 r            60 8
        10 K            27 b            44 s            61 9
        11 L            28 c            45 t            62 +
        12 M            29 d            46 u            63 /
        13 N            30 e            47 v
        14 O            31 f            48 w         (pad) =
        15 P            32 g            49 x
        16 Q            33 h            50 y
```

We can see all numbers from 0-63 are mapped to characters. To encode binary data into Base64 we simply loop over our data in 7-bit chunks and output the correct character. To decode we can build a table of characters that maps to the correct 7-bit binary data. Encoding and decoding should be one for loop and a table lookup, along with some bit manipulation to pack/unpack 7 bits at a time.

It can be fairly difficult to find a good, copy-pastableable, Base64 encoding implementation in C. After some searching around and mandatory hair-pulling I made one. It's since been absorbed into Cute Framework and available for anyone to use. The code is under zlib license so feel free to use it basically however you want. It's just two functions, one for encode and one for decode. They look a lot like `memcpy`. Here's my recommended API if you want to grab the code from Cute Framework or write your own from scratch:

{% highlight cpp %}
struct cf_error_t
{
    int code;
    const char* details;
};

cf_error_t cf_base64_encode(void* dst, size_t dst_size, const void* src, size_t src_size);
cf_error_t cf_base64_decode(void* dst, size_t dst_size, const void* src, size_t src_size);
{% endhighlight %}

Cute Framework reference files:

* [Base64 Header File](https://github.com/RandyGaul/cute_framework/blob/master/include/cute_base64.h)
* [Base64 CPP File](https://github.com/RandyGaul/cute_framework/blob/master/src/cute_base64.cpp)

A quick note about optimization and SIMD -- The encode/decode functions are great candidates for further optimizations. You could use [SIMD instructions](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) to increase performance by 2-3x quite easily. Another way would be to re-shuffle around the order of operations to improve [instruction level parallelism](https://en.wikipedia.org/wiki/Instruction-level_parallelism). These could be great ways to practice optimization, though personally I prefer to keep the code as-is for maximizing simplicity and portability. For games Base64 encoding is probably an infrequent operation and will be unlikely to ever show up in a profiler as a hot-path.
