<div align="center">

# 🤖 My First Steps in ROS 2

**Foundational notes, core architecture concepts, and environment setup for ROS 2 Jazzy Jalisco on Ubuntu 24.04 LTS. Information based on resources from [Robótica con Python (Carlos Morales)](https://www.youtube.com/@roboticaconpython), [Articulated Robotics (Josh Newans)](https://www.youtube.com/@ArticulatedRobotics) and Mastering ROS 2 for Robotics Programming from Lentin Joseph and Jonathan Cacace.**

[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![ROS 2](https://img.shields.io/badge/ROS_2-Jazzy_Jalisco-22314E?style=for-the-badge&logo=ros&logoColor=white)](https://docs.ros.org/en/jazzy/)
[![DDS](https://img.shields.io/badge/Middleware-CycloneDDS-0078D7?style=for-the-badge)](https://github.com/eclipse-cyclonedds/cyclonedds)
[![Status](https://img.shields.io/badge/Repo_Type-Knowledge_Base-success?style=for-the-badge)]()

</div>

---

## 📑 Table of Contents
- [1. About ROS](#1-about-ros)
  - [What is ROS?](#what-is-ros)
  - [The ROS Equation](#the-ros-equation)
- [2. Core Concepts](#2-core-concepts)
  - [Communication Architecture Diagram](#communication-architecture-diagram)
  - [Topics vs. Services vs. Actions](#topics-vs-services-vs-actions)
- [3. Environment Setup & Installation](#3-environment-setup--installation)
  - [Operating System Requirements](#operating-system-requirements)
  - [Installing & Configuring ROS 2 Jazzy](#installing--configuring-ros-2-jazzy)
- [4. Upcoming Project Integration](#4-upcoming-project-integration)
- [5. References & Learning Resources](#5-references--learning-resources)

---

## 1. About ROS

### What is ROS?

As Jonathan Cacace describes in *Mastering ROS 2 for Robotics Programming*, **ROS (Robot Operating System)** is an open-source framework designed to develop robot software applications. Rather than being a traditional operating system, it provides a set of robot-specific libraries, software tools, and implementations of high-level robotic capabilities on top of Linux.

ROS is widely adopted across multiple robotics domains:

  - **Logistics:** Enables autonomous mobile robots (AMRs) to transport goods in warehouses through navigation, mapping, motion control, and multi-robot coordination.

  - **Manufacturing:** Powers complex tasks such as automated pick-and-place operations integrated with computer vision systems for precision handling.

  - **Healthcare:** Supports assistive robotic platforms that improve clinical workflows and patient care efficiency.

<p align="center">
  <img src="images/ros_logo.png" alt="ROS Logo" width="400"/>
</p>

### The ROS Equation

The **ROS Equation** summarizes the four core pillars that make up the ROS ecosystem:

$$\text{ROS} = \text{Plumbing} + \text{Tools} + \text{Capabilities} + \text{Community}$$

<p align="center">
  <img src="images/ros_eq.png" alt="ROS Equation" width="400"/>
</p>

  - **Plumbing (IPC & Middleware):** Handles Inter-Process Communication (IPC) between nodes. While ROS 1 relied on `TCPROS` (a TCP-based protocol), **ROS 2** is built on top of **DDS (Data Distribution Service)**, offering real-time capabilities, decentralization, and robust discovery.

  - **Tools:** Includes both Graphical User Interfaces (GUIs) and Command-Line Interface (CLI) tools. Key GUI tools include **RViz2** (3D visualization) and **rqt** (graph introspection and debugging), while CLI tools allow direct node and topic inspection from the Linux terminal.

  - **Capabilities:** High-level open-source stacks ready for deployment, including **Nav2** for autonomous navigation and **MoveIt 2** for robotic manipulation. The perception pipeline integrates seamlessly with **OpenCV**, **PCL (Point Cloud Library)**, **Open3D**, and **BehaviorTree.CPP**.

  - **Community:** Backed by a massive global ecosystem featuring community-maintained packages, **ROS Discourse** for announcements, **Robotics Stack Exchange** for troubleshooting, and the official **ROS Discord** server.

---

## 2. Core Concepts

1. **ROS Distributions:**
   Every year, a new version of ROS 2 is released alphabetically using codenames (e.g., *Foxy*, *Humble*, *Jazzy*).

    - **ROS 1:** Legacy architecture (reaching End-of-Life).

    - **ROS 2:** Modern architecture designed for real-time performance, security, and multi-robot systems.

2. **Nodes:**
   An independent executable program focused on a **single modular task** (e.g., reading a LiDAR sensor, controlling wheel motors, or running an object detector). A full robotic system consists of dozens of nodes running concurrently.

   ```bash
   ros2 run <package_name> <node_name>   # Run a specific node
   ros2 node list                        # List all active nodes
   ```

3. **Topics & Messages:**
  An asynchronous, continuous Publisher/Subscriber communication mechanism. A node publishes structured data (Messages) to a named bus (Topic), and any interested node subscribes to receive that stream.

    ```bash
    ros2 topic list     # List all active topics
    ros2 topic echo <topic_name>      # Print real-time messages from a topic
    ```

4. **Services:**
  A synchronous Request/Response (Client/Server) communication mechanism. Unlike continuous topic streams, services are triggered only when a node requests a specific, discrete action (e.g., "calibrate sensor" or "toggle motor").

   ```bash
   ros2 service list                     # List all active services
   ros2 service call <service_name> <type> <arguments>
   ```

5. **Parameters, Remapping & Namespaces:**

    - **Parameters:**
    Internal configuration variables for a node (e.g., serial port, baud rate, max velocity).

    - **Remapping:**
    Allows dynamically reassigning topic or service names at runtime without modifying source code.

    - N**amespaces (__ns):**
    Groups nodes under a root prefix to prevent naming collisions when deploying multiple identical sensors or robots.

6. **Launch Files:**
  Python (or XML/YAML) scripts that allow starting and configuring multiple nodes, parameters, and remappings simultaneously with a single command.

   ```bash
   ros2 launch <package_name> <launch_file.py>
   ```

7. **Packages:**
  The primary organizational unit in ROS 2. A package bundles source code (nodes), launch files, configuration files, and dependency manifests (package.xml) so functionality can be easily built and shared.

8. **Underlay vs. Overlay (Workspaces):**

    - **Underlay:** The core ROS 2 system installation (typically located at /opt/ros/jazzy).

    - **Overlay (Workspace):** Your personal development directory (e.g., ~/ros2_ws) where custom packages are compiled using colcon. Sourcing an overlay layers your packages on top of the underlay.

9. **QoS (Quality of Service):**
  Network policies that control how messages are transmitted over DDS (e.g., Reliable guarantees packet delivery via retries, whereas Best Effort prioritizes low latency and drops packets if the network is congested). Publishers and subscribers must have compatible QoS profiles to communicate.

## 3. Environment Setup & Installation

### 3.1 Operating System Requirements

Most robotic systems run on Linux—most commonly Ubuntu. Hardware targets generally fall into two architectures: 

  - **x86_64:** Developer workstations, laptops, and industrial PCs.

  - **ARM64:** Embedded compute modules such as NVIDIA Jetson Orin or Raspberry Pi.

While ROS 2 can run on Windows or macOS via Docker, WSL2, or virtual machines (VMware, VirtualBox, UTM), native Ubuntu is the standard for hardware interfacing and real-time development.

#### 3.1.1 Setting up Ubuntu 24.04 LTS

On a Development Workstation (Dual Boot): Download the official Ubuntu 24.04 LTS ISO image, flash it onto a USB drive, and install it alongside Windows. You can follow the Official Ubuntu Installation Guide.

On Embedded Robot Hardware: For boards like Raspberry Pi or NVIDIA Jetson, flash the respective customized Ubuntu ARM64 images using tools like Raspberry Pi Imager or NVIDIA SDK Manager.

### 3.2 Installing & Configuring ROS 2 Jazzy

Open a terminal in Ubuntu 24.04 LTS and execute the following steps:

1. **Set Locale (UTF-8 Support)**

Ensure your system supports UTF-8 encoding:

```Bash
locale  # Check current UTF-8 settings

sudo apt update && sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # Verify settings
```

2. **Enable Ubuntu Universe Repository**

```Bash
sudo apt install -y software-properties-common
sudo add-apt-repository universe
```

3. **Add the ROS 2 GPG Key**

```Bash
sudo apt update && sudo apt install -y curl
sudo curl -sSL [https://raw.githubusercontent.com/ros/rosdistro/master/ros.key](https://raw.githubusercontent.com/ros/rosdistro/master/ros.key) -o /usr/share/keyrings/ros-archive-keyring.gpg
```

4. **Add the ROS 2 Repository to Sources List**

```Bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] [http://packages.ros.org/ros2/ubuntu](http://packages.ros.org/ros2/ubuntu) $(. /etc/os-release && echo$UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

5. **Install Development Tools & ROS 2 Desktop**

```Bash
sudo apt update && sudo apt install -y ros-dev-tools
sudo apt install -y ros-jazzy-desktop
```

6. **Configure Environment Sourcing**

Automatically source ROS 2 Jazzy every time a new terminal session starts:

```Bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

7. **Install Colcon & CycloneDDS Middleware**

Install the colcon build tool and CycloneDDS (often preferred over the default FastDDS for multi-machine and sensor setups):

```Bash
sudo apt install -y python3-colcon-common-extensions ros-jazzy-rmw-cyclonedds-cpp
```

8. **Additional Useful Packages (Simulation, Control & Navigation)**

Install common packages for Gazebo simulation, hardware control, teleoperation, and Nav2:

```Bash
sudo apt install -y \
  mesa-utils \
  python3-pip \
  jstest-gtk \
  ros-jazzy-joy* \
  ros-jazzy-joint-state-publisher \
  ros-jazzy-joint-state-publisher-gui \
  ros-jazzy-ros2-control \
  ros-jazzy-ros2-controllers \
  ros-jazzy-ros-gz \
  ros-jazzy-gz-ros2-control \
  ros-jazzy-navigation2 \
  ros-jazzy-nav2-bringup
```

9. **Verify Installation (Talker / Listener Test)**

Test node communication across two separate terminals:

- Terminal 1 (C++ Publisher):

```Bash
ros2 run demo_nodes_cpp talker
```

- Terminal 2 (Python Subscriber):

```Bash
ros2 run demo_nodes_py listener
```

If the listener prints "I heard: [Hello World: ...]" in sync with the talker, your ROS 2 Jazzy installation is ready.

## 4. Upcoming Project Integration

> [!NOTE]
> **Work in Progress:** This repository serves as the theoretical and setup foundation. The practical implementation repository—featuring custom ROS 2 nodes, sensor integration, and workspace packages—is currently under development and will be linked here soon.
>
> 🔗 **Next Repository:** `[Coming Soon - ROS 2 Practical Workspace]`

---

## 5. References & Learning Resources

- **Book:** Cacace, J. *Mastering ROS 2 for Robotics Programming*. Packt Publishing.
- **Official Documentation:** [ROS 2 Jazzy Jalisco Documentation](https://docs.ros.org/en/jazzy/index.html)
- **Setup Guide Reference:** [cmoralesd - ROS Jazzy Setup Guide](https://github.com/cmoralesd/my_package/blob/main/extras/ros_jazzy_setup.md)
- **YouTube Channels:**
  - [Robótica con Python (Carlos Morales)](https://www.youtube.com/@roboticaconpython) — Practical ROS 2 tutorials and robotics fundamentals in Spanish.
  - [Articulated Robotics (Josh Newans)](https://www.youtube.com/@ArticulatedRobotics) — In-depth guides on URDF, `ros2_control`, Gazebo, and building real autonomous robots.
