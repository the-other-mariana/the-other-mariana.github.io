---
layout: post
title:  "Coding Letters On A Path"
date:   2023-08-21 01:26:00 -0600
categories: mit
modified_date:   2023-08-28 21:27:00 +0000
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

![img]({{site.url}}/img/12/6.png)

They depend on `$matrix->getMarginLeft()`, `$frameMargin`, `$frameDist`, `n`, `r` and the epsilon.

```php
$xTopLeftVertical = $matrix->getMarginLeft() + $frameMargin*$frameDist + $n + $r - $epsilon;
$yTopLeftVertical = $matrix->getMarginLeft() + $frameMargin*$frameDist + $epsilon + $n;

$xTopLeftHorizontal = $matrix->getMarginLeft() + $frameMargin*$frameDist + $n - $epsilon;
$yTopLeftHorizontal = $matrix->getMarginLeft() + $frameMargin*$frameDist + $n + $r - $epsilon + $n;

$xBottomRightVertical = $fullSize - $matrix->getMarginLeft() - $frameMargin*$frameDist - $n - $r + $epsilon;
$yBottomRightVertical = $fullSize - $matrix->getMarginLeft() - $frameMargin*$frameDist - $epsilon - $n*0.5;

$xBottomRightHorizontal = $fullSize - $matrix->getMarginLeft() - $frameMargin*$frameDist - $n + $epsilon;
$yBottomRightHorizontal = $fullSize - $matrix->getMarginLeft() - $frameMargin*$frameDist - $n - $r + $epsilon - $n*0.5;
```

Since, we mentioned `n`, it's important to define the full scheme from which these variables come from:

![img]({{site.url}}/img/12/8.png)

Then, the next step is to call `imageFilledCircledRect(\GdImage $image, int $color, array $verticalRectCoords, array $horizontalRectCoords, array $circleCenters, array $circleRadii)` two times in order to create the two circled rectangles that we frame needs, one in the foreground color (ie black) and the background color (ie white). By changing the **size of the radii** and moving the **top left and bottom right corners**, we can keep the **same circle centers** in *both* circled rectangles.

Now the letter placing algorithm can be defined. The first thing to point out is the **sections**: there are 8 as a total, where 4 are **straight sections (orange)** and 4 are **circled sections (green)**.

![img]({{site.url}}/img/12/9.png)

The general notion is that, in a loop (a while loop), each letter is placed in the *beginning* of each iteration with `imagettftext(\GdImage $image, int $size, int $angle, int $x, int $y, int $color, <FONT_PATH>, string $text)`, and therefore, a coordinate point where the current letter is written must be computed in all iterations. The *beginning* of each iteration writes a letter. Therefore, we must *initialize the start coordinate*, so that in the first iteration it is used, and from that point on, at the *end* of the iterations in the loop, right before moving onto the next, the *next coordinate point* is computed.

How do we perform the computation of the next coordinate point where a letter must be placed? Let $w$ be the letter width in pixels and $y$ the letter separation gap, also in pixels. The $\Delta_p$ then is the change in position that each iteration signifies, but $\Delta_p$ changes depending on the type of section you are currently in.  For **straight sections**, $\Delta_p = w + y$, but for **circled sections**, the idea of $\Delta_p$ morphs to $\Delta_a$ or change in *angle*. The position $(x, y)$ is not accumulated in this sections, and what is accumulated is the angle.

$$
x = Cx_i + width_i \cdot cos(\theta_j + \phi) \\
y = Cy_i + height_i \cdot \sin(\theta_j + \phi)
$$

where

$$
\theta_j = \theta_{j - 1} + \Delta_a
$$

and $i$ refers to the current circle (from a total of 4), $Cx$ and $Cy$ refer to the corresponding x and y values of the coordinate points that make up the 4 known circles' centers, and $j$ would represent the current iteration in the while loop. Then, the computation of $\Delta_a$ is defined as:

$$
\Delta_a = |tan^{-1}\left(\frac{y}{width_i}\right)| + |tan^{-1}\left(\frac{w}{width_i}\right)|
$$

This definition can be best explained with the diagram below, where the two componets of $\Delta_a$ are represented by $z$ and $x$.

![img]({{site.url}}/img/12/10.png)


The angle $\phi$ is not accumulated in the definition of $\theta_j$ because it must be added just one time per circled section, since $\phi$ represents the *starting angle* when the circled section begins. This comes from the fact that, during a straight section, we know the current section's distance is covered and the section type must change when the **distance** between current position plus $\Delta_p \times 0.5$ and the starting coordinate of the current section is **bigger than** the reference masure known as $g$. Therefore, in the last iteration of a straight section, if $\Delta_p \times 0.5$ fits, which means if half a letter fits, we place it as the last letter of that straight section. That means, there exists always a possibility that the straight sections exceed the ideal $g$ by pixels of *half a letter of less*. Thus, if we were to start each circled sections initial $\theta_i$ as 0, the first letter of a circled section would possibly overlap the last staright sections letter by pixels of *half a letter of less*. This originated the need for a starting angle known as $\phi$, in order to calculate the starting angle by knowing how much did the last letter of the previous straight section exceeded $g$. We can now define $\phi$ as

$$
\phi = |tan^{-1}\left(\frac{\hbox{remaining width}}{height_i}\right)|
$$

And graphically, its represented as

![img]({{site.url}}/img/12/11.png)