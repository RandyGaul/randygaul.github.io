---
layout: post
title:  "Distance Point to Line Segment"
date:   2014-07-01
categories: math collision-detection
---
Understanding the dot product will enable one to come up with an algorithm to compute the distance between a point and line in two or three dimensions without much fuss. This is a math problem but discussion and focus is on writing a function to compute distance.

### Dot Product Intuition

The dot product comes from the [law of cosines](http://en.wikipedia.org/wiki/Law_of_cosines). Here’s the formula:

```
Equation 1

c^2 = a^2 + b^2 – 2ab * cos(γ)
```

This is just an equation that relates the cosine of an angle within a triangle to its various side lengths a, b and c. The Wikipedia page (link above) does a nice job of explaining this. Equation 1 can be rewritten as:

```
Equation 2

c^2 – a^2 – b^2 = -2ab * cos(γ)
```

The right hand side equation Equation 2 is interesting! Lets say that instead of writing the equation with side lengths a, b and c, it is written with two vectors: u and v. The third side can be represented as u – v. Re-writing equation Equation 2 in vector notation yields:

```
Equation 3

|u - v|^2 – |u|^2 – |v|^2 = -2|u||v| * cos(γ)
```

Which can be expressed in scalar form as:

```
Equation 4

(u.x - v.x)^2 + (u.y - v.y)^2 + (u.z - v.z)^2 -
((u.x)^2 + (u.y)^2 + (u.z)^2) - ((v.x)^2 + (v.y)^2 + (v.z)^2) =
-2|u||v| * cos(γ)
```

By crossing out some redundant terms, and getting rid of the -2 on each side of the equation, this ugly equation can be turned into a much more approachable version:

```
Equation 5

u.x * v.x + u.y * v.y + u.w * v.w = |u||v| * cos(γ)
```

Equation 5 is the equation for the dot product. If both u and v are unit vectors then the equation will simplify to:

```
Equation 6

dot(u, v) = cos(γ)
```

If u and v are not unit vectors equation 5 says that the dot product between both vectors is equal to `cos(γ)` that has been scaled by the lengths of u and v. This is a nice thing to know! For example: the squared length of a vector is just itself dotted with itself.

If u is a unit vector and v is not, then `dot(u, v)` will return the distance in which v travels in the u direction. This is useful for understanding the plane equation in three dimensions (or any other dimension):

```
Equation 7

ax + by + cz - d = 0
```

The normal of a plane would be the vector: `{ a, b, c }`. If this normal is a unit vector, then d represents the distance to the plane from the origin. If the normal is not a unit vector then d is scaled by the length of the normal.

To compute the distance of a point to this plane any point can be substituted into the plane equation, assuming the normal of the plane equation is of unit length. This operation is computing the distance along the normal a given point travels. The subtraction by d can be viewed as “translating the plane to the origin” in order to convert the distance along the normal, to a distance to the plane.

### Writing the Function: Distance Point to Line

The simplest function for computing the distance to a plane (or line in 2D) would be to place a point into the plane equation. This means that we’ll have to either compute the plane equation in 2D if all we have are two points to represent the plane, and in 3D find a new tactic altogether since planes in 3D are not lines.

In my own experience I’ve found it most common to have a line in the form of two points in order to represent the [parametric equation](https://web.archive.org/web/20200405212731/https://en.wikipedia.org/wiki/Linear_equation#Parametric_form) of a line. Two points can come from a triangle, a mesh edge, or two pieces of world geometry.

To setup the problem lets outline the function to be created as so:

{% highlight cpp %}
float DistancePtLine( Vec2 a, Vec2 b, Vec2 p )
{
}
{% endhighlight %}

The two parameters a and b are used to define the line segment itself. The direction of the line would be the vector b – a.

After a brief visit to the Wikipedia page for this exact problem I quickly wrote down my own derivation of the formula they have on their page. Take a look at this image I drew:

![dist_pt_line_segment](/assets/dist_pt_line_segment.gif)

The problem of finding the distance of a point to a line makes use of finding the vector d that points from p to the closest point on the line ab. From the above picture: a simple way to calculate this vector would be to subtract away the portion of a – p that travels along the vector ab.

The part of a – p that travels along ab can be calculated by projecting a – p onto ab. This projection is described in the previous section about the dot product intuition.

Given the vector d the distance from p to ab is just `sqrt(dot(d, d))`. The sqrt operation can be omit entirely to compute a distance squared. Our function may now look like:

{% highlight cpp %}
float DistancePtLine( Vec2 a, Vec2 b, Vec2 p )
{
	Vec2 n = b - a;
	Vec2 pa = a - p;
	Vec2 c = n * (Dot( pa, n ) / Dot( n, n ));
	Vec2 d = pa - c;
	return sqrt( Dot( d, d ) );
}
{% endhighlight %}

This function is quite nice because it will never return a negative number. There is a popular version of this function that performs a division operation. Given a very small line segment as input for ab it is entirely possible to have the following function return a negative number:

{% highlight cpp %}
float SqDistancePtLine( Vec2 a, Vec2 b, Vec2 p )
{
	Vec2 ab = b - a, ap = p - a, bp = p - b;

	float e = Dot( ap, ab );

	return Dot( ap, ap ) - e * e / Dot( ab, ab );
}
{% endhighlight %}

It’s very misleading to have a function called “square distance” or “distance” to return a negative number. Passing in the result of this function to a sqrt function call can result in NaNs and be really nasty to deal with.

### Barycentric Coordinates – Segments

A full discussion of barycentric coordinates is way out of scope here. However, they can be used to compute distance from a point to line segment. The segment portion of the code just clamps a point projected into the line within the bounds of a and b.

Assuming readers are a little more comfortable with the dot product than I was when I first started programming, the following function should make sense:

{% highlight cpp %}
float SqDistancePtSegment( Vec2 a, Vec2 b, Vec2 p )
{
	Vec2 n = b - a;
	Vec2 pa = a - p;

	float c = Dot( n, pa );

	// Closest point is a
	if ( c > 0.0f )
		return Dot( pa, pa );

	Vec2 bp = p - b;

	// Closest point is b
	if ( Dot( n, bp ) > 0.0f )
		return Dot( bp, bp );

	// Closest point is between a and b
	Vec2 e = pa - n * (c / Dot( n, n ));

	return Dot( e, e );
}
{% endhighlight %}

This function can be adapted pretty easily to compute the closest point on the line segment to p instead of returning a scalar. The idea is to use the vector from p to the closest position on ab to project p onto the segment ab.

The above function works by computing barycentric coordinates of p relative to ab. The coordinates are scaled by the length of ab so the second if statement must be adapted slightly. If the direction ab were normalized then the second if statement would be a comparison with the value 1, which should make sense for barycentric coordinates.

### Sample Program

Here’s a sample program you can try out yourself:

{% highlight cpp %}
#include <cstdio>

struct Vec2
{
	float x;
	float y;

	const Vec2 operator-( const Vec2& a )
	{
		Vec2 v;
		v.x = x - a.x;
		v.y = y - a.y;

		return v;
	}

	const Vec2 operator*( float a ) const
	{
		Vec2 v;
		v.x = x * a;
		v.y = y * a;

		return v;
	}
};

float Dot( const Vec2& a, const Vec2& b )
{
	return a.x * b.x + a.y * b.y;
}

int main( )
{
	Vec2 a, b, p;

	a.x = 1.0f;
	a.y = 1.0f;

	b.x = 5.0f;
	b.y = 2.0f;

	p.x = 3.0f;
	p.y = 3.0f;

	Vec2 n = b - a;
	Vec2 pa = a - p;
	Vec2 c = n * (Dot( n, pa ) / Dot( n, n ));
	Vec2 d = pa - c;
	float d2 = Dot( d, d );

	printf( "Distance squared: %f\n", d2 );
}
{% endhighlight %}
    
The output is: “Distance squared: 2.117647”.
