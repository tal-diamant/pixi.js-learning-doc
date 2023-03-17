# Pixi.js
This is my attempt at simplifying the explanation of what is pixi.js, and how to use it.

## what is it?
Pixi.js is basically a JavaScript library for rendering graphics with good performance.

### Why not just use canvas?
Pixi.js is supposed to be easier and more performant.

## How it works in practice
Pixi.js is basically a tree of objects. The pixi people are calling the tree a "scene graph" and the objects "containers".
At the top of the tree (the root node) you have the application container which they call a "Stage".

### Stage
to get a stage on screen just make sure you have access to the PIXI library through installation or the CDN `<script src="https://pixijs.download/releas/pixi.js"></script>`
and then type or copy the following:
```javascript
const app = PIXI.Application({width: 800, height:600, backgroundColor:0xaaaaaa});
document.body.appendChild(app.view);
```

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
		 //The loader function made Texture objects out of our .png files.
	 
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
container.filters = [blurFilter]; //this works for con

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
-   Torus - `drawTorus(this, x, y, innerRadius, outerRadius, startArc, endArc)`
-   Chamfer Rectangle - `drawChamferRect(this, x, y, width, height, chamfer)`
-   Fillet Rectangle - `drawFilletRect(this, x, y, width, height, fillet)`
-   Regular Polygon - `drawRegularPolygon(this, x, y, radius, sides, rotation)`
-   Rounded Polygon - `drawRoundedPolygon(this, x, y, radius, sides, corner, rotation)`
-   Star - `drawStar(this, x, y, points, radius, innerRadius, rotation)`\

As you may or may not noticed, the extra shapes all have a `this` as their first argument, that's because they are not built-in methods of the `Graphics` object.\
You use them like this:
```javascript
import { drawStar } from "@pixi/graphics-extras";
// create app and add to DOM ...

// create a Graphics instance
const graphics = new PIXI.Graphics();
graphics.beginFill(0xffff00);
// create the star in graphics
drawStar(graphics, 400, 300, 5, 50, 30);

// add graphics to the stage
app.stage.addChild(graphics);
```
Note* - using the packages that are **not** built-in to pixi may not be as simple as installing and importing. When I was trying to use it I couldn't get it to work like that at all so I resorted to some very dirty solutions that I don't recommend anyone to use.
Once I find a more straight forward solution I'll update this section but for now either avoid using them if they don't just work or try downloading the raw js files and importing from there but there might be cases you'll have to tinker with the package code yourself.

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
You could also use this method Inside a ticker callback to create an animation but the pixi docs warn about performance that could arise from that. Instead, for shapes that need to be changed every frame, they recommend using the `PIXI.Mesh` class which requires some WebGL knowledge.

#### Graphics performance
The pixi docs advise you to not put too many shapes in one `Graphics` instance as it can hinder performance. Instead, they suggest that you create more `Graphics` instances and spread your shapes between them.\
Also, you can create instances of the `Graphics` object by passing the geometry of another `Graphics` instance into their constructor, doing this will make both instances share the geometry of the first instance:
```javascript
// create the instance that will own the geometry
const geometryOwner = new PIXI.Graphics();
graphics.beginFill(0xffff00);
graphics.drawCircle(400, 300, 50);

// create the instance that will refrence the geometry
const geometryUser = new PIXI.Graphics(geometryOwner.geometry);

// the geometry object of both instances is the same one
geometryOwner.geometry === geometryUser.geometry // true
```
Just make sure to call `geometryUser.destroy()` when you're done with it, otherwise the `geometry` object will not be garbage collected even if the geometryOwner is destroyed, causing a memory leak.

### Particle effects
coming soon...


> Written with [StackEdit](https://stackedit.io/).
