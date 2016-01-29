---
title: Randomly generated world map
date: 1390782423444
tags:
---
When I first started designing a Sci-fi browser strategy game, I knew I wanted it to be grand in scope. I wanted players to be able to explore a large, randomly generated galaxy filled with stars, planets, moons, and asteroid belts. Generating all of this was easy, just pick star and planet attributes from a predetermined list and you're all set. Except in order to be immersive, I knew I'd need to generate the surfaces of these planets and moons.

It all comes down to what's better from a game design standpoint. Sure, I could simply let the player build buildings by selecting them from a dropdown list or something similarly bland, but it would be better to have a realistic-looking world map constructed for each world: generate landmasses, place rivers, approximate temperature and humidity, and determine biomes.

### Terrain generation
First you start with a [heightmap image](http://en.wikipedia.org/wiki/Heightmap). These are very common in computer graphics, especially game development. Basically, we use an algorithm to randomly generate an image of a desired size with greyscale pixels. Darker pixels represent low areas and lighter pixels represent higher areas. I use the [Diamond-square algorithm](http://en.wikipedia.org/wiki/Diamond-square_algorithm) to generate my heightmaps.

<div style="text-align: center;">
<a href="heightmap.png"><img style="border: 1px solid black; width: 300px; height: 300px;" src="heightmap.png"></a>
<br />
<i>A typical Heightmap image</i>
</div>

Next we determine a sea level. Earth has over 70% of its surface covered in water, but exoplanets likely have a very wide range of ocean-percentages. We pick a number between 0 and 255 which corresponds to the sea level that would give us our desired percent. Every pixel on the heightmap whose greyscale value is below this number will be ocean, and rest will be land.

### Rivers
There are over 150 major rivers on Earth, and our random world wouldn't be complete without them. River generation is actually pretty simple. Randomly select a starting river pixel at the higher elevations on land, make new river pixels at each lowest neighbor and "pool" by rising the water level if there are no neighboring pixels lower than you. This has the added benefit of creating lakes.

<div style="text-align: center;">
<a href="terrainmap.png"><img style="border: 1px solid black; width: 500px; height: 500px;" src="terrainmap.png"></a>
<br />
<i>Terrain map with rivers, land, and oceans.</i>
</div>

### Climate
If I were a little bit more insane, I would create a complete climate model with wind directions, oceanic currents, and atmospheric effects. But I draw the line at approximation of temperature and rainfall.

We have rivers, so rainfall approximation is easier. In my algorithm, Every river or lake pixel increases the rainfall value in a 30 by 30 area. Areas around coastlines also receive a more modest increase in rainfall.

<div style="text-align: center;">
<a href="rainfall.png"><img style="border: 1px solid black; width: 500px; height: 500px;" src="rainfall.png"></a>
<br />
<i>Rainfall map</i>
</div>

Surface temperature is simple to approximate, but one could go crazy with complexity to be realistic. It's colder at the poles and warmer at the equator. It also changes dramatically with ocean currents. I try to replicate this by increasing temperature near large oceans and over lakes and rivers. Elevation also impacts temperature, which is easier to model.

<div style="text-align: center;">
<a href="temperature.png"><img style="border: 1px solid black; width: 500px; height: 500px;" src="temperature.png"></a>
<br />
<i>Temperature map</i>
</div>

Another important metric that we need to define are biomes. There is a substantial difference between a desert near the equator and one near the poles; they occupy completely separate biomes. Deciding which biome a particular pixel belongs to involves a little bit of climate science, which thankfully [Wikipedia does well](http://en.wikipedia.org/wiki/Biome).

<div style="text-align: center;">
<a href="http://upload.wikimedia.org/wikipedia/en/2/29/PrecipitationTempBiomes.jpg"><img style="border: 1px solid black; width: 300px; height: 300px;" src="http://upload.wikimedia.org/wikipedia/en/2/29/PrecipitationTempBiomes.jpg"></a>
<br />
<i>Temperature + Precipitation = Biome!</i>
</div>

In addition to realism, biomes will also play an important part of the gameplay.

<div style="text-align: center;">
<a href="biomes.png"><img style="border: 1px solid black; width: 500px; height: 500px;" src="biomes.png"></a>
<br />
<i>Biome map</i>
</div>

<div style="text-align: center;">
<a href="key.png"><img style="border: 1px solid black; width: 150px; height: 193px;" src="key.png"></a>
<br />
<i>Biome key</i>
</div>

### Putting it all together
The previous images were simply a easy way of debugging, what we really want is a realistic-looking "satellite map" of our randomly generated world. We combine everything we know about each pixel: the elevation, temperature, rainfall, biome and blend these values together to get a RGB value to represent the pixel.

<div style="text-align: center;">
<a href="satellite.png"><img style="border: 1px solid black; width: 500px; height: 500px;" src="satellite.png"></a>
<br />
<i>"Satellite" map, work in progress. It could use more distinct deserts and ice caps.</i>
</div>

### Political maps, because grids are boring
In previous iterations of this game project, I stopped here. Players simply constructed buildings on a grid system overlayed on the satellite map. But having a grid cell the size of Europe and constructing only one building there isn't very realistic.

Now I randomly generate "territories", as I call them, to replace these grid cells. Finding a suitable algorithm to efficiently generate these was tricky. Basically, each territory is randomly placed as a single pixel which then "grows" by claiming surrounding unowned land pixels, avoiding rivers and oceans, until it stops at the border of another territory.

<div style="text-align: center;">
<a href="territories.png"><img style="border: 1px solid black; width: 500px; height: 500px;" src="territories.png"></a>
<br />
<i>Map of only territories</i>
</div>

You'll notice that not every land is claimed by a territory. There in lies the deficiency of my algorithm, and where further work is to be done. Ideally I would merge all unowned land pixels with the closest territory, but I haven't found an efficient way of doing that yet.

All of this takes around 120 seconds to run for a 1024 by 1024 map, but eventually the map size will be in proportion to the size of the planet. Obviously, I can't generate one for each world, only ones a player actually occupies. This program wouldn't be ran by the player directly, rather in a back-end service that will handle all universe and world generation.

Thanks for reading!
