# First Assignment Of Research Track 

The aim of this project was to code a Python script capable of making an holonomic robot :

    • constantly driving the robot around the environment in the counter-clockwise direction,
    • Avoiding the gold tokens and do not collide with the golden token walls
    • And once the robot will get close enough to a silver token, it should grab it and move it behind itself.
## Running the Program
To run one or more scripts in the simulator use run.py, passing it the file names.<br />
`python run.py assignment.py`
# Robot API
The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api].
### Motors
The simulated robot has two motors configured for skid steering, connected to a two-output Motor Board. The left motor is connected to output `0` and the right motor to output `1`.
The Motor Board API is identical to that of the SR API, except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following: <br />
```ruby
R.motors[0].m0.power = 25 
R.motors[0].m1.power = -25
```
### The Grabber
The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the R.grab method: <br />
```ruby
success = R.grab()
```
The `R.grab` function returns `True` if a token was successfully picked up, or `False` otherwise. If the robot is already holding a token, it will throw an `AlreadyHoldingSomethingException`.<br />

To drop the token, call the `R.release` method.
Cable-tie flails are not implemented.
### Vision
To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The `R.see` method returns a list of all the markers the robot can see, as `Marker` objects. The robot can only see markers which it is facing towards.<br />
Each `Marker` object has the following attributes:<br />
    • info: a `MarkerInfo` object describing the marker itself. Has the following attributes:
            ◦ code: the numeric code of the marker.<br />
        ◦ marker_type: the type of object the marker is attached to (either `MARKER_TOKEN_GOLD`, `MARKER_TOKEN_SILVER`or `MARKER_ARENA`).<br />
        ◦ offset: offset of the numeric code of the marker from the lowest numbered marker of its type. For example, token number 3 has the code 43, but offset 3.<br />
        ◦ size: the size that the marker would be in the real game, for compatibility with the SR API.<br />
    • centre: the location of the marker in polar coordinates, as a `PolarCoord` object. Has the following attributes:<br />
        ◦ length: the distance from the centre of the robot to the object (in metres).<br />
        ◦ rot_y: rotation about the Y axis in degrees. <br />
    • dist: an alias for `centre.length` <br />
    • res: the value of the `res` parameter of `R.see`, for compatibility with the SR API. <br />
    • rot_y: an alias for `centre.rot_y` <br />
    • timestamp: the time at which the marker was seen (when `R.see` was called). <br />
    
 ## Functions 
 Functions to drive the robot in a straight path without getting stuck in golden token are as follows:
 ### drive()
 
The `drive(speed , seconds)`sets a linear velocity to the robot resulting in a straight shifting. To do so, it makes the robot's motors run at the same speed for a certain amount of time.<br />

Arguments:<br />
    • speed: represents the speed at which the wheels will spin. The velocity of the spin assigned to the wheels is settable within the interval -100<speed<100.
    • second: represents the time interval in seconds [s] during which the wheels will spin.
    
 ### Turn()
  
The `turn(speed , seconds)` sets an angular velocity to the robot resulting in a rotation around the y axis (perpendicular to the map). To achieve this behavior, the function makes the robot's motors run at an opposite speed for a certain amount of time.
Arguments:<br />
    • speed: represents the module of the speed at which the wheels will spin. To make the robot spin around its vertical axis, the velocity of the spin assigned to the right wheel is opposite to the velocity of the left one. If the `speed` argument is `positive` the rotation will be counter-clockwise. Given a `negative` speed, the robot will rotate `clockwise`.
    • second: represents the time interval in seconds [s] during which the wheels will spin.
