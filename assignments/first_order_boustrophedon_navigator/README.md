# First-Order Boustrophedon Navigator - ROS2 Tuning

This repository contains the ROS2-based implementation of a first-order Boustrophedon Navigator for a Lawnmower pattern, tuned to minimize cross-track error and optimize the trajectory. The objective was to fine-tune the controller parameters to achieve smooth, efficient path-following with minimal deviations from the target trajectory.

## Final Parameter Values and Justification

### **1. Kp_linear = 3.0**
- **Higher Values:** When the proportional gain was increased beyond 3.0, the robot exhibited unstable behavior, often overshooting the target and overlapping previously covered lines. It also failed to stop within the designated area at the endpoints, moving beyond the restricted zone before halting.
- **Optimal Value (2.2):** This value provided smooth linear tracking, preventing overshooting and ensuring that the robot effectively stopped within the restricted endpoint area, ensuring precise coverage.

### **2. Kd_linear = 0.1**
- **At Kd = 0.0:** Without damping, the robot experienced overlap at turning points due to residual momentum from linear motion.
- **At Higher Values (e.g., 2.0):** The robot decelerated excessively before turns, introducing jerky motion and struggling to follow the trajectory smoothly.
- **Optimal Value (0.1):** The chosen value effectively reduced overlap without introducing excessive deceleration, ensuring smooth transitions at turning points and balanced row-to-row motion.

### **3. Kp_angular = 11.45**
- **Lower Values (e.g., 5.0):** Caused the robot to take wider turns, leading to inefficient coverage and path deviations.
- **Higher Values:** Increased values led to overcorrection and oscillatory behavior during sharp turns.
- **Optimal Value (11.45):** This value produced tight and precise cornering, eliminating oscillations and ensuring smooth, controlled transitions between rows.

### **4. Kd_angular = 0.0**
- **At Non-Zero Values:** Even small values like 0.1 led to wobbling during transitions between rows due to oscillations introduced by the damping factor.
- **Optimal Value (0.0):** Setting the derivative term to zero resulted in smooth, stable turns with no wobbling or oscillations observed during row transitions.

### **5. Spacing = 1.0**
- **Optimal Spacing Justification:** This parameter was crucial for achieving efficient coverage. Smaller values led to redundant coverage, while larger values caused gaps between rows, resulting in incomplete workspace coverage.
- **Optimal Value (1.0):** A spacing of 1.0 balanced coverage efficiency and avoided both overlap and gaps, ensuring that the robot effectively covered the entire workspace.

## Performance Metrics and Analysis

### **1. Cross-Track Error**

- **Current:** -0.001
- **Average:** 0.029
- **Minimum:** 0.001
- **Maximum:** 0.104

![Cross-Track Error Plot](https://github.com/user-attachments/assets/ca531aec-8a6e-4e7a-930f-ecb0804c533e
)
These metrics demonstrate that the robot maintained low deviations from the desired path with minimal cross-track error, indicating effective tracking performance.

### **2. Trajectory Plot**

The trajectory plot shows the robot's path across the boustrophedon pattern. Smooth transitions at turning points can be observed, with spikes in angular velocity corresponding to right and left turns.

![Trajectory Plot](https://github.com/user-attachments/assets/a5713e81-2d47-4dfe-b245-6a8ceb48e925)
![Reconfigure Plot](https://github.com/user-attachments/assets/b80bff93-7445-4dda-9c7d-f4731c1be5c9)
![screenshot](https://github.com/user-attachments/assets/2ef33f71-c33e-448c-9749-f6f10fbf0389)



### **3. Velocity Profiles**

- **Linear Velocity:** Exhibits smooth acceleration and deceleration near turns, maintaining a steady speed during straight-line motion.
- **Angular Velocity:** Shows sharp spikes at turns, indicating tight cornering.


## Discussion of Tuning Methodology

### **1. Initial Tuning**
The process began with reasonable default values to establish a baseline. Observing the robot’s behavior allowed for the identification of issues such as overshooting, wide turns, and wobbling, which guided the fine-tuning process.

### **2. Single Parameter Tuning**
Each parameter was tuned individually, starting with `Kp_linear` and progressing through `Kd_linear`, `Kp_angular`, and `Kd_angular`. This approach helped isolate the effect of each parameter on the robot's motion and enabled targeted improvements.

- **Linear Velocity Tuning (`Kp_linear` & `Kd_linear`):** A balance between responsiveness and smoothness was achieved by setting `Kp_linear = 3.0` and `Kd_linear = 0.1`.
- **Angular Velocity Tuning (`Kp_angular` & `Kd_angular`):** `Kp_angular = 11.2` ensured tight turns, while `Kd_angular = 0.0` eliminated wobbling and oscillations.

### **3. Real-Time Visualization**
Using `rqt_plot` and `rqt_reconfigure`, parameters were dynamically adjusted during runtime. This allowed for real-time observation of performance metrics, which accelerated the tuning process and provided immediate feedback on each change.

### **4. Final Iterations**
After several rounds of testing, parameter adjustments were refined to ensure consistency across multiple cycles, with particular attention to maintaining even row spacing and smooth transitions between rows.

## Challenges and Solutions

### **1. Overshooting at Row Endpoints**
- **Problem:** The robot moved beyond the designated area at row endpoints, causing overlap and reduced trajectory accuracy.
- **Solution:** Reduced `Kp_linear` to prevent excessive acceleration and increased `Kd_linear` to apply smooth braking at row endpoints, eliminating the overshooting behavior.

### **2. Wobbling During Turns**
- **Problem:** The robot exhibited oscillations during sharp turns, affecting stability.
- **Solution:** Set `Kd_angular = 0.0` to eliminate damping-related oscillations. Additionally, fine-tuned `Kp_angular` for precise cornering without wobbling.

### **3. Uneven Spacing Between Rows**
- **Problem:** Inconsistent row spacing led to inefficient coverage with gaps or overlaps.
- **Solution:** Adjusted the `spacing` parameter to 1.0, ensuring even coverage while preventing redundant coverage or gaps.

## Conclusion

The tuning process allowed for fine control over the robot's trajectory and motion, ensuring smooth path following with minimal cross-track error. The final parameter set enabled efficient workspace coverage with tight, controlled turns and smooth linear motion. Visualization tools and iterative tuning helped in achieving a balance between performance and accuracy, ensuring the successful implementation of the boustrophedon pattern.

A video on launching the boustrophedon navigator  can be seen in the following Google Drive Link
![Video](https://drive.google.com/file/d/1v8pKqW07pFh6UCpk2wj5z-b0Es-U7hRq/view?usp=drivesdk)

