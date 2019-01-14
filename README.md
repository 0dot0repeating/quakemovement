# Quake Movement

**Compatible with GZDoom 3.7.0+**

This mini-project is a modding resource that allows you to quickly and easily add
Quake physics to your mod. This is more of a resource than a playable mod, although
you can play it to see what it's like.

**Latest package:** http://jinotra.in/downloads/mods/doom/quakemovement/quakemovement-v1.2.pk3

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
Most of them are self-explanatory, but `CSlide` is only familiar to those familiar with Quake 4,
or Slash in Quake Champions. The contexts are:

- `Ground`: When on the ground.
- `Crouch`: When on the ground and crouching.
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
    When enabled, use Quake physics. When disabled, use Doom physics.


- `GroundSpeed`, `CrouchSpeed`, `AirSpeed`, `FlySpeed`, `WaterSpeed`, `CSlideSpeed`: **double**  
    Your maximum acceleration in the five contexts. These values are in u/s².


- `MaxGroundSpeed`, `MaxCrouchSpeed`, `MaxAirSpeed`, `MaxFlySpeed`, `MaxWaterSpeed`, `MaxCSlideSpeed`: **double**  
    Your maximum speed in the five contexts. These values are in u/s.
    Negative values represent no hard speed cap.


- `GroundFriction`, `CrouchFriction`, `AirFriction`, `FlyFriction`, `WaterFriction`, `CSlideFriction`: **double**  
    Controls how quickly your speed degenerates in the five contexts. These values are
    in half-times (in seconds): that is to say, the values represent how many seconds it
    takes for friction to reduce your velocity to half of its current value. Higher values
    result in less friction. A table to quickly convert Quake friction values to half-time
    values is at the bottom of this README.  

    Values equal to or less than 0 mean "no friction".


- `StopGroundSpeed`, `StopCrouchSpeed`, `StopAirSpeed`, `StopFlySpeed`, `StopWaterSpeed`, `StopCSlideSpeed`: **double**  
    The friction functions will act like your speed is at least this value when slowing you down.
    For example, a value of 200 means that you'll be treated as moving at 200 u/s if you're
    moving slower than that, and you will be slowed down accordingly. These values are in u/s.
    

- `ForwardScale`, `BackwardScale`, `SideScale`, `UpScale`: **double**  
    Scales how fast you can move in a given direction. Most of them are self-explanatory,
    but UpScale only applies to the thrust you gain from +moveup/+movedown/+jump/+crouch,
    not from flying up/down with +forward/+back. Acceleration and max speed are scaled
    by these properties.


- `Ground{Forward,Backward,Side,Up}Scale`, `Air{Forward,Backward,Side,Up}Scale`, `Water{Forward,Backward,Side,Up}Scale`, `Fly{Forward,Backward,Side,Up}Scale`, `CSlide{Forward,Backward,Side,Up}Scale`: **double**  
    Scales the above DirectionScale properties in the given contexts. They multiply
    together, so a ForwardScale of 1.2 and a GroundForwardScale of 1.25 gets you a
    net forward scale of 1.5 on the ground.

    
- `WadingSpeedScale`: **double**  
    Scales your speed when wading through water (waterlevel = 1).


- `QGravity`: **double**  
    When Quake physics are enabled, your `Gravity` value is set to this. This value
    acts the same as the standard `Gravity` property.


- `QJumpHeight`: **double**  
    When Quake physics are enabled, your `Player.JumpZ` value is set so that you jump
    as high as this value says. This value is in units.


- `MidairStepHeight`: **double**  
    When Quake physics are enabled and you're in midair (*not* when you're swimming),
    your `MaxStepHeight` is set to this value. This value is in units.


- `DoubleJumpFactor`: **double**  
    Quake and Quake 2 have a physics quirk where jumping adds to your Z velocity if it's
    positive, rather than simply setting it. This setting scales how much of your starting
    Z velocity is added to your jump velocity. For example, 0.5 means 50% of your Z velocity
    gets added to your jump velocity. If you want half the distance gained, use 0.7071
    (roughly √0.5).


- `RampJumpFactor`: **double**  
    Similarly to double jumping, jumping while moving up or down a ramp added your jump velocity
    to however fast you were going up/down the ramp, allowing for dramatically boosted jump heights
    when jumping up a ramp. This setting scales how much that factors in; for example, 0.5 means
    50% of your Z velocity from running up a ramp gets added to your jump velocity.


- `CrouchSlideTime`: **int**  
    Sets the minimum amount of time you can crouch slide when hitting the ground.
    This value is in tics.


- `CrouchSlideTimeMax`: **int**  
    Sets the maximum amount of time you can crouch slide. This value is in tics.


- `CrouchSlideTimeScale`: **double**  
    In Quake 4 and Quake Champions, the amount of time you fall has a positive and direct
    correlation to how long you can crouch slide. This value scales how much your fall time
    adds to your crouch slide time; for example, a value of 2 means for every tic you fall,
    you can crouch slide for two more tics. This rounds down when added to your crouch slide time.


- `MagneticLedgeScale`: **boolean**  
    Quake 1, and to my knowledge none of the other Quake games, has a nice feature where
    if it detects that you're about to run off a ledge, it doubles your friction in an
    attempt to prevent you from doing so accidentally. When set to a value greater than
    0, this recreates that behavior (higher values means higher ledge friction).
    
    Don't set this to -1; that'll cause a divide by zero error.


- `Autohop`: **boolean**  
    When off, you need to let go and re-press jump between jumps. When on, you don't.


- `AutoSlide`: **boolean**  
    When off, you need to let go and re-press crouch between slides. When on, you don't.


- `VQ1Bunnyhop`: **boolean**  
    In Quakeworld and every Quake afterwards, ground friction never applies when bunnyhopping.
    In vanilla Quake, ground friction applies for a single frame when you hit the ground,
    even if you jump immediately. This boolean enables vanilla Quake behavior.


- `EasyCrouchSlide`: **boolean**  
    In Quake 4 and Quake Champions, your crouch slide time ticks down when holding crouch,
    even in midair. When this is on, crouch slide only ticks down when actually sliding.


- `FlawedAirMove`: **boolean**  
    This replicates a flaw in the Quake games where holding jump or crouch in midair lowers
    your air acceleration, due to the engine zeroing out the Z component of your desired
    movement vector and reducing its length in the process. If you ever wondered why
    holding jump made your strafejumping suck... this is why.


- `InstantZAdjust`: **boolean**  
    Quake 1 and Quake 2 set your Z velocity to constant values if you hit jump or crouch
    when in the water. Quake 3 and beyond just make your wish direction point upwards.
    Turning this on uses Quake 1's behavior, and keeping it off uses Quake 3's.


- `Q2SurfaceTension`: **boolean**  
    In Quake 2, when swimming on the surface of a body of water and holding jump, your
    Z velocity gets clamped, keeping you closer to the water than in other Quake games.
    This simulates that.


By default, QuakeAccelPlayer is configured so that all of its movement styles act similarly
to the game they first appeared in - so air movement acts like Quake 1, crouch movement acts
like Quake 2, fly movement acts like Quake 3, slide movement acts like Quake 4, among other
examples. See `pk3/zscript/testclasses.txt` for examples of how to make the player act like
the other games in their entirety.


## Useful info I guess


### Doom friction/speed

Doom's default friction value is 0.90625, which at its fixed ticrate of 35 translates to
a halftime value of approxiamtely 0.2012.

Max speed (in u/tic) can be solved easily given acceleration (in u/tic²) and friction
(in percentage of velocity kept per tic) with the equation `v = af/(1-f)`.

 Movement mode  | Accel (u/tic²)  | Accel (u/s²)  | Max speed (u/tic) | Max speed (u/s)
:--------------:|:---------------:|:-------------:|:-----------------:|:---------------:
Forward run     | 1.5625          | 1914.0625     | 15.1041           | 528.646
Forward walk    | 0.78125         |  957.03125    |  7.5520           | 264.323
Sideways run    | 1.25            | 1531.25       | 12.0833           | 422.917
Sideways walk   | 0.75            |  918.75       |  7.25             | 253.75
Sideways mouse¹ | 1.5625          | 1914.0625     | 15.1041           | 528.646
SR40            | 2.00098         | 2451.156      | 19.3427           | 676.997
SR50            | 2.20971         | 2706.8931     | 21.3605           | 747.618

¹ Max strafe speed when using the mouse.


### Quake max speed

Calculating max speed with Quake physics is different than in Doom, because Quake
applies friction before acceleration rather than Doom's friction after acceleration.
Therefore, the equation for max speed given friction and acceleration (same units as
above) is `v = a/(1-f)`.

If you have a max speed you want to reach with a given acceleration, use `f = (v-a)/v`.

If you have a max speed you want to reach with a given friction, use `a = v(1-f)`.


### Quake friction

In general, you can convert Quake friction values to half-time values with this Python function:

```python
halftime = lambda friction, ticrate: math.log(0.5, 1-(friction/ticrate))/ticrate```

For quick reference, here are some pre-converted values. Each row corresponds to a Quake
friction value, and each column corresponds to a set framerate the engine's physics would be running at.

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