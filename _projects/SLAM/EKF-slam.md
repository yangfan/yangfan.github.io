---
layout: page
title: Online SLAM with Extended Kalman Filter
description: |
  The first SLAM solution in history is based on the Extended Kalman Filter. This project implements an EKF SLAM system. Given the sensor readings and odometry information, the robot computes a feature-based map while simultaneously localizing itself.
img: /assets/img/projects/SLAM/EKF-slam/ekf_slam.gif
importance: 1
category: SLAM
github: https://github.com/yangfan/probabilitics-robotics/tree/master/EKF_SLAM
---

The SLAM problem is one of the most fundamental problems in robotics. SLAM problems arise when both map of the environment and the pose of robot are not available. Robots are only given the measurement $$z_{1:t}$$ and controls $$u_{1:t}$$. In SLAM problem, the robot should learn a map of the environment and simultaneously estimate its pose in the environment.

There are two main forms of SLAM problem:

1. Full SLAM problem: robot should computes a posterior over the entire path along with the map, i.e.,
   $$P(x_{1:t}, m|z_{1:t},u_{1:t})$$.

2. Online SLAM problem: robot computes a posterior over the pose in the current time step along with the map, i.e.,
   $$P(x_t, m|z_{1:t},u_{1:t})$$.

In this project, we implement the EKF SLAM algorithm for the online SLAM problem. The map in this problem is the feature-based map, which is composed of point landmarks. The data set<sup>[1]</sup> contains the odometry information, the range-bearing sensor readings and the correspondence in each time step. Using EKF, the robot estimates the current pose of robot (illustrated by the red covariance ellipse and bar) and the positions of all landmarks (illustrated by the blue covariance ellipse). The gray lines show the observed landmarks in the current time step.

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/SLAM/EKF-slam/ekf_slam.gif' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
  Extended Kalman Filter Algorithm for the Online SLAM Problem
  </div>

## EKF Overview

Similar to EKF for localization, the algorithm include two steps: predication by the motion model and correction by the sensor model. Unlike EKF for localization problem, the state includes the position of all landmarks in addition to the robot's pose, i.e.,

$$
(x,y,\theta, m_{1,x}, m_{1,y}, ..., m_{n,x}, m_{n,y})
$$ where $$n$$ is the number of landmarks.

Therefore the nonlinear motion function $$g(u_t, x_{t-1})$$, obvervation function $$h(x_t)$$, and the corresponding Jacobian $$G_t$$ and $$H_t$$ should be changed accordingly.

<div class="row justify-content-center">
    <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/localization/EKF/extended-kalman-filter.png' | relative_url }}" alt=""/>
        </div>
    </div>
</div>
<div class="caption">
The Extended Kalman Filter for Localization
</div>
<br/>

More specifically for the odometry motion model, the prediction step is


$$

\bar{\mu}_t = g(u_t, \mu_{t-1}) = \mu*{t-1} + F^T*{x}
\begin{bmatrix}
\delta*{tran} \cos(\mu*{\theta*{t-1,\theta}}+\delta*{rot1}) \\
\delta*{tran} \sin(\mu*{\theta*{t-1,\theta}}+\delta*{rot1}) \\
\delta*{rot1}+ \delta*{rot2}
\end{bmatrix}

$$

where $$\mu_{t-1}\in \mathbb{R}^{(3+2n)\times 1}, F_x \in \mathbb{R}^{3\times (3+2n)}$$


$$

\mu*{t-1} =
\begin{bmatrix}
\mu*{t-1, x} \\
\mu*{t-1, y} \\
\mu*{t-1, \theta} \\
\mu*{m_1,x} \\
\vdots \\
\mu*{m_n,y}
\end{bmatrix},
\ \ F_x =
\begin{bmatrix}
1 & 0 & 0 & \cdots & 0 \\
0 & 1 & 0 & \cdots & 0 \\
0 & 0 & 1 & \cdots & 0 \\
\end{bmatrix}

$$

Note that the estimate of positions of landmarks do not change due to the control.

The Jacobian of $$g(u_t, x_{t-1})$$ with respect to $$x_{t-1}$$ at $$\mu_{t-1}$$ is


$$

G*t =
\begin{bmatrix}
G_t^x & 0 \\
0 & I*{2n\times 2n}
\end{bmatrix}, \ \
G*t^x =
\begin{bmatrix}
1 & 0 & -\delta*{tran} \sin(\mu*{\theta*{t-1}}+\delta*{rot1}) \\
0 & 1 & \delta*{tran} \cos(\mu*{\theta*{t-1}}+\delta\_{rot1}) \\
0 & 0 & 1
\end{bmatrix}

$$

For the range-bearing sensor model, consider the range-bearing sensor model for each landmark $$i$$ (which returns the expected sensor reading),


$$

\hat{z}_t^i = h^i(\bar{\mu}\_t) =
\begin{bmatrix}
\sqrt{q} \\
\mathrm{atan2}(\delta_y, \delta_x) - \bar{\mu}_{t,\theta}
\end{bmatrix}

$$

where
$$\delta =
\begin{bmatrix}
    \delta_x \\
    \delta_y
\end{bmatrix} =
\begin{bmatrix}
\bar{\mu}_{m_j,x} - \bar{\mu}_{t,x} \\
\bar{\mu}_{m_j,y} - \bar{\mu}_{t,y}
\end{bmatrix},
q = \delta^T \delta
$$

and $$j$$ is the correspondence of the measurement $$i$$.

The Jacobian of $$h^i(x_t)$$ at $$\bar{\mu}_t$$ is

$$
H_t^i = \frac{1}{q}
\begin{bmatrix}
-\sqrt{q} \delta_x & -\sqrt{q} \delta_x & 0 & \sqrt{q} \delta_x & \sqrt{q} \delta_y \\
\delta_y & -\delta_x & -q & -\delta_y & \delta_x
\end{bmatrix}
F_{x,j}, \
H_t^i \in \mathbb{R}^{2\times (3+2n)}
$$

where

$$
F_{x,j} =
\begin{bmatrix}
1 & 0 & 0 & 0 & \cdots & 0 & 0 & \cdots & 0 \\
0 & 1 & 0 & 0 & \cdots & 0 & 0 & \cdots & 0 \\
0 & 0 & 1 & 0 & \cdots & 0 & 0 & \cdots & 0 \\
0 & 0 & 0 & 0 & \cdots & 1 & 0 & \cdots & 0 \\
0 & 0 & 0 & 0 & \cdots & 0 & 1 & \cdots & 0 \\
\end{bmatrix}, \
F_{x,j} \in \mathbb{R}^{5\times (3+2n)}
$$.

The non-zeros columns are at $$2j+2$$ and $$2j+3$$.

Now we can obtained the EKF algorithm for SLAM problem below by changing the motion function, observation function and the corresponding Jacobian in EKF for localization.

<div class="row justify-content-center">
    <div class="col">
        <div class="w-75 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/SLAM/EKF-slam/EKF-slam.png' | relative_url }}" alt=""/>
        </div>
    </div>
</div>
<div class="caption">
The Extended Kalman Filter for SLAM
</div>
<br/>

## Code Explanation

1. `[mu, sigma] = prediction_step(mu, sigma, u)`: Update the belief of the robot's pose based on the motion model. Implement line 1 to 4 in EKF SLAM Algorithm.

2. `[mu, sigma, observedLandmarks] = correction_step(mu, sigma, z, observedLandmarks)`: Updates the belief of the robot's pose and landmark positions according to the sensor model. Implement line 6 to 19 in EKF SLAM Algorithm.

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/probabilitics-robotics/tree/master/EKF_SLAM" %}
</div>

### References

1. **Robot Mapping**
   University of Freiburg, WS 2013/14, [http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/)

2. **Probabilistic Robotics**
   S. Thrun, W. Burgard, and D. Fox. MIT Press, Cambridge, Mass., (2005)
$$
