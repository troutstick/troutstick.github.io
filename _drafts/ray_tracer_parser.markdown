---
layout: post
title:  "Simple Ray Tracing in Rust 2: A .obj Parser"
categories: ray_tracer
---

In Part 1, we wrote some basic structs to
represent our scenes, and covered the essentials of
the algorithm we'll use to render them into an image.
This begs the question: How do we construct these scenes?

Building a 3D scene from scratch would be an involved effort, of course.
Thankfully, we don't have to. Artists, animators, and computer graphics
researchers alike have built many file formats to store objects
in exportable files that can be shared from computer to computer
and parsed by your favorite rendering suite, such as Blender.

For my ray tracer, I'll implement a subset of Wavefront's `.obj` file format,
which is open source and widely used.

## Argument Parsing

Let's start simple. We compile and run our program with Rust's trusty package manager,
`cargo`, and we pass in an argument: the path to our `.obj` file.

{% highlight bash %}
cargo r ./path/to/file.obj
{% endhighlight %}