---
layout: post
title: Systems Engineering Project 1
description:  
    As part of a team project, I worked on developing an autonomous navigation system for the Agilex LIMO robot within a custom-designed arena. Using the RTAB-Map SLAM approach, the robot was able to build a map of the environment and localize itself in real time. Waypoints for navigation were identified directly through the RTAB-Map interface, allowing us to set specific goal locations within the mapped environment. I contributed by writing Python code to enable the robot to follow these waypoints using ROS, while the move_base navigation stack handled global and local path planning to avoid obstacles and determine the most efficient route. This project gave me hands-on experience in mapping, waypoint navigation, and system integration with real-world robot hardware.
skills: 
  - ROS 1 (Melodic)
  - RTAB-Map SLAM
  - Python

main-image: /Limo-top-AgileX.webp
---

---
## Project Overview
{% include image-gallery.html images="/assets/images/Terminal1.jpg, /assets/images/kinetic_rain.jpg"  height="400" %}

For this project, our goal was to create an autonomous navigation system using the Agilex LIMO robot in a team-designed arena. We based the layout on Changi Airport Terminal 1, including recognizable features like the iconic Kinetic Rain setup to make it more engaging and realistic. The idea was to build a fun and meaningful environment where the robot could map the area using RTAB-Map SLAM and move between waypoints while avoiding obstacles using the move_base navigation stack. 

### **Initial Arena Design**
{% include image-gallery.html 
   images="/assets/images/Initial_design_plan.jpg", /assets/images/Initial_design_solidworks.jpg" 
   height="400" 
%}
<span style="font-size: 10px">Team-designed testing arena (left) and Designed in Solidworks (right)</span>


### **Final Arena Design**
{% include image-gallery.html 
   images="/assets/images/Final_arena_design_solidworks.jpg, /assets/images/Team7_Arena.jpg" 
   height="400" 
%}
<span style="font-size: 10px">Final design plan in SolidWorks (left) and implemented design (right)</span>


---
## Using RTAB-Mapping for Mapping the Arena

---

## Technical Implementation

### **Core Architecture**
```python
class CombinedNavigator:
    def __init__(self):
        rospy.init_node('combined_navigator')
        self.client = actionlib.SimpleActionClient('move_base', MoveBaseAction)
```
---
### **Framework Components:**

 - move_base action server for global/local planning
 - TF transforms for coordinate handling
 - Actionlib for asynchronous goal management


**1. Waypoint Management**
```python
self.plot_paths = {
    "Plot1": [{"x": "1.15", "y": "1.09", "yaw": "0.0"}],
    # ... 8 predefined plots ...
    "Plot0": [{"x": "0.0", "y": "0.0", "yaw": "0.0"}]  # Home
}
```
**Features:**

 - Preconfigured coordinate system
 - String-to-float auto-conversion
 - Bidirectional path planning (forward/reverse)

**2. Navigation State Machine**
```python
def navigate_to(self, from_plot, to_plot):
    if from_plot == self.home_plot:
        path = self.plot_paths[to_plot]
    elif to_plot == self.home_plot:
        path = list(reversed(self.plot_paths[from_plot]))
```
**Logic Flow:**

- Start from current location
- Calculate path segments
- Execute sequential waypoints
- Handle return-to-home specially

**3. Fault Recovery System**
```python
if state != actionlib.GoalStatus.SUCCEEDED:
    self.clear_costmaps()  # Reset obstacle data
    self.client.send_goal(goal)  # Retry
```
**Error Handling:**

- Automatic costmap clearing
- Single retry attempt
- Detailed ROS logging

**Goal Formulation**
```python
def create_goal(self, x, y, yaw):
    goal = MoveBaseGoal()
    goal.target_pose.header.frame_id = "map"
    q = quaternion_from_euler(0, 0, float(yaw))
    # ... orientation assignment ...
```
**Key Aspects:**

 - Frame-aware targeting (map frame)
 - Proper quaternion conversion
 - Type-safe coordinate handling


