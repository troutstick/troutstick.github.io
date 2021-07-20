---
layout: post
title:  "Simple Ray Tracing in Rust 1: Basics"
categories: ray_tracer
---

Many a programmer have stumbled into the rabbit hole of building a ray tracer. It brings the creative allure of beautiful
graphics, and hides an astonishing amount of depth within its possible implementations. So let's try our hand at building one,
all the while taking advantage of the Rust programming language's incredible speed, flexibility, and modernity.

If you want to get into the conceptual weeds, Steven Waterman already made [an excellent blogpost](https://blog.scottlogic.com/2020/03/10/raytracer-how-to.html) on intuitively building a ray tracer. This series will be about translating these high
level concepts into working Rust code.

## 3D Objects

|![Triangles together form a dolphin](/assets/Dolphin_triangle_mesh.png){:height="200px"}|
|*Many triangles form a dolphin. [Source.](https://en.wikipedia.org/wiki/File:Dolphin_triangle_mesh.png)*|

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

/// A wrapper around a float that allows us to represent angles.
#[derive(Debug, Copy, Clone, PartialEq)]
struct Radian(f64);

impl Radian {
    #[inline]
    fn sin(&self) -> f64 { self.0.sin() }
    
    #[inline]
    fn cos(&self) -> f64 { self.0.cos() }
}
{% endhighlight %}

#### Safety Using Rust's Abstractions

Notice that we define a `Radian` struct
instead of just using an `f64` (a 64-bit 
floating point number,
equivalent to
a `double` in languages like C or Java). This
showcases one of the great strengths of Rust:
zero cost abstractions. When compiled down
into machine code, the `Radian` struct will
be treated no differently than any other `f64`,
enabling operations on it to run at blazing speed.
Yet with this code, we ensure compiler-enforced safety.
By specifically writing only `sin` and `cos` methods
for `Radian`, we make sure that we won't be able
to interact with it in any unintended ways, such
as performing a square root operation on it:

{% highlight rust %}
let r = Radian(0.0);
let r_sin = r.sin();

// won't compile since we do not define a sqrt method
let r_sqrt = r.sqrt();
{% endhighlight %}

## So... How Do We Ray Trace?

What does ray tracing actually entail? First, we
must have a basic understanding of how rays of light are transformed into
our vision in the real world.

Of course, we do some idealization so that
we don't bog ourselves in the details of quantum particles.
We instead can approximate light as traveling from
straight lines out of a light source.

### A Simple Brightness Model

When this light hits a surface, some of it will
be scattered, and a tiny proportion of this might
be lucky enough to be sent towards your eyes (or a camera).
There are many factors in the real world that affect apparent
brightness and color, but
for the sake of simplicity, we will 
assume that our surfaces exhibit [Lambertian reflection](
    https://en.wikipedia.org/wiki/Lambertian_reflectance
). In this model, a surface scatters light equally in all
directions, and the only factors affecting brightness are:

1. The apparent brightness of the light source
2. The angle between the light source and the surface
    * A surface perpendicular to the light will be
    maximally bright
    * A surface parallel to the light will be completely dark

### From Reflection To Camera

|![Camera and View Plane](/assets/ViewFrustum.svg){:height="200px"}|
|*A camera with two view planes. [Source.](https://commons.wikimedia.org/wiki/File:ViewFrustum.svg)*|

After calculating the brightness of a surface, we
trace the path that the light would take to the camera.
The intersection point between this path and the view
plane corresponds to a pixel that we draw on our screen.
From here, we simply repeat until we have calculated the
brightness and color of every single pixel.

## Drawing Backwards

Of course, you might have noticed that the above method
could be very computationally wasteful. What if
the light ray never even intersects our view plane?

This naturally leads to a clever first optimization that
pretty much any ray tracer will implement. Instead of drawing
light from the light source to the camera, we draw
*from the camera to the light source*! This saves us
orders of magnitude of ray calculations.

## Conclusion

A quick recap of our basic algorithm:

1. Draw a light ray from the `Camera` through
each pixel-corresponding region of our `ViewPlane`
2. For every surface (`Triangle`), determine if it intersects our ray
    * At the closest intersection between this ray and
    a surface, calculate the brightness of said surface
    * Brightness grows with how perpendicular the
    surface is to light source
3. Repeat for every pixel

Alright, let's get coding!