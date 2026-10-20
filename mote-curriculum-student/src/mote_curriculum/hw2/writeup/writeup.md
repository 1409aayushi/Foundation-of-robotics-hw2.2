<!---
If you're new to markdown, check out this reference: https://www.markdownguide.org/basic-syntax/
-->

# Writeup

## Q0. Investigating Transforms in Foxglove (10 pts)

**Q0.1:**

Part 1: 
- The rover moves forward along the positive X axis (+X) of the base_link frame. In Foxglove, the red (X) axis of base_link points toward the front of the rover, and driving forward with teleop sends a positive linear.x command.

Part 2: 
- Both wheel_left and wheel_right rotate in the positive direction when the rover moves forward. We confirmed this by echoing `/joint_states/velocity` while driving forward with teleop: both wheel velocities were positive (and both were negative when driving backward).

## Q1. Identifying a Kinematic Model (20 pts)

**Q1.1:** 

Left Wheel Velocity Equation:
$v_L = v - \frac{\omega b}{2}$

Right Wheel Velocity Equation:
$v_R = v + \frac{\omega b}{2}$

![Q1.1 Supporting Work](../images/q1.1.jpeg)
<!---

You can use LaTeX syntax like so: $a_{example} = b_{example} + c_{example}$.
You can use $\upsilon$ for linear velocity and $\omega$ for angular velocity.

If you prefer to write out your work, you can include it in an image as follows:
![image description](./image_path.png)
-->

**Q1.2:**
Angular Velocity for Left Wheel Equation:

$\boxed{\dot{\phi}_L = \frac{2v_l}{d}}$

Angular Velocity for Right Wheel Equation:

$\boxed{\dot{\phi}_R = \frac{2v_r}{d}}$

![Q1.2 Supporting Work](../images/q1.2.jpeg)

**Q1.3:** 

Linear Velocity Equation:

$\boxed{v = \frac{v_r + v_l}{2}}$

Angular Velocity Equation:

$\boxed{\omega = \frac{v_r - v_l}{b}}$

![Q1.3 Supporting Work](../images/q1.3.jpeg)

## Q2. Inverse Kinematics (20 pts)

**Q2.2:** 

![Q2.2 IMU Graph](../images/q2.2imu.png)

![Q2.2 Joint States Graph](../images/q2.2jointstates.png)

**Q2.3:** 

The predicted and actual velocities are relatively similar. The trends for both the right and left wheels show similar peaks, increases, and decreases in velocity. One of the major differences is that the predicted velocities respond earlier than the actual velocities due to delays in the physical system. When the joystick is moved, the input signal must first be transmitted via wifi to the robot's control board. The board then processes the signal and sends commands to the motors. Afterward, the measured velocity data must be transmitted back through Bluetooth and displayed graphically. These additional steps introduce delays in the actual velocity response. In contrast, the theoretical model uses the joystick input to instantly predict the robot's velocity without accounting for communication and processing delays. This explains why the predicted velocities generally respond earlier than the actual measured velocities.

Additionally, we observed that the actual wheel velocity appears to be limited. For example, when the predicted velocity ranges from −30 to −32 rad/s, the measured velocity plateaus at approximately −25 rad/s and does not decrease further. This behavior is visible for both wheels in the graph, around time intervals 2184–2190, 2192–2197, and 2198–2205. We suspect this may be because the motors are reaching their maximum rotational speed. At approximately 25 rad/s and a wheel radius of 0.033 m, the corresponding linear wheel speed is about 0.83 m/s. Alternatively, the speed controllers may be limiting the maximum wheel velocity. Since our inverse kinematics equations do not account for these limits, they continue to predict higher wheel speeds than the hardware can achieve. Other factors, such as wheel slip, friction, and imperfect speed tracking by the controllers, may also contribute to the discrepancies between predicted and measured velocities.

## Q3. Forward Kinematics (20 pts)

**Q3.2:** 

In both screenshots, the top plot compares the commanded linear velocity (orange, `/diff_drive_controller/cmd_vel.linear.x`) with the linear velocity computed from the wheel encoders by our forward kinematics node (blue, `/cmd_vel_ground_truth.twist.linear.x`). The bottom plot does the same for angular velocity (`angular.z`). The rover was driven forward, backward, and spun left and right using keyboard teleop (0.3 m/s, 1.5 rad/s).

![Q3.2 Run 1: Forward Kinematics vs Commanded Velocity](../images/q3.2_run1.png)

![Q3.2 Run 2: Forward Kinematics vs Commanded Velocity](../images/q3.2_run2.png)

**How do the commanded and actual velocities compare?**

All in all, the actual velocites follow the commanded velocities closely. In the case when ±0.3 m/s was commanded, it was observed that the measured linear velocity settled between about 0.29 and 0.34 m/s in magnitude. Additionallly, when ±1.5 rad/s was commanded, the measured angular velocity stayed roughly between 1.4 and 1.65 rad/s in magnitude. It was observed that the signs always matched the command, which confirmed that our forward kinematics equations and wheel sign conventions were correct.
The main differences are:

- **Lag:** During both startup and shutdown, we observed a delay of approximately 0.1–0.3 seconds between a change in the command and the corresponding change in the actual velocity.
- **Overshoot:** The rover briefly exceeds the command when starting a motion, for example up to 0.37 m/s linear around t = 330 s in Run 1, about 0.35 m/s around t = 147 s in Run 2, and up to about 1.9 rad/s angular in both runs.
- **Noise:** The measured signals are stair-stepped and jittery instead of flat, and the angular velocity is noticeably noisier than the linear velocity.
- **Coupling:** While spinning in place, the measured linear velocity is not exactly zero (small ±0.01 m/s wobble, Run 1 at 334–339 s). While driving straight, the measured angular velocity also shows small nonzero values.
- **Missed stop (Run 1 only):** Near the end of Run 1 (t ≈ 343–351 s), the command returned to zero, but the rover kept moving at about 0.31 m/s and −1.3 rad/s. In Run 2 the rover stopped correctly every time (actual returns to 0 by t ≈ 154 s and stays there).

Note: the sloped orange segments are an artifact of Foxglove connecting the sparse teleop messages with straight lines. The actual command is a step change.

**What could account for those differences?**

- **Communication and processing delay:** The latency between the command and the measured response is increased because the command must be transmitted to the rover over Wi-Fi and processed by its controller. Additionally, the encoder data must be sent back to the computer for plotting, further contributing to the overall latency.
- **Encoder resolution:** Wheel velocity is estimated from a finite number of encoder ticks per sampling interval, the measured velocity changes in discrete steps rather than continuously. This explains the stair-step pattern and the observed jitter in the velocity measurements.
- **Motor dynamics and inertia:** The motors cannot change the wheel speed instantaneously, and the wheel-speed controller requires time to accelerate the rover. The brief overshoot suggests that the speed controller is slightly underdamped.
- **Amplification in the angular calculation:** angular velocity is computed as $\omega = \frac{v_r - v_l}{b}$ with a small wheel separation $b = 0.15$ m. Any small error in the wheel speeds gets divided by 0.15, which amplifies it about 6.7×. That is why the angular plot is noisier than the linear plot.
- **Wheel slip and motor mismatch:** The two motors are not exactly the same, and the wheels slip on the ground when spinning in place. This causes the small linear velocity while turning and the small angular velocity while driving straight. The forward kinematics model also assumes ideal wheels that roll without slipping and an exact wheel diameter and separation, which is never perfectly true.
- **Lost commands:** The missed stop in Run 1 is most likely a stop command that never reached the rover over the wireless link, so the rover kept executing the last command it received. This shows that the commanded velocity is not always what the rover actually does, which is why measuring the real velocity with forward kinematics is useful.

## Q4. Calculating an Odometry Solution (30 pts)

**Q4.3:** 

**Observations**

- **Experiment 1 (wheels in the air):** The rover moved in Foxglove even though it was not actually moving.
- **Experiment 2 (one wheel stopped by hand):** The rover spun in Foxglove when we held one wheel, instead of going forward as commanded.
- **Experiment 3 (driving into a wall):** The rover kept going forward in Foxglove even though it was blocked by the wall.

**Why the odometry behaves this way**

We just use the wheel encoders to calculate our odometry. Forward kinematics turns the measured wheel speeds into a linear and angular velocity, and the odometry node integrates those velocities over time to estimate the pose. Additionally, this assumes the wheels are always on the ground and roll without slipping. The odometry can only measure how far the wheels turned, not if the rover really moved.

- In Experiment 1, the wheels spun freely in the air. The encoders still measured wheel rotation, so the odometry integrated a velocity and estimated motion that never happened.
- In Experiment 2, one wheel had zero velocity while the other kept turning. From $\upsilon = \frac{\upsilon_r + \upsilon_l}{2}$ and $\omega = \frac{\upsilon_r - \upsilon_l}{b}$, this gives a small linear velocity and a large angular velocity, so the odometry predicts the rover pivoting around the stopped wheel. The odometry has no way of knowing the wheel was being held, so it does not match what was commanded.
- In Experiment 3, the rover could not move forward, but the wheels kept turning and slipped on the floor. The encoders counted that rotation, so the estimated position kept moving forward through the wall.

In all three cases the error accumulates and is never corrected, because there is no other sensor to check the wheel-based estimate against.

**Proposed improvements**

- **IMU:** The gyroscope measures the rover's actual rotation rate, which would show that the rover was not spinning in Experiments 1 and 2. The accelerometer could detect that the rover was lifted off the ground, or the sudden deceleration of hitting the wall in Experiment 3.
- **LiDAR:** Comparing consecutive laser scans (scan matching or SLAM) shows whether the surroundings are actually moving relative to the rover. In Experiment 3 the wall distance stays constant, so LiDAR would show the rover is not moving forward.
- **Sensor fusion:** Combining wheel odometry, IMU, and LiDAR with a filter such as an Extended Kalman Filter (for example the `robot_localization` ROS package) would let the other sensors correct wheel-odometry errors like slip.
- **Motor current/effort:** A large motor effort with little measured motion could be used to detect stalls or wheel slip and to stop trusting the wheel odometry during those periods.


## Feedback

**How much time did you spend on this assignment?** 

5 hours

**Do you have any feedback for this assignment?**

Sometimes, after driving the robot forward for a long time, it kept moving forward even after we tried to stop it (with both the Foxglove joystick and keyboard teleop), and it would not stop on its own. We had to unplug the robot multiple times to stop it.

## Use of AI

- Used AI in question 1 to help convert equations into LaTeX form.
- Used AI in question 2.2 to help clean up gramatical errors in my response.
- Used AI to remove awkward phrasing and polish overall. 
- Additionally used AI to verify our reasoning.
