---
layout: post
title: Computing isosurface volume
---

{:refdef: style="text-align: center;"}
![Pika]({{ site.url }}/GayMePy/images/dmf.png "DMF"){:height="50%" width="50%" .center-image}
{: refdef}

So my first writing is going to be about something that brought me joy. No, seriously, it did really bring me joy! I’m not kidding. Because, as you may have guessed, I do love python programming language and nothing could bring me more joy than figuring out things from nowhere.
{: style="text-align: justify"}

First, be advised that I’m not a professional programmer, neither a computer scientist and most of the things I know now about dealing with computational things related to chemistry I learned by trial-and-error. Therefore, do not expect to find here the state of the art of anything. Now, without further ado let’s dive in.
{: style="text-align: justify"}

Suppose you have computed the electronic density of your system $\rho(\tau)$ and exported it using a Gaussian cube file format - currently, you can do it in almost any software out there as Quantum-ESPRESSO, Orca, GAMESS, VASP, Crystal, the list goes on.
{: style="text-align: justify"}

As you may have noticed, the density is a function of $\tau$, which represents three dimensions $x$, $y$, and $z$.
{: style="text-align: justify"}

Since the gaussian cube file format makes use of the bohr radius unit, a finite representation that encompasses all the electronic density must be sampled in that unit system.
{: style="text-align: justify"}

The quality of the sample will be given by the size of the grid that was used to write each voxel of the cube. A voxel can be understood as “a 3D pixel” - please, forgive me lords of CS and gods of CG (and some academics).
{: style="text-align: justify"}

All those information are typically stored in the header of the cube files, which can be found as the example below:
{: style="text-align: justify"}

<div class="message" style='font-size=50%'>
<pre>
Super duper brilliant output of MOL
More labelling, why not?
 3  0.000000  0.000000  0.000000
40  0.283459  0.000000  0.000000
40  0.000000  0.283459  0.000000
40  0.000000  0.000000  0.283459
 8  5.570575  5.669178  5.593517
 1  5.562867  5.669178  7.428055
 1  7.340606  5.669178  5.111259
-0.25E-04 0.59E-05 0.81E-05 0.10E-04 0.11E-04 0.35E-05
    :         :         :       :        :        :
    :         :         :       :        :        :
    :         :         :       :        :        :
In this case there will be 40x40x40 floating point values
    :         :         :       :        :        :
    :         :         :       :        :        :
    :         :         :       :        :        :
</pre>
</div>

The first two lines are just labels, and the third line is the number of atoms and origin position for the given cube file. Yet, the next three represents the number of points in each axis, along with the vector that describes that axis.
{: style="text-align: justify"}

Finally, you will be given a table that should have the same number of lines as the number of atoms. For three atoms, as our example, you will see three lines. Each line starts with the atom $Z$ number and its position.
{: style="text-align: justify"}

All the remaining data will be printed in a very peculiar order. As a cube comprises three dimensions, one should expect that each slice of the cube will be printed as a 2D matrix.
{: style="text-align: justify"}

Indeed, that's what happens, and roughly speaking that matrix is written as a n$\times$6 table, according to the code below:
{: style="text-align: justify"}

{% highlight python %}
for ix in range(NX):
   for iy in range(NY):
      for iz in range(NZ):
         print("{:15.10f} ".format(data[ix][iy][iz])
         if iz % 6 == 5:
            print("\n")
      print("\n")
{% endhighlight %}

Therefore, as we are going to compute only the volume of a given isosurface, we are interested in identifying which value of $\rho(\tau)$ is within the isosurface value of our choice.   
{: style="text-align: justify"}

Once we identified those values, we can sum them up and multiply them to the cubic value of one voxel. However, we need to compute the voxel volume before going on in our quest.
{: style="text-align: justify"}

Hence, let's retrieve the size of each axis, the matrix of the cube lattice and the sum of all the values that are higher than a specified isosurface value.
{: style="text-align: justify"}

{% highlight python %}
from numpy import array, sqrt, sum, zeros
points = zeros(3)
M = zeros([3,3])
with open(CUBE, 'r') as cube:
    v = 0
    next(cube)
    next(cube)
    idx = 2
    for line in cube:
        if idx >= 3 and idx <= 5:
            l = line.split()
            points[idx-3] = float(l[0])
            for j in range(3):
                M[idx-3,j] = float(l[j+1])
        if len(line.split()) > 5:
            for val in line.split():
                if float(val) > iso:
                    v += 1
        else: pass
        idx += 1
{% endhighlight %}

Once we got those values we can proceed to some computations. For instance, to determine the voxel volume we need to compute the total volume of the cube and divide it by the number of voxels within the cube.
{: style="text-align: justify"}

As matrix **M**, from our code, is the reduced matrix of the cube, we need to get the original matrix back by multiplying it by the number of points of each axis. So, here is the trick:
{: style="text-align: justify"}

{% highlight python %}
M = (M.T * points).T
Vm = sqrt(sum(M**2, axis = 1))
voxel = Vm.prod() / points.prod()
{% endhighlight %}

Now we can calculate the bohr volume, then convert it to any unit we like. I will present the conversion to &#8491; and cm$^3$/mol.
{: style="text-align: justify"}

{% highlight python %}
A2m = 1e-30
Na = 6.02214076e23
m3tocm3 = 1e6
bohr2A = 0.14818474347690475
Vbohr = v * voxel
Va = Vbohr * bohr2A
Vcmmol = ((Va * A2m) * Na) * m3tocm3
{% endhighlight %}

Now, we can print the result and get the volume from variables Vbohr, Va, and Vcmmol for  bohr$^3$, angstrom$^3$ and cm$^3$/mol, respectively.
{: style="text-align: justify"}

Below are some results that I got by using Quantum-ESPRESSO generated cube files for Ar, CO$_2$ and DMF by employing an isosurface value of 0.001 bohr radius.
{: style="text-align: justify"}

<table>
  <thead>
    <tr>
      <th>Molecule</th>
      <th>&#8491;$^3$</th>
      <th>bohr$^3$</th>
      <th>cm$^3$/mol</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Ar</td>
      <td>32.41</td>
      <td>218.69</td>
      <td>19.52</td>
    </tr>
    <tr>
      <td>CO$_2$</td>
      <td>45.80</td>
      <td>309.11</td>
      <td>27.58</td>
    </tr>
    <tr>
      <td>DMF</td>
      <td>108.42</td>
      <td>731.68</td>
      <td>65.29</td>
    </tr>
  </tbody>
</table>

Sure you can easily plug and play with this code, create your own script that automates the process and make functions out of those routines. I also recommend you to use the package prettytables from python. It is wonderful. :)
