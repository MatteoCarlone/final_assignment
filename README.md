# ROS Robotics Simulator  <img src="https://media3.giphy.com/media/j5bNquEGoYjPYe7ss1/giphy.gif?cid=ecf05e47ct2lji1wa1xaye059a6m3nljj84tuf4ctzssmdza&rid=giphy.gif&ct=s" width="50"></h2>
## First Assignment of the course [Research_Track_1](https://unige.it/en/off.f/2021/ins/51201.html?codcla=10635) , [Robotics Engineering](https://courses.unige.it/10635). 
###  Professor. [Carmine Recchiuto](https://github.com/CarmineD8).

-----------------------

This is a ROS Robotics Simulator which use Gazebo and Rviz.
The aim of this project was to implement three modality of movement that the user can select in order to let the robot exploring an unknown but limited map.
The user can select the desired behaviour of the robot on a user-interface terminal window, the possible modalities are:

__1. Autonomous Drive:__
		
__The user can set a goal position and the robot will autonomously reach it__

__2. Free Drive:__

__The user can control the robot in the enviroment using its keyboard__

__3. Driver Assistant:__
		
__The user can control the robot movement using its keayboard,__

__but an algorithm of obstacle avoidance will prevent the robot to smash into walls__

Installing and Running 
--------

The implementation of this simulator is based on ROS (Robot-Operating-System), specifically the NOETIC version.
In addition, the project makes use of some packages and tools that need to be installed before starting the project:

* [Slam Gmapping package](https://github.com/CarmineD8/slam_gmapping)

	To Install:

	```bash

		$ git clone https://github.com/CarmineD8/slam_gmapping.git

	```

* ros navigation stack
	
	To Install:

	```bash

		$ sudo apt-get install ros-<your_ros_distro>-navigation

	```

* xterm
	
	To Install:

	```bash

		$ sudo apt install xterm

	```

Once the user has all the needed packages, the simulation starts by running a .launch file called

__run.launch__

```xml
<launch>

	<include file="$(find final_assignment)/launch/launch_nodes.launch" />
	<include file="$(find final_assignment)/launch/simulation_gmapping.launch" />
	<include file="$(find final_assignment)/launch/move_base.launch" />

</launch>
```

Enviroment 
--------

As soon as the simulator starts, Rviz (a 3D visualizer for the Robot Operating System (ROS) framework) and Gazebo (an open-source 3D Robotics simulator) create on the screen respectively: 
What the robot sees of the surrounding space thanks to its sensors
The entire explorable environment 

ROS create the environment described in the file `house.world`, stored into the __world__ folder.

The world view given by the Gazebo simulation turns out to be:

Not knowing all the boundaries of the map, the robot initially knows only the portion of the environment that its sensors allow it to perceive from its starting position.

As the robot moves its knowledge of the map increases. On Rviz the environment known to the robot is graphed and updated at each time instant.

Once the robot has seen all the house_map the view on Rviz turns out to be:

User-Interface
--------------

This is the main node and the first one spawned.

This script provides the user with a small graphic explaining how to select the movement modes of the robot. It also manages the user input by going to modify, if the command is correct, the ros parameters that allow the activation of the nodes set for each mode.

```python
	rospy.set_param('mode', mode_number)
```
This mechanism is managed with a switch-case that makes use of standard Python dictionaries. 
The possible commands are the following:

* __[1]__ - Reach autonomousely a user's desired position
* __[2]__ - Free drive the robot with the keyboard
* __[3]__ - Drive the robot with the keyboard with an active collision avoidance
* __[4]__ - STOP the simulation

When the first modality is called the user will be able to set the desired goal position and to cancel it during the reaching by pressing __[0]__ .

Mode_1 - Autonomous Drive
-------------------------

The Desired goal position is set by the user in the UI by modifying two ROS_Parameters.
```python
	rospy.set_param('des_pos_x', desired_x_position) 
	rospy.set_param('des_pos_y', desired_y_position)
```
The Mode_1 node sends the goal-position to the ActionServer /move_base, the client receives the feedbacks which is print on screen in the current xterm window. Moreover the action sends the the status which is: managed using python dictionaries and print, the main status are [1] the goal achievement and [2] the goal cancelation .
Then the move_base node evaluate the shortest path via the Dijkstra's path algorithm. The robot starts moving toward the goal beacause the move_base node publishes the right velocity and orientation on the cmd_vel topic.

If the user decide to change modality from the UI, the goal is correctly cancelled by sending the proper cancelation message to the ActionServer and this Node turns to an Idle state.

Mode_2 - Free Drive
-------------------

The Second modality is just the [teleop_twist_keyboard](https://github.com/ros-teleop/teleop_twist_keyboard) node adapted on this package by checking the status of the ROS parameter "/mode" which represent the current modality.

The user can control the robot movement by pressing the following keys:

<center>

|| Turn left | Stop turn | Turn right|							
|:--------:|:--------:|:----------:|:----------:|
|__Go forward__|`u`|`i`|`o`
|__Stop__|`j`|`k`|`l`
|__Go backward__|`m`|`,`|`.`

</center>

The user can change the robot velocity of 10% by pressing the following keys:

<center>

|| Linear and Angular | Linear only | Angular only|
|:--------:|:--------:|:----------:|:----------:|
|__Increase__|`q`|`w`|`e`
|__Reset__|`a`|`s`|`d`
|__Decrease__|`z`|`x`|`c`

</center>

If the user decide to change modality from the UI, The Free Drive Node turns to an Idle state.


Mode_3 - Driver Assistant
-------------------------

The Driver Assistant Node starts from the Free Drive Node but is able to disable the commands that, if pressed, would cause the robot to collide into the walls of the map.

The robot is equipped with a laser scanner. The scanner publishes a message of type sensor_msgs/LaserScan.msg on the /scan which is read in a CallBack function by the Node Driver Assistant. Here the algorithm checks weather there's an obstacle or not. The user will be able to select only the direction that wouldn't bring the robot to the walls, this part has been implemented by popping out from a python dictionary the defective commands.

The User can control the robot movement by pressing the following keys:

<center>

|| Turn left | ------ | Turn right|							
|:--------:|:--------:|:----------:|:----------:|
|__Go forward__|`-`|`i`|`-`
|------|`j`|`-`|`l`
|__Go backward__|`-`|`k`|`-`

</center>

The change of velocity is still implemented in this modality and uses the same set of keys.

If the user decide to change modality from the UI, The Driver Assistant Node turns to an Idle state.

Possible Improvements
---------------------

I would like in the future, to implement motion algorithm based on the total knowledge of the map that the robot acquires with these modes.
I think it is possible to implement the A* (A-star) path planning algorithm, based on finding the fastest Belmand-Ford path to a point , considering the walls of the room as obstacles.
This set of implementations would make the movement of the robot in the space more efficient and could be an opportunity to confront myself with such types of algorithms.







