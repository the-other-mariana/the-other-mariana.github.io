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

- Uses QR image size and resizes host to it.

- Guassian noise constants:

    - Mean = 0. **Conclusion: 0**

        The function to create a gaussian noise image is:

        ```python
        def gaussian_noise_2d(mean, stdev, size):
        gauss = np.random.normal(mean, stdev, (size[0], size[1]))
        gauss = gauss.reshape(size[0],size[1])
        # (-1,1) to (0,1)
        gauss = 0.5 * (gauss + 1.0)
        # (0,1) to (0, 255)
        gauss = (gauss * 255).astype('uint8')
        return gauss
        ```

        The `np.random.normal(loc, scale, size)` defines as `loc` the center of the distribution, around which the 68% of all randoms generated will fall into. We defined mean = 0 in the proof of concept program, and then all those numbers are generated in a range [-1, 1], **centered at mean = 0**. Then, the array gets moved to a domain of range [0, 1], meaning the center is now at **0.5 or 127 in a scale of 0 to 255**. From this, we can conclude that for the general program, and for any host image, the center or mean of the gaussian noise must be $\mu = 0$.

    - Std dev = 0.5

- Blue noise constants:

    - Iterations: 5

    - Sigma: 0.8

- Overlay mask alpha: 0.2

- Factor to increase/decrease from average luminance + rgb: 0.3

- $\beta$, $\alpha$, $\beta_{c}$ and $\alpha_c$. These are for the non-center module pixels.

## References

- Video processing idea: https://mathematica.stackexchange.com/questions/60292/how-to-build-a-bvh-a-motion-capture-file-format-player-in-mathematica/60942#60942