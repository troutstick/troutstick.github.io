---
layout: post
title:  "A Simple Ray Tracer in Rust: Shapes"
categories: ray_tracer
---

Many a programmer have stumbled into the rabbit hole of building a ray tracer. It brings the creative allure of beautiful
graphics, and hides an astonishing amount of depth within its possible implementations. So let's try our hand at building one,
all the while taking advantage of the Rust programming language's incredible speed, flexibility, and modernity.

I would love to get into the conceptual weeds, but Steven Waterman already made [an excellent blogpost](https://blog.scottlogic.com/2020/03/10/raytracer-how-to.html) on intuitively building a ray tracer. This post will instead focus on translating high
level concepts into working Rust code.

## 3D Objects

![Triangles together form a dolphin](/assets/Dolphin_triangle_mesh.png){:height="200px"}

First, we need to be able to represent the 3D scenes that we want to render. In computer graphics,
the most basic representation of an object is a bunch of triangles.

First we define a `Vector` struct, which
represents an offset in good old 3D Euclidean space.

{% highlight rust %}
struct Vector {
    dx: f64,
    dy: f64,
    dz: f64,
}
{% endhighlight %}

Each `Vector` is a 3D coordinate, and 3 of them
combine to form the vertices of a triangle.

{% highlight rust %}
struct Triangle {
    v1: Vector,
    v2: Vector,
    v3: Vector,
}
{% endhighlight %}

Although it may sound simple, practically any 3D object
that you can think of can be approximated as just a
collection of many (possibly thousands, or even millions)
triangles. Let's call a collection of triangles a `Scene`;
we'll keep our rendering code neatly organized by implementing our rendering functions on our `Scene` structs.

{% highlight rust %}
/// A set of triangles together make up a scene that can be viewed by a camera.
struct Scene {
    camera: Camera,
    triangles: Vec<Triangle>,
}

impl Scene {
    // We'll slowly build up our rendering around this scaffolding
}
{% endhighlight %}


Now let's set up the `Camera` for our scene,
from which we will view our triangles and convert
them into pixels on a screen.

{% highlight rust %}
struct Camera {
    // where our camera rays converge
    pos: Vector,
    // `pitch` and `yaw` determine where the camera is pointed
    pitch: Radian,
    yaw: Radian,
    // the virtual screen onto which our pixels are projected.
    view_plane: ViewPlane,
}

/// A view plane for a camera. The view plane is situated 1.0 units away from the camera.
struct ViewPlane {
    /// The height and width of each pixel.
    pixel_size: f64,
    /// How many pixels wide a view is.
    res_width: usize,
    /// How many pixels tall a view is.
    res_height: usize,
}
{% endhighlight %}
