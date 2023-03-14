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
```
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

You can see the full list of `DisplayObject`s [here](https://pixijs.download/release/docs/PIXI.DisplayObject.html).
*You can also see the properties and methods that all `DisplayObject`s share.

If you nest objects within other objects their display properties become relative to the ones of their parents so for example:
If you rotate the parent you rotate the child.
If you move the parent you move the child.
If you change the opacity of the parent you change the opacity of the child.
And so on...

### Put something on the stage
Now that we have a stage setup let's add a Graphics object to the stage, say a circle/s?
```
const circles = new PIXI.Graphics()
circles.beginFill(0x00ff00); // hex-color to fill the circle
circles.drawCircle(400,300,50); // x, y, radius
circles.drawCircle(300,300,20);
circles.endFill(); // stop color from leaking to other shapes.
circles.beginFill(0x0000ff);
circles.drawCircle(500,300,35);
app.stage.addChild(circles)
```
a single Graphics object can make more than one shape.
*a little note** - the `drawCircle` function we just used to create the circles doesn't actually draws them to the screen. Instead, think of it as a "buildCircle" function.
The Graphics object has a few more of these "draw" functions and you should treat them all as "build" functions instead.
So what actually draws them to the screen? that's what we talk about next.

### addChild(child)
As you just saw, to add anything to the stage we just use the `app.stage.addChild` function. Actually that function is not special to the stage, it's a container function, and since all `DisplayObjects` extend the container class anytime you want to nest one object in another just use its `addChild` function.

### Basic animation
To add basic animations to your display objects (changing their display properties over time) we use the 'Ticker' class.
When you initialize a ticker instance you pass it a callback function that runs on every 'tick'. in that function you change the display properties of whatever DisplayObject you want.
The callback gets as an argument the delta-time (The time since the last tick).
lets make our circles rotate:
```
let angle = 0
circles.position.set(400,300); //set our Graphics container position to that of our middle circle
circles.pivot.set(400,300); // set the origin of our container to be the same as our position

// add the ticker
app.ticker.add((delta) => {
	angle = angle < 360? angle + 1: 0; //just to keep the number from climbing (if it will climb too high it can cause a crash)
	circles.angle = angle; //update the angle of the container every tick.
})
```

### Preloading
#### why?
Let's say you have bunch of assets in your game (or whatever it is you're making). If you don't load them ahead of time your app will not function right, images will not appear on screen and then jump out of nowhere, sounds will not play and things will generally won't work properly and hurt the user experience.

#### how?

use the Pixi 'Assets' class and its functions: `Assets.init({manifest})`, `Assets.load()`,  `Assets.loadBundle()`, `Assets.backgroundLoad()`, `Assets.backgroundLoadBundle()`.
All of the above functions return a promise so you need to use either "async await" or the ".then" syntax.

- `Assets.init({manifest})` / `Assets.addBundle(bundleId,assets)`:
  bundles allow you to load a bunch of assets in one go, but to load bundles you first have to give them an ID. Using `Assets.init` you can define all your bundles when the app starts or if you want to add bundles later you can use `addBundle`.
  A `bundleId` can be any string you want.
  `assets` is an object `{name1: resource, name2: resource2, ... }`
  A `manifest` is an object or JSON file of the following structure:
  ```
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
```
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

    const animation = new AnimatedSprite([PIXI.Texture.from('sprite1.png'), PIXI.Texture.from('sprite2.png')]);
    app.stage.addChild(animation);
    animation.play();

> Written with [StackEdit](https://stackedit.io/).
