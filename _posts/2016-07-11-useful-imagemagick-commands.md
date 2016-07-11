---
layout: post
title: Useful Imagemagick Commands
date: 2016-07-11 22:30:00 +5:30 GMT
share: y
img-header: "https://s3-ap-southeast-1.amazonaws.com/blob.blankcursor.com/uploads/medium/766/26cfedf2-b476-4bcf-8357-594d29c579e4.png"
---

[Imagemagick](http://www.imagemagick.org/script/index.php) is one of those tools
that's there on literally every developers computer. It's such a nifty and
powerful tool, there's so much you can do with it.

Imagemagick is a suite of command line tools that can edit, create and convert
various image formats. Here's a few commands I use regularly.

### Combine two images together

```
convert +append a.png b.png c.png
```
This takes **a.png** and **b.png** and combines them horizontally to create **c.png**.
For best results, make sure both the images are of same or comparable ratios.

To convert the vertically,

```
convert -append a.png b.png c.png
```

You can even combine images of different types (like a PNG and a JPG)!

### Resizing images

```
convert a.png -resize 100x100 a_100.png
```

Easily [resize images]((http://www.imagemagick.org/Usage/resize/)) by providing
the width and height values. If you attempt to resize your images to a larger
ratio, they will often loose quality and appear pixelated. Resize always seeks
to preserve your aspect ratio even if given vales do not comply.

### Bluring images

```
convert a.png -blur 0x2 a_blur_2.png
```

Add [blur effects]((http://www.imagemagick.org/Usage/blur/)) to your images
quickly with this. The first value controls the radius of your blur, while the
second value controls the strength. As a quick rule of the thumb, increase the
second value to increase the blurriness.

### Making your white PNGs transparent

```
convert a.png -fuzz 10% -transparent white a_transparent.png
```

I've used this a million times. If you ever convert a JPG image to a PNG, you'll
notice the whites aren't transparent, which ultimately ruins the purpose of a PNG.
Use this command to make the white parts of your PNG transparent. The **fuzz**
parameter is to smoothen out the conversion. Use a smaller value to keep the conversion
closer to true white, or a larger value to allow more variation from white to become
transparent.

### Grayscale your images

```
convert a.jpg -type Grayscale a_gray.jpg
```

That's it! Simple as pie.

### Square out your uneven images by adding white padding

```
convert -define png:size=513x294 a.png \
-thumbnail '200x200>' -gravity center \
-crop 120x120+0+0\! -background white \
-flatten a_thumb.png
```

Define the size of the PNG you're converting in the **png:size=wxh** block. Where
**w** is the width and **h** is the height of the image. In the **-thumbnail**
block, define the resultant thumbnail dimensions.

That's it for now! I'll keep updating this post as and when I find more useful
commands.
