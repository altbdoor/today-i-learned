Tile-based movement in game with Kontra
===

Apr 18, 2023

So there's a [game jam in April](https://itch.io/jam/gamedevjs-2023)
(in which I'll probably not make it, changing the engine in the middle of the implementation...),
where I fiddled around with [Kontra](https://straker.github.io/kontra/). I do not exactly have
a rough game idea at hand, but I wanted to try out game development a little.

_(P.S., I know it is rough)_

I quickly familiarized myself with the concept of Sprites, Tiles, and the other factories in Kontra.
But I wanted to emulate the movement that was in Pokemon red. Sprites in Kontra can be repositioned
via:

1. updating the `x`/`y` coordinates, or
1. specifying the speed with `dx`/`dy`, and call `update()`

This works well for most games, but with tile-based movement, the player needs to be "snapped" to
the next coordinates. Assuming a grid of 32 pixels square, we would have:

| \# | Action | x coord | y coord |
| --- | --- | --: | --: |
| 1 | Player renders        | 128 | 32 |
| 2 | Player presses right  | 160 | 32 |
| 3 | Player presses left   | 128 | 32 |
| 4 | Player presses down   | 128 | 64 |

Updating the `x`/`y` coordinates can become a trouble. If the game accepts movement inputs
at every interval of 60 FPS, then quick presses of <kbd>Up</kbd> and <kbd>Right</kbd>
might throw off the final coordinate. The sprite will then probably move diagonally,
which in my case, is not favorable.

So the key idea is then to _block_ the movement inputs, until the sprite reaches the final
coordinate. Assuming that the sprite moves to the right, then the `x` coordinate needs to be
updated incrementally. There are many ways around it... especially when the easing comes into
consideration.

Fortunately, Kontra provides a `dx`/`dy` variable, which automatically updates the coordinate,
whenever the sprite gets `.update()`-ed. It will be a linear animation, but that is good enough
for me.

Once the sprite reaches the final ~~destination~~ coordinates, reset the `dx`/`dy` variable to
zero, which will halt all movement on subsequent `.update()`s.

Here's a basic example:

```ts
// i used a `st` prefix to denote "state" variables in the sprite

if (player.stIsMoving && player.x === player.stTargetX && player.y === player.stTargetY) {
    // allow movement
    player.stIsMoving = false;
    
    // stop all speed
    player.dy = 0;
    player.dx = 0;
}

if (!player.stIsMoving && keyPressed(['arrowup', 'arrowdown', 'arrowleft', 'arrowright'])) {
    // block movement
    player.stIsMoving = true;

    // copy over the current position
    player.stTargetX = player.x;
    player.stTargetY = player.y;

    // update speed and target coord
    if (keyPressed('arrowup')) {
        player.dy = -2;
        player.stTargetY -= 32;
    } else if (keyPressed('arrowdown')) {
        player.dy = 2;
        player.stTargetY += 32;
    } else if (keyPressed('arrowleft')) {
        player.dx = -2;
        player.stTargetX -= 32;
    } else if (keyPressed('arrowright')) {
        player.dx = 2;
        player.stTargetX += 32;
    }
}
```

Depending on how fast the sprite needs to move, the `dx`/`dy` variable can be updated with higher
values. Ditto for the target coordinates for distance.

This was pretty fun, that I cooked up a small playable demo, and added some basic animations in
as well. Check it out:

- Source code in https://github.com/altbdoor/kontra-move-tile
- Playable demo in https://altbdoor.github.io/kontra-move-tile/

As an addendum, I did seek ChatGPT for help. However the cutoff date only had an old version of
Kontra. I had a hard time translating the logic across versions, but the suggestions from
ChatGPT did serve as a good starting point :star:

One particular thing that ChatGPT got right was to update the `dx`/`dy` variables, and let
the game engine handle the movement.

---

#### References

- https://itch.io/jam/gamedevjs-2023
- https://straker.github.io/kontra/
- https://github.com/altbdoor/kontra-move-tile
