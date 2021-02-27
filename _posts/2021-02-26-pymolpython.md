---
layout: page
title: Tips and Tricks with PyMOL and Python
---

{:refdef: style="text-align: center;"}
![Pika]({{ site.url }}/GayMePy/images/pp.png "PyMOL-Python"){:height="70%" width="70%" .center-image}
{: refdef}

Konnichiwa minasan! Hello everyone! Hola todos! Olá todos! After a pause in my writings here, I'm back! And I would like to say that everything is fine, and I'm dealing well with my recent loss, but well... you know that it's difficult...
{: style="text-align: justify"}

So, let's skip that drama and go to something more interesting. I've been using this for a while already, and recently I was doing it again and caught myself in that situation thinking "oh, people might like to know about it". Then, here I'm writing about the wonders of PyMOL and how you could take some advantages from it by using plain Python. Doesn't that sound cool?
{: style="text-align: justify"}

Not only for making pretty pictures, but for doing actual computations, PyMOL can be a very handy tool if you know python, and one of the things that chemists and biologists that deals with protein, molecular, and solid crystals structures might like to do is to average the bonding distances among specific atom species.
{: style="text-align: justify"}

For instance, say you have a radius of size $r$ within which you want to know the amount of nearest neighbours and you also want to know the average distance between pair of species within this coordination sphere. Let's name it RNN for now and think about it a little more.
{: style="text-align: justify"}

It's quite a simple concept if you pay attention to it. Besides, doing it with plain python could be a little painful, not impossible though. However, why not use PyMOL to help you with that? As PyMOL already has many methods to do what we want and we can just call them for it.
{: style="text-align: justify"}

First, we will need to have PyMOL installed and it can be easily done by following any of the suggestions available [here](https://pymolwiki.org/index.php/Linux_Install). Sure it can vary, depending on what OS you are running. But I assure you, life will be easier if you just use Linux - I particularly like to use conda to install PyMOL and it is as easy as 'conda install -c conda-forge pymol-open-source'.
{: style="text-align: justify"}

Then, once you have your PyMOL ready to go you can start by calling your PyMOL module and proceed to the problem you would like to tackle. For example, we will deal with a solid structure made out of Pt and Ni atoms. But I have a further problem to think about. Our systems are not a single file in our current working directory. What we have is a set of five directories, and inside each one of them, there is our *xyz* file we want. So, let's say our folders are: A, B, C and D; and inside each one of them there is a file labelled a.xyz, b.xyz, c.xyz, and d.xyz. We will need to get the current parent directory for all those 4 folders and iterate in each one of them. That will look like this:
{: style="text-align: justify"}

{% highlight python %}
from pymol import cmd
from os import chdir, getcwd

wd = getcwd()
for folder in ['A', 'B', 'C', 'D']:
    chdir('%s/%s'.format(wd, folder))
    cmd.load('%s.xyz' % folder.lower())
{% endhighlight %}

So, note that we have used two ways of giving python the string variables. When using python's module chdir we used 'format', but when using pymol's module cmd we used the old-fashioned way with % symbol. Why, would you ask? Well, I might not be the all-knowing clever person in this world, but as far as I have experienced, sometimes pymol just crashes if I don't use this old fashion style and it is most likely to be linked with how it handles special symbols ¯\\_(ツ)_/¯.
{: style="text-align: justify"}

So, now that we have that done, we need to define a dictionary of atoms to be used when iterating among the atomic species of our structure. In my case, as I said, we have just Pt and Ni, thus we can proceed like this:
{: style="text-align: justify"}

{% highlight python %}
    atoms = {'platinum': [], 'nickel': []}
    cmd.iterate('symbol Pt', 'platinum.append(rank)', space = atoms)
    cmd.iterate('symbol Ni', 'nickel.append(rank)', space = atoms)
{% endhighlight %}

You see, what we have done so far was to say to PyMOL to look for any atomic species that has a given atomic symbol and store its rank number in a dictionary called atoms, where the appended list has the same name as one of the keys within that given dictionary. I know it seems quite ugly and strange, but somehow I got so used to it that to me it's quite simple. Thus, I won't mind if someone could suggest anything better than this, despite assuring everybody that it's not that difficult to understand.
{: style="text-align: justify"}

Now, we have to set up some variables that we will be using over our measuring procedure. We will need a variable to store the amount of nearest neighbours that fall within the coordination sphere we wish to evaluate, which is going to be *C* for Pt-Ni, Pt-Pt and Ni-Ni - one for each pair of atoms. Then we will need a variable to store each one of these distances, we'll call it *D*, again one for each pair of atoms. Finally, a variable to store the average of those distances, which can be just called the atomic pair name. Only then we can start iterating over those atom pairs, let's do it:
{: style="text-align: justify"}

{% highlight python %}
def distance(atom1, atom2, d, c, r):
    dist = cmd.distance('rank %d' % atom1, 'rank %d' % atom2)
    if dist < r and atom1 != atom2:
        d += dist
        c += 1
    return d, c

    Cptni, Dptni, ptni = 0, 0, 0
    Cptpt, Dptpt, ptpt = 0, 0, 0
    Cnini, Dnini, nini = 0, 0, 0
    Lpt, Lni = [], []
    for pt1 in atoms['platinum']:
        for pt2 in atoms['platinum']:
            if pt2 not in Lpt:
                Dptpt, Cptpt = distance(pt1, pt2, Dptpt, Cptpt, 3.0)
                Lpt.append(pt1)
    for ni1 in atoms['nickel']:
        for pt1 in atoms['platinum']:
            Dptni, Cptni = distance(pt1, ni1, Dptni, Cptni, 3.0)
    for ni1 in atoms['nickel']:
        for ni2 in atoms['nickel']:
            if ni2 not in Lni:
                Dnini, Cnini = distance(ni1, ni2, Dnini, Cnini, 3.0)
                Lni.append(ni1)
{% endhighlight %}

There there! Look that we have defined a function called *distance* and it takes four arguments, where the first and second are the pair of atoms to be measured then it gets the distance and counting variables we defined, and finally, it gets the radius of the coordination sphere. For us, a value of 3.0 &#8491; is enough.
{: style="text-align: justify"}

Now that we have all the values we want, we might be able to compute the average of the distances we wanted. But, lo! Remind yourself that you might find some coordination spheres where there might be pure Pt or pure Ni, thus the Pt-Ni pairs shall present zero as value. For now, I'm not quite sure if the solution I use is the best one out there, but I'm sure it works. So, forgive me all the gods of computer science out there. Here it goes:
{: style="text-align: justify"}

{% highlight python %}
    try: ptpt = Dptpt / Cptpt
    except: pass
    try: nini = Dnini / Cnini
    except: pass
    try: ptni = Dptni / Cptni
    except: pass
    print('{:<10s} {:>4.2f} {:>4.2f} {:>4.2f}'.format(nc, ptpt, nini, ptni))
    cmd.delete('all')
{% endhighlight %}

Perfect! Once we have computed everything through the try-except block we printed it out! As a final point, we must not forget that this is the routine for the first iteration, thus for the next iteration, our pymol ambient must be clean, without any trace of everything we did. That's why we evoked from the pymol cmd a delete function, passing to it the word 'all'.
{: style="text-align: justify"}

Now, if you look it closely you might think the way we are printing it is somehow ugly. Honestly speaking, I do prefer to use prettytable library, which you can find [here](https://pypi.org/project/prettytable/), this is one of my favourite libraries for printing pretty stuff directly on the terminal. It is quite easy to use prettytable, we can simply play with that as shown below:
{: style="text-align: justify"}

{% highlight python %}
from prettytable import PrettyTable

table = PrettyTable()
table.field_names = ['Structure', 'ptpt', 'ptni', 'nini']
table.float_format = '6.2'
{% endhighlight %}

See, we have created a table and added the headers to it, also we have set a specific format for any float number that could appear in the table. What is missing is the rows for each atomic pair iteration we assessed. For that, we just need to add at our main for loop the following:
{: style="text-align: justify"}

{% highlight python %}
    table.add_row(['%.xyz' % folder.lower(), ptpt, ptni, nini, Cptpt, Cnini, Cptni])
{% endhighlight %}

At the end of everything you will just need to print it to see this wonder:
{: style="text-align: justify"}

{% highlight bash %}
+-----------+--------+--------+--------+
| Structure |  ptpt  |  ptni  |  nini  |
+-----------+--------+--------+--------+
|   a.xyz   |   2.74 |   0    |   0    |
|   b.xyz   |   2.73 |   2.58 |   2.36 |
|   c.xyz   |   2.74 |   2.57 |   2.42 |
|   d.xyz   |   2.83 |   2.56 |   2.42 |
+-----------+--------+--------+--------+
{% endhighlight %}

Isn't that beautiful? I do appreciate it a lot. By the way, it also can be possible to implement a method to get the effective coordination number (ECON) for each atom of your system as proposed by [Hoppe, Z. Kristallogr. 150 (1979) 23](http://dx.doi.org/10.1524/zkri.1979.150.14.23). But I will let it for another time.
{: style="text-align: justify"}

Now, just to wrapping-up all our code until here:
{: style="text-align: justify"}

{% highlight python %}
from pymol import cmd
from os import chdir, getcwd
from prettytable import PrettyTable

def distance(atom1, atom2, d, c, r):
    dist = cmd.distance('rank %d' % atom1, 'rank %d' % atom2)
    if dist < r and atom1 != atom2:
        d += dist
        c += 1
    return d, c

table = PrettyTable()
table.field_names = ['Structure', 'ptpt', 'ptni', 'nini']
table.float_format = '6.2'
wd = getcwd()

for folder in ['A', 'B', 'C', 'D']:
    chdir('%s/%s'.format(wd, folder))
    cmd.load('%s.xyz' % folder.lower())
    atoms = {'platinum': [], 'nickel': []}
    cmd.iterate('symbol Pt', 'platinum.append(rank)', space = atoms)
    cmd.iterate('symbol Ni', 'nickel.append(rank)', space = atoms)

    Cptni, Dptni, ptni = 0, 0, 0
    Cptpt, Dptpt, ptpt = 0, 0, 0
    Cnini, Dnini, nini = 0, 0, 0
    Lpt, Lni = [], []
    for pt1 in atoms['platinum']:
        for pt2 in atoms['platinum']:
            if pt2 not in Lpt:
                Dptpt, Cptpt = distance(pt1, pt2, Dptpt, Cptpt, 3.0)
                Lpt.append(pt1)
    for ni1 in atoms['nickel']:
        for pt1 in atoms['platinum']:
            Dptni, Cptni = distance(pt1, ni1, Dptni, Cptni, 3.0)
    for ni1 in atoms['nickel']:
        for ni2 in atoms['nickel']:
            if ni2 not in Lni:
                Dnini, Cnini = distance(ni1, ni2, Dnini, Cnini, 3.0)
                Lni.append(ni1)
    try: ptpt = Dptpt / Cptpt
    except: pass
    try: nini = Dnini / Cnini
    except: pass
    try: ptni = Dptni / Cptni
    except: pass
    table.add_row(['%.xyz' % folder.lower(), ptpt, ptni, nini, Cptpt, Cnini, Cptni])
    cmd.delete('all')
{% endhighlight %}

That's all, folks! Hope you all liked it. :)
{: style="text-align: justify"}
