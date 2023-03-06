---
layout: post
title:  "Generalizing The QR System"
date:   2023-02-24 12:30:12 -0600
categories: qr code embedding
modified_date:   2023-03-06 13:30:00 +0000
---

The Embedded QR code system so far has been outlined in terms of pipeline, but it always relies on the assumption that all images will behave as the lena image, which as a fact, is not true. So the next step is to generalize the system to work with any image.

The different constants used so far are:

- Squared host image

- Uses QR image size and resizes host to it.

- Guassian noise constants:

    - Mean = 0. **Conclusion: always 0**

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

        The following image shows the results from shifting $\mu$ to values -0.5, 0 and 0.5, and keeping a standard deviation of $\sigma = 0.5$.

        ![img]({{site.url}}/img/5/mean-tests.png)

    - Std dev = 0.5. **Conclusion: always 0.5**

        The `np.random.normal(loc, scale, size)` defines as `scale` the standard deviation of the distribution, that is, the speard or width of the gaussian bell. The standard deviation is the square root of the variance, and thus we are eliminating the sign of the differences of the numbers in the dataset with the mean, because we only care about its distance to the center. The mayority of data is within the range of $[\mu - \sigma, \mu + \sigma]$. We say the mayority instead of a percentage because the exact percentage depends on the distribution that the data has. For example, if it was a normal distribution we would say that the 68% of data points are within the range mentioned.

        In the proof of concept program we used a standard deviation of $\sigma = 0.5$, and by following the function for the gaussian noise above, we can say that the 68% of the data points are 0.5 units from the center 0. With the first domain shift, this becomes 0.25 from the center at 0.5; with the second domain shift, this becomes 64 units from the center at 127. Thus, **68% of the data points lie in a range of [127-64, 127+64]**. Since this is the range of color in RGB (0 to 255), the $\sigma = 0.5$ for the gaussian noise must stay the same for all cases.

        The following image shows the results from shifting $\sigma$ to values 0.25, 0.5 and 0.75, and keeping a mean of $\mu = 0$.

        ![img]({{site.url}}/img/5/stdev-tests.png)

- Blue noise constants:

    - Iterations: 5

    - Sigma: 0.8

- Overlay mask alpha: 0.2

- Factor to increase/decrease from average luminance + rgb: 0.3

- $\beta$, $\alpha$, $\beta_{c}$ and $\alpha_c$. These are for the non-center module pixels.

## References

- Video processing idea: https://mathematica.stackexchange.com/questions/60292/how-to-build-a-bvh-a-motion-capture-file-format-player-in-mathematica/60942#60942