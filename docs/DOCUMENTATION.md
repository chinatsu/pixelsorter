
# PixelSorting – Documentation

A detailed description of the possible sort parameters.

## Command Line Options

To run the script, use the command `python pixelsort.py <image> -o <result>` to sort a given image
and store it in `<result>`. Use the `--log` flag to view progress for sorting particularly large images.
The other settings and options are described below.
For the example figures, the original image used is this:

![Original Image][original]

#### Sorting Interval:

The `-i` or `--max-interval` argument specified the length of individual intervals of pixels to sort.
By default, whole rows of the image are sorted at once. However, when `-i <INTEGER>` is used,
pixels are sorted in consecutive groups of the given length.

With `-i` omitted:

![Default sorting interval][default]

With `-i 50`:

![Sorting interval of 50px][sort50]

With `-i 100`:

![Sorting interval of 100px][sort100]

The `-r` or `--randomize` argument will cause pixel intervals to have random length between 1 and `INTERVAL` pixels,
where `INTERVAL` is the amount specified by the `-i` flag. If the `-i` flag is omitted, then `-r` has no effect.

With `-i 50`:

![Uniform sorting interval of 50px][sort50]

With `-i 50 -r`:

![Random sorting interval of 50px][sort50random]

The `--progressive-amount <AMOUNT>` flag takes a floating point amount,
and increases the sorting interval by the given amount each row, as a fraction of the total interval.
For instance, with `-i 100` and `--progressive-amount 0.01`,
the sorting interval would increase by 1 pixel each row.
Since there are hundreds of rows in an image, values less than `0.001` usually work best.

With `-i 100 -r` and no progressive sorting:

![Sorting with random interval of 100][sort100random]

With `-i 100 -r --progressive-amount 0.0001`:

![Sorting with random interval of 100 and progressive sorting][sort100-progressive]

(It is difficult to see, but the sorting intervals near the top rows are smaller than at the bottom.)

#### Sorting Keys:

One can apply a custom function to pixels before they are sorted,
allowing one to sort by brightness, saturation, or some other metric.
This is done using the `-s <KEY>` or `--sortkey <KEY>` flag, where `<KEY>`
is one of a set of predefined function names.
By default, pixels are sorted simply using their numerical value.
Note that sorting is stable, so if pixels are mapped to the same value,
their relative position will be unchanged.

Default sorting method:

![Default sorting method][sort-default]

Using `-s sum` will sort by the sum of the pixel's (R,G,B) values:

![Sorting by pixel sum][sort-sum]

Using `-s red|green|blue` will sort using only the given channel of each pixel.
Sorting with `-s blue` is shown below.

![Sorting by pixel sum][sort-blue]

Other sorting methods include:
 - `chroma`
 - `hue`
 - `intensity`
 - `lightness`
 - `luma`
 - `random`
 - `saturation`
 - `value`

Using the `-R` or `--reverse` flag will reverse the sorting order of the pixels:

With `-i 50`:

![Uniform sorting interval of 50px][sort50]

With `-i 50 -R`:

![Reversed sorting interval of 50px][sort50reverse]

The `-d` or `--discretize <INTEGER>` flag will take pixel values,
divide them by the given amount, and cast to an integer.
This means that pixels with small variations will be binned into the same categories,
and not sorted relative to each other.
This allows one, for instance, to only move the "brightest" pixels while the other pixels stay in place.

With `-s sum` and `-d` omitted:

![Sorting by sum][sort-sum]

With `-s sum -d 100`:

![Sorting by sum with -d 100][sort-sum-d100]

With `-s sum -d 200`:

![Sorting by sum with -d 200][sort-sum-d200]

#### Sorting Paths

By default, all sorting is done in horizontal intervals through the image.
However, several other sorting paths are available.

The `-v` flag will sort the image vertically instead of horizontally.
Otherwise, one can explicitly specify what type of path to use with the `-p` or `--path` flag:
 - `concentric` Instead of rows, sorts concentric rectangles around the image.

![Sorting with concentric path][sort100-concentric]

 - `diagonal` Sorts in diagonal lines that move from the top left to the bottom right.

![Sorting with diagonal path][sort100-diagonal]

 - `horizontal`: The default, sorts horizontally row by row

![Sorting with horizontal path][sort100random]

 - `vertical`: Sorts vertically, column by column

![Sorting with concentric path][sort100-vertical]

Sorting paths are explained in detail in [the paths documentation.](docs/PATHS.md)

#### Path modifiers

Several modifiers can be applied to individual intervals of pixels.
 - `mirror` - The mirror modifier takes an interval of pixels,
and moves each successive pixel to either the beginning or end of the interval.
 This makes intervals look symmetric, and borders between intervals are more continuous.
 This is especially useful for intervals that form loops, such as the `concentric` path.

   With `-s luma -p concentric --mirror`:
   ![Sorting with concentric paths and --mirror][sort-concentric-mirror]

 - `splice` - The splice modifier splits an interval of pixels at a certain position,
 and places the front half after the second half.
 The split position is given as a number between `0` and `1`, with `0` reprsenting the start of the list
  and `1` representing the end.

   With `-s luma -p concentric --splice 0`:
   ![Sorting with concentric paths and splice=0][sort-concentric-splice-0]

   With `-s luma -p concentric --splice 0.3`:
   ![Sorting with concentric paths and splice=0.3][sort-concentric-splice-0.3]

  - `splice-random` - Randomly splices each interval at a different position.

   With `-s luma -p concentric --splice-random`:
   ![Sorting with concentric paths and --splice-random][sort-concentric-splice-random]


#### Edge Detection

Edge detection allows one to break up sorting intervals are points of high contrast
(i.e. where there are edges in the image). This creates the effect of only sorting within low-contrast regions.
The `-e <THRESHOLD>` or `--edge-threshold <THRESHOLD>` flag can be used to specify a numerical threshold above
pixels are considered "edges". Intuitively, this means that low thresholds will cause images with very short sorting
intervals, while very high threholds will cause the image to be almost completely sorted, as if `-e` had
not been specified at all. However, the exact effects depend on the image specified.

The original image:

![Original Image][original]

Sorting with `-e 50`:

![Sorting with -e 50][sort-edge-detect-50]

Sorting with `-e 100`:

![Sorting with -e 100][sort-edge-detect-100]

Sorting with `-e 200`:

![Sorting with -e 200][sort-edge-detect-200]

Sorting without the `-e` flag:

![Sorting without -e][default]

Note that with low `-e` values, nearly the whole image is undisturbed,
since almost everything is considered an 'edge',
but with high `-e` values everything is sorted except for the starkly-contrasted trees.

### Image Thresholds

Image colors can also be used to determine sorting intervals. When `--image-threshold <FLOAT>`
is specified, pixels that are too dark or too bright are not sorted,
and delimit sorting intervals. The value given determines the thresholds for pixel brightness.
This value ranges from 0 (meaning all pixels are within the brightness threshold, and everything gets sorted),
to 1 (meaning all pixels are out of the brightness threshold, and nothing is sorted).

The original image:

![Original Image][original]

Sorting with `--image-threshold 0`:

![Sorting with --image-threshold 0][sort-image-threshold-0]

Sorting with `--image-threshold 0.6`:

![Sorting with --image-threshold 0.6][sort-image-threshold-0.6]

Sorting with `--image-threshold 1`:

![Sorting with --image-threshold 1][sort-image-threshold-1]

### Image Masks

One can also specify a custom image that will be used as a sort mask.
In this case, pixels that with a brightness of more than half will be used to delimit sort intervals,
while darker pixels will be ignored. This can be done with the `--image-mask <FILENAME>` flag.
Note that the image mask must have *exactly* the same size

Sorting without `--image-mask` and with `-R`:

![Sorting without image mask][default]

An image mask that can be applied to an image:

![Image mask][image-mask]

Sorting with `--image-mask image-mask.jpg` and `-R`:

![Sorting with image mask][sort-image-mask]

The parts of the image corresponding to black pixels in the pixel mask are sorted,
while the parts corresponding to white pixels are not.

### Tiles

One can also sort individual tiles in the image. With the `--use-tiles` flag,
the image will be broken into a grid of recantgular tiles, and each tile will be sorted as if it were a separate image.
By default, tiles are 50 pixels on a size, but this can be changed with the `--tile-x <INT>` and `--tile-y <INT>` flags

Sorting with `--use-tiles` and `-p diagonal`

![Sorting with tiles with default arguments][sort-tiles-default]

Sorting with `--use-tiles --tile-x 20 --tile-y 30` and `-p diagonal`

![Sorting with custom sized tiles][sort-tiles-custom-size]

The `--tile-density <FLOAT>` flag will only sort the given fraction of tiles. For instance, with `--tile-density 0.5`,
only half of the tiles will be sorted, while the others are unmodified. By default, all tiles are sorted.
Also, if the density is less than 1, tiles are sorted uniformly across the image.
If the `--randomize-tiles` flag is specified, then tiles will be randomly sorted,
with the probability given by `--tile-density <FLOAT>`.

Sorting with `--use-tiles --tile-x 20 --tile-y 20 --tile-density 0.5` and `-p diagonal-single`

![Sorting with tiles and density 0.5][sort-tiles-half]

Sorting with `--use-tiles --tile-x 20 --tile-y 20 --tile-density 0.5 --randomize-tiles` and `-p diagonal-single`

![Sorting with custom sized tiles][sort-tiles-half-random]


### Channels

It is also possible to sort individual channels of an RGB image, using the `--channel {red,green,blue}` flag.

Sorting with `--channel red` and `-s sum -i 100 -r`

![Sorting red channel only][sort-channel-red]

### Animation

This pixelsorting library also includes tools for creating and modifying animated gifs.
To change a certain parameter over time, on can use the `--animate "PARAM START STOP N_STEPS"` flag.
For instance, to change the sorting interval from 10 to 100 over 15 frames, one could specify
`--animage "max_interval 10 100 15"`.
Note that currently, this is only possible for parameters that have numerical values.
Also, only one parameter can be animated at a time.

Animated sort, with  `--animate "max_interval 2 30 15"` and `-s sum -r`:

![Animated sort][sort-animated]

To save intermediate frames, use the `--save-frames` flag.

One can also input animated gifs, and each frame of the gif will be modified according to the command line arguments.
If `--animate "..."` is specified as well, the animated settings will be applied frame-by-frame to the gif.

[//]: # "Figures"
[original]: figures/original.jpg
[default]: figures/sort-sum.jpg
[sort50]: figures/sort-50.jpg
[sort100]: figures/sort-100.jpg
[sort50random]: figures/sort-50-random.jpg
[sort100random]: figures/sort-100-random.jpg
[sort-default]: figures/sort-default.jpg
[sort-sum]: figures/sort-sum.jpg
[sort-blue]: figures/sort-blue.jpg
[sort50reverse]: figures/sort-50-reverse.jpg
[sort-sum-d100]: figures/sort-sum-d100.jpg
[sort-sum-d200]: figures/sort-sum-d200.jpg
[sort100-progressive]: figures/sort-100-progressive.jpg
[sort100-concentric]: figures/sort-100-concentric.jpg
[sort100-diagonal]: figures/sort-100-diagonal.jpg
[sort-diagonal-single]: figures/sort-diagonal-single.jpg
[sort100-random-walk]: figures/sort-100-random-walk.jpg
[sort100-random-walk-horizontal]: figures/sort-100-random-walk-horizontal.jpg
[sort100-random-walk-vertical]: figures/sort-100-random-walk-vertical.jpg
[sort100-vertical]: figures/sort-100-vertical.jpg
[sort-concentric-mirror]: figures/sort-concentric-mirror.jpg
[sort-concentric-splice-0]: figures/sort-concentric-splice-0.jpg
[sort-concentric-splice-0.3]: figures/sort-concentric-splice-0.3.jpg
[sort-concentric-splice-random]: figures/sort-concentric-splice-random.jpg
[sort-edge-detect-50]: figures/sort-edge-detect-50.jpg
[sort-edge-detect-100]: figures/sort-edge-detect-100.jpg
[sort-edge-detect-200]: figures/sort-edge-detect-200.jpg
[sort-image-threshold-0]: figures/sort-image-threshold-0.jpg
[sort-image-threshold-0.6]: figures/sort-image-threshold-0.6.jpg
[sort-image-threshold-1]: figures/sort-image-threshold-1.jpg
[image-mask]: figures/image-mask.jpg
[sort-image-mask]: figures/sort-image-mask.jpg
[sort-tiles-default]: figures/sort-tiles-default.jpg
[sort-tiles-custom-size]: figures/sort-tiles-custom-size.jpg
[sort-tiles-half]: figures/sort-tiles-half.jpg
[sort-tiles-half-random]: figures/sort-tiles-half-random.jpg
[sort-channel-red]: figures/sort-channel-red.jpg
[sort-animated]: figures/sort-animated.gif
