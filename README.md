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

``` Ctrl+O ``` , ``` Enter ``` and ``` Ctrl+X ```



## 4. Edit the CMakeLists.txt File.

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
``` Ctrl+O ``` , ``` Enter ``` and ``` Ctrl+X ```



## 5. Introduce the Environment (Build)

Register code with ROS. That's, build the workspace. Then, refresh the sources.


```bash
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```


## 6. Visualize the Robot

Run launch file.

```bash
ros2 launch my_robot_description display.launch.py
```

## 7. Required RViz Settings

When rviz is first turned on, a blank screen appears. Follow these steps to bring the robot into the environment:

1. Global Options -> Fixed Frame: Use ``` base_link ``` instead of ``` map ```
2. Add -> RobotModel
3. RobotModel -> Description Topic: Write ``` /robot_description ``` and Enter

   **BOOOM!** Our body has arrived.

 
 ## 8. Load the Wheel Control Panel.
In the URDF code, you will set the wheels to ``` type=continuous ```. This tells ROS that "these wheels are motorized and can rotate around their own axis". You need to send the angle information (Joint State) of the wheels. Add a control panel (Joint State Publisher GUI) to the screen where you can rotate the wheels by dragging them manually.


(URDF kodunda tekerlekleri ``` type=continuous ```  olarak ayarlayacaksın. Bu, ROS’a “bu tekerlekler motorlu ve kendi ekseni etrafında dönebilir” demektir. Tekerleklerin açı bilgisini göndermeniz gerekir. Ekrana, tekerlekleri el ile sürükleyerek döndürebileceğiniz bir kontrol paneli ekleyin.

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
        # İŞTE YENİ EKLEDİĞİMİZ KONTROL PANELİ DÜĞÜMÜ
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

## 9. Adding to a URDF File (Like Wheels, etc.)

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
    <axis xyz="0 0 -1"/> </joint>

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
When you run the command, the three-wheeled robot will appear on the screen. A control panel will also appear to move the wheels.
