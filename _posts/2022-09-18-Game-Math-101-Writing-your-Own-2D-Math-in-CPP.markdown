---
layout: post
title:  "Game Math 101, Writing your Own 2D Math in C++"
date:   2022-09-18
categories: math
---
THIS POST IS A WIP

Do you want to make a game but don't know any math? Do you want to skip endless hours of reading arcane math texts and go straight to the useful stuff? Good! Me too. Let's just skip straight ahead to the useful stuff. Sit back, grab yourself a lunchable, and get ready to become a wizard.

We will be using C++ and writing our own math to create some demonstrations and animations from scratch. By the end of this article you will produce an interactive experience like the one seen just here below in this cute gif!

gif here

### Prerequisites

You'll need to know some basic C++ (well, more like just some basic C stuff), but not much so don't be worried. If you're not quite comfortable feel free to visit some online C++ tutorials and then come back here later to try again. At a minimum I'd recommend learning about these topics:

* Loops
* If-statements
* Functions
* Structs

### Get a Compiler

Since we're using C++ we need a compiler. If you already have one and are familiar with how to use them, skip ahead. For those on Mac/Windows here's how I recommend setting up the `g++` compiler.

#### Make a Folder for this Article

Go ahead and make your own personal folder for holding all the files for this article. I'd recommend naming it `math_101`.

#### Windows

If you're on Windows I recommend to use [TDM-GCC](https://jmeubank.github.io/tdm-gcc/download/), a dead-simple compiler distribution for `g++`. Just download the installer and open up a command prompt ***after you install the compiler***. Here's a gif showing how to open a command prompt in a specific window (below). You can also use the [cd command](https://www.youtube.com/watch?v=BfXh11ryBJg) to move the command prompt to another folder. Move it to your `math_101` folder.

![command_prompt](/assets/command_prompt.gif)

#### Mac

On Mac just download Xcode from the Asset Store. This will automatically install the command line tools for you, including `g++`. Open up the terminal application ***after you install Xcode***. You can also use the [cd command](https://www.youtube.com/watch?v=DvwWJw6Ppns) to move the terminal to another folder. Move it to your `math_101` folder.

Most people reading this article will be using a Windows machine. One Windows they use command prompt instead of Terminal. Later when reading, if you see "command prompt" just think Terminal instead. The rest of the steps are 99% the same, despite this difference.


#### Linux

If you're on Linux then I'll assume you're already quite familiar with C/C++ and compilers.

### Grab a Copy of tigr

We're using the tigr library. It's the easiest possible graphics library to get running and use. It handles a bunch of [OpenGL](https://en.wikipedia.org/wiki/OpenGL) stuff for us, in case you were curious. It consists of merely one header file and one source file. Grab a copy of each and put them into your `math_101` folder.

* [tigr.h](https://raw.githubusercontent.com/erkkah/tigr/master/tigr.h)
* [tigr.c](https://raw.githubusercontent.com/erkkah/tigr/master/tigr.c)

### Build and Run the Math 101 Program

Make a file called `main.cpp` in your `math_101` folder. Copy + paste this code snippet into your `main.cpp` file.

{% highlight cpp %}
#include "tigr.h"

int main()
{
	Tigr* screen = tigrWindow(640, 480, "Math 101", 0);

	while (!tigrClosed(screen) && !tigrKeyDown(screen, TK_ESCAPE)) {
		tigrClear(screen, tigrRGB(0x80, 0x90, 0xa0));
		tigrUpdate(screen);
	}

	tigrFree(screen);

	return 0;
}
{% endhighlight %}

This code sets up our nice little window to create cool animations and visualize our math as we learn. We must compile this code to create an executable. Make sure your command prompt is opened in the `math_101` folder along with tigr.c, tigr.h and main.cpp. It should look something like this (you can ignore the `.git` file).

![math_101_folder](/assets/math_101_folder.png)

Here is the command we will use to compile the code on Windows.

```
g++ main.cpp tigr.c -o math_101_program -s -lopengl32 -lgdi32
```

And here is the one for Mac.

```
g++ main.cpp tigr.c -o math_101_program -framework OpenGL -framework Cocoa
```

And finally for you Linux weirdos (joking, haha).

```
g++ main.cpp tigr.c -o math_101_program -s -lGLU -lGL -lX11
```

If you already know how to use a [make and a makefile](https://www.gnu.org/software/make/manual/make.html) you can optionally use this makefile. But this is completely optional and not necessary.

{% highlight make %}
ifeq ($(OS),Windows_NT)
	LDFLAGS = -s -lopengl32 -lgdi32
else
	UNAME_S := $(shell uname -s)
	ifeq ($(UNAME_S),Darwin)
		LDFLAGS = -framework OpenGL -framework Cocoa
	else ifeq ($(UNAME_S),Linux)
		LDFLAGS = -s -lGLU -lGL -lX11
	endif
endif

demo : demo.cpp ../../tigr.c
	g++ $^ -Os -o $@ $(CFLAGS) $(LDFLAGS)
{% endhighlight %}

This will produce an executable file called `math_101_program`. Go ahead and run it! You should see a nice ugly window like this one.

![math_101_program](/assets/math_101_program.png)

Congrats!

### Drawing some Lines

The function `tigrLine` will draw us lines. All we need to do is specify where the line starts, ends, and it's color. Here's the function signature from the header file along with the documentation comments found in tigr.h.

{% highlight cpp %}
// Draws a line.
// Start pixel is drawn, end pixel is not.
// Clips and blends.
void tigrLine(Tigr *bmp, int x0, int y0, int x1, int y1, TPixel color);
{% endhighlight %}

`x0` and `y0` is the first point of the line, while `x1` and `y` are the end point. Add a few calls to `tigrLine` to our main.cpp program like so:

{% highlight cpp %}
#include "tigr.h"

int main()
{
	Tigr* screen = tigrWindow(640, 480, "Math 101", 0);

	while (!tigrClosed(screen) && !tigrKeyDown(screen, TK_ESCAPE)) {
		tigrClear(screen, tigrRGB(0x80, 0x90, 0xa0));

        	// Add these two lines here!
		tigrLine(screen, 0, 0, 100, 100, tigrRGB(0xFF, 0xFF, 0xFF));
		tigrLine(screen, 0, 0, 30, 200, tigrRGB(0xFF, 0xFF, 0xFF));

		tigrUpdate(screen);
	}

	tigrFree(screen);

	return 0;
}
{% endhighlight %}

It should look like this:

![math_101_lines](/assets/math_101_lines.png)

### Positions (points) and Vectors

You've already used them in the last section on drawing lines! A vector is just a collection of numbers, each called a component. For games it's very common to use 2D, 3D, and occassionally 4D vectors. For this article let's just stick with 2D vectors. Usually we use vectors to represent positions, velocities, and directions. For each of these vectors there would be an x and y a component.

We can write a vector like `{x, y}`, or `(x, y)`, or even just `v`. In our program we can use a struct to represent a vector as a pair of two floats.

{% highlight cpp %}
struct v2
{
    v2() { }
    v2(float x, float y) { this->x = x; this->y = y; }
	float x;
	float y;
};
{% endhighlight %}

`v2` stands for vector 2, for a 2D vector. We will be writing v2 a lot so it's best to choose a very short and convenient name. Many other folks like to use `vec2` or `Vec2`, but I'll recommend a nice a clean `v2`.

Operations on vectors are what make them useful. The simplest ones are addition and subtraction. We can add two vectors together or subtract two vectors from each other by adding and subtracting their components. By using operator overloading we can easily implement these operations in C++.

{% highlight cpp %}
v2 operator+(v2 a, v2 b)
{
    return v2(a.x + b.x, a.y + b.y);
}

v2 operator-(v2 a, v2 b)
{
    return v2(a.x - b.x, a.y - b.y);
}
{% endhighlight %}

Though from here on out let's just pack these functions up onto a single line each. We're going to write a lot of them!

{% highlight cpp %}
v2 operator+(v2 a, v2 b) { return v2(a.x + b.x, a.y + b.y); }
v2 operator-(v2 a, v2 b) { return v2(a.x - b.x, a.y - b.y); }
{% endhighlight %}

In our program the coordinate (0, 0) represents the origin of the screen and sits a the top-left. The positive x-axis points to the right, while the positive y-axis points down.

![math_101_origin_top_left](/assets/math_101_origin_top_left.png)

When we drew lines in the last section we actually used vectors. Technically speaking a vector is a direction while a *point* has a position (or location). (0, 0) is a point at the origin, and (100, 100) is also a point at the end of one of our lines. The difference between these two points is a vector. In this way we can say the vector (a direction) of (100, 100) is used to point at the position (100, 100) from the position (0, 0).

If we wanted to be completely strict about our distinction of points and vectors we could adjust our type definitions and operators. We're not going to *actually* do this though, because it's just not practical. But for learning it's critical to really nail the difference of points and vectors. Here are a couple quick rules:

1. A position has two components and represents a point in 2D space. We can write it as (x, y), and represent it in code as a `v2` float pair. For example, the top-left pixel in our program (the origin) has the position of (0, 0).
2. A position has no orientation or direction. It's merely a point. An infinitesimally tiny spot.
3. A vector also has two components, and as far as the computer is concerned it looks the same as vector: two floats as a pair, such as (x, y).
4. A vector is a direction, as if it points in a certain direction. For example, so far the direction (0, 1) in our program point straight donwnward.
5. A vector has no position.

So a position has no direction, and a direction has no position. To be somewhere in the world we need a position. To know where something else is in the world, we need not only a position but a direction. Directions are like differences between positions.

When we drew a line in the last section, we could say we drew a vector between two input points (x0, y0) and (x1, y1). However, we could also say we drew a point and vector, or a vector and a point. These are all mathematically equivalent. Let us name a couple variables.

```
pa = (30, 50)
pb = (200, 350)
```

We have pa and pb, standing for point A and point B. We can subtract two points to get a vector.

```
v = pb - ab
-v = pa - pb
```

In the above example v would have the value of (170, 300). If we do `pb - pa` this should read like so: *the vector between two points is the endpoint minus the start point*. Memorize this and burn it into your skull. If we flip the terms around and do `pa - pb` we get a negative vector (-170, -300). Some more rules pop up!

1. The difference (subtraction) of two points is a vector.
2. The vector between two points is the endpoint minus the start point.

Similarly we can move a points position by addition. We can add a vector to a point and get a new point. Much like how we can subtract a vector from a point and get a new point.

1. A point added with a vector moves the point along the vector.
2. A point subtracted with a vector moves the point along the negative vector.

### Drawing some Points

We can draw a single pixel at a specified point with the function `tigrPlot`. Here's the signature and docs from the header tigr.h:

{% highlight cpp %}
// Plots a pixel.
// Clips and blends.
// For high performance, just access bmp->pix directly.
void tigrPlot(Tigr *bmp, int x, int y, TPixel pix);
{% endhighlight %}

Let's use this function to draw a couple pixels. The color of the background is set to black and the points will be white pixels, so we can see them a little easier. `tigrPlot` takes integers to draw an exact pixel while our `v2` struct is floats. We can just typecast the floats to ints to snap each float to specific pixel and draw it as a point.

{% highlight cpp %}
void draw_point(v2 p, TPixel color)
{
    tigrPlot(screen, (int)p.x, (int)p.y, color);
}
{% endhighlight %}

Go ahead and add it to our program, along with our math functions. Let's also add in some convenience functions to define specific colors for us, like `color_white` and `color_black`.

{% highlight cpp %}
#include "tigr.h"

struct v2
{
	v2() { }
	v2(float x, float y) { this->x = x; this->y = y; }
	float x;
	float y;
};

v2 operator+(v2 a, v2 b) { return v2(a.x + b.x, a.y + b.y); }
v2 operator-(v2 a, v2 b) { return v2(a.x - b.x, a.y - b.y); }

Tigr* screen;

void draw_point(v2 p, TPixel color)
{
    tigrPlot(screen, (int)p.x, (int)p.y, color);
}

TPixel color_white() { return tigrRGB(0xFF, 0xFF, 0xFF); }
TPixel color_black() { return tigrRGB(0, 0, 0); }

int main()
{
	screen = tigrWindow(640, 480, "Math 101", 0);

	while (!tigrClosed(screen) && !tigrKeyDown(screen, TK_ESCAPE)) {
		tigrClear(screen, color_black());
  
        v2 a = v2(200, 300);
        v2 b = v2(100, 150);
        v2 c = v2(400, 600);

		draw_point(a, color_white());
		draw_point(b, color_white());
		draw_point(c, color_white());

		tigrUpdate(screen);
	}

	tigrFree(screen);

	return 0;
}
{% endhighlight %}

![math_101_points](/assets/math_101_points.png)

We can use the above rules to move these points. We can add or subtract a vector. Here's a program to move all three white points and draw the moved ones as green offset to the right by 20 pixels.

{% highlight cpp %}
#include "tigr.h"

struct v2
{
	v2() { }
	v2(float x, float y) { this->x = x; this->y = y; }
	float x;
	float y;
};

v2 operator+(v2 a, v2 b) { return v2(a.x + b.x, a.y + b.y); }
v2 operator-(v2 a, v2 b) { return v2(a.x - b.x, a.y - b.y); }

Tigr* screen;

void draw_point(v2 p, TPixel color)
{
	tigrPlot(screen, (int)p.x, (int)p.y, color);
}

TPixel color_white() { return tigrRGB(0xFF, 0xFF, 0xFF); }
TPixel color_black() { return tigrRGB(0, 0, 0); }
TPixel color_red() { return tigrRGB(0xFF, 0, 0); }
TPixel color_green() { return tigrRGB(0, 0xFF, 0); }
TPixel color_blue() { return tigrRGB(0, 0, 0xFF); }

int main()
{
	screen = tigrWindow(640, 480, "Math 101", 0);

	while (!tigrClosed(screen) && !tigrKeyDown(screen, TK_ESCAPE)) {
		tigrClear(screen, color_black());

		v2 a = v2(200, 300);
		v2 b = v2(100, 150);
		v2 c = v2(400, 600);

		draw_point(a, color_white());
		draw_point(b, color_white());
		draw_point(c, color_white());

		// Draw our points moved by the `offset` with green.
		v2 offset = v2(20, 0);

		draw_point(a + offset, color_green());
		draw_point(b + offset, color_green());
		draw_point(c + offset, color_green());

		tigrUpdate(screen);
	}

	tigrFree(screen);

	return 0;
}
{% endhighlight %}

![math_101_point_offset](/assets/math_101_point_offset.png)

### Animating

In games we often used what's called an axis-aligned bounding box (or aabb for short). It means a box that's aligned onto the x and y axis. We can define an aabb struct in a few ways by using our knowledge of points (positions) and vectors (direction).

{% highlight cpp %}
// Option 1.
struct aabb
{
    v2 min;
    v2 max;
};

// Option 2.
struct aabb
{
    v2 p;    // position
    float w; // width
    float h; // height
};

// Option 3.
struct aabb
{
    v2 p; // position
    v2 e; // extent vector (width, height)
};

// Option 4.
struct aabb
{
    v2 p;  // position
    v2 he; // half-extent vector (width * 0.5, height * 0.5)
};
{% endhighlight %}

Each option is functionally equivalent, but which we choose is largely up to preference. By storing 4 floats we can store the minimum amount of information necessary to represent the entire box. Since an axis aligned bounding box has four corners and a location we only need to know two opposing corners and can calculate the others from that info.

Option 1 stores two opposing corners, the lowest x and lowest y as a pair, and the highest x and highest y as a pair. This is my personal favorite, so let's go with that. We can also define some useful functions for the aabb:

{% highlight cpp %}
float width(aabb box) { return box.max.x - box.min.x; }
float height(aabb box) { return box.max.y - box.min.y; }
v2 center(aabb box) { return (box.min + box.max) * 0.5f; }
{% endhighlight %}

However, for that last function to work properly we must expand our vector operations. It's possible to multiply a vector with a scalar. A scalar means a single component number, or a float. Simply multiply with the x component, and then the y component. This adjusts the size, or scale of a vector.

{% highlight cpp %}
v2 operator*(v2 a, float b) { return v2(a.x * b, a.y * b); }
{% endhighlight %}

In math notation we can the length of a vector v is `|v|`. When we multiply a vector v, we are multiplying each component, which in turn scales the length by the same value. We can calculate the length of a vector by utilizing the Pythagorean Theorem `a^2 + b^c = c^2`, where a, b and c are sides of a right triangle.

![pythagorean](/assets/pythagorean.png)

We can think of our vector's length as c, where a and b are equal to our vectors individual components x and y. We can write down a function to solve for the hypotenuse (or the length of our vector).

{% highlight cpp %}
#include <math.h> // For sqrtf function.

float length(v2 v)
{
    float a = v.x;
    float b = v.y;
    float c = sqrtf(a * a + b * b);
    return c;
}
{% endhighlight %}

Or the shortened version:

{% highlight cpp %}
float len(v2 v) { return sqrtf(v.x * v.x + v.y * v.y); }
{% endhighlight %}

We will make use of this `len` function later. Now, back to our box drawing function, which will make use of a nifty line drawing function.

{% highlight cpp %}
void draw_line(v2 a, v2 b, TPixel color)
{
	tigrLine(screen, (int)a.x, (int)a.y, (int)b.x, (int)b.y, color);
}

void draw_box(aabb box, TPixel color)
{
	float w = width(box) + 1;
	float h = height(box) + 1;
	draw_line(box.min, box.min + v2(w, 0), color);
	draw_line(box.min, box.min + v2(0, h), color);
	draw_line(box.max, box.max - v2(w, 0), color);
	draw_line(box.max, box.max - v2(0, h), color);
}
{% endhighlight %}

Make a quick mental note that the docs for `tigrLine` made it clear the last pixel is *not drawn*, so we add +1 to w/h values to draw one pixel farther.

And here's the program for drawing our box.

{% highlight cpp %}
#include <math.h>
#include "tigr.h"

struct v2
{
	v2() { }
	v2(float x, float y) { this->x = x; this->y = y; }
	float x;
	float y;
};

v2 operator+(v2 a, v2 b) { return v2(a.x + b.x, a.y + b.y); }
v2 operator-(v2 a, v2 b) { return v2(a.x - b.x, a.y - b.y); }
v2 operator*(v2 a, float b) { return v2(a.x * b, a.y * b); }
float len(v2 v) { return sqrtf(v.x * v.x + v.y * v.y); }

struct aabb
{
	aabb() { }
	aabb(v2 min, v2 max) { this->min = min; this->max = max; }
	v2 min;
	v2 max;
};

float width(aabb box) { return box.max.x - box.min.x; }
float height(aabb box) { return box.max.y - box.min.y; }
v2 center(aabb box) { return (box.min + box.max) * 0.5f; }

Tigr* screen;

void draw_point(v2 p, TPixel color) { tigrPlot(screen, (int)p.x, (int)p.y, color); }
void draw_line(v2 a, v2 b, TPixel color) { tigrLine(screen, (int)a.x, (int)a.y, (int)b.x, (int)b.y, color); }

void draw_box(aabb box, TPixel color)
{
	float w = width(box) + 1;
	float h = height(box) + 1;
	draw_line(box.min, box.min + v2(w, 0), color);
	draw_line(box.min, box.min + v2(0, h), color);
	draw_line(box.max, box.max - v2(w, 0), color);
	draw_line(box.max, box.max - v2(0, h), color);
}

TPixel color_white() { return tigrRGB(0xFF, 0xFF, 0xFF); }
TPixel color_black() { return tigrRGB(0, 0, 0); }
TPixel color_red() { return tigrRGB(0xFF, 0, 0); }
TPixel color_green() { return tigrRGB(0, 0xFF, 0); }
TPixel color_blue() { return tigrRGB(0, 0, 0xFF); }

int main()
{
	screen = tigrWindow(640, 480, "Math 101", 0);

	while (!tigrClosed(screen) && !tigrKeyDown(screen, TK_ESCAPE)) {
		tigrClear(screen, color_black());

		aabb box = aabb(v2(120, 120), v2(540, 360));
		draw_box(box, color_white());

		tigrUpdate(screen);
	}

	tigrFree(screen);

	return 0;
}
{% endhighlight %}

![draw_box](/assets/draw_box.png)

And finally let's animate this box a bit. It's time to introduce time... Haha. I'm practicing my dad jokes. `tigrTime` is a nice function that returns the number of seconds since it was last called as a float. We can use it to calculate a `dt` variable once per game loop, standing for `delta time`. This will be used a whole lot later in this article for animating things and calculating movements.

For now we can use time and pass it into cosine and sine functions `cosf` and `sinf` from the `math.h` C runtime header. Go ahead and check out these edits to our program. I added `// New code!` comments on all the edited lines since the last version.

{% highlight cpp %}
#include <math.h>
#include "tigr.h"

struct v2
{
	v2() { }
	v2(float x, float y) { this->x = x; this->y = y; }
	float x;
	float y;
};

v2 operator+(v2 a, v2 b) { return v2(a.x + b.x, a.y + b.y); }
v2 operator-(v2 a, v2 b) { return v2(a.x - b.x, a.y - b.y); }
v2 operator*(v2 a, float b) { return v2(a.x * b, a.y * b); }
float len(v2 v) { return sqrtf(v.x * v.x + v.y * v.y); }

struct aabb
{
	aabb() { }
	aabb(v2 min, v2 max) { this->min = min; this->max = max; }
	v2 min;
	v2 max;
};

float width(aabb box) { return box.max.x - box.min.x; }
float height(aabb box) { return box.max.y - box.min.y; }
v2 center(aabb box) { return (box.min + box.max) * 0.5f; }

Tigr* screen;

void draw_point(v2 p, TPixel color) { tigrPlot(screen, (int)p.x, (int)p.y, color); }
void draw_line(v2 a, v2 b, TPixel color) { tigrLine(screen, (int)a.x, (int)a.y, (int)b.x, (int)b.y, color); }

void draw_box(aabb box, TPixel color)
{
	float w = width(box) + 1;
	float h = height(box) + 1;
	draw_line(box.min, box.min + v2(w, 0), color);
	draw_line(box.min, box.min + v2(0, h), color);
	draw_line(box.max, box.max - v2(w, 0), color);
	draw_line(box.max, box.max - v2(0, h), color);
}

TPixel color_white() { return tigrRGB(0xFF, 0xFF, 0xFF); }
TPixel color_black() { return tigrRGB(0, 0, 0); }
TPixel color_red() { return tigrRGB(0xFF, 0, 0); }
TPixel color_green() { return tigrRGB(0, 0xFF, 0); }
TPixel color_blue() { return tigrRGB(0, 0, 0xFF); }

int main()
{
	screen = tigrWindow(640, 480, "Math 101", 0);

	float t = 0; // New code!
	while (!tigrClosed(screen) && !tigrKeyDown(screen, TK_ESCAPE)) {
		float dt = tigrTime(); // New code!
		t += dt; // New code!
		tigrClear(screen, color_black());

		aabb box = aabb(v2(120, 120), v2(540, 360));
		v2 offset = v2(cosf(t), sinf(t)) * 100.0f; // New code!
		box.min = box.min - offset; // New code!
		box.max = box.max + offset; // New code!
		draw_box(box, color_white());

		tigrUpdate(screen);
	}

	tigrFree(screen);

	return 0;
}
{% endhighlight %}

![morphing_box](/assets/morphing_box.gif)

### More to Come!

THIS POST IS A WIP

I'LL ADD MORE SOON
