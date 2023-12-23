---
title: "Kalman Filters on Orbit Trajectory"
layout: single
excerpt: "Linear, Extended, and Unscented KF to estimate spacecraft state in presence of noise"
header:
   teaser: ../assets/images/project_KF_logo.png

---

This was a team project where we were tasked with analyzing the performance of the proposed autonomous-navigation system and ultimately determine whether it is suitable for the company’s mission to Bennu. We did so using numerical simulations of (i) the spacecraft orbital motion about the asteroid, (ii) landmark-based line-of-sight measurements, and (iii) the navigation filter itself.

<img src="/assets/images/project_KF_validate.png" alt= "Visualization Tool for Checking Landmark Visibility">


We utilized a pinhole-camera model to convert pixel locations on a focal plane to extract the focal point location, which we assumed was at the center of the spacecraft. 

<img src="/assets/images/project_KF_camera.png" alt= "Simplified 2D pinhole camera model to represent measurement model and observation geometry">


Using both non-linear and linearized two-body gravity models, we propagated perturbations and created trajectorys to estimate the spacecraft state at certain timesteps. We utilized both Linear, Extended, and Unscented kalman filters to update the correction using the measured information. 

Given the high non-linearity of orbit trajectories, the linear KF proved to be unstable when measurement and process noise was introduced into the system. However, the extended KF and unscented KF utilized a nominal trajectory that was updated according to each step's new state estimate. This allowed the EKF and UKF to track and estimate the actual trajectory with minimal error.

<img src="/assets/images/project_KF_errors.png" alt= "EKF estimate errors when estimating Spacecraft state in presence of process and measurement noise">

In the presence of measurement and process noise, Q tuning was performed to adjust the error uncertainy plus\minus 2-sigma bounds to capture the state estimate very well. Normalized Error Estimation Squared (NEES) and Normalized Innovation Squared (NIS) testing was additionally performed to ensure the filters were consistent and producing quality estimates.

<img src="/assets/images/project_KF_NEESNIS.png" alt= "EKF NEES and NIS results. The jagged bounds of the NIS plot is due to the varying number of measurements (i.e. visible landmarks) as the asteroid rotates and as the Spacecraft orbits.">
