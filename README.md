# MPC-Project-SDC-Term2-P5-Udacity
Last project of the second term of the Self Driving Car Nanodegree Program by Udacity

---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.
  
## Start the simulator
Open the simulator and select Project 5: MPC controller.
![P5_simulator](./images/P5_simulator.PNG)

## Video Example
  [YouTube video](https://youtu.be/k1vFmap3mXk)
  
## State, actuators and update
  The model used a kinematic model (not considering the way tyres and road interact).
`
x[t] = x[t-1] + v[t-1] * cos(psi[t-1]) * dt  
y[t] = y[t-1] + v[t-1] * sin(psi[t-1]) * dt  
psi[t] = psi[t-1] + v[t-1] / Lf * delta[t-1] * dt  
v[t] = v[t-1] + a[t-1] * dt  
cte[t] = f(x[t-1]) - y[t-1] + v[t-1] * sin(epsi[t-1]) * dt  
epsi[t] = psi[t] - psides[t-1] + v[t-1] * delta[t-1] / Lf * dt  
`

The state vector is given by:
- `x` and `y` position
- `psi` heading direction
- `v` velocity
- `cte` cross-track error
- `epsi` orientation error

My actuator vector : `[delta, a]`
`Delta` is the steering angle and `a` is the acceleration.

The model update equation is:
![model update equaion](./images/Update_MPC.PNG)
The solver uses the initial state to feed the model the constraints and the cost to return a vector that minimize the cost function.

## N e dt
  I choose `N`equals 12 and `dt` equals 0.08. I found that the model worked pretty well using these numbers. Previously, I tried decreasing `dt`, but the model was overfitting and the car was moving a lot. I, then, incremented `dt` to 0.2 but the car was not reacting fast enough and went off road. Regarding `N`, I found that a lower numbers didn't work fine.  
## Polynomial fitting and MPC preprocessing
  The waypoints provided by the simulator are transformed to the car coordinates in the `main.cpp` file (lines 104-113). Then a 3rd-degree polynomial is fitted to the transformed waypoints (line 116). The polynomial coefficients are used to calculate `cte` and `epsi` (lines 118-122). They are used by `mpc.Solve` to create a reference trajectory and the predicted trajectory (lines 142-173).
    
## Latency
A contributing factor to latency is actuator dynamics. For example the time elapsed between when you command a steering angle to when that angle is actually achieved. This could easily be modeled by a simple dynamic system and incorporated into the vehicle model. One approach would be running a simulation using the vehicle model starting from the current state for the duration of the latency. The resulting state from the simulation is the new initial state for MPC.
This was the approach I used. The code can be found in the `main.cpp` file (lines 127-140).
