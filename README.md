# Fishies - UE5 project on Stylized Ocean Environment 

This project is a standalone educational project on the topic on creating a stylized ocean environment in Unreal Engine 5. 


## Preface 
This project consist of multiple areas of shader works including waves, water effects, WPO animation shaders, postprocessing effects, multiple other shaders used for water related environment as well as using UE5's Niagara particle system grid3D interface for simulating boids.


In this README, I will only list out techniques I've experimented with and some visual of the stages of the implementation.

***The full article will be more detailed about implementation stages and techniques I've experimented. To read it, please check my github blog under the following url below.***
\<Url here soon \>

# Here is actually kinda the full part lol ↓ will move to website later

In this article, I will briefly go through the following techniques I've experimented with in several stages in chronological order. 

The area of coverage is larger than I initially expected. With that being said, it's possible that I have made some mistakes and misunderstanding, in which case, please feel free to correct my understanding　(´・ω・｀).

# Implementation and Stages

## Part 1: Waves - Sine Waves, Gerstner Waves, Scrolling waves

### Waves
Initially, this project was meant to only be focused on this section, making a simple ocean waves simulation where I try a few techniques on implementing wave shaders.

The idea of animating ocean-wave in general is to offset the vertices of the ocean-wave plane in a specific direction using a wave function. From here, we can use wave superposition by adding multiple waves with different amplitudes, wavelength, speed, etc.. together to create a more interesting shape. For this part, I've tried 2 different implementations.

If you really want to understand the waves I would recommend you read GPU Gems. 

### Sine Waves 
The first one is using sine waves where I offset the plane's vertices in z direction only. This is probably the simplest method that uses World Position Offset for simulating waves.

### Gerstner Waves
The second method here I tried and decided to use is gerstner waves, this method creates a more realistic waves with wider trough. In contrary to the sine wave implementation, this wave offset moves the waves vertex in all xyz direction. 


### Comparing the two waves
One of the benefits of using Gerstner waves is that, waves here creates wider trough than the sine waves. 

However, this does not come without a  drawback. When the steepness value gets too high $(>1)$ the wave will curl upon itself. 
(There are modified versions of sine wave that gives wider trough too but I didn't use that.)

For this implementation, I choose to continue with gerstner wave since it produces sharper peaks and also that I simply wanted to try it out. 

### Calculating wave normals

After the offset are calculated, I use cross product of the Binormal and Bitangent to calculate the normal vector where binormal and bitangent can be derived from the partial derivative of the x and y offset function. Luckily for my case, the literature from GPU Gems have already provided the complete form of the normal vector calculation which saved alot of time. 

### Extra effects - Scrolling wave normals
**Scrolling textures:** Now to improve the visuals I have added scrolling textures as additional normals to give the waves surface a more interesting look.

**Fresnel:** Afterwards, a Fresnel term is added to ensure the wave is more opaque (more reflection) when viewing at a near-grazing incedence. 

**Depth-Opacity:** To make the object underwater look less visible when it is deeper, an opacity-based depth is also calculated using scene-depth (translucent-material ignored) - pixel-depth (translucent material tested)
and scale it down by the desired depth factor. This gives a good impression of depth under water surface. 

**Refraction:** For refraction, I just set it to water's refraction index which is 1.33.
Now we can combine these method and make a pretty nice waves/water shader .

### Issues, Remedies, and Considerations

#### Two sided waves
However, given that our wave is translucent. Since Unreal Engine uses Deferred rendering, the backface of the translucent waves are no longer culled giving a triangle looking visual artifacts. 

To remedy this issue, I instead uses 2 water plane, made the mateirial one-sided and disable the depth effect on the under-side of water.

#### Mesh density and Nanite

The quality of the wave depends on the mesh density using this world position offset method. This means if this method were to be implement for a game, we should use higher vertex density mesh near the player or use a custom tesselation shader. 

I have to note that unreal engine's nanite cannot be applied to this also since we uses the world position offset on the wave's vertices and nanite would change the vertex position and density in a highly undesirable way. 

#### Tiling and Alternatives 
Both method of offsetting wave here introduces the undesirable effects of tiling. In this example it looks alright since it kind of resembles a calm ocean. However it cannot be denied that at certain angles the tiling became visible. 

This means if we want a more realistic simulation, certain techniques suchas oceanographic spectra and FFT. 

## Part 2: Fishes - Boids

Now comes the ***should-be-extra*** part which takes wayyy longer than expected. 

### Neighbor Grid3D Interface

Ever since I worked on building my own particle simulation system (N-body simulation) on GPU, I've been intersted in using a GPU accelerated data structures ever since. And this time I've stumbled on a video on UE5 Neighbor Grid3D Interface by Ghislain Girardot. Since I have never implemented this data structure myself, I will not go into details but here are the premises and use case of it.  

The Neighbor Grid3D Interface appears to be an implementation of a Bounded Spatial Hash Grid where the regions are divided into identical sized cells in xyz, direction. 

To figure out which cell the particle is in, you can applying a function based on the particle position which is stored inside a flatten 1d array.
To make the flatten position works for dense grid, there are multiple techniques employed such as partial/prefix sum and so on... 

In our use case, from there forward you can fetch the desired data of the neighbouring particle by iterating through the neighboring cells. (I'm glossing over a whole lot of things.) 

This is particulary useful for things that utilizes local particle interaction rather than global interactions which makes it especially useful for fluid simulations, and in our case, boids.

### Boids

Boids (bird-oid object) or a way to simulate flocking behaviour based on set of rules. There are three main factors when it comes to boids. 

| Rules | Description| 
| -----| ----- |
| Separation | Each particle avoids each other| 
| Alignment | Each particle steers in the direction of the flock | 
| Cohesion | Each particle steers towards the center of the flock | 

With these three rules, the particle will form an emerging behaviour 

In this final implementation, the force are calculated and applied into 2 separated stages. 

### Rules Force calculation \[Force 1\]

In this implementation, the Neighbour Grid3D Interface is used to fetch the cell of the current particle. Afterwards, I loop through the neighbouring cells index of the current cell and then iterates through each neighbouring particle in the cells.

After we get the neighboring particle positions, we calculate whether the particle is in the field of view using the dot product of the normalized current particle velocity and the direction of the current particle position to the neighbour. 

If the neighboring particle is in the FOV, we can calculate the following forces.

```cpp
// Calculate separation value
float separationRatio = 1 - clamp(length(directionParticleToNbrs) / inSeparationFalloff, 0, 1);

if (separationRatio > 1e-5)
{
    outSeparationForce -= separationRatio * directionParticleToNbrs; // -= since we are avoiding
    separationCount += 1.0;
}

// Calculate alignment value (flock direction)
float alignmentRatio = 1 - clamp(length(directionParticleToNbrs) / inAlignmentFalloff, 0, 1);

if (alignmentRatio > 1e-5)
{
    outAlignmentForce += alignmentRatio * normalize(nbrsVelocity);
    alignmentCount += 1.0;
}

// Calculate Cohesion value (flock center of mass)
float cohesionRatio = (1 - clamp(length(directionParticleToNbrs) / inCohesionFalloff, 0, 1)) ? 1 : 0;

if (cohesionRatio > 1e-5)
{
    outCohesionForce += nbrsPos * cohesionRatio;
    cohesionCount += 1.0;
}
```

We then take each ratio and divide them by the total count of particle, normalize the value, then scale it by a user defined factor.

### Introducing several species, Predator-Prey force calculation \[Force 2\]

Boids species are distributed based on a user-defined curve. 
The rule in this implementation is simple and forces calculated very similary for intra species boids. 

For particles that are in different species, they will avoid each other and will not align or cohese. 
Prey will try to avoid predator and predator will try to follow prey. 

### Setting boundaries \[Force3\]
I use 3 ways initially to create boundary for keeping boids inside. 

The first method uses a box scaled by the scale of the particle system. If the particle goes out of bound, it will immediately reflects back in. This creates an undesirable effect of extremely sharp turn.

The second method also uses a box but then only apply forces when the particle goes out of bound for a smoother turn. 

The third method creates a spherical region where the particles inside the region will not be affected by the force. However, when the particle leaves the region, it will apply a weak force to increase the tendency of particles to stay inside the region.

### Applying Forces

The forces are applied into 2 stages

\[Force1] and \[Force2] are added together with the velocity, scaled by delta time and a constant speed. (This somewhat gives a smoother movement)

Then in another pass the same is done with \[Force3], this is to ensure the bounding force stays relatively strong.

### WPO animation on fish vertex
The animation is done simply by using the wave function to offset the world positon
$$f(x,t) = Asin(kx+\omega t)$$
Where x is the world position. I should've vertex paint the face to mask the movement but it looks funnier this way. 

### Results 
This made me realize boids are so difficult to control and to make it work as desired.. 

## Part 3: Effects and Props - Caustics, Post-Processing, Foilage WPO animation, Setting up final scenes

### Caustics 
The caustics are created with 2 textures.

The first texture is the base caustics, created using 2 smooth f1 voronoids of the same position setup subtracing the one with lower scale, this create a smooth pattern resembling caustics. Using the same setup, I created 3 other textures with minimal offset in xy direction to combine and create chromatic abberation effect. 

The other texture is simply a scrolling wave texture which is added into the UV position of caustics to distort it.

### Post-Processing, Depth, Distance fog

The post procesing effect uses distance fog based on the depth buffer to make position further away from screen become less visible. This however poses one issue, meaning that for areas above the ground, the sky, the sun will be entirely invisible. 

To remedy this issue, I have tried a few method
#### Method 1: Ray-Plane intersection
I've tried drawing a ray plane intersection test based on sea level to see if the direction that ray points to is above the sea level or not. This works fine in vertex shades but it would mean I would have to find a way to write to a custom texture to use in the later stages. Since this project is already getting quite long I've decided to try other methods instead. 

#### Method 2: Custom depth pass
In this method I chose to create a few objects that only draw to the custom depth pass. I then set the scene and moves the mesh with custom depth to the desired sea level. This actually worked quite well lol.

So in the end I opted in to use Method 2 which is simpler. 

I've also added the sea-level based depth color which simply uses the sea-level and lerp the color based on how far one is from the sea-level.

The final post-processing object is now done with the custom depth pass governing the scene depth based visibility and the distance form sea level governing the color.

### Seagrass WPO animation - Global wind, Localwind

For local wind: 

I used wave function with higher wavelength on Z direction to maximizes movement speed (the feel) on xy direction instead of z. 


For global wind: 

I just used a few normal wave funcitions for this, one to mask the position of the wind in sections. The other to move the wind, both are simple implementations of wave function.

### Setting up final scenes

Just add images with description for this section (this is getting too long for real)
- Blockins 1-2
- Sculpted some stone mesh and bake it 
- Made some foiliage cards
- Added boids
- Added camera 
- Added volumetric lighting 
- Done!

# References
- https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models
- https://www.youtube.com/watch?v=PH9q0HNBjT4
- https://www.youtube.com/watch?v=kGEqaX4Y4bQ
- https://cs.stanford.edu/people/eroberts/courses/soco/projects/2008-09/modeling-natural-systems/boids.html
- https://people.ece.cornell.edu/land/courses/ece4760/labs/s2021/Boids/Boids.html
- https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf
- https://www.youtube.com/watch?v=82asza6Kv24
- https://www.youtube.com/@BenCloward