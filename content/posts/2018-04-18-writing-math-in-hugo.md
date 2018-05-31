---
title: "Writing Math in Hugo"
date: 2018-04-18T12:00:00-04:00
draft: false
tags: [ "hugo" , "mathjax", "hyde-hyde" ]
categories: [ "programming", "fun"]
---

In this blog, I am going to write a lot of math. I used Typora as my markdown editor and it supports MathJax out of the box. I just need to wrap my math blocks with $$ (following this [reference)](https://math.meta.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference). However, I will need to do a quick step to make Hugo work with MathJax as discribed in the Hugo [documentation](https://gohugo.io/content-management/formats/#mathjax-with-hugo). I copied `add-mathjax-to-page.html`  and pasted in the beginning of `post_footer.html` in the `layout/partials` folder.

add-mathjax-to-page.html:

```html
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

There are some minor issues with how Hugo render MathJax, below are some tests and small modifications I need to make for each cases.

#  Mathjax test

My solution to this [puzzle](https://puzzling.stackexchange.com/questions/1963/6-the-magic-number).
$$
( 1 + 1 + 1)! = 6\\\\\\
2 + 2 + 2 = 6\\\\\\
3 * 3 - 3 = 6\\\\\\
4! / 4 * 4^0 = 6\\\\\\
5 + (5 / 5) = 6\\\\\\
6 + 6 - 6 = 6\\\\\\
7 - (7 / 7) = 6\\\\\\
8 * 8^0 - 8^{\frac 13} = 6\\\\\\
{\sqrt 9} + {\sqrt 9} + {\sqrt 9} = 6\\\\\\
{\log (10 * 10 * 10)}! = 6
$$
As flagged in this [issue](https://github.com/gcushen/hugo-academic/issues/291), we need to use six backslashes for line break.

How about summation and subscripts? (Cost function with regularization)


$$
J(\theta) =\frac{1}{2m}
[\sum^m\_{i=1}(h\_\theta(x^{(i)}) - 
y^{(i)})^2] + \lambda\sum^n\_{j=1}\theta^2\_j
$$
The way Hugo handles italics is the same with subscript in MathJax, they both use underscore, to work around this, I just need to add an extra backslash before each underscore to escape them.

Matrix operations?
$$
Av=\begin{bmatrix} 
c & c & c & \cdots \\\\\\
o & o & o & \cdots \\\\\\ 
l & l & l & \cdots \\\\\\ 
u & u & u & \cdots \\\\\\ 
m & m & m & \cdots \\\\\\ 
n & n & n & \cdots
\end{bmatrix} 
\begin{bmatrix} 
a  \\\\\\ 
b \\\\\\ 
c \\\\\\ 
\vdots
\end{bmatrix}
= a* \begin{bmatrix} 
1^{st}  \\\\\\ 
c \\\\\\ 
o \\\\\\ 
l \\\\\\ 
u \\\\\\ 
m \\\\\\ 
n 
\end{bmatrix}
+b*  \begin{bmatrix} 
2^{nd}  \\\\\\ 
c \\\\\\ 
o \\\\\\ 
l \\\\\\ 
u \\\\\\ 
m \\\\\\ 
n 
\end{bmatrix} + \cdots
$$
Again, for each line break at the end of row, we need six backslashes. Original LaTex from [here](https://blogs.ams.org/mathgradblog/2017/06/12/matrix-multiplication-human-way/).



How about some physics? (Maxwell's Equations)
$$
\begin{align}
\nabla \times \vec{\mathbf{B}} -\, \frac1c\, \frac{\partial\vec{\mathbf{E}}}{\partial t} & = \frac{4\pi}{c}\vec{\mathbf{j}} \\\\\\   \nabla \cdot \vec{\mathbf{E}} & = 4 \pi \rho \\\\\\
\nabla \times \vec{\mathbf{E}}\, +\, \frac1c\, \frac{\partial\vec{\mathbf{B}}}{\partial t} & = \vec{\mathbf{0}} \\\\\\
\nabla \cdot \vec{\mathbf{B}} & = 0
\end{align}
$$
Although it looks good here, it differs from standard MathJax and in my editor so I will need to spend a bit of time re-formatting the math. Luckily, the LiveLoad feature of Hugo is perfect for editing markdown and rebuild the site instantly to see how it looks on the site.

