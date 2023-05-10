---
layout: post
title:  "A Study of QR Codes"
date:   2023-05-05 12:15:00 -0600
categories: cmake
modified_date:   2023-05-05 12:20:00 +0000
---

## What do we  know?

The QR code's purpose is to embed information into a 2D barcode, and it encodes not only the information you provide but also extra information that aids in a process known as **error correction**. This extra information is included in a QR code so that the code can still be decoded even when an area has been damaged. The size of this *forgettable* area is determined by the **error correction level** parameter. The article by Karrach et al., titled *Recognition of Perspective Distorted QR Codes with a Partially Damaged Finder Pattern in Real Scene Images* states:

> QR Code has error correction capability to restore data if the code is partially damaged. Four correction levels are available (L–Low, M–Medium, Q–Quartile, and H–High). The error level determines how much of the QR Code can be corrupted to keep the data still recoverable (L–7%, M–15%, Q–25%, and H–30%) [5]. The QR Code error correction feature is implemented by adding a Reed–Solomon Code to the original data.

Then, it also mentions a parameter known as the **version of the QR code**. This parameter is encoded specifically in ortogonal bands connecting the finder patterns, so that decoder systems know which error should they expect. Coming back to the article, it mentions:

> The size of the QR code is determined by the number of modules and can vary from 21 x 21 modules (version 1) to 177 x 177 (version 40). Each higher version number comprises four additional modules per side.

Meaning that the version can be determined by knowing the length of the encoded data and the error correction level. 

## Concern

We could ignore this for a long time, but the dynamic QR code system needs to specify the size of the data modules so that it works as intended. If there is a very very dense QR code, the modules become so small that when we want to embed an image onto it, it would distort it making it unscannable.

In order to determine the optimal size or range of sizes with which the embedding of a QR in an image results successful, we need to determine the **optimal size of the url** to encode, the **size of the final QR code image**, and the **error correction level** so that we can also determine the **version or amount of modules per size**, and thus we can finally know the resulting **size of the modules**. In other words, we know:

- The size of the QR code's image (pixels)

- The error correction level we want (L, M, H)

- The size of the url to encode (integer)

so that we can know

- The version (integer)

- The size of the modules (pixels)

and therefore we can say if a QR code image can be embedded in an image successfully.

For this purpose, I performed a study of how the QR code's data module size changes as the length and error correction level changes. 

- For QR codes with **Low error correction level**:

![img]({{site.url}}/img/7/qr-study_low.png)

- For QR codes with **Medium error correction level**:

![img]({{site.url}}/img/7/qr-study_medium.png)

- For QR codes with **High error correction level**:

![img]({{site.url}}/img/7/qr-study_high.png)

What was done for the plots began with creating 40/70/80-image batches with Endroid's PHP QR code library by adding 1 letter to the url per iteration in a for loop. This library allows you to input the **url to encode**, the **image size** in pixels and the **error correction level**, and the library will fit a QR to the image size you provided. Most python libraries do not let you just input the full image size, but they ask you to specify the **module size**. Since the purpose of this study is to see how the size of the modules changes with the URL length, then these libs would not be useful as we would have to set the size of the modules. 

Then, in Python I coded a little computer vision routine to determine the size of the modules in all these images. This involved the Probabilistic Hough Transform to store all vertical line candidates and measuring the minimum distance found among them. This distance was in fact the size of the QR modules. 

With all this information, I created a csv file for each correction level (Low, Medium, and High) where the **url**, its **length**, the **size of the resulting module**, the **block count** and the **image size** were stored. These three csv files were used for the above plots.

## Analysis

QR decoding tests proved that those QR codes that have **module size <= 23 pixels** failed to be detected in around 50% of the cases. Therefore, the accepted **module size must be 24 or higher**. 

By looking at the plots above, the conclusion is the following:

A QR code will be scanned successfully if its URL length is

| URL Length | URL Example | Error Correction Level | Module Size | Image Size | 
| --- | --- | --- | --- | --- |
| 25 - 31 | https://pokme.wispok.mx/x - https://pokme.wispok.mx/xxxxxxx | Low | 31, 26 | 900 |
| 25 - 40 | https://pokme.wispok.mx/x - https://pokme.wispok.mx/xxxxxxxxxxxxxxxx | Medium | 26 | 900 |


Since the error correction level increases the chances of decoding a QR code successfully the higher it is, then the suggested error correction should be **Medium**.
