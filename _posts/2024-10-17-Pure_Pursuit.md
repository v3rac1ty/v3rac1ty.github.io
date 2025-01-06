---
layout: post
title: Pure Pursuit
date: 17-10-2024
categories: [Robotics]
tags: [robotics, algorithms, localization, control_systems, autonomous-navigation]
---
The **Pure Pursuit** algorithm is a popular and efficient path-following algorithm used in autonomous robotics. It enables the robot to follow a desired path by continuously steering toward a lookahead point, which is calculated based on a fixed lookahead distance from the robot’s current position. This allows the robot to follow a smooth curve and navigate through predefined paths, making it ideal for dynamic environments.

In this project, the **Illini VEX Robotics** team implemented the **Pure Pursuit Algorithm** to enable precise path-following for our robot during autonomous tasks in robotics competitions. This implementation is designed to be efficient and can run asynchronously, allowing the robot to perform other tasks while following the path.

## Key Features of the Algorithm

1. **Path Following**: The robot follows a predetermined path by continuously recalculating the lookahead point along the path.
2. **Curvature Control**: The robot calculates the curvature needed to steer towards the lookahead point, ensuring smooth turns and path-following.
3. **Asynchronous Execution**: The algorithm runs asynchronously, allowing the robot to perform other tasks while executing path-following.
4. **Real-Time Adjustments**: The robot constantly recalculates its lookahead point and adjusts its velocity and heading to stay on the path.

## C++ Code Implementation

Below is the C++ implementation of the **Pure Pursuit Algorithm**. The code includes functions for reading path data, finding the closest point to the robot on the path, calculating the lookahead point, and following the path.

### Key Code Snippets:

1. **Reading Path Data**: This function reads a series of waypoints from a file and stores them as poses (x, y, theta).

```cpp
// Function to read elements from a line, split by a delimiter
std::vector<std::string> readElement(const std::string& input, const std::string& delimiter) {
    // Code for splitting string into elements
}

// Get path data from file
std::vector<pursuit::Pose> getData(const asset& path) {
    // Code for reading path points from a file
}
```

2. **Finding the Closest Point**: This function calculates the closest point on the path to the robot’s current position.

```cpp
// Find the closest point on the path to the robot
int findClosest(pursuit::Pose pose, std::vector<pursuit::Pose> path) {
    // Code for finding the closest point
}
```

3. **Calculating the Lookahead Point**: This function calculates the "lookahead" point by finding the intersection between a circle (representing the robot’s future position) and the path.

```cpp
// Function to calculate intersection between a circle and a line
float circleIntersect(pursuit::Pose p1, pursuit::Pose p2, pursuit::Pose pose, float lookaheadDist) {
    // Code for calculating intersection
    return -1; // return value if no intersection found
}

// Returns the lookahead point
pursuit::Pose lookaheadPoint(pursuit::Pose lastLookahead, pursuit::Pose pose, std::vector<pursuit::Pose> path, int closest, float lookaheadDist) {
    // Code for calculating the lookahead point
    return lastLookahead; // return the updated lookahead
}
```

4. **Following the Path**: This function is the main driver for the Pure Pursuit algorithm. It calculates the necessary velocities based on the lookahead point and adjusts the robot’s movements accordingly.

```cpp
// Function to follow the path using the Pure Pursuit algorithm
void pursuit::Chassis::follow(const asset& path, float lookahead, int timeout, bool forwards, bool async) {
    if (async) {
        pros::Task task([&]() { follow(path, lookahead, timeout, forwards, false); });
        pros::delay(10); 
        return;
    }

    std::vector<pursuit::Pose> pathPoints = getData(path);
    if (pathPoints.size() == 0) {
        return;
    }

    pursuit::Pose pose = this->getPose(true);
    pursuit::Pose lastPose = pose;
    pursuit::Pose lookaheadPose(0, 0, 0);
    pursuit::Pose lastLookahead = pathPoints.at(0);
    lastLookahead.theta = 0;

    int closestPoint;
    for (int i = 0; i < timeout / 10 && this->runningPath; i++) {
        pose = this->getPose(true);
        if (!forwards) pose.theta -= M_PI;

        distTraveled += pose.distance(lastPose);
        lastPose = pose;

        closestPoint = findClosest(pose, pathPoints);
        if (pathPoints.at(closestPoint).theta == 0) break;

        lookaheadPose = lookaheadPoint(lastLookahead, pose, pathPoints, closestPoint, lookahead);
        lastLookahead = lookaheadPose;

        float curvatureHeading = M_PI / 2 - pose.theta;
        float curvature = findLookaheadCurvature(pose, curvatureHeading, lookaheadPose);

        float targetVel = pathPoints.at(closestPoint).theta;
        targetVel = slew(targetVel, prevVel, lateralSettings.slew);
        prevVel = targetVel;

        float targetLeftVel = targetVel * (2 + curvature * drivetrain.trackWidth) / 2;
        float targetRightVel = targetVel * (2 - curvature * drivetrain.trackWidth) / 2;

        float ratio = std::max(std::fabs(targetLeftVel), std::fabs(targetRightVel)) / 127;
        if (ratio > 1) {
            targetLeftVel /= ratio;
            targetRightVel /= ratio;
        }

        prevLeftVel = targetLeftVel;
        prevRightVel = targetRightVel;

        if (forwards) {
            drivetrain.leftMotors->move(targetLeftVel);
            drivetrain.rightMotors->move(targetRightVel);
        } else {
            drivetrain.leftMotors->move(-targetRightVel);
            drivetrain.rightMotors->move(-targetLeftVel);
        }

        pros::delay(10);
    }

    drivetrain.leftMotors->brake();
    drivetrain.rightMotors->brake();
    distTraveled = -1;
}
```

## Explanation of the Code

1. **Path Data**: The `getData` function reads the path points from an external source (e.g., an SD card) and stores them as `pursuit::Pose` objects, which contain the `x`, `y`, and `theta` (angle) values for each point on the path.

2. **Finding the Closest Point**: The `findClosest` function calculates the distance between the robot’s current position and each point on the path, returning the index of the closest point.

3. **Lookahead Point**: The `lookaheadPoint` function calculates the lookahead point by finding the intersection between a circle (representing the robot’s reach) and the path. This ensures that the robot always has a target point ahead of it, which it will steer towards.

4. **Curvature Calculation**: The `findLookaheadCurvature` function calculates the curvature needed to steer the robot towards the lookahead point. The curvature is based on the position of the robot and the lookahead point relative to the robot’s current heading.

5. **Motion Control**: The `follow` function adjusts the robot's velocity and direction in real-time to follow the path. It uses the lookahead point and the calculated curvature to determine the robot's steering and speed.

## Conclusion

The **Pure Pursuit Algorithm** is a highly effective method for autonomous path-following, ensuring smooth and efficient navigation for the robot. This implementation allows the **Illini VEX Robotics** team’s robot to autonomously follow complex trajectories with high precision. By using the lookahead point and adjusting its steering and speed dynamically, the robot can navigate obstacle courses and reach specific locations with minimal intervention.

This algorithm has been crucial for our team, enabling us to execute autonomous tasks with confidence and accuracy in competitions.

For more information on Pure Pursuit, check out the [Purdue SIGBots Wiki on Pure Pursuit](https://wiki.purduesigbots.com/software/control-algorithms/basic-pure-pursuit).
