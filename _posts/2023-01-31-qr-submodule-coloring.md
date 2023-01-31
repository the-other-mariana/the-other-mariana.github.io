---
layout: post
title:  "QR Code Sub-module Coloring"
date:   2023-01-31 09:31:13 -0600
categories: qr code embedding
---

ZXing library (Zebra Crossing) can be seen as a dependency on the Android Open Source Project, and is used for the decoding of QR codes. If we go to analyze the algorithm that the library follows to decode a QR code, the first step is to binarize the image coming from a camera, and this is done using a 'dynamic' threshold based on the luminance of the image.

![img](/img/1.png)