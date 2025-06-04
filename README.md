# Fishies - UE5 project on Stylized Ocean Environment 

This project is a standalone educational project on the topic on creating a stylized ocean environment in Unreal Engine 5. 

## Preface 
This project consist of multiple areas of shader works including waves, water effects, WPO animation shaders, postprocessing effects, multiple other shaders used for water related environment with Unreal Engine material system and custom HLSL nodes, as well as using UE5's Niagara particle system grid3D interface for simulating boids.

In this README, I will only list out techniques I've experimented with and some visual of the stages of the implementation.
The area of coverage is larger than I initially expected. With that being said, it’s possible that I have made some mistakes and misunderstanding, in which case, please feel free to correct my understanding　(´・ω・｀).

***The full article will be more detailed about implementation stages and techniques I've experimented. To read it, please check my github blog under the following url below.***
\<Url here soon \>

## Implementation stages 
### Part 1: Waves- Sine Waves, Gerstner Waves, Scrolling waves
- Here I've tried multiple implementation of WPO wave shaders such as sums of sine waves and sums of gerstner waves.
- I've also used scrolling normals, 2 sided plane waves, fresnel, depth-opacity, and added refraction for water shader.

### Part 2: Fishes - Boids
- I've used UE5 Niagara Particle system's Neighbor Grid3D as the acceleration structure for boids. 
- There are 3 variants of boids implementation I've done
  - 1 Species boids
  - Multi-Species boids
  - Predator-Prey system

### Part 3: Effects and Props - Caustics, Post-Processing, Foilage, Setting up final Scene
- The feature of caustics are created with voronoi based scrolling texture distorted by another scrolling texture
- The postprocessing material have scene depth based visibility and depth color based on the distance from the sea-level. 
  - The depth-based visibility uses a combination of custom depth and scene depth to fix issue where the area that points to the sky is invisible. 
- Foliage is animated using combination of wave functions for WPO 

## Simplified Directory Structure

```
├───blueprints
├───levels
├───materials
│   ├───boids
│   ├───caustics
│   ├───landscape_sand
│   ├───placeholder_materials
│   ├───post_process
│   ├───rocks
│   ├───seagrass
│   ├───test
│   │
│   ├───textures
│   │   ├───rocks_1024_FINAL
│   │   ├───sacabambaspis_tex
│   │   ├───sand_1024
│   │   └───seagrass_256
│   └───waves
│       ├───gerstner
│       └───sum_of_sines
├───mesh
└───particles
    ├───boids
    └───grid3dDebug
```

### Levels
| Level | Description | 
| ----- | ----- | 
| waves_test | room for testing waves and ocean related materials |
| boids_test | room for testing boids | 
| demo_scene | final scene with all the older assets intact |
| final_scene1 | Setup for camera 1 with some assets removed | 
| final_scene2 | Setup for camera 2 with some assets removed| 


# References
- https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models
- https://www.youtube.com/watch?v=PH9q0HNBjT4
- https://www.youtube.com/watch?v=kGEqaX4Y4bQ
- https://cs.stanford.edu/people/eroberts/courses/soco/projects/2008-09/modeling-natural-systems/boids.html
- https://people.ece.cornell.edu/land/courses/ece4760/labs/s2021/Boids/Boids.html
- https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf
- https://www.youtube.com/watch?v=82asza6Kv24
- https://www.youtube.com/@BenCloward