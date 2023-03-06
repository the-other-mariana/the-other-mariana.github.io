---
layout: post
title:  "Generalizing The QR System"
date:   2023-02-24 12:30:12 -0600
categories: qr code embedding
modified_date:   2023-02-24 13:30:00 +0000
---

The Embedded QR code system so far has been outlined in terms of pipeline, but it always relies on the assumption that all images will behave as the lena image, which as a fact, is not true. So the next step is to generalize the system to work with any image.

The different constants used so far are:

- Squared host image

- Gets the maximum height and width from the two images (qr and host)

- Guassian noise constants:

    - Mean = 0 

    - Std dev = 0.5

- Blue noise constants:

    - Iterations: 5

    - Sigma: 0.8

- Overlay mask alpha: 0.2

- Factor to increase/decrease from average luminance + rgb: 0.3

- $\beta$, $\alpha$, $\beta_{c}$ and $\alpha_c$. These are for the non-center module pixels.

## References

- Video processing idea: https://mathematica.stackexchange.com/questions/60292/how-to-build-a-bvh-a-motion-capture-file-format-player-in-mathematica/60942#60942