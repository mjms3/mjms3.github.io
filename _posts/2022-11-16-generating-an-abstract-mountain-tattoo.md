---
title:  Generating an abstract mountain tattoo
tags:
  - coding
---

Back in 2018 I had an idea for an abstract tattoo representing
mountains based on 3D photogrammetry data. I had a go at generating an image 
then and came up with this:

<div class="card mb-3">
    <img class="card-img-top" src="{{ '/assets/images/armband_flattened.png' | relative_url }}"/>
    <div class="card-body bg-light">
        <div class="card-text">
            Initial (infeasible) tattoo.
        </div>
    </div>
</div>

When I took this in to the tattoo studio I was told, perhaps unsurprisingly, that it wouldn't
work as a tattoo. Unfortunately, how I originally generated it is now lost to the sands of time 
(and several OS reinstalls etc) so I had to start again from scratch.

<div class="card mb-3">
    <img class="card-img-top" src="{{ '/assets/images/sketchfab_model.png' | relative_url }}"/>
    <div class="card-body bg-light">
        <div class="card-text">
            3D Model of the Mont Blanc Massif
        </div>
    </div>
</div>

I found a <a href="https://sketchfab.com/3d-models/mont-blanc-massif-photographed-from-iss-c66a5a559d3844eaac942939211a4b8d">
3D Model of the Mont Blanc Massif</a> available online, made available with a CC attribution license. 
I downloaded the model and loaded it up in a jupyter notebook using 
<a href="https://trimsh.org/trimesh.html">trimesh</a>.

{% highlight python %}
filename = "./source/MontBlanc.glb"
import trimesh
mesh = trimesh.load(filename, force='mesh')
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
ax.plot_trisurf(mesh.vertices[:,0], 
                mesh.vertices[:,2], 
                triangles=mesh.faces, 
                Z=mesh.vertices[:,1],
               ) 
plt.show()
{% endhighlight %}

<div class="card mb-3">
    <img class="card-img-top" src="{{ '/assets/images/imported_mesh.png' | relative_url }}"/>
    <div class="card-body bg-light">
        <div class="card-text">
            Mesh imported using trimesh
        </div>
    </div>
</div>

I had decided that I wanted to base my tattoo on the 
<a href="https://en.wikipedia.org/wiki/Grandes_Jorasses">Grandes Jorasses</a> 
and surrounding mountains. I attempted to manipulate the mesh in my jupyter notebook in 
order to find a suitable center point for the basin containing the Grandes Jorasses, but 
I hit performance issues so I ended up downloading 
<a href="https://www.meshlab.net/">MeshLab</a> and exploring the model there.

<div class="card mb-3">
    <img class="card-img-top" src="{{ '/assets/images/meshlab.png' | relative_url }}"/>
    <div class="card-body bg-light">
        <div class="card-text">
            Finding the center of the Grandes Jorasses basin.
        </div>
    </div>
</div>

This allowed me to determine a suitable center point to use for the next step, which was
to extract the mesh vertices in a cylindrical ring around the center point.
{% highlight python %}
def generate_ring_vertices(in_mesh, center, r_outer, r_inner):
    vertex_mask = np.logical_and(
        (in_mesh.vertices[:, 0] - center[0]) ** 2 + \
        (in_mesh.vertices[:, 2] - center[1]) ** 2 \
        < r_outer ** 2,
        (in_mesh.vertices[:, 0] - center[0]) ** 2 + \
        (in_mesh.vertices[:, 2] - center[1]) ** 2 \
        > r_inner ** 2
    )
    return in_mesh.vertices[vertex_mask]
{% endhighlight %}
Note that the indices 0 and 2 correspond to the x and y dimensions in the model (a fact which
surprised me initially).

Having extracted the vertices, I projected them on to a cylinder then needed to post 
process them to generate an image that could actually be used as a tattoo.
{% highlight python %}
def generate_flattened_image_data(vertices,
                                  center,
                                  BINS=500,
                                  STEP=1,
                                  sigma=0,
                                  ):
    X = vertices[::STEP, 0] - center[0]
    Y = vertices[::STEP, 2] - center[1]
    Z = vertices[::STEP, 1]
    X_flatten = np.arctan2(X, Y)
    H, _, _, _ = scipy.stats.binned_statistic_2d(X_flatten,
                                                 Z,
                                                 np.ones(np.shape(X_flatten)),
                                                 bins=BINS,
                                                 statistic='count'
                                                 )
    H = scipy.ndimage.zoom(H, 5, order=1)
    H = scipy.ndimage.gaussian_filter(H, sigma=sigma)
    H = np.ma.masked_where(H == 0, H)

    return H


filename = "./source/MontBlanc.glb"
mesh = trimesh.load(filename, force='mesh')

gj_center = (-0.383453, 4.553252)
vertices = generate_ring_vertices(mesh,
                                  gj_center,
                                  r_outer=4.5,
                                  r_inner=0
                                  )

bins = 500
step = 12
sigma = 1
points = generate_flattened_image_data(vertices,
                                       gj_center,
                                       BINS=bins,
                                       STEP=step,
                                       sigma=(sigma, sigma)
                                       )
to_plot = points.T
plt.imshow(to_plot, cmap='magma', vmax=100)
plt.axis('off')
plt.savefig(f'bins{bins}_step{step}_sigma{sigma}.jpg', bbox_inches='tight')
plt.show()
{% endhighlight %}

<div class="card mb-3">
    <img class="card-img-top" src="{{ '/assets/images/bins500_step12_sigma1.jpg' | relative_url }}"/>
    <div class="card-body bg-light">
        <div class="card-text">
            Final image used for tattoo stencil
        </div>
    </div>
</div>

Generating this image was a process of trial and error using different techniques and ideas to come up with something
that still captured the essential idea of a mountain whilst being sufficiently simple to do as a tattoo. It's not really
possible (or probably useful) for me to go through all the steps that didn't work but the code snippet above shows
what did work!

The tattoo was done by <a href="https://www.instagram.com/jag__ink/">Jay Green</a> and looks exactly like I imagined it.

<div align="center" class="embed-responsive embed-responsive-16by9">
    <video autoplay loop class="embed-responsive-item">
        <source src="{{ '/assets/videos/tattoo.mp4' | relative_url }}" type="video/mp4">
    </video>
</div>

