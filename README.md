# ROS2 l Gazebo l RViz l Remote Control Car
This guide aims to show you how to build a remote-controlled car step by step.

## 1. Setting Up the Workspace

First, create the workspace.
```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

Create a package. This package will contain the robot definition (URDF).

```bash
ros2 pkg create --build-type ament_cmake my_robot_description
```


## 2. Create Robot Definition (URDF)

Define the robot's physical structure in ROS.

```bash

cd my_robot_description
mkdir urdf
nano urdf/my_robot.urdf

```

Create a simple body.
```xml


<?xml version="1.0"?>
<robot name="simple_robot">

  <link name="base_link">
    <visual>
      <geometry>
        <box size="0.5 0.3 0.1"/>
      </geometry>
      <material name="blue">
        <color rgba="0 0 1 1"/>
      </material>
    </visual>
  </link>

</robot>

```
``` Ctrl+O ``` , ``` Enter ``` and ``` Ctrl+X ```


## 3. Viewing the Robot in RViz

Check the result of the code you wrote in RViz. Create a launch file so that everything can be started with a single command.

```bash

mkdir launch
nano launch/display.launch.py

```

```Python

from launch import LaunchDescription
from launch_ros.actions import Node
import os
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():
    pkg_path = get_package_share_directory('my_robot_description')
    urdf_file = os.path.join(pkg_path, 'urdf', 'my_robot.urdf')

    with open(urdf_file, 'r') as infp:
        robot_desc = infp.read()

    return LaunchDescription([
        Node(
            package='robot_state_publisher',
            executable='robot_state_publisher',
            parameters=[{'robot_description': robot_desc}]
        ),
        Node(
            package='rviz2',
            executable='rviz2',
            name='rviz2'
        )
    ])

```


- **Edit the CMakeLists.txt File.**

In ROS2, files must be copied to the install directory during build, which we specify in CMakeLists.txt.

```bash
nano ~/ros2_ws/src/my_robot_description/CMakeLists.txt
```

Go to the bottom of the file. At the very bottom, you'll find the line ``` ament_package() ```. Paste this code above it:


```Cmake
install(DIRECTORY urdf launch
  DESTINATION share/${PROJECT_NAME}
)
```


- **Introduce the Environment (Build)**

Register code with ROS. That's, build the workspace. Then, refresh the sources.


```bash
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```


## 4. Visualize the Robot

Run launch file.

```bash
ros2 launch my_robot_description display.launch.py
```

## 5. Required RViz Settings

When rviz is first turned on, a blank screen appears. Follow these steps to bring the robot into the environment:

1. Global Options -> Fixed Frame: Use ``` base_link ``` instead of ``` map ```
2. Add -> RobotModel
3. RobotModel -> Description Topic: Write ``` /robot_description ``` and Enter

   **BOOOM!** Our body has arrived.

 
 ## 6. Load the Wheel Control Panel.

[ENG]

In the URDF code, you will set the wheels to ``` type=continuous ```. This tells ROS that "these wheels are motorized and can rotate around their own axis". You need to send the angle information (Joint State) of the wheels. Add a control panel (Joint State Publisher GUI) to the screen where you can rotate the wheels by dragging them manually.

[TR]

(URDF kodunda tekerlekleri ``` type=continuous ```  olarak ayarlayacaksın. Bu, ROS’a “bu tekerlekler motorlu ve kendi ekseni etrafında dönebilir” demektir. Tekerleklerin açı bilgisini göndermeniz gerekir. Ekrana, tekerlekleri el ile sürükleyerek döndürebileceğiniz bir kontrol paneli ekleyin.)

- Install Control Panel

```bash
sudo apt install ros-humble-joint-state-publisher-gui
```

- Update launch file

```bash
nano ~/ros2_ws/src/my_robot_description/launch/display.launch.py
```

Delete everything inside (line by line= Ctrl+K) and paste the new code (``` joint_state_publisher_gui ``` added)

```Python

from launch import LaunchDescription
from launch_ros.actions import Node
import os
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():
    pkg_path = get_package_share_directory('my_robot_description')
    urdf_file = os.path.join(pkg_path, 'urdf', 'my_robot.urdf')

    with open(urdf_file, 'r') as infp:
        robot_desc = infp.read()

    return LaunchDescription([
        Node(
            package='robot_state_publisher',
            executable='robot_state_publisher',
            parameters=[{'robot_description': robot_desc}]
        ),
        Node(
            package='joint_state_publisher_gui',
            executable='joint_state_publisher_gui',
            name='joint_state_publisher_gui'
        ),
        Node(
            package='rviz2',
            executable='rviz2',
            name='rviz2'
        )
    ])


```

## 7. Adding to a URDF File (Like Wheels, etc.)

```bash
nano ~/ros2_ws/src/my_robot_description/urdf/my_robot.urdf
```
Delete everything inside (line by line= Ctrl+K). Add wheel code.


```xml

<?xml version="1.0"?>
<robot name="simple_robot">

  <material name="blue">
    <color rgba="0 0 1 1"/>
  </material>
  <material name="black">
    <color rgba="0 0 0 1"/>
  </material>

  <link name="base_link">
    <visual>
      <geometry>
        <box size="0.5 0.3 0.1"/>
      </geometry>
      <material name="blue"/>
    </visual>
  </link>

  <link name="right_wheel">
    <visual>
      <geometry>
        <cylinder radius="0.1" length="0.05"/>
      </geometry>
      <material name="black"/>
    </visual>
  </link>

  <joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="right_wheel"/>
    <origin xyz="-0.15 -0.175 0" rpy="1.5708 0 0"/>
    <axis xyz="0 0 1"/> </joint>

  <link name="left_wheel">
    <visual>
      <geometry>
        <cylinder radius="0.1" length="0.05"/>
      </geometry>
      <material name="black"/>
    </visual>
  </link>

  <joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="left_wheel"/>
    <origin xyz="-0.15 0.175 0" rpy="1.5708 0 0"/>
    <axis xyz="0 0 1"/> </joint>

<link name="caster_wheel">
    <visual>
      <geometry>
        <sphere radius="0.05"/>
      </geometry>
      <material name="black"/>
    </visual>
  </link>

  <joint name="caster_wheel_joint" type="fixed">
    <parent link="base_link"/>
    <child link="caster_wheel"/>
    <origin xyz="0.2 0 -0.05" rpy="0 0 0"/>
  </joint>

</robot>
```

**Build and Execution**

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
ros2 launch my_robot_description display.launch.py
```
When you run the command, the three-wheeled robot will appear on the screen. A control panel will also appear to move the wheels. The visualization in RViz has been completed. we will add weight and collision properties in Gazebo.

## 8. Adding Physical Properties in Gazebo Simulation

We will add weight and collision properties in Gazebo.

```bash
nano ~/ros2_ws/src/my_robot_description/urdf/my_robot.urdf
```

Delete everything inside (line by line= Ctrl+K). Add weight and collision properties.
```xml
<?xml version="1.0"?>
<robot name="simple_robot">

  <material name="blue"><color rgba="0 0 1 1"/></material>
  <material name="black"><color rgba="0 0 0 1"/></material>

  <link name="base_link">
    <visual>
      <geometry><box size="0.5 0.3 0.1"/></geometry>
      <material name="blue"/>
    </visual>
    <collision>
      <geometry><box size="0.5 0.3 0.1"/></geometry>
    </collision>
    <inertial>
      <mass value="5.0"/>
      <inertia ixx="0.04" ixy="0.0" ixz="0.0" iyy="0.1" iyz="0.0" izz="0.14"/>
    </inertial>
  </link>

  <link name="right_wheel">
    <visual>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
      <material name="black"/>
    </visual>
    <collision>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
    </collision>
    <inertial>
      <mass value="1.0"/>
      <inertia ixx="0.0027" ixy="0.0" ixz="0.0" iyy="0.0027" iyz="0.0" izz="0.005"/>
    </inertial>
  </link>

  <joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="right_wheel"/>
    <origin xyz="-0.15 -0.175 0" rpy="1.5708 0 0"/>
    <axis xyz="0 0 1"/>
  </joint>

  <link name="left_wheel">
    <visual>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
      <material name="black"/>
    </visual>
    <collision>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
    </collision>
    <inertial>
      <mass value="1.0"/>
      <inertia ixx="0.0027" ixy="0.0" ixz="0.0" iyy="0.0027" iyz="0.0" izz="0.005"/>
    </inertial>
  </link>

  <joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="left_wheel"/>
    <origin xyz="-0.15 0.175 0" rpy="1.5708 0 0"/>
    <axis xyz="0 0 1"/>
  </joint>

  <link name="caster_wheel">
    <visual>
      <geometry><sphere radius="0.05"/></geometry>
      <material name="black"/>
    </visual>
    <collision>
      <geometry><sphere radius="0.05"/></geometry>
    </collision>
    <inertial>
      <mass value="0.5"/>
      <inertia ixx="0.0005" ixy="0.0" ixz="0.0" iyy="0.0005" iyz="0.0" izz="0.0005"/>
    </inertial>
  </link>

  <joint name="caster_wheel_joint" type="fixed">
    <parent link="base_link"/>
    <child link="caster_wheel"/>
    <origin xyz="0.2 0 -0.05" rpy="0 0 0"/>
  </joint>

  <gazebo reference="caster_wheel">
    <mu1 value="0.001"/>
    <mu2 value="0.001"/>
  </gazebo>

</robot>

```
The remote-controlled car now has a mass of 7.5 kg and includes collision properties.

## 9. Gazebo Simulation World


Now open a new launch file. We're going to the Gazebo simulation world!
```bash

nano ~/ros2_ws/src/my_robot_description/launch/gazebo.launch.py

```
This code will first open the Gazebo Harmonic world, then process the URDF file you wrote and bring the robot into the simulation world.

```python

import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.actions import Node

def generate_launch_description():
    # Get the package paths.
    pkg_my_robot = get_package_share_directory('my_robot_description')
    pkg_ros_gz_sim = get_package_share_directory('ros_gz_sim')

    urdf_file = os.path.join(pkg_my_robot, 'urdf', 'my_robot.urdf')
    with open(urdf_file, 'r') as infp:
        robot_desc = infp.read()

    # 1. Launch Gazebo Harmonic (start with an empty world and use the -r option to start the simulation).
    gazebo = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(pkg_ros_gz_sim, 'launch', 'gz_sim.launch.py')
        ),
        launch_arguments={'gz_args': 'empty.sdf -r'}.items()
    )

    # 2. Robot State Publisher 
    rsp = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        parameters=[{'robot_description': robot_desc}]
    )

    # 3. Create the robot in Gazebo. (Spawn)
    spawn = Node(
        package='ros_gz_sim',
        executable='create',
        # We drop the robot 0.5 meters above the Z-axis so we can see it fall to the ground!
        arguments=['-name', 'my_simple_robot',
                   '-topic', 'robot_description',
                   '-z', '0.5'],
        output='screen'
    )

    return LaunchDescription([
        gazebo,
        rsp,
        spawn
    ])

```

- Build, Source and Run
  
```bash

cd ~/ros2_ws
colcon build
source install/setup.bash
ros2 launch my_robot_description gazebo.launch.py

```

**If Gazebo did not open, follow the step below.**

```bash
export LIBGL_ALWAYS_SOFTWARE=1
```

This command tells the system to "force software rendering, handle graphics via the CPU." Now, run the launch file again.


If you can see your car on the screen, the next step is the final one: adding the motor driver.

## 10. Adding the Motor Driver
```bash

nano ~/ros2_ws/src/my_robot_description/urdf/my_robot.urdf

```
Delete everything inside (line by line= Ctrl+K). Add motor driver.
```xml

<?xml version="1.0"?>
<robot name="simple_robot">

  <material name="blue"><color rgba="0 0 1 1"/></material>
  <material name="black"><color rgba="0 0 0 1"/></material>

  <link name="base_link">
    <visual>
      <geometry><box size="0.5 0.3 0.1"/></geometry>
      <material name="blue"/>
    </visual>
    <collision>
      <geometry><box size="0.5 0.3 0.1"/></geometry>
    </collision>
    <inertial>
      <mass value="5.0"/>
      <inertia ixx="0.04" ixy="0.0" ixz="0.0" iyy="0.1" iyz="0.0" izz="0.14"/>
    </inertial>
  </link>

  <link name="right_wheel">
    <visual>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
      <material name="black"/>
    </visual>
    <collision>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
    </collision>
    <inertial>
      <mass value="1.0"/>
      <inertia ixx="0.0027" ixy="0.0" ixz="0.0" iyy="0.0027" iyz="0.0" izz="0.005"/>
    </inertial>
  </link>

  <joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="right_wheel"/>
    <origin xyz="-0.15 -0.175 0" rpy="1.5708 0 0"/>
    <axis xyz="0 0 1"/>
  </joint>

  <link name="left_wheel">
    <visual>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
      <material name="black"/>
    </visual>
    <collision>
      <geometry><cylinder radius="0.1" length="0.05"/></geometry>
    </collision>
    <inertial>
      <mass value="1.0"/>
      <inertia ixx="0.0027" ixy="0.0" ixz="0.0" iyy="0.0027" iyz="0.0" izz="0.005"/>
    </inertial>
  </link>

  <joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="left_wheel"/>
    <origin xyz="-0.15 0.175 0" rpy="1.5708 0 0"/>
    <axis xyz="0 0 1"/>
  </joint>

  <link name="caster_wheel">
    <visual>
      <geometry><sphere radius="0.05"/></geometry>
      <material name="black"/>
    </visual>
    <collision>
      <geometry><sphere radius="0.05"/></geometry>
    </collision>
    <inertial>
      <mass value="0.5"/>
      <inertia ixx="0.0005" ixy="0.0" ixz="0.0" iyy="0.0005" iyz="0.0" izz="0.0005"/>
    </inertial>
  </link>

  <joint name="caster_wheel_joint" type="fixed">
    <parent link="base_link"/>
    <child link="caster_wheel"/>
    <origin xyz="0.2 0 -0.05" rpy="0 0 0"/>
  </joint>

  <gazebo reference="caster_wheel">
    <mu1 value="0.001"/>
    <mu2 value="0.001"/>
  </gazebo>

<gazebo>
    <plugin filename="gz-sim-diff-drive-system" name="gz::sim::systems::DiffDrive">
      <left_joint>left_wheel_joint</left_joint>
      <right_joint>right_wheel_joint</right_joint>
      <wheel_separation>0.35</wheel_separation>
      <wheel_radius>0.1</wheel_radius>
      <topic>cmd_vel</topic>
    </plugin>
  </gazebo>

</robot>


```
We have completed all the steps; now it’s time to test the remote-controlled car.

## 11. Remote-Controlled Car Testing

Close all terminals and open a new terminal. Follow the steps below

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
export LIBGL_ALWAYS_SOFTWARE=1
ros2 launch my_robot_description gazebo.launch.py
```
Open a new terminal. This terminal will serve as the bridge between ROS and Gazebo.

```bash

source ~/ros2_ws/install/setup.bash
ros2 run ros_gz_bridge parameter_bridge /cmd_vel@geometry_msgs/msg/Twist@gz.msgs.Twist

```

Open a third terminal and run the keyboard controller.
```bash
source ~/ros2_ws/install/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

**BOOOM!** Your remote-controlled car is ready to drive. 

Controls:

* **i** : Move forward 
* **,** (comma) : Move backward
* **j** : Turn left
* **l** : Turn right
* **k** : Brake (Stop)

  The remote-controlled car project has been **successfully** completed.


  For any questions or feedback:

  
<div align="center">
  <a href="https://linkedin.com/in/azatkaya1">
    <img src="https://img.shields.io/badge/LinkedIn-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn">
  </a>




