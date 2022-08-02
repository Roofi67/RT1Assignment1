# First Assignment Of Research Track 

The aim of this project was to code a Python script capable of making an holonomic robot :

    • constantly driving the robot around the environment in the counter-clockwise direction,
    • Avoiding the gold tokens and do not collide with the golden token walls
    • And once the robot will get close enough to a silver token, it should grab it and move it behind itself.
## Running the Program
To run one or more scripts in the simulator use run.py, passing it the file names.<br />
```python2 run.py assignment.py```
# Robot API
The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api].
<img width="1192" alt="arena" src="https://user-images.githubusercontent.com/95746070/151961449-1b3a6b80-e540-42f0-a785-577da302496f.png">


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
    • speed: represents the module of the speed at which the wheels will spin. To make the robot spin around its vertical axis, the velocity of the spin assigned to the right wheel is opposite to the velocity of the left one. If the `speed` argument is `positive` the rotation will be counter-clockwise. Given a `negative` speed, the robot will rotate `clockwise`.<br />
    • second: represents the time interval in seconds [s] during which the wheels will spin.
    
 ### find_silver_token()
 The find_silver_token() function is used to study all the silver tokens that are around the robot. The function checks all the tokens that the robot, we can say, sees thanks to R.see() method. The function only takes the tokens that are closer than 3 (which is pretty close inside the enviroment) and inside the angle φ, which is -70°<φ<70°. Obviously, as long as we want only silver tokens, we want to have as marker_type MARKER_TOKEN_SILVER, because it's what it differentiates it from the golden ones.<br />

Arguments<br />
None.<br />
Returns<br />
Returns distance of the closest silver token and angle between the robot and the closest silver token (dist, rot_y).<br />
Code
```ruby
def find_silver_token():
    dist=3
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_SILVER and -70<token.rot_y<70:
            dist=token.dist
	    rot_y=token.rot_y
    if dist==3:
	return -1, -1
    else:
   	return dist, rot_y
```
### find_golden_token()
The find_golden_token() function is crucial for the movement of the robot close to a wall. As the find_silver_token() function, it uses the same methods and code structure of it. We can underline the paramaters that change which are the dist which is going to be higher in order to always check where is the closest golden box (dist=100), the angle which is going to be more limited (-40°<φ<40°) because we want to check only the golden boxes in front of the robot, and, of course, the marker_type which is going to be MARKER_TOKEN_GOLD.

Arguments<br />
    None.<br />
Returns<br />
    Returns distance of the closest golden token and angle between the robot and the closest golden token (dist, rot_y).<br />
Code<br />
```ruby
def find_golden_token():
    dist=100
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_GOLD and -40<token.rot_y<40:
            dist=token.dist
	    rot_y=token.rot_y
    if dist==100:
	return -1, -1
    else:
   	return dist, rot_y
```
### find_golden_token_left()
The find_golden_token_left() function is used to check how far is the wall of golden boxes on the left, used with its sister, find_golden_token_right() we can check where do we have to turn when getting closer to a wall. Nothing is really new because again only one important parameter changes which is the angle, which now is -105°<φ<-75°, as the angle is negative on the left.<br />

Arguments<br />
None.<br />
Returns<br />
Returns distance of the closest golden token on the left (dist).<br />
Code<br />
```ruby
def find_golden_token_left():
    dist=100
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_GOLD and -105<token.rot_y<-75:
            dist=token.dist
    if dist==100:
	return -1
    else:
   	return dist
```
### find_golden_token_right()
The find_golden_token_right() function is identical to the one just explained, it only changes the angle which is 75°<φ<105°, as the angle is positive on the right.<br />

Arguments<br />
None.<br />
Returns<br />
Returns distance of the closest golden token on the right (dist).<br />
Code<br />
```ruby
def find_golden_token_right():

    dist=100
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_GOLD and 75<token.rot_y<105:
            dist=token.dist
    if dist==100:
	return -1
    else:
   	return dist
```
## Main function 

With the help of Flow chart it can be seen that program flow is quite smple. 
![Robot (1)](https://user-images.githubusercontent.com/95746070/151898739-2269086b-04f9-48d0-b95d-12dd751594dd.png)

We nee to run the code endlessly therefore for every true value of 1 robot will keep doing the given task unless there is no error.With the following command functions we can get the distance information after some instances
```ruby
while(1):

	dist_silver, rot_silver = find_silver_token()
	dist_gold, rot_gold =find_golden_token()
	left_dist=find_golden_token_left()
	right_dist=find_golden_token_right()
```
By setting the threshold values silver_th and gold_th we can easily move the robot in the maze with high speed unless it reaches closer toany one of the token.
```ruby
if (dist_gold>gold_th and dist_silver>silver_th) or (dist_gold>gold_th and dist_silver==-1):
			print("Drive straight 0.05 seconds")
			drive(100,0.05)
```
Now, when the robot reaches close to the wall or golden token, which is detected with the front visual angles width of 80 degrees(-40 to 40 degrees). The golden token or wall can be deflected by using the functions below where left side and right side of robots are compared and then turned the robot towards the side whose distance is greater so that the robot can run without hitting the walls.
```ruby
if dist_gold<gold_th and dist_gold!=-1:
			if left_dist>right_dist:
				turn(-25, 0.1)
			elif right_dist>left_dist:
				turn(25, 0.1)
```
Hence the main part of the code when the robot arrives close to the silver token. When we are within the threshold(silver_th) of silver token firstly let's check if the robot is aligned with the token or not. If the robot is nor aligned so we aligned it and then check weather the robot is really close(d_th) to grab the toke and if it not then move the robot a little further. Now we can finally grab the token,turn back,move a little bit further, release the token, move back again and then turn back again to move to the next token
```ruby
if dist_silver<silver_th and dist_silver!=-1:
			if dist_silver < d_th:
				if R.grab():
                    print("Gotcha!")
                    turn(20, 3)
                    drive(20, 0.9)
                    R.release()
                    drive(-20,0.9)
                    turn(-20,3)
	    		elif -a_th<=rot_silver<=a_th:
	    			drive(40, 0.1)
		    	elif rot_silver < -a_th:
				    turn(-10, 0.1)
			    elif rot_silver > a_th:
				    turn(10, 0.1)
	
main()
```
## Possible Improvements
- One possible improvement is that robot should be able to gather the information of silver token from a large distance and direct it towards the token without deviating from a straight line
- Another addition is that robot should release the silver token at the same position which is at the center of maze and run smoothly in the center
