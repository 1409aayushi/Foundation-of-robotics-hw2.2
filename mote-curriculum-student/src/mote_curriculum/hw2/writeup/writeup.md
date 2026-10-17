<!---
If you're new to markdown, check out this reference: https://www.markdownguide.org/basic-syntax/
-->

# Writeup

## Q0. Investigating Transforms in Foxglove (10 pts)

**Q0.1:**

Part 1: 
- The base_link frame represents the robots negative X direction (-X).

Part 2: 
- The wheel_left and wheel_right wheels will rotate in the negative direction to move the car forward.

## Q1. Identifying a Kinematic Model (20 pts)

**Q1.1:** 

Left Wheel Velocity Equation:
$v_L = v - \frac{\omega b}{2}$

Right Wheel Velocity Equation:
$v_R = v + \frac{\omega b}{2}$

![Q1.1 Supporting Work](..\images\q1.1.jpeg)
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

![Q1.2 Supporting Work](..\images\q1.2.jpeg)

**Q1.3:** 

Linear Velocity Equation:

$\boxed{v = \frac{v_r + v_l}{2}}$

Angular Velocity Equation:

$\boxed{\omega = \frac{v_r - v_l}{b}}$

![Q1.3 Supporting Work](../images/q1.3.jpeg)

## Q2. Inverse Kinematics (20 pts)

**Q2.2:** 

![Q2.2 IMU Graph](..\images\q2.2imu.png)

![Q2.2 Joint States Graph](..\images\q2.2jointstates.png)

**Q2.3:** 

The predicted and actual velocities are relatively similar. The trends for both the right and left wheels show similar peaks, increases, and decreases in velocity. One of the major differences is that the predicted velocities respond earlier than the actual velocities due to delays in the physical system. When the joystick is moved, the input signal must first be transmitted via Bluetooth to the robot's control board. The board then processes the signal and sends commands to the motors. Afterward, the measured velocity data must be transmitted back through Bluetooth and displayed graphically. These additional steps introduce delays in the actual velocity response. In contrast, the theoretical model uses the joystick input to instantly predict the robot's velocity without accounting for communication and processing delays. This explains why the predicted velocities generally respond earlier than the actual measured velocities.

## Q3. Forward Kinematics (20 pts)

**Q3.2:** 

## Q4. Calculating an Odometry Solution (30 pts)

**Q4.3:** 


## Feedback

**How much time did you spend on this assignment?** 

**Do you have any feedback for this assignment?**

## Use of AI

- Used AI in question 1 to help convert equations into LaTeX form.
- Used AI in question 2.2 to help clean up gramatical errors in my response.
