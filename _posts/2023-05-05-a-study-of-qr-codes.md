---
layout: post
title:  "Migrate OpenCV System To C++ (Server)"
date:   2023-05-05 12:15:00 -0600
categories: cmake
modified_date:   2023-05-05 12:20:00 +0000
---

# A Study of QR Codes

The QR code's purpose is to embed information into a 2D barcode, and it encodes not only the information you provide but also extra information that aids in a process known as **error correction**. This extra information is included in a QR code so that the code can still be decoded even when an area has been damaged. The size of this *forgettable* area is determined by the **error correction level** parameter. The article by Karrach et al., titled *Recognition of Perspective Distorted QR Codes with a Partially Damaged Finder Pattern in Real Scene Images* states:

> QR Code has error correction capability to restore data if the code is partially damaged. Four error
QR Code has error correction capability to restore data if the code is partially damaged. Four
correction levels are available (L–Low, M–Medium, Q–Quartile, and H–High). The error correction
error correction levels are available (L–Low, M–Medium, Q–Quartile, and H–High). The error
level determines how much of the QR Code can be corrupted to keep the data still recoverable (L–7%,
correction level determines how much of the QR Code can be corrupted to keep the data still
M–15%, Q–25%, and H–30%) [5]. The QR Code error correction feature is implemented by adding
recoverable (L–7%, M–15%, Q–25%, and H–30%) [5]. The QR Code error correction feature is
a Reed–Solomon Code to the original data.

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

- For QR codes with **High error correction level**:

![img]({{site.url}}/img/7/qr-study_high.png)

