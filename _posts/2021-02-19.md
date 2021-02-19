---
layout: post
title: Computing isosurface volume
---

So my first writing is going to be about something that brought me joy. No, seriously, it doest really brought me joy! I'm not kidding. Because, as you may have guessed, I do love python programming language and nothing could bring me more joy then figuring out things from nowhere.

First, be advised that I'm not a professional programmer, neither a computer scientist, and most of the things I know now about dealing with computational things related to chemistry I learned by trial-and-error. Therefore, do not expect to find here the state of the art of anything. Now, without further ado let's dive in.

Suppose you have computed the electronic density of your system $\rho(\tau)$ and exported it using a Gaussian cube file format. As you may have noticed, the density is a function of $\tau$, which represents three dimensions $x$, $y$, and $z$. Since the gaussian cube file format makes use of the bohr radius unit, a finite representation that encompasses all the electronic density must be sampled. The quality of the sample will be given by the size of the grid that was used to write each voxel of the cube. A voxel can be understood as "a 3D pixel" - please, forgive me lords of CS.

All those informations are typically stored in the header of the cube files, which can be found as the example bellow:

<div class="message">
CPMD CUBE FILE.<br>
 OUTER LOOP: X, MIDDLE LOOP: Y, INNER LOOP: Z<br>
    3    0.000000    0.000000    0.000000<br>
   40    0.283459    0.000000    0.000000<br>
   40    0.000000    0.283459    0.000000<br>
   40    0.000000    0.000000    0.283459<br>
    8    0.000000    5.570575    5.669178    5.593517<br>
    1    0.000000    5.562867    5.669178    7.428055<br>
    1    0.000000    7.340606    5.669178    5.111259<br>
 -0.25568E-04  0.59213E-05  0.81068E-05  0.10868E-04  0.11313E-04  0.35999E-05<br>
      :             :             :           :            :            :<br>
      :             :             :           :            :            :<br>
      :             :             :           :            :            :<br>
        In this case there will be 40 x 40 x 40 floating point values<br>
      :             :             :           :            :            :<br>
      :             :             :           :            :            :<br>
      :             :             :           :            :            :<br>
</div>
