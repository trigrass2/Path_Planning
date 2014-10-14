Summary and Objectives
-----
The goal for this roject was to develop code that would enable a root to plan a path through a traversable space. Idealy, this space would have obstacles in it. We chose to focus on a more algorythmically based approach as opposed to one which foccused on directing the physical robots to do things. While physical robots were involved, the main objective was to gain a deeper understanding of the core algorythms by implementing them. 

-----
Running and Installing Code
-----
First, clone the Path_Planning repo onto your local instalation of Ubuntu 12.04 with:
```Shell
git clone https://github.com/YOUR_GITHUB_USER_NAME/Path_Planning.git
```
Alternatively, clone the repository with SSH keys or Subversion! 

Next, you'll need to establish symlinks between the working file system you just cloned and your catkin workspace. If you're new to ROS, the Catkin workspace handles running your ROS Packages. If you don't already have a Catkin workspace, a good guide to how to create one is [here](https://sites.google.com/site/comprobofall14/home/howto/setting-up-your-environment). 

Once you have a Catkin workspace, or if you have one already, create symlinks between the catkin workspace and the repository. Symlinks effectively allow the same files to exist in two different locations on your computer. Create them with:
```Shell
ln -s path_to_your_github_repo/my_pf ~/catkin_ws/src
ln -s path_to_your_github_repo/hector_slam ~/catkin_ws/src
ln -s path_to_your_github_repo/geomtery ~/catkin_ws/src
```
<!---
/home/jasper/comprobo2014/src  # Jasper, do we really need this?
-->

Now, it's time to make the catkin workspace so we can run the code with ROS!
```Shell
cd ~/catkin_ws/src
catkin_make
catkin_make install
```

To use localize robot on map:
roslaunch neato_2dnav amcl_builtin.launch map_file:=/home/jasper/catkin_ws/src/hector_slam/hector_mapping/maps/newmap.yaml ($PATHOFYAMLFILE)

Set frame to map on Rviz.

rosrun my_pf pf.py

Add a topic to listen to.
Select /particle, whose type is Pose. This is the mean of all particles predicted by pf.py in my_pf.

To allow the path planning code to receive the map generated by SLAM maping, run:
```Shell
roslaunch path_planning start_pathplanner.launch 
```
**Note: if you get an error about there not being a ROS topic about map_static, this is most likely the issue. Make sure you have started this topic. This is a distinct step from begining the ROS operating system.**

-------
Code Ideology
-------
The general idea behi
