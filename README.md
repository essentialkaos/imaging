<p align="center"><a href="#readme"><img src=".github/images/card.svg"/></a></p>

<p align="center">
  <a href="https://kaos.sh/g/imaging"><img src=".github/images/godoc.svg"/></a>
  <a href="https://kaos.sh/w/imaging/ci"><img src="https://kaos.sh/w/imaging/ci.svg" alt="GitHub Actions CI Status" /></a>
  <a href="https://kaos.sh/w/imaging/codeql"><img src="https://kaos.sh/w/imaging/codeql.svg" alt="GitHub Actions CodeQL Status" /></a>
  <a href="#license"><img src=".github/images/license.svg"/></a>
</p>

<p align="center"><a href="#image-processing">Image processing</a> • <a href="#faq">FAQ</a> • <a href="#usage-example">Usage example</a> • <a href="#license">License</a></p>

<br/>

Package `imaging` provides basic image processing functions (_resize, rotate, crop, brightness/contrast adjustments, etc._).

All the image processing functions provided by the package accept any image type that implements `image.Image` interface as an input, and return a new image of `*image.NRGBA` type (_32bit RGBA colors, non-premultiplied alpha_).

### Image processing
#### Image resizing

```go
// Resize srcImage to size = 128x128px using the Lanczos filter.
dstImage128 := imaging.Resize(srcImage, 128, 128, imaging.Lanczos)

// Resize srcImage to width = 800px preserving the aspect ratio.
dstImage800 := imaging.Resize(srcImage, 800, 0, imaging.Lanczos)

// Scale down srcImage to fit the 800x600px bounding box.
dstImageFit := imaging.Fit(srcImage, 800, 600, imaging.Lanczos)

// Resize and crop the srcImage to fill the 100x100px area.
dstImageFill := imaging.Fill(srcImage, 100, 100, imaging.Center, imaging.Lanczos)
```

Imaging supports image resizing using various resampling filters. The most notable ones:
- `Lanczos` - A high-quality resampling filter for photographic images yielding sharp results.
- `CatmullRom` - A sharp cubic filter that is faster than Lanczos filter while providing similar results.
- `MitchellNetravali` - A cubic filter that produces smoother results with less ringing artifacts than CatmullRom.
- `Linear` - Bilinear resampling filter, produces smooth output. Faster than cubic filters.
- `Box` - Simple and fast averaging filter appropriate for downscaling. When upscaling it's similar to NearestNeighbor.
- `NearestNeighbor` - Fastest resampling filter, no antialiasing.

The full list of supported filters: `NearestNeighbor`, `Box`, `Linear`, `Hermite`, `MitchellNetravali`, `CatmullRom`, `BSpline`, `Gaussian`, `Lanczos`, `Hann`, `Hamming`, `Blackman`, `Bartlett`, `Welch`, `Cosine`.

Custom filters can be created using ResampleFilter struct.

**Resampling filters comparison**

Original image:

![srcImage](testdata/branches.png)

The same image resized from 600x400px to 150x100px using different resampling filters. From faster (_lower quality_) to slower (_higher quality_):

Filter                    | Resize result
--------------------------|---------------------------------------------
`imaging.NearestNeighbor` | ![dstImage](testdata/out_resize_nearest.png)
`imaging.Linear`          | ![dstImage](testdata/out_resize_linear.png)
`imaging.CatmullRom`      | ![dstImage](testdata/out_resize_catrom.png)
`imaging.Lanczos`         | ![dstImage](testdata/out_resize_lanczos.png)

#### Gaussian Blur

```go
dstImage := imaging.Blur(srcImage, 0.5)
```

Sigma parameter allows to control the strength of the blurring effect.

Original image | Sigma = 0.5 | Sigma = 1.5
---------------|------------ |------------
![srcImage](testdata/flowers_small.png) | ![dstImage](testdata/out_blur_0.5.png) | ![dstImage](testdata/out_blur_1.5.png)

#### Sharpening

```go
dstImage := imaging.Sharpen(srcImage, 0.5)
```

`Sharpen` uses gaussian function internally. Sigma parameter allows to control the strength of the sharpening effect.

Original image | Sigma = 0.5 | Sigma = 1.5
---------------|-------------|------------
![srcImage](testdata/flowers_small.png) | ![dstImage](testdata/out_sharpen_0.5.png) | ![dstImage](testdata/out_sharpen_1.5.png)

#### Gamma correction

```go
dstImage := imaging.AdjustGamma(srcImage, 0.75)
```

Original image | Gamma = 0.75 | Gamma = 1.25
---------------|--------------|-------------
![srcImage](testdata/flowers_small.png) | ![dstImage](testdata/out_gamma_0.75.png) | ![dstImage](testdata/out_gamma_1.25.png)

#### Contrast adjustment

```go
dstImage := imaging.AdjustContrast(srcImage, 20)
```

Original image | Contrast = 15 | Contrast = -15
---------------|---------------|---------------
![srcImage](testdata/flowers_small.png) | ![dstImage](testdata/out_contrast_p15.png) | ![dstImage](testdata/out_contrast_m15.png)

#### Brightness adjustment

```go
dstImage := imaging.AdjustBrightness(srcImage, 20)
```

Original image | Brightness = 10 | Brightness = -10
---------------|-----------------|-----------------
![srcImage](testdata/flowers_small.png) | ![dstImage](testdata/out_brightness_p10.png) | ![dstImage](testdata/out_brightness_m10.png)

#### Saturation adjustment

```go
dstImage := imaging.AdjustSaturation(srcImage, 20)
```

Original image | Saturation = 30 | Saturation = -30
---------------|-----------------|-----------------
![srcImage](testdata/flowers_small.png) | ![dstImage](testdata/out_saturation_p30.png) | ![dstImage](testdata/out_saturation_m30.png)

#### Hue adjustment

```go
dstImage := imaging.AdjustHue(srcImage, 20)
```

Original image                     | Hue = 60                                     | Hue = -60
-----------------------------------|----------------------------------------------|---------------------------------------------
![srcImage](testdata/flowers_small.png) | ![dstImage](testdata/out_hue_p60.png) | ![dstImage](testdata/out_hue_m60.png)

### FAQ

#### Incorrect image orientation after processing (e.g. an image appears rotated after resizing)

Most probably, the given image contains the EXIF orientation tag.
The standard `image/*` packages do not support loading and saving
this kind of information. To fix the issue, try opening images with
the `AutoOrientation` decode option. If this option is set to `true`,
the image orientation is changed after decoding, according to the
orientation tag (if present). Here's the example:

```go
img, err := imaging.Open("test.jpg", imaging.AutoOrientation(true))
```

### Usage example

```go
package main

import (
	"image"
	"image/color"
	"log"

	"github.com/essentialkaos/imaging"
)

func main() {
	// Open a test image.
	src, err := imaging.Open("testdata/flowers.png")
	if err != nil {
		log.Fatalf("failed to open image: %v", err)
	}

	// Crop the original image to 300x300px size using the center anchor.
	src = imaging.CropAnchor(src, 300, 300, imaging.Center)

	// Resize the cropped image to width = 200px preserving the aspect ratio.
	src = imaging.Resize(src, 200, 0, imaging.Lanczos)

	// Create a blurred version of the image.
	img1 := imaging.Blur(src, 5)

	// Create a grayscale version of the image with higher contrast and sharpness.
	img2 := imaging.Grayscale(src)
	img2 = imaging.AdjustContrast(img2, 20)
	img2 = imaging.Sharpen(img2, 2)

	// Create an inverted version of the image.
	img3 := imaging.Invert(src)

	// Create an embossed version of the image using a convolution filter.
	img4 := imaging.Convolve3x3(
		src,
		[9]float64{
			-1, -1, 0,
			-1, 1, 1,
			0, 1, 1,
		},
		nil,
	)

	// Create a new image and paste the four produced images into it.
	dst := imaging.New(400, 400, color.NRGBA{0, 0, 0, 0})
	dst = imaging.Paste(dst, img1, image.Pt(0, 0))
	dst = imaging.Paste(dst, img2, image.Pt(0, 200))
	dst = imaging.Paste(dst, img3, image.Pt(200, 0))
	dst = imaging.Paste(dst, img4, image.Pt(200, 200))

	// Save the resulting image as JPEG.
	err = imaging.Save(dst, "testdata/out_example.jpg")
	if err != nil {
		log.Fatalf("failed to save image: %v", err)
	}
}
```

Output:

![dstImage](testdata/out_example.jpg)

### License

[MIT Licence](LICENSE)
