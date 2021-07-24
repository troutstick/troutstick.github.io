---
layout: post
title:  "Simple Ray Tracing in Rust 4: Exporting PNG Outputs"
categories: ray_tracer
---

Now that we're able to output the pixels of our ray traced image,
we can work on exporting it in a format that can be easily displayed,
such as a `.png` file. Of course, building a `.png` from scratch
would be quite an undertaking. Instead, we will output our
image as a `.ppm` file, a much simpler format.

## .ppm Formatting

Let's take a look at an example `.ppm` file.

```
P3          # This is an RGB image
3 2         # This image is 3 pixels wide and 2 pixels tall
255         # RGB values go from 0 to 255
255 255 255 # The top left pixel is white
255 255 255 
0   0   255 # The top right pixel is blue
100 0   64
10  10  10
255 0   0   # The bottom right pixel is red
```

After 3 initial metadata lines,
each line describes a pixel's color.
The pixels are ordered from left to right, top to bottom.

## Our .ppm Writer

Recall that our main function currently looks like this:

{% highlight rust %}
fn main() {
    let args: Vec<String> = args().collect();

    let input_path = if args.len() != 2 {
        panic!("wrong number of args provided");
    } else {
        args[1].as_str()
    };

    let triangles = if let Ok(lines) = read_lines(input_path) {
        parse_input(lines)
    } else {
        panic!("Failed to parse path. Did you enter a valid filename?");
    };

    let scene = Scene::new(triangles);

    let pixels = scene.iterate_over_rays();

    // ...
}
{% endhighlight %}

Let's add a simple helper method on scene that will write our output.
We'll implement it as a method on `Scene`, and move the
call to `iterate_over_rays` from `main` to here:

{% highlight rust %}
impl Scene {
    fn render_to_output(&self, mut writer: BufWriter<File>) {
        let pixels = self.iterate_over_rays();

        let num_cols = self.camera.view_plane.res_width;
        let num_rows = self.camera.view_plane.res_height;
        writer.write(format!("P3\n{} {}\n255\n", num_cols, num_rows).as_bytes()).unwrap();
        for p in pixels {
            let s = format!("{} {} {}\n", p.r, p.g, p.b);
            writer.write(s.as_bytes()).unwrap();
        }
        writer.flush().unwrap();
    }
}
{% endhighlight %}

With this helper and a couple of new imports
from Rust's standard library, we can put together a compact
and robust `main` function. It creates an output file
and writes our rendering to it in a `.ppm` format,
just as we want:

{%- highlight rust -%}
// update import
use std::io::{BufWriter, Write, BufRead};

// new const
const OUTPUT_FOLDER: &str = "./images/output";

fn main() {
    // define triangles...

    let scene = Scene::new(triangles);

    // create a file
    let f = File::create(format!("{}/{}_{}.ppm", OUTPUT_FOLDER, input_filename, i))
            .expect("Unable to create file");
    let f = BufWriter::new(f);
    // update iterate_over_rays logic
    scene.render_to_output(f);
}
{%- endhighlight -%}

Incredibly, this is essentially all we need for our program.
With just this, we're now able to crunch a `.obj` input
into a `.ppm` output that a good image viewer will
be able to display on your computer screen!

## Exporting as PNG Files

There's one more thing we can do here. For convenience,
we can convert our `.ppm` output file into a much
more universally recognized format like `.png`.
This can be easily accomplished with a tool
like ImageMagick; using the command line on Linux,
we can write `convert path/to/x.ppm path/to/y.png`
to create a new `.png` file from `x.ppm`.

For this program, we can use a Bash script to
automate the typing away:

{%- highlight bash -%}
#!/bin/bash
# convert program output into PNGs to be exported

output="./images/output"

echo "Exporting images..."

shopt -s nullglob
for filename in $output/*.ppm
do
  filename=$(basename -- "$filename")
  filename="${filename%.*}"
  echo "Converting $filename.ppm to .png"
  convert $output/$filename.ppm $output/$filename.png
  rm $output/$filename.ppm
done
{%- endhighlight -%}

This little script will look through all generated
outputs, convert them into `.png` files,
and delete the originals. Nice! Let's put
it in `./scripts/export.sh`.

We can finally organize everything in a Makefile
so that we can execute our program from
the command line without having to memorize long,
tedious
sequences to type in.

{%- highlight make -%}
OUTPUT=images/output/
INPUT=images/input/
IMAGE_NAME=teapot

run : src $(INPUT)
	cargo r --release $(IMAGE_NAME)

all : run
	./scripts/export.sh
	eog ./$(OUTPUT)

display : $(OUTPUT)
	./scripts/export.sh
	eog ./$(OUTPUT)

clean : $(OUTPUT)
	rm -r ./$(OUTPUT)
{%- endhighlight -%}

Here, we make use of `eog` as an image displayer.

* `make run` produces `.ppm` outputs for given inputs
* `make all` produces `.png` outputs for given inputs and displays them
* `make display` displays pre-rendered output files
* `make clean` removes all files in the output folder

## Conclusion

Well this was short and sweet. And... what's this,
we can actually see our renderings now! Awesome!
And we even added some convenience scripts
to smooth out our workflow.
Now we have the meat of our program done. Time to optimize...