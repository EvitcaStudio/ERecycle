# Collector
This plugin gives you an easy and beneficial way to reuse instances inside of the [Vylocity Game Engine](https://vylocity.com), as well as slow down the garbage collector.

## Installation

### ES Module

```js
import { Collector } from './collector.mjs';
```

### IIFE (Immediately Invoked Function Expression)

```js
<script src="collector.js"></script>;
window.CollectorBundle.Collector;
```

### CommonJS (CJS) Module

```js
const { Collector } = require('./collector.cjs.js');
```

## API  
###  Collector.setMaxLimit(pLimit)
   - `pLimit`: The new max limit on collections. 
   - `desc`: Sets a limit on how much each collection array can recycle before deleting the access. The max is `20` by default. 
> [!CAUTION]  
> (ONLY SET THIS TO A HIGH VALUE IF YOU KNOW WHAT YOU ARE DOING. HIGH VALUES LEAD TO INSTANCES JUST SITTING IN ARRAYS AND TAKING UP MEMORY)  

###  Collector.collect(pCollected, pCollection)
  - `pCollected`: The instance|array to collect into the collection array pCollection.
  - `pCollection`: The collection array this instance is going to. If the collection's length is `>=` then the set max limit then pCollected will be deleted.
  - `desc`: Recycle an instance|array for reuse into pCollection. If an array is passed, its contents will be added to the collection array.

###  Collector.grab(pType, pNum, pCollection, [pArg[, pArg[, ... pArg]]])
  - `pType`: The type of instance you want to get
  - `pNum`: The amount of pType(s) you want to get out of this collection
  - `pCollection`: The collection array to check inside of
  - `pArg`: Any argument(s) to pass to the `instance.onNew` or the `instance.onDumped` event function.
  - `desc`: This is the plugin equivalent of `new Diob(pType)`. 
  - `returns`: This returns either the instance you ask for (if it was only 1) Or it returns an array full of instances that you asked for.

###  Diob|Object.clean() `event`
   - `desc`: This is a user defined event function that will be called when this instance is collected. This should reset this instance to it's default state. This plugin does some basic cleaning of this instance when it is collected, but cannot do things it doesn't know about. Internal cleaning involves:
      - `Removing color&text`
      - `Removing overlays&filters`
      - `Removing it from an interface if it is a interface element.`
      - `Removing it from the ticker system`
      - `Resetting animations&transitions`
      - `Resetting angle&scale&composite&positionalData`
      - `Cancels movement`

###  Diob|Object.onDumped(...pParam) `event`
   - `pParam`: The argument(s) that were passed inside of `Collector.grab`.
   - `desc`: Event function that is called when this object has been removed from a collection and is ready for use. This is the plugin's equivalent of `Object.onNew`

###  Diob|Object.onCollected() `event`
   - `desc`: Event function that is called when this object is added to a collection. This is the plugin's equivalent of `Object.onDel`

##  Examples  
####  Get an instance 
```js
const toyBin = []
Diob
   Toy
World
   onNew()
      const toyForParty = JS.CollectorBundle.Collector.grab('Diob/Toy', 1, toyBin) // DiobToyInstance 
```

#### Get an array of instances

```js
const toyBin = []
Diob
   Toy
World
   onNew()
      const toysForParty = JS.CollectorBundle.Collector.grab('Diob/Toy', 3, toyBin) // [DiobToyInstance, DiobToyInstance, DiobToyInstance]
```
#### Recycle an instance  

```js
const toyBin = []
Diob
   Toy
World
   onNew()
      const toyForParty = JS.CollectorBundle.Collector.grab('Diob/Toy', 1, toyBin) // DiobToyInstance
      // a few moments later
      JS.CollectorBundle.Collector.collect(toyForParty, toyBin)
      // We have gotten a toy from the toy bin, did something with it for a while, and returned it to the toy bin. Recycling rocks!
```

#### How to use `instance.onDumped`  
Any time an instance is removed from a collection array it will call `instance.onDumped()` if it is a defined function.  
```js
const toyBin = []
Diob
   Toy
      onNew()
         World.log('toyBin was empty so i was created!')
      function onDumped()
         World.log('I was removed from toyBin so you can use me again!')
World
   onNew()
      // At this point in time toyBin has no toys in it so Collector creates the instance for you
      const toyForParty = JS.CollectorBundle.Collector.grab('Diob/Toy', 1, toyBin) // DiobToyInstance
      // We have just put the toy into the toybin
      JS.CollectorBundle.Collector.collect(toyForParty, toyBin)
      // A few moments later we decide we want to get that toy back out
      // Instead of creating a new instance of the toy, aRecyle cleaned and removed the toy you previously put into toyBin
      const toy = JS.CollectorBundle.Collector.grab('Diob/Toy', 1, toyBin) // DiobToyInstance
```

#### How to use `instance.onDumped` with parameters  
If your collection array starts off empty you must handle the condition of Collector calling `new Diob` and in return the event function `instance.onNew` being called!  
```js
const toyBin = []
Diob
   Toy
      onNew(pName)
         this.name = pName // 'toyBall'
      function onDumped(pName)
         this.name = pName // 'toyBall'
World
   onNew()
      const toyBall = JS.CollectorBundle.Collector.grab('Diob/Toy', 1, toyBin, 'toyBall') // DiobToyInstance  
```

#### Using `instance.onDumped` like a pro  
Since `instance.onNew` and `instance.onDumped` are virtually the same, they can run the EXACT same code under each of them. So it is best to create a handler function so you don't have to duplicate code. In this case `setup` is the handler function.  
```js
const toyBin = []
Diob
   Toy
      onNew(pName)
         this.setup(pName)
      function setup(pName)
         this.name = pName // 'toyBall'
      function onDumped(pName)
         this.setup(pName)
World
   onNew()
      const toyBall = JS.CollectorBundle.Collector.grab('Diob/Toy', 1, toyBin, 'toyBall') // DiobToyInstance  
```

#### Using `instance.onCollected`  
Any time a diob is collected, Collector will call `instance.onCollected` if it is a defined function.  
```js
const toyBin = []
Diob
   Toy
      function onCollected()
         World.log('I have been collected')
World
   onNew()
      const toyBall = JS.CollectorBundle.Collector.grab('Diob/Toy', 1, toyBall) // DiobToyInstance
      JS.CollectorBundle.Collector.collect(toyBall, toyBin)
```

      
### Global Dependency

Collector relies on the `VYLO` variable being globally accessible.