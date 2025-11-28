---
layout: page
title: Full SLAM with FastSLAM Algorithm
description: |
  This project implements FastSLAM for full SLAM problem. Unlike the online SLAM algorithms such as EKF that computes the posterior in the current time step, FastSLAM estimates the full path posterior and the locations of features given the range-bearing sensor reading and odometry information.
img: /assets/img/projects/SLAM/Fast-slam/fastslam.gif
importance: 3
category: SLAM
github: https://github.com/yangfan/probabilitics-robotics/tree/master/FastSLAM
images:
  - path:
    column:
    text:
not_empty: true
---

The FastSLAM algorithm implemented in this project solves the full SLAM problem. It uses particle filters for estimating the robot path while each particle (which is a hypothesis of robot pose) uses separate EKFs to estimate each map feature location, based on the conditional independence between any two disjoint sets of features in the map, given the robot pose. The particle in FastSLAM is denoted as

$$
\langle x_t^{[k]}, \mu_{1,t}^{[k]}, \Sigma_{1,t}^{[k]}, ..., \mu_{N,t}^{[k]}, \Sigma_{N,t}^{[k]} \rangle
$$

where $$k$$ is the index of the particle, $$\mu_{i,t}^{[k]}, \Sigma_{i,t}^{[k]}$$ refer to the mean and covariance of the i-th map feature.

The advantage of FastSLAM is

1. The data association decision can be made on a per-particle basis. The FastSLAM maintains multiple data associations, in contrast to EKF that tracks only one at any time step

2. The particle filter used in FastSLAM allows the algorithm to deal with the highly nonlinear motion models.
<div class="row justify-content-center">
    <div class="col">
      <div class="w-50 mx-auto" style="background-color: white;">
          <img class="img-fluid" src="{{ '/assets/img/projects/SLAM/Fast-slam/fastslam.gif' | relative_url }}" alt=""/>
      </div>
    </div>
</div>
<div class="caption">
Full SLAM by FastSLAM algorithm
</div>

The particles with the highest weight are in red color. The arrow refers to the current pose of the particle with the highest weight. All other particles are in gray color (some of paths may be disappear because of the resampling step). The blue crosses represent the ground-true position of the landmarks. The blue dots represent the estimated landmark positions of each particle. The ellipse is the error ellipse of estimated landmark positions of the particle with the highest weight.

## FastSLAM Overview

The full SLAM problem considered in this project can be written as a posterior:

$$
P(x_{0:t},m_{1:N}|z_{1:t},u_{1:t}) = P(x_{0:t}|z_{1:t},u_{1:t}) P(m_{1:N}|x_{0:t},z_{1:t})
$$.

Based on the conditionally independence of the landmarks given the poses, the latter posterior on the right hand side can be further written as


$$

\prod*{i=1}^{N} P(m_i|x*{0:t},z\_{1:t})

$$

Therefore the full SLAM posterior is equivalent to:


$$

P(x*{0:t},m*{1:N}|z*{1:t},u*{1:t}) = P(x*{0:t}|z*{1:t},u*{1:t})\prod*{i=1}^{N} P(m*i|x*{0:t},z\_{1:t})

$$

The path posterior can be represented by a set of the particles while each particle maintains separate map posterior by EKF, i.e.,


$$

\langle x*t^{[k]}, \mu*{1,t}^{[k]}, \Sigma*{1,t}^{[k]}, ..., \mu*{N,t}^{[k]}, \Sigma\_{N,t}^{[k]} \rangle

$$

The key steps of FastSLAM are following:

1. Prediction: Extend the path posterior by sampling a new pose for each particle, based on the motion model (odometry motion model in this project).


$$

    x_t^{[k]} \sim P(x_t|x_{t-1}^{[k]}, u_t), \ \forall k = 1, ..., N.
    $$

2. Measurement update: Update belief of each observed landmark by EKF, compute measurement predication by

   $$
   \hat{z}^{[k]} = h(\mu^{[k]}_{j,t-1}, x_t^{[k]})
   $$

3. Compute the particle weight by

   $$
   w^{[k]} = |2\pi Q|^{-\frac{1}{2}} \exp{(-\frac{1}{2}(z_t - \hat{z}^{[k]})^T Q^{-1} (z_t-\hat{z}^{[k]}))}
   $$

The detailed algorithm is presented in the pseudocode<sup>[2]</sup> below.

<div class="row justify-content-center">
    <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/SLAM/Fast-slam/fast-slam.png'| relative_url }}" alt=""/>
        </div>
    </div>
</div>
<div class="caption">
The FastSLAM 1.0 
</div>
<br/>

## Code Explanation

1. `sample_motion_model(odometry, particles)`: Sample a new pose for each particle by the odometry motion model. Implement line 3 to 4.

2. `measurement_model(particle, landmark)`: Compute the expected measurement for the given landmark and the Jacobian with respect to the landmark. Implement line 12 to 13.

3. `eval_sensor_model(sensor_data, particles)`: Update the estimate of the landmark position based on the range-bearing measurement<sup>[1]</sup> and compute the particle weight.

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/probabilitics-robotics/tree/master/FastSLAM" %}
</div>

### References

1. **Introduction to Mobile Robotics**  
   University of Freiburg, Spring, 2020, [http://ais.informatik.uni-freiburg.de/teaching/ss20/robotics/index_en.php](http://ais.informatik.uni-freiburg.de/teaching/ss20/robotics/index_en.php)

2. **Probabilistic Robotics**  
   S. Thrun, W. Burgard, and D. Fox. MIT Press, Cambridge, Mass., (2005)
