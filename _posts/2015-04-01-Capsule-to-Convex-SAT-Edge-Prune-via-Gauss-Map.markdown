---
layout: post
title:  "Capsule to Convex – SAT Edge Prune via Gauss Map"
date:   2015-04-01
categories: math collision-detection
---
In 2013 Dirk Gregorius of Valve presented at GDC on the topic of the Separating Axis Theorem. In his talk he studied an useful property of the Minkowski Difference between two 3D convexes: edge pairs from each shape may or may not contribute to the convex hull of the Minkowski Difference.

It is possible to derive simple predicate functions to reduce the number of edge pair queries, and to simplify implementation, during collision detection via the Separating Axis Theorem. This article shows a derivation of this predicate for the Capsule to Convex case.

Here is the [PDF containing my own derivation](https://github.com/RandyGaul/randygaul.github.io/raw/gh-pages/assets/R.Gaul-Capsule-to-Convex.pdf).
