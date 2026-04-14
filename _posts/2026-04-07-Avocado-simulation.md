---
title: "Kepler Ptolemy Simulation"
classes: wide
tags:
  - prog
---

This is a subsection of my <a href="{{site.url}}/Kepler-ptolemy">Kepler Ptolemy projecy</a>, in the math section. This is a big part of what I worked on in that project, but I felt it was more fitting in the programming section. This will be a shorter descritption, but it still taught me some valuable things.

What inspired this project was simply a desire to see what the Kepler Ptolemy system could look like when we eventually solved the system of differential equations. Cris (Cristopher Moore, who was my mentor for the larger Kepler Ptolemy project) delegated it to me, as it's relatively simple and I'm familiar enough with python to do it easily (or so I thought). My initial approach was to choose some angle step from the equant, then step out along that line until the area swept out by the most recent step is equal to our initial area step (this ensures that angular momentum is conserved, as discussed in the main post):

<details>
<summary>
{% highlight python %}
import matplotlib.pyplot as plt
import numpy as np
import math

def main():
{% endhighlight %}
</summary>
{% highlight python %}
    final_angle = np.pi
    angle_step = 0.001 * np.pi
    distance_step = 0.0001

    r = np.array((2, 0))
    new_r = r + (0, math.sin(angle_step) * mag(r))

    l = np.array((1, 0))
    fig, ax = plt.subplots(1, 1)
    ax.plot((0, l[0]), (0, l[1]), 'o', color='grey')
    semi_perim = 0.5 * (mag(r - l) + mag(r - new_r) + mag(new_r - l))
    area_step = math.sqrt(semi_perim * (semi_perim - mag(r - l)) * (semi_perim - mag(r - new_r)) * (semi_perim - mag(new_r - l)))
    print(area_step)

    points = [r]
    for i in np.arange(0, final_angle, angle_step):
        ## update and record the position
        r = step(r, l, angle_step, area_step, distance_step)
        points.append(r)
    
    points = np.array(points)
    ax.plot(points[:, 0], points[:, 1])
        
    plt.show()
    
def step(r, l, angle_step, area_step, distance_step):
    ## uses the geometry of a single step to determine the length of the next radius
    pol_r = [mag(r), math.acos(r[0] / mag(r))]
    if r[1] < 0: ## adjust range of acos properly
        pol_r[1] = 2 * np.pi - pol_r[1]
    
    calc_area = 0
    pol_new_r = np.array([0, pol_r[1] + angle_step])
    len_base = mag(r - l)
    while calc_area < area_step:
        pol_new_r[0] += distance_step
        new_r = polar_to_cart(pol_new_r)
        len_short_side = mag(new_r - r)
        len_long_side = mag(new_r - l)
        semi_perim = 0.5 * (len_base + len_short_side + len_long_side)
        calc_area = math.sqrt(semi_perim * 
                             (semi_perim - len_base) *
                             (semi_perim - len_short_side) *
                             (semi_perim - len_long_side)) ## Heron's formula
        print(calc_area)
        
    return new_r

def mag(vec):
    ## returns the magnitutde of a vector
    return math.sqrt(vec[0] ** 2 + vec[1] ** 2)
    
def angle(vec):
    ## returns the angle of a vector
    if mag(vec) == 0:
        return 0

    angle = math.acos(vec[0] / mag(vec))
    if vec[1] < 0:
        ## adjust range of acos
        angle = 2 * np.pi - angle
    return angle

def polar_to_cart(point):
    ## converts a point from polar to cartesian coordinates
    r, theta = point
    x, y = r*math.cos(theta), r*math.sin(theta)
    return np.array((x, y))

if __name__ == '__main__':
    main()
{% endhighlight %}

</details>

I now believe this method does produce correct results, as the system sometimes does have a pole and diverge to infinity. However, when I was first programming it, I believed this was a bug and spent a lot of time attempting to tweak parameters to fix it. Eventually, I found starting conditions that allowed for a complete 180 degree arc before encountering issues. From there, I flipped it to the other side (which is allowable due to symmetry), and I had the first idea of what a solution to this system could look like. A reproduction of that initial code is pictured below:

{% highlight python linenos %}
import numpy as np
x = np.array([1, 2])
{% endhighlight %}