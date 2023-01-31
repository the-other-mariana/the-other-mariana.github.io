---
layout: post
title:  "QR Code Sub-module Coloring"
date:   2023-01-31 09:31:13 -0600
categories: qr code embedding
---

![img]({{site.url}}/img/1.png)

ZXing library (Zebra Crossing) can be seen as a dependency on the Android Open Source Project [[1]](https://github.com/zxing/zxing/blob/master/core/src/main/java/com/google/zxing/common/HybridBinarizer.java), and is used for the decoding of QR codes. If we go to analyze the algorithm that the library follows to decode a QR code, the first step is to binarize the image coming from a camera, and this is done using a 'dynamic' threshold based on the luminance of the image.

The class `HybridBinarizer` uses 5x5 blocks to compute local luminance where each block is 8x8 pixels. The following algorithm is based on such class. The image is divided in **blocks** of 8x8 pixels. Thus we have the number of blocks $N$ that fit inside an image, and it computes a matrix of *black points* with size $N \times N$, which seems to be the average luminance of the values in the blocks. The average is calculated generally as:

```
for i in N:
    for j in N:
        avg
        for r in BLOCK_SIZE:
            for c in BLOCK_SIZE:
                avg += luminance
        avg /= BLOCK_SIZE * BLOCK_SIZE
        ...
```

Then, with the above calculation we can get the maximum and minimum values of luminance of the computed block, and if the difference between that maximum and minimum is smaller than $\frac{255}{24}$, since ZXing's equivalent variable is 24 in a scale from 0 to 255 and we are working with a scale from 0 to 1, then the average is equal to $\frac{min}{2}$. This means that the variation within the block is low, and thus this must be a block with only light or dark pixels. Therefore, if we use the average as its threshold, we are creating noise data instead of guessing the actual module color. That is why in this case the average is half the minimum, assuming that the block is white. Then this assumption is corrected by computing the average black point from the 8 neighbours around the current block's black point, and if that average is larger than the minimum luminance, the black point becomes that average. That is how the matrix of black points $bp$ with size $N \times N$ is calculated.

```
for i in N:
    for j in N:
        avg
        for r in BLOCK_SIZE:
            for c in BLOCK_SIZE:
                avg += luminance
        avg /= BLOCK_SIZE * BLOCK_SIZE
        if max - min < MIN_DYNAMIC_RANGE:
            avg = min / 2
        avgBP
        for r in range 3:
            for c in range 3:
                avgBP += bp[i, j]
        avgBP /= 9
        if avgBP > min:
            avg = avgBP
        bp[i, j] = avg
```

The black point $bp$ is used now to calculate the **average black point** for each block using a 5x5 grid of the blocks around each block. This is the matrix of block averages that is used as thresholds for the binarization. The averages for the blocks tells us the limit from which a higher or lower value is binarized to be white or black. Thus, the averages are the limits of the color from which we need to base the color of the submodule. That is, the average of a block is the mid point from which a value is considered white or black.

![img]({{site.url}}/img/2.png)

