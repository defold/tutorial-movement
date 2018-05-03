# Movement tutorial

This beginner's tutorial will walk you through the steps of creating a simple object that moves in a natural manner when you give keyboard and mouse input. After having completed this tutorial you will have a good understanding of what vectors are, how to use vectors to represent positions, velocity and acceleration, and how to tie these things together into a game embryo that you can experiment with and develop further.

The tutorial assumes some basic understanding of physics concepts such as velocity and acceleration. You will also need some basic understanding of Lua programming.

## Making the spaceship move

Before digging into any details, let's first create a simple playable experiment. This project is prepared in advance for you so there is no setup to bother about. [Run the game](defold://build) to get an overview of what you have to work with:

- It include graphics: an animated spaceship and a background.
- Input is set up for arrow keys and mouse clicks.
- There is a "spaceship" game object that has a script attached to it.
- The script has code in place to react to player input. Initially, it only prints messages to the console as a reaction to input.

With the game running, try pressing the arrow buttons, or click in the game window, then check the editor console for input results. Notice that different text is printed in the console depending on which button you pressed:

<img src="doc/input.png" srcset="doc/input@2x.png 2x">

Now open ["spaceship.script"](defold://open?path=/main/spaceship.script) and scroll down to the beginning of the function `on_input()` where you find this code:

```lua
function on_input(self, action_id, action)
    if action_id == hash("up") then
        print("UP!")
    elseif...
```

Remove the print statement and replace it so the beginning of the function instead looks like this:

```lua
function on_input(self, action_id, action)
    if action_id == hash("up") then
        local p = go.get_position()
        p.y = p.y + 1
        go.set_position(p)
    elseif...
```

[Run the game](defold://build) again and press the <kbd>up</kbd> arrow key and see how the spaceship moves up at each press. The code is very simple, but let's look at it line by line to get a proper idea of what's going on here:

```lua
if action_id == hash("up") then
```

The input bindings that are set up in the project (in the file "/input/game.input_binding") binds the <kbd>up</kbd> arrow key to an action name "up". The game runs at 60 frames per second and each frame you push <kbd>up</kbd>, an "up" action name is sent to the `on_input()` function. Holding the button down will thus make the code that follows the `if` statement execute 60 times each second.

```lua
local p = go.get_position()
```

Since the function `go.get_position()` is called without any specific argument, it returns the position of the current game object. In this case that is the spaceship. The position is assigned to a local variable `p` so it is possible to manipulate it. The position object is a `vector3`, a vector that holds three values. If you have no idea what a vector is, don't worry, we will get there soon.

```lua
p.y = p.y + 1
```

The `vector3` object in `p` describes a point in 3D space, consisting of an X coordinate, an Y coordinate and a Z coordinate. Since pressing the <kbd>up</kbd> button should move the ship up along the Y axis, the `y` component of the position is increased by 1. Try changing the value that is added to the `y` component and run the game again.

```lua
go.set_position(p)
```

Finally, the new changed position value is written back to the current game object.

## Vectors

A vector is a mathematical entity that has both a _direction_ and a _magnitude_, or length. It describes an entity in a vector space, or a specific point. In practice, a vector is nothing more than a set of numbers that taken together refer to a point in a space. For a two dimensional space, you need two numbers to describe any arbitrary point so a 2 vector consists of two numbers: one for the X axis and one for the Y axis:

<img src="doc/2d-vector.png" srcset="doc/2d-vector@2x.png 2x">

For a three dimensional space, you need three numbers to describe any point so a 3 vector consists of three numbers: one for the X axis, one for the Y axis and one for the Z axis:

<img src="doc/3d-vector.png" srcset="doc/3d-vector@2x.png 2x">

The magnitude of a vector *v* is calculated by using the Pythagorean theorum:

<img src="doc/magnitude.png" srcset="doc/magnitude@2x.png 2x">

A vector with magnitude (or length) 1 is called a *normalized vector*.

Vectors for higher number of dimensions are certainly possible. Defold uses four value large vectors to encode colors (the first three values give the amount of red, green, and blue in the color. The last value describes the amount of translucency, also called "alpha".)

We are all used to scalar values, real numbers that describes points on the number line. Real numbers are used to mean many different things. The number 12 could mean a number of meters, kilograms, seconds, meters per second, volts, dollars or horses. The same is true for vectors. Since a vector describes a point in space it can be used to mean a position, motion through space (along the direction of the vector) or acceleration among many other things.

If you want to express movement on the screen you need two values to encode the motion (since the computer screen is 2d plane): The speed along the X axis and the speed along the Y axis. You can very well use two scalar values for this and add the speed values to the X and Y positions separately:

```lua
-- update position by adding the speed (meters/s) multiplied with the number of
-- seconds that has elapsed.
position_x = position_x + speed_x * elapsed_seconds
position_y = position_y + speed_y * elapsed_seconds
```

This is roughly what you did when you made the spaceship move upwards earlier. There is absolutely nothing mathematically wrong with this way of calculating movement, but vectors allow you to express the motion in a clearer and more concise way. Since vectors have both a direction and a *magnitude* (length) using them to describe motion is intuitive: the direction of the vector equals the motion direction and the magnitude describes the speed of motion.

```lua
-- update position by adding the speed (meters/s) multiplied with the number of
-- seconds that has elapsed.
position = position + speed * elapsed_seconds
```

As long as the `position` and `speed` values are expressed as vectors in the same space there is a well defined algebra that allows you to add or subtract vectors, multiply vectors with scalar values, and more. We are soon going to look into that algebra.

Even though Defold has a toolset that is tailored for 2D games, the engine is a true 3D engine. All positions for sprites, particle emitters, tile maps and spine objects are expressed as 3 element vectors of the type `vector3`. The X and Y value determine the position of the object along the "width" and "height" axis, and the Z position determines the "depth" position, which in effect allows you to order overlapping objects. For instance, a sprite with a Z value of 1 will appear in front of a sprite at Z position 0. Defold by default uses a coordinate system where Z is between -1 and 1 and a higher Z is in front of lower Z:

<img src="doc/z-order.png" srcset="doc/z-order@2x.png 2x">

The Defold Lua library [`vmath`](https://defold.com/ref/vmath) contains functions to create and manipulate [`vmath.vector3`](https://defold.com/ref/vmath/#vmath.vector3) objects:

```lua
-- create a new vector3 at X position 100 and Y position 350.
local position = vmath.vector3(100, 350, 0)

-- set the position of game object "player" to the new vector.
go.set_position("player", position)
```

## Vector algebra

Vector algebra defines mathematical operations on vectors. Beginning with the simplest, negation, addition and subtraction.

Negation
: Negating a vector *v*, denoted by -*v*, negates each component of the vector. This makes a vector that points in the opposite direction with the same magnitude as the original vector:

  <img src="doc/vector-negate.png" srcset="doc/vector-neg@2x.png 2x">

Addition
: Adding vector *u* to vector *v* (*u* + *v*) adds each component of *u* to *v*. The result is a new vector:

  <img src="doc/vector-add-cs.png" srcset="doc/vector-add-cs@2x.png 2x">

  Vectors are often drawn displaced from the coordinate system. Vector operations becomes intuitive and easy to understand this way:

  <img src="doc/vector-add.png" srcset="doc/vector-add@2x.png 2x">

Subtraction
: Subtracting vector *v* from vector *u* (*u* - *v*) is equal to adding the negation of *v* to *u*. So *u* - *v* = *u* + (-*v*):

  <img src="doc/vector-sub.png" srcset="doc/vector-sub@2x.png 2x">

Multiplication with scalar
: Multiplying a vector *v* with a real number *r* produces a new vector of the same orientation, but with the magnitude scaled: the vector is streched out by a factor *r*. Multiplying with a negative *r* flips the vector 180 degrees:
  
  <img src="doc/vector-mul.png" srcset="doc/vector-mul@2x.png 2x">

These are the basic operations on vectors that you need to model motion. In addition, there are two special operations that come in handy if you, for instance, want to check if two vectors are parallell or at right angles of each other:

Dot product
: The dot product of two vectors *u* and *v*, denoted by *u ∙ v*, is a scalar value. It is defined as:

  <img src="doc/dot.png" srcset="doc/dot@2x.png 2x">

  - *‖u‖* is the magnitude of vector *u*.
  - *‖v‖* is the magnitude of vector *v*.
  - *θ* is the angle between the vectors.

  <img src="doc/vector-dot.png" srcset="doc/vector-dot@2x.png 2x">

  If the vectors are orthogonal (the angle between them is 90 degrees), then the dot product is zero.

Cross product
: The cross product of two vectors *u* and *v*, denoted by *u* × *v*, is a vector that is perinpendicular to both *u* and *v*:

  <img src="doc/vector-cross.png" srcset="doc/vector-cross@2x.png 2x">

  The resulting vector is a zero vector if:

  - Either one or both of the input vectors is the zero vector, (*u* = 0 or *v* = 0)
  - The two input vectors are parallel (θ = 0°)
  - The two input vectors are antiparallel (θ = 180°)

## Creating movement with vectors

With some vector algebra under your belt, rewriting the spaceship's script should be pretty easy. Open ["spaceship.script"](defold://open?path=/main/spaceship.script) and modify the `init()`, `update()` and `on_input()` functions:

```lua
function init(self)
    msg.post(".", "acquire_input_focus")
    self.input = vmath.vector3()                -- [1]
end
```
1. Create a new `vector3` (with all components set to 0) for storing the input direction. It is placed it in the current script instance (`self`) so you can use it through the lifetime of the spaceship game object.

```lua
function update(self, dt)
    local movement = self.input * 3             -- [1]
    local p = go.get_position()                 -- [2]
    go.set_position(p + movement)               -- [3]
    self.input = vmath.vector3()                -- [4]
end
```
1. Calculate the movement vector based on the player's input vector.
2. Retrieve the position of the current game object (the spaceship). The position is a `vector3`.
2. Set the position of the current game object to `p` plus the movement vector.
3. Reset the input vector. It will be set in the `on_input()` function that is called each frame before `update()`.

```lua
function on_input(self, action_id, action)
    if action_id == hash("up") then
        self.input.y = 1                     -- [1]
    elseif action_id == hash("down") then
        self.input.y = -1                    -- [1]
    elseif action_id == hash("left") then
        self.input.x = -1                    -- [1]
    elseif action_id == hash("right") then
        self.input.x = 1                     -- [1]
    elseif action_id == hash("click") and action.pressed then
        print("CLICK!")
    end
end
```
1. Set the x or y component of the input vector depending on player input. If the player presses `up` and `left` at the same time, both components are set and the input direction is diagonal.

There are two issues with this code:

1. The input vector has length 1 if you move vertically or horizontally, but diagonally the length is 1.4142 (square root of 2) so diagonal movement is faster. You probably don't want that.
2. The scale of the velocity is a bit arbitrary. Currently, it's 3 pixels of movement each frame (or 4.2 diagonally). To make the ship go faster, change the 3 to a higher value. If you want it to go slower, decrease the value. The values are a bit hard to reason about though, and there are other time based problems that may appear.

The first problem is easy to fix, just normalize the input vector so the input length is always 1:

lua
```
function update(self, dt)
    if vmath.length_sqr(self.input) > 1 then        -- [1]
        self.input = vmath.normalize(self.input)
    end
    local movement = self.input * 3
    local p = go.get_position()
    go.set_position(p + movement)
    self.input = vmath.vector3()
end
```
1. If the squared length of the input vector is larger than 1, normalize the vector so it is of magnitude 1. Compare against square length since it's faster than comparing against length.

The second problem requires the use of a timestep value.

## Time step

Each frame the Defold engine calls the `update()` function of each script. A Defold game usually runs at 60 frames per second, so each frame is 0.016666 seconds long. That is the time elapsed between each call to `update()`. A velocity vector with a magnitude of 3 will then represent a speed of 3 * 60 = 180 pixels per second (with the regular render script). However, it would be better if you work with pixels per second: you would then be able to time your game with a stopwatch, reason about distances and timing without having to scale all values against the framerate. And a bigger problem is what happens if there, for whatever reason, is a hitch in the framerate. With the current code movement will be uneven and unpredictable.

Defold provides a time step value to each call to `update()` allowing you to easily fix both of these problems. The time step value is usually called `dt` (for "delta time") and is set to the number of seconds that elapsed since the last frame. You should scale your values against `dt` to get proper units:

```lua
function update(self, dt)
    if vmath.length_sqr(self.input) > 1 then
        self.input = vmath.normalize(self.input)
    end
    local movement = self.input * 150 * dt              -- [1]  
    local p = go.get_position()
    go.set_position(p + movement)
    self.input = vmath.vector3()
end
```
1. Since input is normalized and the velocity (150) is scaled with `dt` we know that the velocity is 150 pixels per second. The screen is 1280 pixels wide so it should take the ship 8.53 seconds to fly across. You can check that with a stopwatch.

The current movement works but is very stiff. To give a sense of weight to the spaceship you can use acceleration based movement.

## Acceleration

In the code you wrote above, the velocity was constant so the resulting movement of the velocity acting over the time-step (`dt`) was simply the velocity multiplied with the timestep, denoted by the orange area in this diagram:

<img src="doc/integration-constant.png" srcset="doc/integration-constant@2x.png 2x">

Acceleration defines how fast something can reach its max speed and change direction. The acceleration is acting over the frame time-step (`dt`) and then added to the velocity. The velocity acts over the frame and the resulting translation (movement) is added to the position. We see that the movement now has to be calculated as the area under a curve. In maths, this is called [integration over time](http://en.wikipedia.org/wiki/Integral).

<img src="doc/integration.png" srcset="doc/integration@2x.png 2x">

With a small enough time-step a good enough geometric approximation of the area can be calculated by assuming that the acceleration acting between *v0* and *v1* is constant. By that assumption *v1* can be calculated as *v0* + *acceleration* * *dt* and the resulting movement becomes:

<img src="doc/movement.png" srcset="doc/movement@2x.png 2x">

You can now write the final code for `init()` and `update()`:

```lua
function init(self)
    msg.post(".", "acquire_input_focus")
    self.velocity = vmath.vector3()             -- [1]
    self.input = vmath.vector3()
end

function update(self, dt)
    if vmath.length_sqr(self.input) > 1 then
        self.input = vmath.normalize(self.input)
    end
    
    local acceleration = self.input * 200       -- [2]
    
    local dv = acceleration * dt                -- [3]
    local v0 = self.velocity                    -- [4]
    local v1 = self.velocity + dv               -- [5]
    local movement = (v0 + v1) * dt * 0.5       -- [6]

    local p = go.get_position()
    go.set_position(p + movement)

    self.velocity = v1                          -- [7]
    self.input = vmath.vector3()
end
```
1. Create a vector for storing velocity over time.
2. Acceleration set to 200 pixels per second per second in input direction.
3. Calculate velocity change for this time-step.
4. v0 is velocity last time-step.
5. v1 is v0 plus change this time-step.
6. Calculate movement this time-step.
7. Store velocity for next time-step.

[Run the game](defold://build) again and try controlling the spaceship with the arrow keys. Notice how the ship feels heavy.

Congratulations! You are now finished with this tutorial. But we encourage you to continue experimenting with the code.

Here are some ideas what you can try:

1. Put a cap on the velocity.
2. Make the spaceship bounce off the edges of the screen.
3. Allow mouse clicks to dictate the input direction.

Check out [the documentation pages](https://defold.com/learn) for more examples, tutorials, manuals and API docs.

If you run into trouble, help is available in [our forum](https://forum.defold.com).

Happy Defolding!

----

This project is released under the Creative Commons CC0 1.0 Universal license.

You’re free to use these assets in any project, personal or commercial. There’s no need to ask permission before using these. Giving attribution is not required, but is greatly appreciated!
[Full license text](https://creativecommons.org/publicdomain/zero/1.0)
