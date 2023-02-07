---
layout: post
title:  "QR Code Eye Coloring"
date:   2023-02-03 11:25:12 -0600
categories: qr code embedding
---
Finder Patterns, which are located in the three corners of a QR Code, are important for determining the position and rotation of the QR Code. The size of the QR code is determined by the number of modules and can vary from 21 × 21 modules (Version 1) to 177 × 177 (Version 1). Each higher version number comprises four additional modules per side. QR Code has error correction capability to restore data if the code is partially damaged. Four error
correction levels are available (L–Low, M–Medium, Q–Quartile, and H–High). The error correction level determines how much of the QR Code can be corrupted to keep the data still recoverable (L–7%, M–15%, Q–25%, and H–30%). The QR Code error correction feature is implemented by adding a Reed–Solomon Code to the original data. The higher the error correction level is the less storage capacity is available for data. Each QR Code symbol version has the maximum data capacity according to the amount of data, character type and the error correction level. The data capacity ranges from 10 alphanumeric (or 17 numeric) characters for the smallest QR Code up to 1852 alphanumeric/3057 numeric characters for the largest QR Code at highest error correction level (Karrach et al., 2020).

![img]({{site.url}}/img/3/finder.png)

Finder Pattern based location methods locate a QR Code based on the location of its typical Finder Patterns that are present in its three corners. A shape of the Finder Pattern was deliberately chosen by the authors of the QR Code, because it was the pattern least likely to appear on various business forms and the like. They found out that black and white areas that alternate in a 1:1:3:1:1 ratio are the least common on printed materials (Karrach et al., 2020).

Most algorithms analyzed in Karrach's article base the Finder Pattern detection on the 1:1:3:1:1 ratio detection with the identification of candidate points located in areas that fulfill said ratio horizontally, vertically and often this means diagonally (due to rotation of the code).

![img]({{site.url}}/img/3/ratio.png)

The existing QR code decoders require the symbol to be properly “framed”, that is, the symbol must correspond to at least 30% of the image area to be suitable for decoding. The pixels, where the brightness is lower than the threshold, are declared as 1 and the others are declared as 0.

### References

(Karrach et al., 2020) Karrach, L., Pivarčiová, E., & Bozek, P. (2020). Recognition of perspective distorted QR codes with a partially damaged finder pattern in real scene images. Applied Sciences, 10(21), 7814.