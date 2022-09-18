---
layout: post
title:  "Character Controllers"
date:   2019-03-09
categories: collision-detection
---
I created a [demo called player2d](https://github.com/RandyGaul/player2d) by implementing a swept 2D character controller, showcasing a bunch of [cute_headers](https://github.com/RandyGaul/cute_headers) in action.

![fabulous_demo](/assets/fabulous_demo.gif)

There are also a set of slides made for a talk at a local university available here: [download link](https://github.com/RandyGaul/randygaul.github.io/blob/gh-pages/assets/R.Gaul_Character_Controllers.pptx?raw=true). The demo should be a good educational tool to showcase one way to implement a robust character controller in 2D that can gracefully avoid tunneling problems, and [edge-catching problems](https://box2d.org/posts/2020/06/ghost-collisions/), all the while exposing a useful API to the rest of the project.

Hopefully someone finds the content useful! As always, feel free to ask questions or post comments on github! I enjoy talking about game physics whenever I get a chance :)
