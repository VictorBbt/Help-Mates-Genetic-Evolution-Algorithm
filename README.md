# Help-Mates! Is flocking or herding a survival behaviour ?
This is a Unity project that provides tools for terrain generation, and proposes a simulation to observe flocking/herding behaviors of animals in the presence of predators

<p align="center">
  <img src="https://github.com/VictorBbt/Help-Mates-Genetic-Evolution-Algorithm/blob/main/SimulationResults/gifs/FinalRenderingGif.gif" alt="FinalRenderingGif">
</p>

This work was done as the final project for the course **INF633: Advanced 3D Graphics** given by Marie-Paule Cani at Ecole Polytechnique. The repository containing the description and the expectations for this project can be found [here](https://edualvarado.github.io/inf633-2022-2023/).

## 1 - Setup the project 🛠️

To run the project on your computer, clone [this repository](https://github.com/VictorBbt/inf633-2023-2024), open [UnityHub](https://unity.com/fr/download) and open the project **inf633-2023-2024**. The project was compiled with the **2021.3.8f1** version. It is very likely to run on more recent Long Term Support (LTS) versions, but there are not any guarantees for that. You can manage the different Unity versions you installed in UnityHub, and you can click [here](unityhub://2021.3.8f1/b30333d56e81) to install the 2021.3.8f1 that corresponds to this project.

Right now, you should be able to play around with the scene in Unity. The different tools and features you can use to make your own virtual ecosystem are detailed below.

## 2 - Make it yours 🎨

There are mainly three ways to change the ecosystem. You can either:
1.  **Shape the terrain** (new topography, new textures)
2. **Add objects** by instantiating trees, rocks (and even sakura trees !)
3. **Run the genetic algorithm** with custom species parameters, and monitor their evolution through time

All the brushes, instance brushes and the genetic algorithm work in **play mode**. Just press the start button and select whatever tool you want to try !

### 2.1 - Terrain generation 🏔️

There are a lot of custom brushes that you can use to design your terrain. The pipeline we advise to use to design a terrain is the following:
- Use the **Terrain Creation Brush** to automatically generate a whole terrain. You can play with a whole bunch of parameters to create hills or sharp ridges. Then, you can use a gaussian filter (*Number of Peaks* and *CircularFiltering* in the GUI inspector) to have moutains and valleys.

<p align="center">
  <img src="https://github.com/VictorBbt/Help-Mates-Genetic-Evolution-Algorithm/blob/main/SimulationResults/gifs/TerrainEdition.gif" alt="TerrainCreationGif">
</p>

- Use the local brushes (you can change the radius in the *CustomTerrain* GUI inspector) to locally change the topography of your terrain. There are a lot of custom brushes including *Smooth Brush, Well Brush, ...*

<p align="center">
  <img src="https://github.com/VictorBbt/Help-Mates-Genetic-Evolution-Algorithm/blob/main/SimulationResults/gifs/smooth-brush.gif" alt="LocalBrush" width="300">
</p>

- Use the **Erosion Brush** to erode the terrain (globally or locally). This is a physical simulation of droplets falling down the terrain and capturing or releasing sediments.

  
<p align="center">
  <img src="https://github.com/VictorBbt/Help-Mates-Genetic-Evolution-Algorithm/blob/main/SimulationResults/gifs/Erosion.gif" alt="ErosionBrushGif" width="400">
</p>

- Use the **Texture Brush** to add beautiful, custom textures to your terrain. You can set the height threshold of each texture, blend them and add importance to some on flat terrains or on shaded areas (by specifying the sun direction). As we can not override the *Terrain Shader* built-in with the Terrain tool, we wrote a custom script where we manually change the splatmaps. Thus, the indices you provide within the GUI inspector of this script (basically, the index of the array you initialize) are the indices that corresponds to those built-in the Terrain tool. You can find in the *Texture tab* in the Terrain inspector. We have created 6 textures, but if you want more/less/change the order, you will have to manually specify it within the brush (this will not automatically update).

<p align="center">
  <img src="https://github.com/VictorBbt/Help-Mates-Genetic-Evolution-Algorithm/blob/main/SimulationResults/gifs/Texture.gif" alt="FinalTerrainGif" width="600">
</p>

And there you go! You can now have nice looking terrain topography with custom textures. 

### 2.2 - Terrain instanciation 🌴

We now want to spawn some objects on the terrain. We downloaded several models of trees (conifers, plam trees, sakura trees, cypress), rocks and bushes, but you can download and plug in whatever asset you want from the [Unity Asset Store](https://assetstore.unity.com). However, please note that the animals that we will pop in the next part will only detect and avoid the obstacles that have a *Collider*, as well as the *Obstacle Layer*. Would you want to add objects, you have to set this up manually.

On top of that, the mechanism is a bit the same as the texture brush, that's to say that the `prefab index` you have to input in the Inspector GUI of the custom brushes corresponds to the place in the list of the loaded instances within the Terrain tool (you can find them within the *Tree Tab*).

That said, you can now use several *Instance Brushes* to spawn objects like *Cluster Brush, Height Instance Brush, Steepness Instance Brush, Poisson Instance Brush...*.

One that is worth noting is the **Poisson Moisture Instance Brush**. It first generate an invisible Perlin noise map to represent the moisture. Then, as the tree distribution in nature is close to a [Poisson Disc Distribution](https://link.springer.com/chapter/10.1007/978-3-662-56233-8_18#:~:text=Distribution%20patterns%20will%20typically%20fall,with%20plant%20establishment%20and%20growth.), the brush spawns instances following a Poisson distribution, but whether pops an object from a modifiable list `Dry Zone Objects` or `Moist Zone Objects` depending on the moisture map.

<p align="center">
  <img src="https://github.com/VictorBbt/Help-Mates-Genetic-Evolution-Algorithm/blob/main/SimulationResults/gifs/PoissonMoistureInstanceBrush.gif" alt="PoissonMoistureGif" width="400">
</p>

At that point, you should have generated a nice-looking terrain, all is left is to populate it with preys and predators 😈

### 2.3 - Genetic algorithm 🧠

I have always been fascinated by flocks and herds. These behaviours stem from simple rules as the famous [BOIDS model](https://dl.acm.org/doi/pdf/10.1145/37401.37406) that we used here. The object of this study is to see if flocking is a survival behaviour. To that end, we simulate preys with a brain that learn through time how to eat, and see how they adapt to the sudden arrival of predators. Both prey and predartors have a lot of customabke settings that you can modify within the GUI Inspector to change the simulation

#### Preys 🐏

Our approach is to let some preys learn:
1. How to eat. Grass is spawned on the terrain and preys must learn that finding this food is core to reproduce and thrive. They have a `foodBrain` that takes as input the content of the *Detail Layer* within their vision range and outputs a direction where they must go to eat.
2. How to behave in group. This second brain `reactionBrain` takes as input the vision of predators and their energy level (hunger) and outputs four weights: the weight to give to the direction computed with the `foodBrain` and three weights to give to the different components of the BOIDS behaviour (this is parallelized on a **compute buffer** to speed up the simulation). This characterizes how a single animal behaves with relation to the group.

#### Predators 🐺
The predators have no brain: their behaviour is harcoded (you can see it as a Finite State Machine). Based on their urge to eat or to reproduce, they whether look for food or for a mate. Once they locked their target, they relentlessly track it if it stays within their visions range.

#### Simulation 📈
You can find the jupyter Notebook **SimulationAnalysis.ipynb** in the SimulationResults folder if you want to have an idea of how the parameters of the Neural Networks evolve depending on the characteristics of the agents, and how it is hard to reach an equilibrium between preys and predators! We also provide a little conclusion on the limits and validity of our validation.

The rays to display the vision of the agents are drawn with `Debug.DrawRay`, so you must run play mode and come back in the scene while it is playing to display them

Whenever you want, you can save the simulation data by clicking on the *Save Data* button in the GUI Inspector. However, you must specifiy the **absolute path manually within the SaveSystem.cs script**. The code to unpack this data is provided within the notebook.

<p align="center">
  <img src="https://github.com/VictorBbt/Help-Mates-Genetic-Evolution-Algorithm/blob/main/SimulationResults/img/SuddenPredators.png" alt="GraphImageofEvolution" width="700">
</p>

Would you like to improve this method, you can find some commented code to tweak the grass and prey spawns on the map based on height and steepness, that we did not uncomment to save simulation time. We also coded **full inverse kinematics** for a quadruped. It could be nice to make our preys and/or predators really walking during the simulation! There are plenty of other ideas in the provided links listed below.

If you have any issue, insights or comments about this work, don't hesitate to [reach me](https://github.com/VictorBbt/VictorBbt), I am always happy to share 😄

# References

Here is a small bibliography with useful insights/codes that I built on to do this projects. 

**Terrain Generation**
- [Noise Map 1](https://www.redblobgames.com/maps/terrain-from-noise/)
- [Noise Map 2](https://www.decarpentier.nl/scape-procedural-basics)
- [Noise Map 3](https://www.decarpentier.nl/scape-procedural-extensions)
- [Hydraulic Erosion](https://www.youtube.com/watch?v=eaXk97ujbPQ&list=PLFt_AvWsXl0ehjAfLFsp1PGaatzAwo0uK&index=19)
- [Texture with Custom Shader](https://www.youtube.com/watch?v=XjH-UoyaTgs) (doesn't work with the Terrain tool)
- [Manual Texture Mapping](https://alastaira.wordpress.com/2013/11/14/procedural-terrain-splatmapping/)

**Terrain Instanciation**
- [Poisson Disc Sampler for Unity](http://gregschlom.com/devlog/2014/06/29/Poisson-disc-sampling-Unity.html)
- [Bridson's Algorithm](https://sighack.com/post/poisson-disk-sampling-bridsons-algorithm)
- [Spatial distribution of vegtation](https://link.springer.com/chapter/10.1007/978-3-662-56233-8_18#:~:text=Distribution%20patterns%20will%20typically%20fall,with%20plant%20establishment%20and%20growth.)

**Genetic Algorithm**
- [BOIDS in Unity](https://www.youtube.com/watch?v=bqtqltqcQhw)
- [Ecosystem Simulation 1](https://www.youtube.com/watch?v=r_It_X7v-1E): still Sebastian Lague, you should definitely check his [amazing work](https://www.youtube.com/@SebastianLague)!
- [Ecosystem Simulation 2](https://www.youtube.com/watch?v=NHk7klC-p7k)
- [Ecosystem Simulation 3](https://www.youtube.com/watch?v=0ZGbIKd0XrM) (in French)
- [Save and Load System in Unity](https://www.youtube.com/watch?v=XOjd_qU2Ido)

**Flocking behaviour**
- [Flocking and survival](https://www.jstor.org/stable/3537?typeAccessWorkflow=login&seq=3)
- [Another one](https://www.researchgate.net/publication/354475153_Flocking_in_birds_increases_annual_adult_survival_in_a_global_analysis)
- [Implementation of Delgado-Mata's model](https://www.diva-portal.org/smash/get/diva2:1105912/FULLTEXT01.pdf) (an improvement over the standard BOIDS)
