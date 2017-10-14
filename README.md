This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).


### Overall project overview

TODO Salim

TODO: maybe overview of project on how Perception - Planning - Control work together. nicely reflected in a distributed tool like ROS.

### Perception - the Traffic light detector

![waypoint updater block](./tl-detector-ros-graph.png)

The traffic detection ROS node purpose is to provide obstacle information to the planning nodes. In this project, traffic lights are the first obstacle that we focus on.



This node takes in data from the /image_color, /current_pose, and /base_waypoints topics and publishes the locations to stop for red traffic lights to the /traffic_waypoint topic.

The /current_pose topic provides the vehicle's current position, and /base_waypoints provides a complete list of waypoints the car will be following.

You will build both a traffic light detection node and a traffic light classification node. Traffic light detection should take place within tl_detector.py, whereas traffic light classification should take place within ../tl_detector/light_classification_model/tl_classfier.py.


Traffic Light Detection: This can be split into 2 parts:
Detection: Detect the traffic light and its color from the /image_color. The topic /vehicle/traffic_lights contains the exact location and status of all traffic lights in simulator, so you can test your output.
Waypoint publishing: Once you have correctly identified the traffic light and determined its position, you can convert it to a waypoint index and publish it.


TODO:

* FSM

* Time delay



### Planning - Waypoint Updater

![waypoint updater block](./waypoint-updater-ros-graph.png)

The waypoint updater node takes the car's current pose and waypoint information, to compute and publish the future set of final waypoints which the car's drive by wire (dbw) system can follow.

Before the waypoint updater can start, it needs to receive the base_waypoints -- the set of points which describes the path on the road which the car should follow.  For the simulator, the base_waypoints describes a loop around the test road. The initial base_waypoints also contains a set of vehicle velocities which can be used to initially test the waypoint updater, however, for the complete waypoint updater, those values will be recomputed.  

When the waypoint updater node first receives the base_waypoints, it resets all of the velocities to a minimum since it doesn't necessarily know the current car pose and consequently closest waypoint.  It must also accelerate the vehicle from a standstill. Since the car's starting position in the simulator is relatively close to a traffic light, it could actually receive an upcoming traffic waypoint while it's still accelerating from a standstill.  If the car responds to these traffic waypoints and starts to slow down too early, it will "crawl" until it reaches the first traffic light.  On the other hand, if the waypoint updater allows the car to reach its cruise velocity (about 24mph in the simulator), it will not have enough  time to slow down.  Consequently, the waypoint updater goes through an initialiaization state where it will ignore traffic waypoints.  Once it reaches an initial velocity of about 1/5 the cruise velocity, it will then go into normal mode and respond to traffic waypoint messages.

When the waypoint updater receives the current car pose, it must first calculate the closest waypoint.  To do this efficiently, if it previously computed a closest waypoint, it will use that as a starting point and search around that; otherwise it will search the entire list of base_waypoints.  Once it computes a candidate for closest waypoint, it compares the directional vector from the car's current position to that waypoint versus the car's current orientation (yaw) in order to determine if the waypoint is in front or in back of the car.  If the waypoint is behind the car, it will increment the waypoint by one.  If by chance, the waypoint matches the current car location, it will increment it again so that the car can move forward.

Once the waypoint updater finds the next closest waypoint to follow, it checks to see if there is an upcoming traffic waypoint.  If there is no traffic waypoint, it will take the closest waypoint's velocity and for the next LOOKAHEAD_WPS number of waypoints calculate the appropriate velocities such that it either accelerates to the cruise velocity or stays at the cruise velocity.  If there is an upcoming traffic waypoint, however, it will calculate how far it is from its stopping point and how long it would take to stop using maximum deceleration.  If there is sufficient margin, it will continue to cruise at its current velocity; otherwise it will calculate a trajectory where it slows down.

For both accelerating and decelerating, the waypoint updater uses the distance between two waypoints and the simple kinematic equation: *Vf <sup>2</sup>* = *Vi <sup>2</sup>* + *2&middot;a&middot;s*, where :

* *Vf* : next waypoint velocity
* *Vi* : current waypoint velocity
* *a*  : acceleration or deceleration where acceleration is > 0 and deceleration < 0
* *s*  : distance between waypoints

When the waypoint updater receives a traffic waypoint, it also performs some checks to ensure that it is valid.  If the traffic waypoint is behind the car or outside of it's projected trajectory, then it will ignore it.  If the traffic waypoint is -1 and it's stopped at a traffic waypoint, then it clears its current traffic waypoint and can start accelerating to its target cruise velocity once again.

In the simulator, the base_waypoints forms a loop around the test road.  To support looping around the track, modular arithmetic is used when dealing with waypoint indices.  For example if there are NUM_WAYPOINTS which are indexed from 0 to NUM_WAYPOINTS-1, NUM_WAYPOINTS and NUM_WAYPOINTS+1 will map back to index 0 and 1 respectively.

### Traffic classifier

Traffic classifier code is used which was used in Udacity AI nano-degree course and by Vulture team.
(in Traffic classifier/gan_semi_supervised_kb.ipynb)

Number of epochs was set to 55, as the trade-off when the classifier accuracy on the test set starts to diminish (while the accuracy on the testing set still increases - therefore representint when the classifier starts to overfit the data.)

Overall 5 models were trained and the best model retained.

An example image below:
