# Pixi.js
This is my attempt at simplifying the explanation of what is pixi.js, and how to use it.

## what is it?
Pixi.js is basically a JavaScript library for rendering graphics with good performance.

### Why not just use canvas?
Pixi.js is supposed to be easier and more performant.

## How it works in practice
Pixi.js is basically a tree of objects. The pixi people are calling the tree a "scene graph" and the objects "containers".
At the top of the tree (the root node) you have the application container which they call a "Stage".

### Getting started
Pixi is divided to core and extra packages, to make sure we can use them all with the least amount of headache we need to use a modern buildtool, we'll be using vite.\
Create a vite app, choose whatever framework you want in the prompts, I'll be using React (if you're using a framework other than React or Vanilla JS you'll have to figure where to put the pixi code yourself).
```
npm create vite@latest
or
yarn create vite
or
pnpm create vite
```
*using a buildtool prevents a lot of headache dealing with the pixi extra packages.

### Stage
to get a stage on screen just make sure you have access to the PIXI library through installation:
```javascript
npm i pixi.js
```
 or the CDN `<script src="https://pixijs.download/releas/pixi.js"></script>`
and then type or copy the following:\
**react**
```javascript
/*
for the rest of this guide I'll assume usage of this template for react and just write the PIXI code.
*/
import { useEffect, useState, useRef } from "react";
import * as PIXI from "pixi.js";

export default function App() {
  const [pixiApp, setPixiApp] = useState(null);
  const appDiv = useRef(null);

  useEffect(() => {
    if (!pixiApp && appDiv.current) {
      // PIXI CODE GOES HERE

      const app = new PIXI.Application({width: 800,height: 600,});
      appDiv.current.appendChild(app.view);

      // PIXI CODE ENDS HERE

	  setPixiApp(app);
    }
    return () => {
      pixiApp?.destroy(true, true);
      setPixiApp(null);
    };
  }, []);

  return <div ref={appDiv}></div>;
}
```
**vanilla js**
```javascript
const app = PIXI.Application({width: 800, height:600});
document.body.appendChild(app.view);
```
You should see a black box apear on your screen.\

Any descendant of the stage which is a `DisplayObject` is what will get rendered to the screen.
The main objects that can get rendered to the screen are:
- `Container` - boxes to hold and group stuff
- `Graphics` - basic shapes (circle, square, line, polygon, ...)
- `Sprite` - image files
- `Text` - using any font and css style you want (expensive to change at runtime)
- `BitmapText` - letters are taken from an image file (very cheap to change at runtime but can't be styled anyway you want).

You can see the full list of `DisplayObject`s [here](https://pixijs.download/release/docs/PIXI.DisplayObject.html).\
*You can also see the properties and methods that all `DisplayObject`s share.

If you nest objects within other objects their display properties become relative to the ones of their parents so for example:
If you rotate the parent you rotate the child.
If you move the parent you move the child.
If you change the opacity of the parent you change the opacity of the child.
And so on...

### Put something on the stage
Now that we have a stage setup let's add a Graphics object to the stage, say a circle/s?
```javascript
const circles = new PIXI.Graphics()
circles.beginFill(0x00ff00); // hex-color to fill the circle
circles.drawCircle(400,300,50); // x, y, radius
circles.drawCircle(300,300,20);
circles.endFill(); // stop color from leaking to other shapes.
circles.beginFill(0x0000ff);
circles.drawCircle(500,300,35);
app.stage.addChild(circles)
```
a single Graphics object can make more than one shape.\
*a little note** - the `drawCircle` function we just used to create the circles doesn't actually draws them to the screen. Instead, think of it as a "buildCircle" function.
The Graphics object has a few more of these "draw" functions and you should treat them all as "build" functions instead.
So what actually draws them to the screen? that's what we talk about next.

### addChild(child)
As you just saw, to add anything to the stage we just use the `app.stage.addChild` function and this is what made the circles appear on screen.\
So when things get added to the stage, thats when they get rendered.\
Also `addChild` is not special to the stage, it's a `Container` function, and since all `DisplayObjects` extend the `Container` class anytime you want to nest one object in another just use its `addChild` function.

### Basic animation
To add basic animations to your display objects (changing their display properties over time) we use the 'Ticker' class.
When you initialize a ticker instance you pass it a callback function that runs on every 'tick'. in that function you change the display properties of whatever DisplayObject you want.
The callback gets as an argument the "delta-time" (I'll expand on that in a bit).
lets make our circles rotate:
```javascript
let angle = 0
circles.position.set(400,300); // set our Graphics container position to that of our middle circle
circles.pivot.set(400,300); // set the origin of our container to be the same as our position

// add the ticker
app.ticker.add((delta) => {
	angle = angle < 360? angle + 1: 0; //just to keep the number from climbing (if it will climb too high it can cause a crash)
	circles.angle = angle; //update the angle of the container every tick.
})
```
#### About delta-time
delta-time is a misleading name, the actual value it holds is the number of frames
that have passed since the last time the callback was invoked.\
what? shouldn't the callback be invoked on every frame?\
Yes! it should!, unfortunately, sometimes due technical issues or overload the
GPU can't keep up with the screen's refresh rate and ends up missing a frame
or more. To avoid your animations lagging behind you can multiply your update values by the delta argument.
```javascript
app.ticker.add(delta => {
  circle.angle += 1 * delta;
})
```
usually it returns 1 or 0.99XX... which means no frames were skipped and it wouldn't change your update value. but in case some frames were skipped it will compensate for it by increasing your update value.

#### About frame rate
By default the ticker tries to match your screen's refresh rate, for some screens its 60 fps but for other screens this can go up to 120 fps, 144 fps and even 240 fps. This means that you might want to limit the ticker's maximum fps as otherwise people with higher refresh rate might experience your game at 2 or even 4 times the speed it's supposed to be played at. To do it all you need to do is:
```javascript
app.ticker.maxFPS = 60;
```

### Preloading
#### why?
Let's say you have bunch of assets in your game (or whatever it is you're making). If you don't load them ahead of time your app will not function right, images will not appear on screen and then jump out of nowhere, sounds will not play and things will generally won't work properly and hurt the user experience.

#### how?

use the Pixi 'Assets' class and its functions: `Assets.init({manifest})`, `Assets.load()`,  `Assets.loadBundle()`, `Assets.backgroundLoad()`, `Assets.backgroundLoadBundle()`.
All of the above functions return a promise so you need to use either "async await" or the ".then" syntax.

- `Assets.init({manifest})` / `Assets.addBundle(bundleId,assets)`:
  bundles allow you to load a bunch of assets in one go, but to load bundles you first have to give them an ID. Using `Assets.init` you can define all your bundles when the app starts or if you want to add bundles later you can use `addBundle`.\
  A `bundleId` can be any string you want.\
  `assets` is an object `{name1: resource, name2: resource2, ... }`\
  A `manifest` is an object or JSON file of the following structure:
```javascript
const manifest = {
  bundles: [
	{
	  name: "game-bundle",
	  assets: [
		{
		  name: "player",
		  srcs: "player.png"
		},
		{
		  name: "enemy",
		  srcs: "enemy.png"
		},
	  ]
	},
	{
	  name: "bundle-2",
	  assets: [
		{
		  name: "loading-bar",
		  srcs: "loadingbar.png"
		},
		{
		  name: "loading-font",
		  srcs: "loading.font"
		},
	  ]
	}
  ]
}
```

preloading example with `Assets.init()`:
```javascript
 const  app = new  PIXI.Application({
	width:  800,
	height:  600,
	backgroundColor:  0xaaaaaa,
});
document.body.appendChild(app.view);

 PIXI.Assests.init({manifest: 'manifest.json'})
 .then(() => {
	 PIXI.Assets.loadBundle('game-bundle')
	 .then(loaded_resources => {
		 // the code in here will run only once our bundle has loaded.
		 
		 console.log(loaded_resources) // { player: Texture, enemy: Texture }
		 //The loadBundle function made Texture objects out of our .png files.
	 
		 //we can now safely add our sprites to the stage knowing they won't
		 // cause any problems
		 const sprite1 = new PIXI.Sprite(loaded_resources.player);
		 const sprite2 = new PIXI.Sprite(loaded_resources.enemy);
		 app.stage.addChild(sprite1, sprite2);
	 })
 })
```

### Sprite animation
create an instance of the `AnimatedSprite` class with an array of textures
in the order you want it to play.
you can change the speed of the animation by assigning a value to the `animationSpeed` property (0 - 1).

```javascript
const animation = new AnimatedSprite([PIXI.Texture.from('sprite1.png'), PIXI.Texture.from('sprite2.png')]);
app.stage.addChild(animation);
animation.play();
```

### Containers
Containers in PIXI are basically boxes to put other stuff in such as sprites, text, and graphics (basic shapes or composites). and they serve 3 main purposes:
1. grouping
2. masking
3. filtering

#### Grouping
when you want a few elements to move or change appearance all together (like how a weapon sprite might move with the player sprite) you can nest them in a container using the `addChild` function we mentioned before and then make the changes to the container instead of each of them separately:
```javascript
const manWithSword = new PIXI.Container()
const man = new PIXI.Sprite.from('man.png');
const sword = new PIXI.Sprite.from('sword.png');
manWithSword.addChild(man, sword);
manWithSword.x = 300;
```
Remember, children's "display properties" (properties that all display objects have) are relative to those of their parents so if the parent moves the children move with it.

#### Masking
To "mask" something in PIXI means to make it visible only through the mask it has put on. To illustrate, if I make a `Graphics` instance that renders a circle and I set it as the mask for a container with a sprite in it. I will only be able to see the sprite where the circle is overlapping the container.
here is a somewhat more elaborate example:
```javascript
// create the mask
const  mask = new  PIXI.Graphics();
mask.beginFill(0xffffff);
mask.drawCircle(400, 300, 50);

// create the container that will be masked
const  maskedContainer = new  PIXI.Container();
maskedContainer.mask = mask;

// create some stuff to put in the container
const  circles = new  PIXI.Graphics();
circles.beginFill(0x0000ff);
circles.drawCircle(400, 200, 35);
circles.beginFill(0x00ff00);
circles.drawCircle(300, 300, 35);
circles.beginFill(0xff00ff);
circles.drawCircle(500, 300, 35);
circles.beginFill(0x00ffff);
circles.drawCircle(400, 400, 35);

// more stuff to put in the container (to make the difference more obvious)
const  background = new  PIXI.Graphics();
background.beginFill(0xaa2222);
background.drawRect(0,0,app.screen.width,app.screen.height)

// put the stuff in the container
maskedContainer.addChild(background, circles);

// add the container to the stage
app.stage.addChild(maskedContainer);

// animate the mask (to help illustrate)
let  angle = 0;
app.ticker.add(delta  => {
  angle = (angle + 1 * delta) % 360;
  const  radians = angle * Math.PI/180;

  mask.clear();
  mask.beginFill(0xffffff);
  mask.drawCircle(400, 300, 100 + Math.cos(radians) * 34);
})
```

#### Filtering
Filters are special effects that can be applied to `DisplayObjects` like sprites and text. Filters can change the appearance of objects in many ways, for example by adding a blur effect, adjusting the color saturation and much more!\
pixi has two types of filters: core filters and community filters. Core filters are included in the `PIXI` library and are maintained by the pixi team. Community filters are created by members of the pixi community.
To see a list of all the available filters in pixi and live examples of them check out this [repo](https://github.com/pixijs/filters). It's really cool!\

Note* - most of the filters in this list are community filters that are not part of the core `PIXI` library which means they need to be installed or downloaded separately.
for the Built-in Filters list, just scroll to the end of the list and you'll see them.

Let's see how to use a built-in filter (`BlurFilter`):
```javascript
// Create a filter instance
const blurFilter = new PIXI.BlurFilter();

// create something to blur
const circle = new PIXI.Graphics();
circle.beginFill(0x0000ff);
circle.drawCircle(400,300,50);

// create a container to applay the filter to
const container = new PIXI.Container()

// add the circle to the container
container.addChild(circle);

// apply the filter to the container
container.filters = [blurFilter];

// put the container on the stage
app.stage.addChild(container);
```
Now every child of the container will have a blur filter applied to it.
*You don't have to apply filters to containers, you can also apply them directly to other display objects (sprites, graphics, shapes, text, etc...)

To use community filters is pretty much the same except you have to first install or download them and when you instantiate them you do it directly instead of through `PIXI` so instead of:
```javascript
const filter = new PIXI.BloomFilter()
```
you'll code:
```javascript
const filter = new BloomFilter()
```
To learn how to install the community filters check the repo I mentioned earlier [here](https://github.com/pixijs/filters) and go all the way to the bottom.

### BaseTextures, Textures and Sprites
You may have seen `Texture` appear in the above code here and there but I still didn't explain it, let's fix that now!\
when dealing with images in pixi there are 4 things in play:
1. the image file that contains all the data of the image
2. the `BaseTexture` which takes all the data from the image file and loads it to the GPU
3. the `Texture` which references all or a portion of the data from the `BaseTexture`
4. the `Sprite` which is a `DisplayObject` that "wears" a `Texture` as its "face" and can move and be seen in the 'game world'/stage\

Now you might be thinking to yourself (I know I did), why so complicated?
In short, performance. In length, maybe later in this guide, but in case you don't really care why and just want to understand how (to use it), think about it this way:\
The `BaseTexture` is a piece of paper with a drawing on it.\
The `Texture` is a smaller piece of paper that we cut from the `BaseTexture`.\
The `Sprite` is a piece of plastic that we stick the small piece of paper to.\
Now we can use our piece of plastic with a picture stuck to it as a 'character' on our board game.\

Thankfully, because this is all digital, we can cut as many small pieces of paper from the bigger piece as we want, even if we already cut that area before.\
And in the same way we can also stick the small piece of paper to as many pieces of plasic as we want.\

So now hopefully even if you don't understand the technical details underneath, you at least understand the relationship between all the parts of this system and why we use it like we do.\

Ok, so using the preloading techniques we used earlier is the best way to load images into your project, but if you want to be more quick and dirty (for example when prototyping or during development) you can use the following:\
to load a single image to a sprite
```javascript
// when served through the internet this will cause a popin effect as
// it takes time for the image to download to the browser then loaded into
// the GPU
const sprite = new PIXI.Sprite.from('image.png')
```
Once you're done with your sprites and textures (you have no more need for them in your app) you should free up the memory they take:
```javascript
sprite.destroy();
texture.destroy();
baseTexture.destroy();

// or alternatively
sprite.destroy({
	children: true, // destroy children if there are any [default is false]
	texture: true, // destroy the texture the sprite has been using [default is false]
	baseTexture: true, // destroy the baseTexture used by the sprite's texture [default is false]
})
```
this removes references to the texture in your app and lets the garbage collector remove it from memory. When all textures of a `BaseTexture` have been destroyed the `BaseTexture` will also be removed from memory and from the GPU.

### Graphics
As we mentioned before `Graphics` in pixi is the name of an Object that lets you 'draw'/build simple shapes. A single `Graphics` instance can build one or more shapes that can be combined to create composites. here is a list of the shapes a `Graphics` object can build,\
Built in shapes:
-   Line - `lineTo(x,y)` (use `moveTo(x,y)` to where you want the line to start)
-   Rectangle - `drawRect(x,y,width,height)`
-   Round Rectangle - `drawRoundedRect(x,y,width,height,radius)`
-   Circle - `drawCircle(x,y,radius)`
-   Ellipse - `drawEllipse(x,y,width,height)`
-   Polygon - `drawPolygon(…path)`
-   Arc - `arc(cx, cy, radius, startAngle, endAngle, anticlockwise)`
-   Bezier Curve - `bezierCurveTo(cpX, cpY, cpX2, cpY2, toX, toY)`
-   Quadratic Curve - `quadraticCurveTo(cpX, cpY, toX, toY)`\

Extra shapes that come with the `@pixi/graphics-extras` package:
-   Torus - `drawTorus(x, y, innerRadius, outerRadius, startArc, endArc)`
-   Chamfer Rectangle - `drawChamferRect(x, y, width, height, chamfer)`
-   Fillet Rectangle - `drawFilletRect(x, y, width, height, fillet)`
-   Regular Polygon - `drawRegularPolygon(x, y, radius, sides, rotation)`
-   Rounded Polygon - `drawRoundedPolygon(x, y, radius, sides, corner, rotation)`
-   Star - `drawStar(x, y, points, radius, innerRadius, rotation)`\

You use the extra graphics exactly like you would the built-in graphics except you have to add an import of the `@pixi/graphics-extras`:
```javascript
import * as PIXI from "pixi.js";
import "@pixi/graphics-extras"; // <-- add extras import

// const app = new PIXI.Applic...

// create a Graphics instance
const graphics = new PIXI.Graphics();
graphics.beginFill(0xffff00);

// create the star in graphics
graphics.drawStar(400, 300, 5, 50, 30) // <-- use extras like built-in graphics

// add graphics to the stage
app.stage.addChild(graphics);
```
Note* - the draw functions in the `@pixi/graphics-extras` package can cause trouble with TypeScript, you can try to fix it yourself or use the hacky way I used:\
`graphics.drawStar ? graphics.drawStar(400, 300, 5, 50, 30) : null;`\
Note* - If you are not using a modern build tool (such as vite or webpack) the Pixi packages that are not built-in to Pixi can cause a lot of trouble.

#### Changing Graphics after creation
To change the geometries (shapes) of a `Graphics` instance you have to use the `clear` method and then rebuild the shape/s the way you want it to be.
```javascript
const graphics = new PIXI.Graphics();
graphics.beginFill(0x00ff00);
graphics.drawCircle(400,300,50);

app.stage.add(graphics);

// then some event happnes and I want chnage the circle to be a square
graphics.clear();
graphics.beginFill(0x00ff00);
graphics.drawRect(375,275,50,50);
```
You could also use this method Inside a ticker callback to create an animation but the pixi docs warn about performance issues that could arise from that. Instead, for shapes that need to be changed every frame, they recommend using the `PIXI.Mesh` class which requires some WebGL knowledge (this will be covered later).

#### Graphics performance
The pixi docs advise you to not put too many shapes in one `Graphics` instance as it can hinder performance. Instead, they suggest that you create more `Graphics` instances and spread your shapes between them.\
Also, you can create instances of the `Graphics` object by passing the geometry of another `Graphics` instance into their constructor, doing this will make both instances share the geometry of the first instance:
```javascript
// create the instance that will own the geometry
const geometryOwner = new PIXI.Graphics();
geometryOwner.beginFill(0xffff00);
geometryOwner.drawCircle(400, 300, 50);

// create the instance that will refrence the geometry
const geometryUser = new PIXI.Graphics(geometryOwner.geometry);

// the geometry object of both instances is the same one
geometryOwner.geometry === geometryUser.geometry // true
```
Just make sure to call `geometryUser.destroy()` when you're done with it, otherwise the `geometry` object will not be garbage collected even if the geometryOwner is destroyed, causing a memory leak.

### Particle effects
To use particles in pixi we'll use the `ParticleContainer` class, it is basically a regular `Container` but with some limitations which makes it more performant (more on the limitations later).\
Using particles can feel very complicated but I'll show you it's not too bad.\
pixi has an online particle editor that you can play with and use to customize your particles.\
Once you are happy with the particles, you can download a JSON config file which we'll feed into the particle emitter to replicate the effect in our project.\
[Particle editor](https://pixijs.io/pixi-particles-editor/).\
Note* that the particle is using an image, you can see it and download it in the "particle properties" section of the editor, or you can upload and use your own.\
Note** the particle editor generates an old version of the emitter configuration but don't worry, they made a function that accepts that old configuration and converts it to the new one so don't get confused when you see it.\

The first step to be able to use particles is to install the `@pixi/particle-emitter` package.
```
npm i @pixi/particle-emitter
```
with the package installed go through the following steps: 
1. download the particle JSON file from the online editor
2. download the particle image from the online editor (if you didn't use your own)
3. add the image file to your public folder - `puclic/images/particle.png`
4. change the JSON file to .js or .ts (whatever you are using) and export the JSON object
```javascript
// src/data/emitter.js

export const emitterConfig = {
 // json data
}
```
5. import all the things we need and use the emitter in the project
```javascript
// src/app.js

import { Emitter, upgradeConfig } from  "@pixi/particle-emitter";
import { emitterConfig } from  "./data/emitter";

const  particleContainer = new  PIXI.ParticleContainer();
const  emitter = new  Emitter(particleContainer,upgradeConfig(emitterConfig,"images/particle.png"));
emitter.autoUpdate = true; // you can have that flase but then you need to control the timings of emittion
emitter.updateSpawnPos(400, 300); // the position of the emitter
emitter.emit = true; // a switch to stop and start emittion
app.stage.addChild(particleContainer);
```
That's it, it should be working now.\
Regarding the limitations of the `ParticleContainer` it cannot be masked or have filters applied to it and some more advanced features won't work with it. I would tell you exactly what these features are but the documentation is quite vague about it so I have no idea.

Be aware that if you go too far with the effects (too many particles or complicated behavior) it can significantly hurt performance so don't overdo it.

### Sprite sheets
- what are they?
- how you create them?
- how you use them?

#### What are sprite sheets and why should I use them?
Sprite sheets are basically just a bunch of individual images bunched up into one image file.\
The benefits of this is faster initial load time for your project and better batch rendering at runtime.\
The initial load time improvement is due to less requests to the server as you only need one file instead of many. This benefit becomes more obvious the more images you have.
The improvement in batch rendering helps WebGL draw our sprites faster and is due to all of our sprites coming from a single `BaseTexture`.

#### How do I create them?
You may think or want to create sprite sheets manually by yourself but you don't need to! and it's better that you don't!\
You see, putting a bunch of images in a file is nice but you then need to also map each sprite to its location on the image and you need to do it in a JSON structure that the pixi spritesheet parser can understand. 
Well instead of doing all that, if you have a bunch of individual images you want to turn into a sprite sheet you can use a spritesheet packer such as one of these:\
[sprite sheet packer](https://www.codeandweb.com/free-sprite-sheet-packer), [Shoebox](https://renderhjs.net/shoebox/), [spritesheet.js](https://github.com/krzysztof-o/spritesheet.js).\
These packers will take your images and arrange them in a single file and they'll also generate the JSON file with all the information about each image's location and other meta data. You just need to download the generated image file (the sprite sheet) and the json that hold all the meta data and load them to your project.

#### How to use sprite sheets in Pixi
You can do it in 2 ways:\
using the `Assets` class:
```javascript
import { Assets } from 'pixi.js';
const sheet = await Assets.load('images/spritesheet.json');
```
or directly with the `Spritesheet` class:
```javascript
import { Spritesheet } from 'pixi.js';

// texture - needs to be a Texture or BaseTexture object
// spritesheetData - is the json file or js object with the meta data
const sheet = new Spritesheet(texture, spritesheetData);
await sheet.parse(); //<-- notice it returns a promise
console.log('Spritesheet ready to use!');
```
Note* - both ways are asynchronous!\
With the `sheet.textures` you can create Sprite objects, and with `sheet.animations` you can create an `AnimatedSprite`:
```javascript
await sheet.parse() // assume this is called inside an async function
const sprite = new PIXI.Sprite(sheet.textures.textureName);
const animation = new PIXI.AnimatedSprite(sheet.animations.animationName);
animation.animationSpeed = 0.05; // speed in seconds
app.stage.addChild(animation, sprite);
animation.play();
```
Sprite sheets can define animations, anchors for each individual sprite and more, here is an example of what a `spritesheet.json` can look like:
```json
{
    "frames": {
        "enemy1.png":
        {
            "frame": {"x":103,"y":1,"w":32,"h":32},
            "spriteSourceSize": {"x":0,"y":0,"w":32,"h":32},
            "sourceSize": {"w":32,"h":32},
            "anchor": {"x":16,"y":16}
        },
        "enemy2.png":
        {
            "frame": {"x":103,"y":35,"w":32,"h":32},
            "spriteSourceSize": {"x":0,"y":0,"w":32,"h":32},
            "sourceSize": {"w":32,"h":32},
            "anchor": {"x":16,"y":16}
        },
        "button.png":
        {
            "frame": {"x":1,"y":1,"w":100,"h":100},
            "spriteSourceSize": {"x":0,"y":0,"w":100,"h":100},
            "sourceSize": {"w":100,"h":100},
            "anchor": {"x":0,"y":0},
            "borders": {"left":35,"top":35,"right":35,"bottom":35}
        }
    },

    "animations": {
        "enemy": ["enemy1.png","enemy2.png"]
    },

    "meta": {
        "image": "sheet.png",
        "format": "RGBA8888",
        "size": {"w":136,"h":102},
        "scale": "1"
    }
}
```

### Text & BitmapText
There are two kinds of text in Pixi:

-  `Text`
-  `BitmapText`

in short:\
`Text` - Good for dynamically styled text that doesn't change too often.\
`BitmapText` - Good for statically styled text that needs to be changed very frequently.

in length:
#### `Text` - performance
`Text` - is basically taking the browser's normal text rendering and rasterizes it (turning vector data into pixel data) into an image. once rasterized it acts like a `Sprite`, meaning you can scale, rotate, position and tint it as you like. Because of the rasterizing, changing the text (content or style) is a somewhat expensive procedure, that is why it is not meant to change often.\
That being said, don't be afraid to change `Text`, You can possibly get away with changing one `Text` object every frame but if you try to do it with a few at once you'll likely get performance issues.\
Test your project and see what's your limit, and take in consideration that your users might have weaker machines than yours.

#### `Text` - preloading
Because `Text` is using the browser's normal text rendering you could encounter a situation where the font you defined is not being applied to your `Text`. That's because fonts need to be loaded and if the browser is ready to display text on the screen before the font is loaded it will default to a different font until the font loading is done. Because `Text` is rasterized into an image it won't change along with the browser's normal text rendering once the font is loaded and you'll get stuck with the default font (unless you re-generate your `Text` after the font loads, but that will cause it to pop-in which is not an ideal user experience).\
To ensure the font is loaded you'll need to use a 3rd party library such as [FontFaceObserver](https://fontfaceobserver.com/).\
Unfortunately, Pixi's `Assets` class is not able to do it for us.

#### `Text` - creation & usage
If you want to visually style your font you can use an online tool that will both style it and generate the necessary code for you to supply Pixi with.\
here is the tool: [pixi-text-style](https://pixijs.io/pixi-text-style/#).
Once you're done styling just copy/download the generated JSON or JavaScript to your project.\
using JSON
```javascript
import  textStyle  from  "./textStyle.json";

const  text = new  PIXI.Text('Styled text content', textStyle);
text.position.set(app.screen.width / 2 - text.width / 2, 30);
app.stage.addChild(text);
```
using JavaScript
```javascript
// textStyle.js
export const textStyle = {
	// styles copied from pixi-text-style online tool
}

// App.js
import  { textStyle }  from  "./textStyle.js";

const  text = new  PIXI.Text('Styled text content', textStyle);
text.position.set(app.screen.width / 2 - text.width / 2, 30);
app.stage.addChild(text);
```

#### `Text` - change text
to change the text content of a `Text` after it has been instantiated:
```javascript
textInstance.text = 'new text';
```

#### `Text` - final notes
Changing the regular display properties of `Text` (position, scale, rotation, etc... ) does not regenerate the text so don't worry about using those.\
One thing to note though is that scaling `Text` up past its default style-size (scale > 1) will cause it to get pixely/blurry. A solution for this could be making the default style as big as you need it to get and then scaling it down to the other sizes you need. It takes more memory but your text will maintain a clean look.

  
#### `BitmapText` - general info
`BitmapText` - this type of text is basically generated from a sprite sheet of all the letters and symbols you are going to use so unlike `Text` it does not need to go through rasterizing. Because of this, changing the text content is much faster so this is a very good option if you need a lot of changing text at once. Of course, because it is an image you cannot change its style without editing/changing the image.\
I still need to test this one but seems like another benefit `BitmapText` has is that unlike `Text` it does not need to wait for a font to load, you just need to pre-load its image and meta data with the `Assets` class.

#### `BitmapText` - creation & usage
To create a `BitmapText` you don't need to create a spritesheet, in this case pixi makes our lives easier. To style your `BitmapText` you can once again use the online tool [pixi-text-style](https://pixijs.io/pixi-text-style/#) just like we did with `Text` and get the JavaScript or JSON files to use in your project. then in your code just do:
```javascript
import textStyle from "./textStyle.json";

// create a bitmap font
PIXI.BitmapFont.from('myFont', textStyle);

// create the BitmapText object with your text and the font we just created
const  text = new  PIXI.BitmapText('My styled BitmapText', {fontName:  'myFont'});

// center the font on the screen
text.position.set(app.screen.width / 2 - text.width / 2, app.screen.height / 2);

app.stage.addChild(text)
```
If you want to create a custom font `BitmapText` you can do so with 3rd party tools such as: [Snowb](https://snowb.org/).\
To use Snowb go to their [online tool](https://snowb.org/) and use their default font or upload your own then design it however you want. Once done click the export button and it will give you the option to name your font and save it in a few different formats.\
Choose the "`fileName.txt (BMFont TEXT)`" format and make sure to **remember the name** you give the **font**. This name is what you need to pass to the `BitmapText` constructor. The Pixi docs say that you can also use a `.xml` file (instead of the `.txt` we're using) but I couldn't get it to work.\
Now that you chose the right format and you remember the font name, you can save the export (should download as a `.zip` file) .\
Inside you'll find a `.txt` with the meta data and a `.png` image spritesheet, add them both to your project files.\
The last thing we need before using it in the project is to convert the `.txt` to a `.js` and export its entire contents as a single `string`.
```javascript
rename yourFileName.txt -> yourFileName.js

// then inside yourFileName.js wrap the entire text with backticks 
// and export it
export const fontData = `
 file contents...
`
```
Finally, we can now use our custom font like this:
```javascript
import { fontData } from "./yourFileName.js";

// create a texture from the font spritesheet
const  fontTexture = PIXI.Texture.from('images/yourFileName.png')

// this adds the font to the available fonts list under font-name you gave when you exported it
PIXI.BitmapFont.install(fontData,fontTexture);

// create the actual BitmapText instance that will apear on screen
const  customFontText = new  PIXI.BitmapText('My custom BitmapText', {fontName:  'theFontNameThatYouRemember'});

// center the font on the screen
customFontText.position.set(app.screen.width / 2 - customFontText.width / 2, app.screen.height / 2 - customFontText.height / 2);

app.stage.addChild(customFontText)
```

#### `BitmapText` - change text
to change the text content of a `BitmapText` after it has been instantiated:
```javascript
bitmapTextInstance.text = 'new text';
```

#### `BitmapText` - notes
- I didn't test this but I think you could use custom fonts using the simpler builtin pixi way, the thing to note about it is that you'll need to ensure your font loads like in the case of using custom fonts with `Text`.



> Written partly with [StackEdit](https://stackedit.io/).
