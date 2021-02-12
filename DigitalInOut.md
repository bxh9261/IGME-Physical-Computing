# Stoplight - Digital In/Outs

![](https://imgur.com/NMcinhE)

Text Rain is an interactive visual experience where the participant controls falling letters which are projected on a wall. They can do so with any part of their body or any object, as the silhouette of anything in front of the wall is projected on the screen. The user can attempt to catch letters at random, or spell out specific words, as the letters fall in the order of a poem ("Talk, You" by Evan Zimroth).
<br>

## Whodunnit, and how did they build it?
Text Rain was built by Romy Achituv & Camille Utterback. It was built in 1999, as part of a collaborative art workshop in New York City called AbacusParts. Achituv was one of the founders of the workshop, and helped guide Utterback, who came up with the idea for the piece. Utterback does not dive into the technical side of her art on her blog, but thanks to Marlena Abraham for Carnegie Mellon University, we can get a good idea of how it may work. She created a replica of Text Rain which uses an object-oriented approach. A string containing the poem is passed in, and each letter is turned into a Letter object. Each Letter object is programmed to fall down the screen, and checks the darkness of the pixels below it. If the darkness reaches a certain threshold, the letter will be pushed up. 
<br>

## What context is it aimed at?
Text Rain is intended to be an art exhibit displaying the interaction between humans and poetry. It is an art peice that can peak the interest of anyone in any demographic. It is currently exhibited at the Simthsonian Exhibit in Washington, DC, as well as museums in São Paulo, Barcelona, Pittsburgh, and Louisville. 
<br>

## What kind of experience is created?
Camille discusses in her blog how she is amazed at the different experiences that can be created. Some people move sporadically, scattering text as it falls down the screen. Others stand perfectly still, allowing the text to accumulate on their body. In groups, some people will link arms to try to catch as much text as possible. Others will focus on the poem aspect, collaborating to spell out the words of the poem. On the flip side, some group members will even attempt to disrupt the creation of the poem. Ultimately, all participants will experience interaction with the poem, but the specifics of the interaction are up to the user.
<br>

[Additional Reference - Marlena Abraham](http://golancourses.net/2013/projects/text-rain-processing-implementation/)
