# Quake Movement

**Compatible with GZDoom 3.7.0+**

This mini-project is a modding resource that allows you to quickly and easily add
Quake physics to your mod. This is more of a resource than a playable mod, although
you can play it to see what it's like.

**Latest package:** http://jinotra.in/downloads/mods/doom/quakemovement/quakemovement-v1.pk3

**Packaged downloads:** http://jinotra.in/downloads/mods/doom/quakemovement


## Building your own PK3

Above the latest-commit bar, on the very right, there's a download icon.

Click that, choose ".ZIP", then unzip the ZIP file into its own directory.

Go into the pk3/ directory, select everything in there, and zip it all up.
Change the extension of the new ZIP file to ".pk3", and you should be good to go.

If, in the ZIP you created, everything's still in the "pk3/" folder, you zipped
the folder, not its contents. Don't do that.


## Documentation

Most of this is covered in quakeaccel.txt as well.

`QuakeAccelPlayer` overrides the following methods:

- `Thinker.Tick`: extended to enable/disable Quake physics when Q_UseQuakeAccel changes.
  
- `PlayerPawn.CheckCrouch`: overridden to speed up crouching and uncrouching. Not strictly
  necessary, but every Quake game has fast crouching, so I'd recommend keeping this.

- `PlayerPawn.DeathThink`: extended to apply Quake style friction when dead.

- `PlayerPawn.HandleMovement`: overridden with Quake physics and movement. Do not override
  this with your own movement code. (what's the point of using this if you do that, anyway?)


There are five contexts where acceleration, max speed, friction, and stopping speed can differ.
Most of them are self-explanatory, but `CSlide` is only familiar to those familiar with Quake 4.
or Slash in Quake Champions. The contexts are:

- `Ground`: When on the ground.
- `Air`: When freefalling.
- `Fly`: When flying (Wings of Wrath, `fly`, `noclip2`, etc).
- `Water`: When swimming.
- `CSlide`: When on the ground and crouch sliding. Crouch sliding was introduced in Quake 4,
  and occurs when you hit the ground from a freefall while crouching. In both Quake 4 and
  Quake Champions, crouch sliding has drastically reduced friction, allowing you to keep and
  even gain speed while rounding corners on the ground. You aren't bound to that, but it *is*
  called a slide, after all.


`QuakeAccelPlayer` provides the following properties:

- `UseQuakeAccel`: **boolean**
  - When enabled, use Quake physics. When disabled, use Doom physics.

- `GroundSpeed`, `AirSpeed`, `FlySpeed`, `WaterSpeed`, `CSlideSpeed`: **double**
  - Your maximum acceleration in the five contexts. These values are in units/s^2.

- `MaxGroundSpeed`, `MaxAirSpeed`, `MaxFlySpeed`, `MaxWaterSpeed`, `MaxCSlideSpeed`: **double**
  - Your maximum speed in the five contexts. These values are in units/s.
    Negative values represent no hard speed cap.

- `GroundFriction`, `AirFriction`, `FlyFriction`, `WaterFriction`, `CSlideFriction`: **double**
  - Controls how quickly your speed degenerates in the five contexts. These values are
    in half-times (in seconds): that is to say, the values represent how many seconds it
    takes for friction to reduce your velocity to half of its current value. Higher values
    result in less friction. A table to quickly convert Quake friction values to half-time
    values is at the bottom of this README.  
    Values equal to or less than 0 mean "no friction".

- `StopGroundSpeed`, `StopAirSpeed`, `StopFlySpeed`, `StopWaterSpeed`, `StopCSlideSpeed`: **double**
  - The friction functions will act like your speed is at least this value when slowing you down.
    For example, a value of 200 means that you'll be treated as moving at 200 units/s if you're
    moving slower than that, and you will be slowed down accordingly. These values are in units/s.

- `QGravity`: **double**
  - When Quake physics are enabled, your `Gravity` value is set to this. This value
    acts the same as the standard `Gravity` property.

- `QJumpHeight`: **double**
  - When Quake physics are enabled, your `Player.JumpZ` value is set so that you jump
    as high as this value says. This value is in units.

- `MidairStepHeight`: **double**
  - When Quake physics are enabled and you're in midair (*not* when you're swimming),
    your `MaxStepHeight` is set to this value. This value is in units.

- `DoubleJumpFactor`: **double**
  - Quake and Quake 2 have a physics quirk where jumping adds to your Z velocity if it's
    positive, rather than simply setting it. This setting scales how much of your starting
    Z velocity is added to your jump velocity. For example, 0.5 means 50% of your Z velocity
    gets added to your jump velocity.

- `RampJumpFactor`: **double**
  - Similarly to double jumping, jumping while moving up or down a ramp added your jump velocity
    to however fast you were going up/down the ramp, allowing for dramatically boosted jump heights
    when jumping up a ramp. This setting scales how much that factors in; for example, 0.5 means
    50% of your Z velocity from running up a ramp gets added to your jump velocity.

- `CrouchSpeedMult`: **double**
  - Scales your speed and max speed values when crouching in all contexts but the `CSlide` context.

- `CrouchSlideTime`: **int**
  - Sets the minimum amount of time you can crouch slide when hitting the ground.
    This value is in tics.

- `CrouchSlideTimeMax`: **int**
  - Sets the maximum amount of time you can crouch slide. This value is in tics.

- `CrouchSlideTimeScale`: **double**
  - In Quake 4 and Quake Champions, the amount of time you fall has a positive and direct
    correlation to how long you can crouch slide. This value scales how much your fall time
    adds to your crouch slide time; for example, a value of 2 means for every tic you fall,
    you can crouch slide for two more tics. This rounds down when added to your crouch slide time.

- `Autohop`: **boolean**
  - When off, you need to let go and re-press jump between jumps. When on, you don't.

- `AutoSlide`: **boolean**
  - When off, you need to let go and re-press crouch between slides. When on, you don't.

- `VQ1Bunnyhop`: **boolean**
  - In Quakeworld and every Quake afterwards, ground friction never applies when bunnyhopping.
    In vanilla Quake, ground friction applies for a single frame when you hit the ground,
    even if you jump immediately. This boolean enables vanilla Quake behavior.

- `EasyCrouchSlide`: **boolean**
  - In Quake 4 and Quake Champions, your crouch slide time ticks down when holding crouch,
    even in midair. When this is on, crouch slide only ticks down when actually sliding.

- `FlawedAirMove`: **boolean**
  - This replicates a flaw in the Quake games where holding jump or crouch in midair lowers
    your air acceleration, due to the engine zeroing out the Z component of your desired
    movement vector and reducing its length in the process. You usually only notice this
    when using the autohop feature in Quake Live and Quake Champions, since otherwise, the
    engine acts like you let go of the jump button the moment you jump.

    
By default, QuakeAccelPlayer is configured to act similarly to Quakeworld's physics.
See `pk3/zscript/testclasses.txt` for examples of how to configure it to act similarly
to the other Quake games.


## Quake friction values to half-time values

In general, you can convert Quake friction values to half-time values with this Python function:

```python
halftime = lambda friction, ticrate: math.log(0.5, 1-(friction/ticrate))/ticrate```

For quick reference, here are some pre-converted values. Each row corresponds to a Quake
friction value, and each column corresponds to a set framerate the engine's physics could be running at.

 QFriction |  60 FPS  |  72 FPS  |  77 FPS  |  85 FPS  |  125 FPS
:---------:|:--------:|:--------:|:--------:|:--------:|:--------:
   **1**   | 0.687355 | 0.688322 | 0.688636 | 0.689062 | 0.690371
   **2**   | 0.340765 | 0.341737 | 0.342053 | 0.342480 | 0.343794
   **3**   | 0.225223 | 0.226201 | 0.226518 | 0.226947 | 0.228265
   **4**   | 0.167444 | 0.168427 | 0.168746 | 0.169177 | 0.170499
   **5**   | 0.132769 | 0.133758 | 0.134078 | 0.134511 | 0.135838
   **6**   | 0.109647 | 0.110641 | 0.110963 | 0.111397 | 0.112729

For reference:
- Quake 1 and Quakeworld typically run at 77 fps.
- Quake 2 typically runs at 85 fps.
- Quake 3 and Quake Live typically run at 125 fps.
- Quake 4 is usually locked to 60 fps, but Q4Max duels run at 90 fps.
- Quake Champions' tick rate isn't certain, but it's around 77 fps.