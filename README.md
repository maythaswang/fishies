# Fishies - UE5 project on Stylized Ocean Environment 

This project is a standalone educational project on the topic on creating a stylized ocean environment in Unreal Engine 5. 


## Preface 
This project consist of multiple areas of shader works including waves, water effects, WPO animation shaders, postprocessing effects, multiple other shaders used for water related environment as well as using UE5's Niagara particle system grid3D interface for simulating boids.

The area of coverage is larger than I initially expected. With that being said, it's possible that I have made some mistakes and misunderstanding, in which case, please feel free to correct my understanding　(´・ω・｀).


In this README, I will very briefly go through the following techniques I've experimented with in several stages in chronological order. 



***For the full article, please check my github blog under the following url below.***
\<Url here soon \>

# Implementation and Stages
The project can chronologically be separated into 3 main stages which are ocean waves shaders, boids, effect shaders. 

## Part 1: Waves - Sine Waves, Gerstner Waves, Scrolling waves

Initially, this project was meant to only be focused on this section, making a simple ocean waves simulation where I try a few techniques on implementing wave shaders.

### Sine waves
In order to simulate an ocean wave, the first technique I tried to use is using sum of sine waves based on what was defined in GPU gems. The idea was simply to take the world position of the vertex's plane and offset them in Z position based on the time. Given multiple simple waves with different wavelength, amplitude, speed, and direction, we can combine them using wave superposition to create an interesting pattern. 

This method does give some intersting patterns but are a bit hard to control and the wave through are not wide enough and it also tiles very quickly.  

### Gerstner waves 
The second method here I tried and decided to use is gerstner waves, this method creates a more realistic waves with wider trough. In contrary to the sine wave implementation, this wave offset moves the waves vertex in all xyz direction.  

TODO: (write the issue on wave curling into itself or sth)

### Scrolling textures, Fresnel, Depth Color 


### 2 Sided waves, Issues, and Remedy

### Mesh density and Quality of wave 



## Part 2: Fishes - Boids, Grid3D interface 

### Grid3D interface


### Boids 


### Introducing several species 


### Predator-Prey 


### Force calculation and setting boundaries

### WPO animation on fish vertex

## Part 3: Effects and Props - Caustics, Post-Processing, Foilage WPO animation

### Caustics 


### Post-Processing, Depth, Distance fog

### Seagrass WPO animation - Global wind, Localwind













<!-- ### Table of contents 
### 1. Waves - Sine waves, Gerstner Waves, Waves shaders 

### 2. Boids, Grid3D interface, Spatial grid 3D 

### 3. Water shaders, Caustics, Wind shaders, Post processing  -->





<!-- The project is comprises of 3 separated parts joined together  -->