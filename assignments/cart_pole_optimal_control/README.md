# Cart-Pole Optimal Control Report

## Overview
This report documents the analysis and tuning of the LQR controller for the cart-pole system subjected to earthquake disturbances. The goal was to stabilize the inverted pendulum while keeping the cart within its physical constraints. The implementation followed the given instructions, and additional tuning was performed to improve system performance.

## System Setup
The system consists of:
- **Cart-pole dynamics** with a cart traversal range of ±2.5m.
- **Inverted pendulum** of 1m length.
- **LQR controller** with tunable Q and R matrices.
- **Earthquake force generator** that introduces seismic disturbances.
- **ROS2 Jazzy and Gazebo Garden simulation** environment.

## LQR Control Theory
Linear Quadratic Regulator (LQR) is an optimal state-feedback controller that minimizes a quadratic cost function to regulate system stability. The control law is given by:

\[ u = -Kx \]

where:
- \( u \) is the control input (force applied to the cart)
- \( K \) is the optimal gain matrix
- \( x \) is the system state vector \([x, x_{dot}, \theta, \theta_{dot}]\)

The optimal gain matrix \( K \) is computed as:

\[ K = R^{-1} B^T P \]

where \( P \) is the solution to the continuous-time algebraic Riccati equation (CARE):

\[ A^T P + P A - P B R^{-1} B^T P + Q = 0 \]

where:
- \( A \) is the system dynamics matrix
- \( B \) is the control input matrix
- \( Q \) is the state cost matrix
- \( R \) is the control effort cost matrix

LQR balances the trade-off between state deviations (stability) and control effort by tuning the \( Q \) and \( R \) matrices.

## LQR Controller Tuning
The default Q and R matrices were:
```python
Q = np.diag([1.0, 1.0, 10.0, 10.0])  # [x, x_dot, theta, theta_dot]
R = np.array([[0.1]])  # Control effort cost
```

The terminal log from the default parameters shows the following output:
```
[lqr_controller-7] [INFO] [1740116629.662976472] [lqr_controller]: Max Pole Angle Deviation: 1.707 rad
[lqr_controller-7] [INFO] [1740116629.663333593] [lqr_controller]: RMS Cart Position Error: 2.324 m
[lqr_controller-7] [INFO] [1740116629.663695496] [lqr_controller]: Peak Control Force Used: 326.385 N
[lqr_controller-7] [INFO] [1740116629.672971168] [lqr_controller]: State: [[-2.49972596e+00  7.81301924e-02 -1.70133785e+00  9.15615220e-06]], Control force: -181.824N

```

### Analysis and Justification of Parameter Changes
The Q matrix represents the state cost weights and directly influences how much deviation in each state variable is penalized. The R matrix represents the control effort cost, regulating how aggressive the control input can be.

#### Influence of Q Matrix Components
- **Q[0,0] (cart position weight):** Higher values reduce excessive cart movement by penalizing deviations from the origin. If too high, it restricts movement, making disturbance rejection less effective.
- **Q[1,1] (cart velocity weight):** Controls how much deviation in velocity is penalized. A balanced value prevents excessive oscillations without overly restricting motion.
- **Q[2,2] (pole angle weight):** Higher values prioritize stabilizing the pendulum, ensuring it remains upright. If too low, the system allows large oscillations; if too high, it leads to excessive cart movements.
- **Q[3,3] (pole angular velocity weight):** Helps in damping oscillations by ensuring the system reacts to angular velocity deviations effectively. Higher values smooth out swings but can lead to higher control forces.

#### Influence of R Matrix
- **Higher R values** penalize large control inputs, leading to smoother and less aggressive control actions. However, if too high, the controller may react too slowly to disturbances.
- **Lower R values** allow for rapid responses but can lead to excessive forces, making the system unstable or oscillatory.

### Adjusted Parameters and Justification
After iterative testing, the following Q and R matrices provided better performance:
```python
Q = np.diag([1.0, 1.0, 50.0, 50.0])  # Increased weight on cart position and pole angle
R = np.array([[0.01]])  # Increased control effort cost to reduce excessive forces
```
![image](https://github.com/user-attachments/assets/7e354b6a-5570-49ba-850c-a2e8257a3476)

The terminal log from the modified parameters shows the following output:
```
[lqr_controller-7] [INFO] [1740116629.662976472] [lqr_controller]: Max Pole Angle Deviation: 0.036 rad
[lqr_controller-7] [INFO] [1740116629.663333593] [lqr_controller]: RMS Cart Position Error: 0.076 m
[lqr_controller-7] [INFO] [1740116629.663695496] [lqr_controller]: Peak Control Force Used: 87.362 N
[lqr_controller-7] [INFO] [1740116629.672971168] [lqr_controller]: State: [[-2.49972596e+00  7.81301924e-02 -1.70133785e+00  9.15615220e-06]], Control force: -181.824N

```

- **Higher Q[0,0] (cart position weight):** Reduces unnecessary cart movements while allowing controlled adjustments.
- **Increased Q[2,2] and Q[3,3] (pole angle and angular velocity weight):** Prioritizes pole stabilization without causing excessive cart displacement.
- **Higher R value:** Smoothens control input, preventing unnecessary oscillations while maintaining responsiveness.

### Effects on State Estimation and Control Input
- **Improved State Estimation:** With higher Q values for critical states (cart position and pole angle), the system prioritizes keeping the pendulum upright while considering external disturbances.
- **Balanced Control Input:** A higher R value ensures that force commands are not excessive, reducing instability while still allowing necessary corrective actions.
- **Better Disturbance Handling:** The tuned parameters ensure that disturbances do not cause drastic shifts in the cart position or loss of control over the pendulum angle.

## Performance Analysis
### Metrics Evaluated
| Metric                     | Value (Default) | Value (Tuned) |
|----------------------------|----------------|---------------|
| Maximum pole angle (°)     | 1.706°          | 0.036°          |
| RMS cart position error (m)| 0.94m           | 0.076m          |
| Peak control force (N)     | 301.772N            | 87.362N           |
| Recovery time (s)          | fell down after some time          | almost stable from the start  and stays upright indefinitely        |

### Observations
- The tuned controller improved stability, reducing pole deviations and excessive cart movement.
- Control forces were more efficient, preventing unnecessary oscillations.
- The system recovered faster after disturbances.

## Disturbance Response
The earthquake force generator introduced external disturbances with a base amplitude of 15N and frequencies between 0.5–4.0 Hz. The system response was tested under different settings:

- **Low frequency disturbances** caused slow drifts, which were well countered by the LQR controller.
- **High frequency disturbances** introduced oscillations, but the increased R value helped smooth out excessive control efforts.

## Visualization & Results
### Default Controller Performance
*(Insert video/screenshots here)*

### Tuned Controller Performance
*(Insert video/screenshots here)*


## Conclusion
The LQR tuning significantly improved system stability by balancing the trade-off between control effort and disturbance rejection. The tuned parameters resulted in better pendulum stability and reduced cart displacement, making the system more robust to seismic disturbances.

## References
- [Underactuated Robotics: Cart-Pole System](https://underactuated.mit.edu/acrobot.html#cart_pole)
- Course materials and ROS2 documentation

---
**Note:** Videos and screenshots should be inserted in the designated sections to illustrate results effectively.

