---
layout: post
title:  "Semantic Segmentation"
date:   2023-07-03 21:26:00 -0600
categories: mit
modified_date:   2023-07-03 21:27:00 +0000
---

**Semantic Segmentation** in Computer Vision is a technique that expects an image as input and in return you get a matrix where each pixel or cell contains an integer that tells you the object that each pixel corresponds to.

![img]({{site.url}}/img/11/1.png)

From tensorflow's words: 

> In an image classification task, the network assigns a label (or class) to each input image. However, suppose you want to know the shape of that object, which pixel belongs to which object, etc. In this case, you need to assign a class to each pixel of the image—this task is known as segmentation. 

## Handy Links

- Image segmentation with tf: https://www.tensorflow.org/tutorials/images/segmentation

- OpenCV: https://pyimagesearch.com/2018/09/03/semantic-segmentation-with-opencv-and-deep-learning/