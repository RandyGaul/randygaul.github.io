---
layout: post
title:  "Robust Parallel Vectors Test"
date:   2014-11-11
categories: math collision-detection
---
In an effort to try to find some robust tests for parallel vectors I’ve decided to create some slides talking about the solutions I’ve come across. Hopefully one day some reader can comment or email about some alternatives or better solutions! I decided to write this blog post in slide format since Microsoft’s Powerpoint is pretty snazzy for making quick diagrams and whatnot :)

Here’s the link to Ericson’s blog post on tolerances: [Tolerances Revisited](http://realtimecollisiondetection.net/pubs/Tolerances/). Please note there’s a little bit of ambiguity about whether the test should care if the vectors point in the same direction or not. In general it doesn’t really matter since the point of the slides is numeric robustness.

The gist of the slides is that the scale of the vectors in question matters for certain applications. Computing a good relative epsilon seems difficult, and maybe different epsilon calculations would be good for different applications. I’m not sure!

Here’s a demo you can try out yourself to test two of the solutions from the slides (tests returning true should be considered false positives):

{% highlight cpp %}
#include <cstdio>
#include <cmath>

struct Vec3
{
        float x;
        float y;
        float z;

        Vec3 operator*( float a )
        {
                Vec3 v;
                v.x = x * a;
                v.y = y * a;
                v.z = z * a;
                return v;
        }
};

float Dot( Vec3 a, Vec3 b )
{
        return a.x * b.x + a.y * b.y + a.z * b.z;
}

Vec3 Normalize( Vec3 a )
{
        return a * (1.0f / std::sqrt( Dot( a, a ) ));
}

bool Parallel0( Vec3 a, Vec3 b, float tol )
{
        a = Normalize( a );
        b = Normalize( b );

        float dot = 1.0f - std::abs( Dot( a, b ) );
        return dot < tol;
}

bool Parallel1( Vec3 a, Vec3 b, float tol )
{
        if ( Dot( a, b ) < 0 )
        {
                b.x = -b.x;
                b.y = -b.y;
                b.z = -b.z;
        }

        float k = a.y / b.y;
        b = b * k;

        float x = std::abs( a.x - b.x );
        float y = std::abs( a.y - b.y );
        float z = std::abs( a.z - b.z );

        if ( x < tol && y < tol && z < tol )
                return true;

        return false;
}

void Tolerance( float tol )
{
        Vec3 u;
        u.x = 0.0f;
        u.y = 0.9812398512f;
        u.z = 0.001f;
        u = Normalize( u );

        Vec3 v;
        v.x = 0.0f;
        v.y = 10.0f;
        v.z = 1.0f;

        printf( "Tolerance: %f\n", tol );
        bool result = Parallel0( u, v, tol );
        printf( "Parallel0: %s\n", result ? "true" : "false" );
        result = Parallel1( u, v, tol );
        printf( "Parallel1: %s\n", result ? "true" : "false" );

        printf( "\n" );
}

int main( )
{
        Tolerance( 1.0e-1f );
        Tolerance( 1.0e-2f );
        Tolerance( 1.0e-3f );
}
{% endhighlight %}

Output of the program is:

{% highlight cpp %}
Tolerance: 0.100000
Parallel0: true
Parallel1: true

Tolerance: 0.010000
Parallel0: true
Parallel1: false

Tolerance: 0.001000
Parallel0: false
Parallel1: false
{% endhighlight %}
                                          
And finally, here are some slides I wrote cover this topic in detail, [available here](https://github.com/RandyGaul/randygaul.github.io/raw/gh-pages/assets/ParallelVectors.pdf).
