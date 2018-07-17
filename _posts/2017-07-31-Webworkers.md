---
layout: post
title: Webworkers
---

I've been playing around with webworkers recently, which are useful javascript files that you can use to run tasks asynchronously, while passing messages back and forth from your main thread. Here's an example of one of their uses. 

A while back I wrote a javascript maze generator. Basically, the maze is a bunch of squares, each with walls on all four sides. All the walls start as up. We visit the top left square, then pick a random direction to go to. If the picked square hasn't been visited, we mark that wall as removed, remove the opposite wall on the next square, then repeat the process. This uses a *ton* of recursion. Javascript is single threaded, so while all these calls are going on the UI is not showing us anything. Here's the [before example with a DOA UI.](https://plnkr.co/edit/vKiTBH?p=info) I've added a simple loading animation, but nothing is happening until the maze generation is complete, so we never see the animation.

Here's where web workers come in. I can move all of the heavy recursion to the webworker, and leave the UI for our main script. Once the generation is done, the webworker sends back an object containing the completed maze, which my main script draws on the canvas. This way, the user sees my lovely loading animation rather than thinking their browser has crashed. [Take a look!](https://plnkr.co/edit/eo71qo?p=info)

It only required a simple reorganization of the code. Create a webworker like so:
```
let worker = new Worker('worker.js');
```

Then send the worker messages:
```
worker.postMessage(something);
```

You need to pass in an argument, but since my maze doesn't require input I just passed in ```null```. In the webworker script, you listen for the message and then do whatever you need to do:
```
onmessage = function(e) {
    // heavy, UI-freezing logic goes here!
}
```

And once your worker is done, it passes the message back in the exact same way:
```
postMessage(maze);
```

Your main script listens to the worker in the same way again:
```
worker.onmessage = function(e) {
    drawMaze(e.data);
}
```

The message here is returned in a wrapper object with some other information, but all I'm looking for is the user defined data stored in ```e.data```. That's all there is to it. You can use workers to keep your UI active, do AJAX calls, behind the scenes number-crunching, whatever you need! Happy coding!