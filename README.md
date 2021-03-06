###Summary and Objectives
-----
The goal for this project was to develop code that would enable a robot to plan a path through a traversable space. Idealy, this space would have obstacles in it. We chose to focus on a more algorythmically based approach as opposed to one which foccused on directing the physical robots to do things. While physical robots were involved, the main objective was to gain a deeper understanding of the core algorithms by implementing them. 

This project was the second of four progressively more involved robotics projects in Computational Robotics at Franklin W. Olin College of Engineering and was developed over 3.5 weeks. Projects focused on particle filtering, mapping, multi-agent systems, and path planning.

For this particurlar project, the algorithms for path planning and localization were explored. [SLAM](http://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping) mapping was used to create a map or "occupancy grid" describing the area around the robot. This occupancy grid was then used as the innput map for finding the shortest path from the robot's current location to a specified goal using [Dijkstra's Algorithm](http://en.wikipedia.org/wiki/Dijkstra%27s_algorithm). Running Dijkstra's Algorithm yielded a list of waypoints. This list of waypoints was then traversed by the robot in the physical world. To obtain the physical location of the robot while traversing waypoints, the robot's current location is determined from SLAM mapping compared to the  generated map as it traverses the waypoints.


-----
#### Particle Filtering
The current position and direction is determined by a particle filtering algorithm (written by Paul Ruvolo). The particle filtering algorithm continuously updates the position and direction of the robot, which are determined by comparatively weighing its lidar and odom readings to the robot's mapped environment. We assumed that the mean of the estimated positions and directions represents an accurate position and direction of the robot in a period of time.

####Path Planning
The best path for the robot to traverse was generated by Dijkstra's Algorithm. This is a graph based search algorithm. Therefore, to make the continuous space of the world descrete and look like a graph, we devided the world into a grid. The center of each square is a node of the graph. The neighbors surrounding that node were considered to be it's "children" and were the places the robot could move to from that node. This meant that the maximum number of places the robot could move from any given node was its 8 surrounding neighbors--the 4 orthagonal ones to the right, left, front and back, and the 4 diagonal ones midway between the orthagonal neighbors. However, sometimes it's not possible to reach a point if, for example, a T-Rex is sitting on top of the node the Neato wants to traverse. This is a sad day for Neatos, but not for our Neato! Fortunately, our Neato has already mapped the environemnt. Therefore, any nodes which the map says are occupied or whose occupancy is unknown we do not consider traversable. This keeps our Neato from hitting things like carniverous dinosaurs that might clutter our search space. However, if the dinosaur or any other object is moved between mapping and the robot moving through the search space, there is a risk the robot will hit them because it has no way of knowing about obstacles. In the future, it would be good to implement a finite state controler which would monitor objects in front of the robot and momentarily switch into obstacle avoidance mode if the robot is about to hit something before moving back to follow the path.

Dijkstra's Algorithm finds the least net cost path between two nodes. In our application, these are the robot's starting location and the goal we want it to reach. The concept of cost for our application is the distance between two locations. Since the floor we traversed is level, made of a uniform material, and the neato's decrease in speed due to battery drain or other factors over the time span of any journey we considered is negligable, this is reasonable. We used the 2D, planar distance between two points equation to calculate the distance between the two points. We calculate the distance between a node and each of it's traversable children. This is the cost of traveling between nodes. In a different implementation, other costs could have been added such as giving a preference toward orthagonal cells or smooth terain. 

One problem with Dijkstra's Algorithm is that it is computationally quite complex--O(|V^2|). Therefore, on too large a graph or search space, it can take a long time to run. Even on a reasonably small search space, it can take a noticable time to complete. Therefore, computation time was considered when implementing the algorythm. Here, Dijkstra's algorythm is implemented recursevely, despite a small performance cost, because it elimanates redundent code. The map is input into the path planning segment of the code as a list of lists (2D array). However, as the size of the search space increases, the computational complexity of traversing a 2D array increases by n squared. Since both the current node and all  of its children would have needed to be looked up in this way, this was really not an acceptable compramise. Therefore, the map was immediatly translated into a much more efficient data structure: a dictionary of dictionaries. Inside the dictionary, each node was represented by a tuple of it's (x,y) location from the map. The first dictionary key was each node. The value for this node was another dictionary containing each traversable child node for the key and the distance from the parent node to the child node as the value for the child node. Implemented, this looks like:

```python
Space={
    (x1,y1):{
        (x2,y2):1,
        (x3,y3):1.43
    }
    (x2,y2):{
        (x3,y3):1.43,
        (x4,y4):1.43,
        (x1,y1):1
    }
}
```
This data structure makes it very easy and not computationally intense to expand the child nodes. Since this is what is mostly done in Dijkstra's Algorythm, this is a significant gain. 

#### Waypoint Following
Assuming that particle filtering works successfully and locations of waypoints are available, there is enough information to navigate a robot to its waypoints. What is left is simple trigonometry to direct where the robot should be directed towards and proportional control.

Below is a snippet of code that illustrates how trigonometry is used to calculate the desired heading from point A to point B in degrees. 

```python
import math
delta_y = y_B - y_A
delta_x = x_B - x_A
angle = atan2(delta_y/delta_x)*180/math.pi
```

Given desired heading and current direction, we use proportional control to command the speed at which it rotates in the clockwise or counter-clockwise direction. The linear speed is set to low in order to maintain an accurate estimation of current position and direction.

#### Waypoint Check-off
The robot keeps track of its current waypoint. When it detects that it is close to its waypoint, it directs itself to the next waypoint. If there are no waypoints, the robot stays in place. 

### Design Decision/ Code Structure

We've combined the scripts that we worked on into one directory. In addition, we established that one script would import the other and call its methods. This script was the path follower, which imported the path planner. Due to this establishment, we were able to gather information from the path planner before calling the path follower. This code flow made sense logically. At one point, we were considering having the path planner publish to a node that the path follower would subscribe to. However, the path planner takes time to run, which would likely cause problems in following waypoints for the path follower.

Our final, integrated system behaved as distinct parts, all communicating intelligently over ROS topics. The mapping first occured and was directed by code in the file `waypoint_follow.py`. From here, `parse_map.py` was imported into `waypoint_follow.py`. When mapping was complete, `waypoint_follow.py` called a small, helper function in `parse_map.py`. This function was responsible for making sure Dijkstra's Algorythm had a map to use, that it planned a path, and that the a resulting list of waypoints was created. To get the list of waypoints from the path planning section of the code over to the section responsible for moving through waypoints, a string of format: `[(x0,y0),(x1,y1),(x2,y2),...(xn,yn)]` was written to the ROS topic waypoint_list. This could have been any topic name, so long as it was consistant. The code to traverse waypoints then parsed the received string with a regular expression. 

### Challenges

We wished we had allocated more time into integrating the code, because during that process, we came across several bugs, which were larger problems than we had expected. For instance, we wanted our path planning code to publish to a node, but to be able to publish to a node, it needed to be casted into a select amount of data types. We had originally tried to publish an array of tuples of waypoints but spent hours looking through documentation and debugging to get waypoints published. In the end, we decided on publishing a string of etc.

### Future / Interesting Lessons
We found that it is important to dedicate more time towards integration in future robotic programming projects. We had largely underestimated integrating code between the path planning system and path following system. In addition to the graphic of the estimated current position and heading, it would be great to visualize the path of the robot. That way, it would be easy to see how far off our robot is from the path and how it is deviating. 

-----
Running and Installing Code
-----
First, clone the Path_Planning repo onto your local instalation of Ubuntu 12.04 with:
```bash
$ git clone https://github.com/YOUR_GITHUB_USER_NAME/Path_Planning.git
```
Alternatively, clone the repository with SSH keys or Subversion! 

Next, you'll need to establish symlinks between the working file system you just cloned and your catkin workspace. If you're new to ROS, the Catkin workspace handles running your ROS Packages. If you don't already have a Catkin workspace, a good guide to how to create one is [here](https://sites.google.com/site/comprobofall14/home/howto/setting-up-your-environment). 

Once you have a Catkin workspace, or if you have one already, create symlinks between the catkin workspace and the repository. Symlinks effectively allow the same files to exist in two different locations on your computer. Create them with:
```bash
$ ln -s path_to_your_github_repo/my_pf ~/catkin_ws/src
$ ln -s path_to_your_github_repo/hector_slam ~/catkin_ws/src
$ ln -s path_to_your_github_repo/geomtery ~/catkin_ws/src
```

Now, it's time to make the catkin workspace so we can run the code with ROS!
```bash
$ cd ~/catkin_ws/src
$ catkin_make
$ catkin_make install
```

To use localize robot on map:
```bash
$ roslaunch neato_2dnav amcl_builtin.launch map_file:=/home/jasper/catkin_ws/src/hector_slam/hector_mapping/maps/newmap.yaml ($PATHOFYAMLFILE)
```

Set frame to map on Rviz.

```bash
$ rosrun my_pf pf.py
```

Add a topic to listen to.
Select /particle, whose type is Pose. This is the mean of all particles predicted by pf.py in my_pf.

To allow the path planning code to receive the map generated by SLAM maping, run:
```bash
$ roslaunch path_planning start_pathplanner.launch 
```
**Note: If you get an error about there not being a ROS topic about map_static, this is most likely the issue. Make sure you have started this topic. This is a distinct step from begining the ROS operating system.**
