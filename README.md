# Dynamic Human-Robot Handover using Deep RL (UR5e)

[![ROS 2](https://img.shields.io/badge/ROS_2-Humble-blue.svg)](https://docs.ros.org/en/humble/)
[![Gazebo](https://img.shields.io/badge/Gazebo-Fortress-orange.svg)](https://gazebosim.org/)
[![Stable Baselines3](https://img.shields.io/badge/RL-Stable_Baselines3-purple.svg)](https://stable-baselines3.readthedocs.io/)

A Deep Reinforcement Learning framework for training a 6-DOF UR5e robotic manipulator to perform dynamic handovers with a moving human target. This project evaluates and compares **Proximal Policy Optimization (PPO)** and **Soft Actor-Critic (SAC)** for continuous, contact-rich control tasks.

## 📖 Project Overview
As collaborative robots enter human workspaces, the ability to adapt to unpredictable human motion in real-time is critical. This project utilizes ROS 2 and Gazebo to simulate a UR5e arm attempting to hand over an object to a human hand moving in a dynamic, sinusoidal trajectory. 

**Key Findings:** While off-policy algorithms like SAC are theoretically more sample-efficient, our experiments demonstrated that the on-policy stability of **PPO** vastly outperformed SAC in this specific high-fidelity simulation. PPO converged to a safe, robust tracking policy, whereas SAC suffered from severe hyperparameter sensitivity and catastrophic forgetting.

### ✨ Key Features
* **Dynamic Target Tracking:** The robot learns to anticipate a moving target rather than statically reaching.
* **"Glued Object" Heuristic:** A custom kinematic attachment approach to isolate trajectory planning from Gazebo friction/contact instability.
* **Safe Launch Infrastructure:** Custom ROS 2 lifecycle timers to stabilize the physics engine prior to RL agent interaction.
* **Algorithm Comparison:** Direct benchmarking of SAC vs. PPO using `stable-baselines3`.

---

## 🛠️ Installation & Setup

### Prerequisites
* **OS:** Ubuntu 22.04
* **Middleware:** ROS 2 Humble
* **Simulator:** Gazebo Classic / Ignition
* **Docker:** Recommended for reproducible environments.

### Building the Workspace
1. Clone this repository into your ROS 2 workspace:
   ```bash
   mkdir -p ~/ws_ros/src && cd ~/ws_ros/src
   git clone "this project"
2. Install Python dependencies:
   ```bash
   pip install stable-baselines3 torch numpy tensorboard PyKDL
   ```
3. Build the package:
   ```bash
   cd ~/ws_ros
   colcon build --symlink-install
   source install/setup.bash
   ```

---

## 🚀 Usage

### 1. Launching the Simulation
Start the Gazebo world, spawn the UR5e, and launch the dynamic human hand simulator:
```bash
ros2 launch hri_control hri_world.launch.py
```
*(Note: The environment includes a 10-second "Safe Launch" delay to allow physics controllers to stabilize).*

### 2. Training the Agent
To train the PPO or SAC agent from scratch:
```bash
ros2 run hri_control train
```
*Modify the `train.py` script to toggle between the `PPO` and `SAC` models.*

### 3. Visualizing Results
To view the training curves and algorithmic stability (PPO vs SAC) in TensorBoard:
```bash
tensorboard --logdir ./eval_logs/
```

To visualize the live robot and hand trajectories in RViz:
```bash
ros2 run hri_control visualize_results
```

---

## 📊 Results: PPO vs. SAC

Both agents were trained for 700,000 timesteps using identical state-spaces and dense reward functions.

* **PPO (Stable):** Achieved monotonic convergence to a high mean reward (~463). Exhibited smooth, robust tracking behavior suitable for real-world deployment.
* **SAC (Unstable):** Demonstrated early sample efficiency but suffered from severe "sawtooth" policy collapse (dropping to <200 reward). The maximum-entropy objective conflicted with the precision required for the handover, leading to catastrophic forgetting.

`[Training Results](SAC vs PPO.png)`

---

## 🏗️ System Architecture
* `hri_env_final.py`: The Custom Gymnasium environment bridging RL to ROS 2.
* `fk_helper.py`: PyKDL-based Forward Kinematics wrapper for real-time dense reward calculation.
* `hand_simulator`: Node generating the parameterized sinusoidal target trajectory.
* `train.py`: The main training loop utilizing `stable-baselines3`.

---

## 🙏 Acknowledgments
This project builds upon the shoulders of giants. We acknowledge the following open-source resources and materials:
* [Stable Baselines3](https://github.com/DLR-RM/stable-baselines3) for the robust RL implementations.
* [Universal Robots ROS 2 Description](https://github.com/UniversalRobots/Universal_Robots_ROS2_Description) for the high-fidelity UR5e models.
* Our course instructors for providing the foundational Docker and ROS 2 setup materials.
```
