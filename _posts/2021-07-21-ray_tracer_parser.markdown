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

Our plan is to parse this file into a 3D scene within our program.
First we collect the argument:

{% highlight rust %}
// in main.rs

use std::env::args;

fn main() {
    let args: Vec<String> = args().collect();

    let input_path = if args.len() != 2 {
        panic!("wrong number of args provided");
    } else {
        args[1].as_str()
    };

    println!("Processing `{}`...", input_path);

    // ...
}
{% endhighlight %}

Let's define a helper function that opens a file and returns a reader
for easy line processing. We can take advantage of some cool
functions that are already provided in Rust's standard library!

{% highlight rust %}
// additional imports
use std::fs::File;
use std::io::BufRead;
use std::io;
use std::path::Path;

fn read_lines<P>(filename: P) -> io::Result<io::Lines<io::BufReader<File>>>
    where P: AsRef<Path>, {
    let file = File::open(filename)?;
    Ok(io::BufReader::new(file).lines())
}
{% endhighlight %}

With this helper, we can lazily process lines of text from our `.obj` file.
Back in the main function, we can use `if let` syntax to idiomatically
instantiate our line reader for further processing:

{% highlight rust %}
fn main() {
    // arg processing...

    let triangles = if let Ok(lines) = read_lines(input_path) {
        // do something with `lines`
    } else {
        panic!("Failed to parse path. Did you enter a valid filename?");
    };

    // ...
}
{% endhighlight %}

Time to get our hands dirty. We now go through our `.obj` file
line by line, translating it into a `Vec<Triangle>`. Conveniently enough,
Wikipedia has a [neat overview](https://en.wikipedia.org/wiki/Wavefront_.obj_file#File_format)
of how `.obj` files store data.

First you go parse each line one by one, translating it into a corresponding data structure.
Rust's `match` blocks are perfect for this task. We parse each line depending on the initial
word:
1. Lines starting with `v` correspond to the coordinates of triangle vertices
2. Lines starting with `f` correspond to vertex triplets that make up triangles

With this information, we can begin building our triangle mesh:
{% highlight rust %}
/// Parse the input into a set of triangles to render.
fn parse_input(lines: io::Lines<io::BufReader<File>>) -> Vec<Triangle> {
    let mut vertex_coords = Vec::new();
    let mut faces = Vec::new();
    for line in lines {
        if let Ok(s) = line {
            let mut line_iter = s.split_ascii_whitespace();
            if let Some(first_word) = line_iter.next() {
                match first_word {
                    "v" => {
                        let coords: Vec<f64> = line_iter
                            .map(|s| s.parse::<f64>().unwrap())
                            .collect();
                        if coords.len() != 3 {
                            panic!("unable to parse non 3d coordinates");
                        }
                        vertex_coords.push(coords);
                    },
                    "f" => {
                        let vertices: Vec<usize> = line_iter
                            .map(|s| s.parse::<usize>().unwrap())
                            .map(|i| i-1) // normalize into 0 index
                            .collect();
                        if vertices.len() != 3 {
                            panic!("unable to parse non-triangle polygons");
                        }
                        faces.push(vertices);
                    },
                    "#" => (), // ignore comment line
                    _ => panic!("only v and f lines readable in `.obj` files"),
                }
            }
        }
    }

    // ...
}
{% endhighlight %}

After extracting triangle vertex and face information
in the form of somewhat disorganized lists of numbers,
we can finally start instantiating the structs we
previously defined.

We take advantage of Rust's awesome anonymous functions, known
to Rustaceans as closures. We functionally map over our vector
of triangle faces, extracting their vertex information,
and turning them into native `Triangle` structs to be processed
by our program:

{% highlight rust %}
// new triangle instantiator
impl Triangle {
    fn new(v1: Vector, v2: Vector, v3: Vector) -> Triangle {
        Triangle { v1, v2, v3 }
    }
}

fn parse_input(lines: io::Lines<io::BufReader<File>>) -> Vec<Triangle> {
    // define `faces` and `vertex_coords`...

    let get_3d_vect = |coord: &Vec<f64>| -> Vector {
        Vector::new(coord[0], coord[1], coord[2])
    };

    let get_vertices = |face: &Vec<usize>| -> Triangle {
        let v1 = get_3d_vect(&vertex_coords[face[0]]);
        let v2 = get_3d_vect(&vertex_coords[face[1]]);
        let v3 = get_3d_vect(&vertex_coords[face[2]]);
        Triangle::new(v1, v2, v3)
    };

    let triangles: Vec<Triangle> = faces
        .iter()
        .map(get_vertices)
        .collect();

    triangles
}
{% endhighlight %}

Finally, we return to our main function, where we can use the nifty
input parser we just defined:

{% highlight rust %}
fn main() {
    // arg processing...

    let triangles = if let Ok(lines) = read_lines(input_path) {
        // NEW 
        parse_input(lines)
    } else {
        panic!("Failed to parse path. Did you enter a valid filename?");
    };

    // ...
}
{% endhighlight %}

## Conclusion

In this part, we learned how to extract a CLI argument, open a file using
tools from Rust's standard library, and parse each of the lines
in our `.obj` file into
coordinate information of our vertices. Then we converted raw vertex
data into `Triangle` structs that can easily be digested by our program later on.

Stay tuned; in the coming parts we will organize our measly `Triangles` into
a 3D scene, and lay down a solid foundation for our ray tracing loop!