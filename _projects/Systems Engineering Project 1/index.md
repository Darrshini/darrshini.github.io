---
layout: post
title: Systems Engineering Project 1
description:  
    As part of a team project, I worked on developing an autonomous navigation system for the Agilex LIMO robot within a custom-designed arena. Using the RTAB-Map SLAM approach, the robot was able to build a map of the environment and localize itself in real time. Waypoints for navigation were identified directly through the RTAB-Map interface, allowing us to set specific goal locations within the mapped environment. I contributed by writing Python code to enable the robot to follow these waypoints using ROS, while the move_base navigation stack handled global and local path planning to avoid obstacles and determine the most efficient route. This project gave me hands-on experience in mapping, waypoint navigation, and system integration with real-world robot hardware.
skills: 
  - ROS 1 (Melodic)
  - RTAB-Map SLAM
  - Python
  - move_base navigation stack

main-image: /limo_robot.jpg
---

---
## Arena Designing process
<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap; margin: 20px 0;">
  {% include image-gallery.html 
     images="/assets/images/Terminal1.jpg, /assets/images/kinetic_rain.jpg"  
     height="375" 
  %}
</div>

For this project, our goal was to create an autonomous navigation system using the Agilex LIMO robot in a team-designed arena. We based the layout on Changi Airport Terminal 1, including recognizable features like the iconic Kinetic Rain setup to make it more engaging and realistic. The idea was to build a fun and meaningful environment where the robot could map the area using RTAB-Map SLAM and move between waypoints while avoiding obstacles using the move_base navigation stack. 

### **Initial Arena Design**
<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap; margin: 20px 0;">
  {% include image-gallery.html 
     images="/assets/images/Initial_design_plan.jpg, /assets/images/Initial_Design_Solidworks.png" 
     height="350" 
  %}
</div>
As seen in the images above, our team initially planned for a more complex design that included two levels, ramps, and gantries. 

### **Final Arena Design**
<div class="arena-slideshow-container">

{% include image-gallery.html 
   images="/assets/images/Final_Design_Solidworks.png, /assets/images/Team7_Arena.jpg, /assets/images/Arena_Entrance.jpg, /assets/images/Arena_Windows.jpg, /assets/images/Arena_KineticRain.jpg"
   height="500"
   mode="slideshow"
   captions="Solidworks Final Design|Arena Side view|Arena Entrance view|Arena Top View|Arena Kinetic Rain"
%}
</div>
After evaluating its feasibility and considering the time and budget constraints, we decided to go with a simpler design. The updated design still met the project requirements such as ensuring the Limo Robot can pass through the center of the plot, provided enough challenges for the LIMO robot to navigate through, and effectively represented the key elements of Changi Terminal 1.


---
## Mapping the Class Arena
The project required us to design an arena that not only allowed our LIMO robot to navigate through our team’s plot but also to move seamlessly across other teams’ plots. The robot needed to be able to pass through the central point of each plot in the class’s combined arena while navigating around the various obstacles placed within each area.

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap; margin: 20px 0;">
  {% include image-gallery.html 
     images="/assets/images/Class_arena.jpg" 
     height="450" 
  %}
</div>

### **Using RTAB-Mapping** 
We chose RTAB-Map instead of alternatives like Cartographer or Gmapping because it is able to localize the robot automatically in the map without us having to manually figure out where it is each time. RTAB-Map also works well with ROS 1 (Melodic) and can handle larger environments accurately, which was important for mapping the entire class arena with multiple obstacles and plots.

<div style="display: flex; justify-content: center; gap: 20px; flex-wrap: wrap; margin: 20px 0;">
  {% include image-gallery.html 
     images="/assets/images/completed_map.jpg, /assets/images/completed_map_navigation.jpg" 
     height="400" 
  %}
</div>
Using RTAB-Map, we were able to map the entire class arena, which included all the teams’ plots combined into one large environment. The mapping process allowed the LIMO robot to build a detailed 2D/3D representation of the arena in real time while it navigated through each plot. This map served as the foundation for waypoint navigation and obstacle avoidance, as the robot relied on it to localize itself and plan efficient paths. The results of the mapping can be seen in the images above, visualized using the RViz tool in ROS.

---

## Navigation Code Implementation

### **Core Architecture**
```python
class WaypointNavigator:
    def __init__(self):
        rospy.init_node('waypoint_navigator')
        self.client = actionlib.SimpleActionClient('move_base', MoveBaseAction)
```
---
### **Framework Components:**

 - move_base action server for global & local path planning
 - TF transforms for map-based coordinate handling
 - Actionlib to send and track navigation goals one by one

**1. Waypoint Management**
```python
self.plot_paths = {
    "Plot1": [{"x": "1.378", "y": "1.427", "yaw": "0.567"}],
    # ... 8 predefined plots ...
    "Plot0": [{"x": "0.0", "y": "0.0", "yaw": "0.0"}]  # Home
}
```
**What this does:**

 - We store predefined coordinates (x, y, yaw) for each plot.
 - Plot0 is the home location, and the others represent specific waypoints around the arena.
 - Plot 0 is the center plot of the entire arena , it is the starting point for the Limo Robot before navigating to other plots.
 - Yaw values are stored as radians (rotation) so the robot faces the correct direction when it reaches each point.

**2.User input handling**
```python
selection = raw_input("Enter your plot sequence (e.g. 134, 1 3 4): ")
selected_plots = re.findall(r'\b[0-9]\b', selection)
```
**How it works:**

 - The user can type multiple waypoints at once (e.g., 1 3 4 or 134).
 - We use a small regex (re.findall) to clean up the input and only keep valid numbers between 0 and 9.
 - It makes the robot more flexible since you can plan multi-stop routes in one go.

**3. Navigation Control**
```python
for plot_num in selected_plots:
    plot_key = "Plot{}".format(plot_num)
    self.navigate_to(self.current_location, plot_key)
    self.current_location = plot_key

```
**What this does:**

 - Loops through each selected waypoint in order.
 - Calls navigate_to() to send a navigation goal for each plot.
 - Updates current_location so the robot knows where it’s coming from next.

   
**4. Waypoint Execution**
```python
def navigate_to(self, from_plot, to_plot):
    path = self.plot_paths[to_plot]
    for wp in path:
        goal = self.create_goal(wp["x"], wp["y"], wp["yaw"])
        self.client.send_goal(goal)
        self.client.wait_for_result()
```
**How it works:**

 - Converts the coordinates into a ROS navigation goal.
 - Waits for the robot to finish moving before sending the next goal (so it doesn’t skip).
 - Retries up to 10 times if it fails (e.g., if there’s an obstacle in the way).

**5. Fault Recovery**
```python
if state != actionlib.GoalStatus.SUCCEEDED:
    self.clear_costmaps()  # Clear obstacles from costmaps
    self.client.send_goal(goal)  # Retry the goal
```
 - If a goal fails, we clear the costmaps to reset any temporary obstacle data.
 - Then we retry the same goal up to 10 times before skipping it.
 - This makes the robot more robust when navigating in changing environments.
   

**6. Goal Creation**
```python
def create_goal(self, x, y, yaw):
    goal = MoveBaseGoal()
    goal.target_pose.header.frame_id = "map"
    q = quaternion_from_euler(0, 0, float(yaw))

```
**How it works:**

 - Converts (x, y, yaw) into a proper ROS navigation goal in the map frame.
 - Uses quaternion_from_euler to get the correct orientation so the robot faces the right way when it arrives.

---

## Example of our Limo Robot moving from one plot to another

{% include youtube-video.html id="kO21XZkHZq8" autoplay = "true" %}

---
**CONCLUSION:**
This project gave me and my team valuable hands-on experience with multi-waypoint navigation using ROS 1, implementing fault recovery mechanisms, and designing flexible user input. It served as a strong foundation for understanding how autonomous robots can plan and adapt in real-world scenarios.
