# RL_Control

## Overview
The code breaks down into 4 main components. 
The first is the agent that is responsible for the learning and converting sensor data into actionable Twist commands.
This is done using a Q Learing model that reduces state from a (wheeled-odometry, desired-odometry,) tuple-pair to 5 functions in the 2D implementation:
- Error along desired axis
- Error off desired axis
- Heading Error
- Linear Velocity Error
- Angular Velocity Error

Then using a transition model, the "optimal" Q can be found by iterating through Twist (just linear x, angular z in 2D case) to find the best resultng Q.
The transition model makes up the last 3 components (roughly ROS nodes) that make up the code.

The first of these 3 is the odometry (usually wheeled) state transition model. 
The node maintains its best estimate of a transition of the form f(state0, action) => state1.
The states are the data from odometry (wheeled or otherwise) that comprise the best estimate of the robot's actual position.
The model is broken into two components: the deterministic estimate of the robot's change in position, and an error term of the same dimensions.
The two of these combine to create a stochastic transition function that best represent's the robot's estimate of how it will move.
The function/model is used to estimate the new location of the robot for a given Twist.

The next node is the desired state transition model.
This node maintains its best estiamte of a transition f(des_state0) => des_state1.
The states are of the Odometry data type, and are published by a higher level planner node to "lead" the lower level controller by a carrot on a stick model.
The model is again broken into two components: the deterministic estimate and error term.
These two combine to create a general estimate of what the control is going to ask for in a future step.
The function/model is used to estimate the new location of the desired state for a given initial state.

Collectively, these two nodes represent the tuple-state transition function f(state, des_state, twist) => (state, des_state) that is used to estimate the values in the learning model.

The last node is auxiliary, and does not have a direct effect on the learning model (yet). 
Instead, what it does is track the actual states that occur, in one stochastic component.
This is used to check the deterministic transition model, because if the mean of the non-adjusted model starts to depart from the trantision model, then the model needs to be adjusted.

## Versions
- v0.1: Working implementation of odom_Transition (deterministic + error), desired_Transition
- v0.2: RL based off of those two components
- v0.3: Integrate 
- v1: Working RL w/ all 4 components. State space constrained to 2D wheeled robotics. Motion model is defined by an estimated deterministic transition function
- v2: State space in 3D.

## Reinforcement Learning
The learning agent attempts to learn the best action for every tuple-state that will minize the Error function (usually demonstrated as maximizing Reward Function, I will generally refer to it as the reward function). It does this by calculating and updating a Q-function that is an estimate of the reward (min error in this case) based on a linear combination of the functions on the tuple-state listed above: Error along desired axis, Error off desired axis, Heading Error, Linear Velocity Error, Angular Velocity Error. In a theoretical case, the agent "knows" the reward function, because it knows that it is the linear combination of the error functions, but in practice, the desired state will change unpredictably, and the robot will not transition as expected, so the projected future reward should be close to but not necessarily the same as the way it is calculated in hindsight.

### Q Learning - Updating the Reward Function estimate
Coming Soon

### Taking Action - Given a Q function, find the best action
Once the robot has identified the best Q function that it can find (in terms of fidelity to the real error), the robot needs to take action. In a finite space, it would check every possible action and then choose the one that had the highest resulting Q value (or in this case, the highest probability weighted Q-value because of the stochastic transition). In the continuous space of possible Twist commands to give, the maximum Q-value is chosen by first heuristically picking an estimated best Twist, and then using gradient ascent (not exactly, but that's what it is in theory) to ascend from there, finding ever higher Q values iteratively until it can't improve, and it takes that Q value to be it's maximum Q value, and the action that generated it is considered the best action, which is then outputed as the decision. 

There are two things to note:
First, the transition is stochastic, so the agent actually queries the transition models for 10 possible options, which are assumed to be normally distributed, and then uses the average Q value for those options to estimate the probability weighted Q over the entired distribution of possible outcomes for that estimated transition. 
Second, the "gradient ascent" is going to be implemented by iterating in the Twist space (linear velocity x and angular velocity z in the 2D case) to single-variable optimums, and then optimizing the next variable, and looping through the Twist variables until the estimated Q value cannot be improved. This will also have to be capped at a certain number of iterations, because with a stochastic transition model, the Q values could come out as some form of alternating cycle. In the end, it should still be the agent's best approximation of the maximum probability weighted Q value for a given action.

## Location-Twist/Odom Transition Model
Coming Soon

## Desired State Transition Model
Coming Soon

## Independent/Audit Location Transition Model
Coming Soon