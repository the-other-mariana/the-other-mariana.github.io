---
layout: post
title:  "Coding Letters On A Path"
date:   2023-08-21 01:26:00 -0600
categories: mit
modified_date:   2023-08-21 21:27:00 +0000
---

The challenge now is to make letters paint the frame on its own. For that, I got the references that Design uses:

![img]({{site.url}}/img/12/1.png)

Therefore, we need to generalize the proportions of the figures for all sizes. The first thing to do was

$$
\frac{B}{A} = 10.83 = \hbox{B is 10.83 times what A is}
$$

By knowing $A$, $B$, and $\frac{B}{A}$, we can know $a$ and $b$, which represent the sections for any image size. But we need to solve a tiny linear equation system:

$$
\begin{cases}
a + b = \hbox{frameHeight} \\
\frac{B}{A}a = b
\end{cases}
$$

where 

$$
\hbox{frameHeight} = (\hbox{inner size} + 2 \cdot \hbox{frame margin}) \cdot \frac{374}{426}
$$

since the reference measures will be constants in all these computations. That's how the term $\frac{374}{426}$ came to be, so that the current measure for the **image height** represented by $(\hbox{inner size} + 2 \cdot \hbox{frame margin})$ is transformed to a $B$ size by following the proportion $\frac{374}{426}$, which is the size of the frame (without the identifiers' rectangles) over the total image size in the reference (426 px).

![img]({{site.url}}/img/12/2.png)

This will allow us to solve for $a$:

$$
a + \frac{B}{A}a =\hbox{frameHeight} \\
(1 + \frac{B}{A})a = \hbox{frameHeight} \\
a = \frac{\hbox{frameHeight}}{(1 + \frac{B}{A})} \\
b = (\frac{B}{A}) a
$$

Other constants taken in account involve the distance from the edge of the reference PNG until the actual frame starts, which looks as below.

![img]({{site.url}}/img/12/5.png)

That is why a constant proportion ratio was established as $\hbox{frameDist} = \frac{27}{65}$, which represents the y distance to advance to the point where the actual frame begins. It is also important to point out that 

$$
\hbox{fullSize} = \hbox{inner size} + (2 \cdot \hbox{frame margin}) + (2 \cdot \hbox{margin left})
$$

where 

$$
\hbox{frame margin} = \hbox{target size} \cdot \frac{65}{426}
$$

since we want to maintain the proportion of an additional 65 px for the frame for an image of size 426x426. Then, it was also noticed that the QR matrix library *approximates* a QR for the size the user desires, but the output image's size isn't necessarily the exact same, there is usually an *epsilon*, defined as:

$$
\epsilon = \hbox{fullSize} \cdot \frac{5}{426}
$$

Once that we defined all constants and figured out a generalization for $a$ and $b$, we can translate any measure in the reference image to any size, maintaining the proportions in the reference. We can do this translation by doing

$$
\hbox{desired measure} = b \cdot (\frac{\hbox{desired measure in reference}}{B})
$$

For example, to get $n$ and $r$, which both are a generalization for reference measures $N$ and $R$, we would do:

$$
n = b \cdot (\frac{N}{B}) \\
r = b \cdot (\frac{R}{B})
$$

These measures are needed to define the following coordinate points: