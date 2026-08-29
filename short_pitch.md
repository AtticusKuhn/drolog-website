# Drolog

Hi, I'm Atticus, and along with my co-founder Adarsh, we're creating Drolog:
the semantic ontology layer for drone simulation.

Drolog translates between the low-level world modeled by simulators and the mission-level concepts used by autonomy teams. Drolog lets your simulator understand missions, not just coordinates.

As Cambridge computer scientists with many connections in the defense-tech industry, we have both
the technical skills and domain-knowledge experience to tackle what we've observed
to be the simulation-gap problem.

Currently, drone manufacturers are limited in their simulations, which are 
built upon general-purpose simulators like Gazebo, IsaacSim, UnrealEngine. 
The current simulators expose physics-level concepts such as distances
and objects, which makes it cumbersome to define RL policies. We've observed this in real-world workflows of drone simulations. They write one-off integration code, or, they hard-code threat zones and mission rules, or they rebuild the same abstractions for every simulator, or they model uncertainty today, but only with substantial engineering effort.

Drolog is a software layer that connects existing drone simulators to autonomy systems. It allows teams to model concepts such as missions, objectives, threat zones, and uncertain location directly, instead of rebuilding them from low-level coordinates, distances, and sensor data.

Most simulators describe the physical world: where a drone is, how fast it is moving, and what its sensors detect. Autonomous systems must answer higher-level questions: Is the objective still reachable? How confident is the drone in its location? What action remains safe if GPS is unavailable?

We've built Drolog (Drones + Prolog), which is the thin semantic layer that sits in-between
the simulator at the physics-level and the RL policy. Drolog defines high-level concepts
such as "Drone", "Mission", and "Objective" directly. 

Drolog bridges those two levels. As a simulation runs, it updates the drone’s modeled beliefs and their probabilities, allowing teams to test decision-making under changing and imperfect information.

Our killer feature: a native understanding of probabilities that is recomputed per-tick. This models how drones are uncertain about their position in a GPS-denied environment.

Drolog is not a simulator, not an autonomy stack, and does not replace your flight software. Even better: it only 
runs in simulation mode, not on a live-flying drone, so you can keep your preexisting workflow.

Example scenario: During a simulated mission, GPS becomes unreliable and the drone’s estimated position begins to drift. Drolog updates the system’s confidence in that position and allows the autonomy software to reason about whether the objective remains reachable and which routes remain safe.

Example Syntax:

```drolog
drone scout_1

zone gps_denied_area {
    shape: polygon([
        (52.2041, 0.1182),
        (52.2048, 0.1201),
        (52.2035, 0.1210)
    ])
}

# Simulator adapters provide these observations every tick.
observe gps_position(scout_1) from isaac_sim.gps
observe visual_odometry(scout_1) from isaac_sim.camera
observe inertial_motion(scout_1) from isaac_sim.imu

# Drolog maintains a distribution, not merely one coordinate.
belief position(scout_1) ~ fuse(
    gps_position(scout_1),
    visual_odometry(scout_1),
    inertial_motion(scout_1)
)

probability inside(scout_1, gps_denied_area) :=
    P(position(scout_1) in gps_denied_area)

probability position_accurate(scout_1, within: 5m) :=
    P(distance(position(scout_1), true_position(scout_1)) < 5m)

boolean gps_denied(scout_1) :=
    inside(scout_1, gps_denied_area) > 0.80
    or gps_signal_quality(scout_1) < 0.20

boolean position_trustworthy(scout_1) :=
    position_accurate(scout_1, within: 5m) >= 0.95
```

An RL environment consumes these directly:

```drolog
rl.observation "gps_denied" =
    gps_denied(scout_1)

rl.observation "position_confidence" =
    position_accurate(scout_1, within: 5m)

rl.action_allowed continue_to_objective =
    position_trustworthy(scout_1)
```



We have built a working prototype that works against IsaacSim and runs on GPU and are looking for a drone or defense organization to join us as a design partner. We will integrate Drolog with a bounded use case in your existing simulation environment at no cost.
We guarantee one bounded scenario in your existing environment, six weeks, no cost.

If you're doing drone simulation, and you're 
looking for any of these benefits
- reducing the custom code required for each simulation;
- making mission logic reusable across simulators;
- creating GPS-denied and uncertain-information scenarios more quickly;
- testing autonomy against mission outcomes rather than only physical behavior;
- making autonomous decisions easier to inspect or evaluate.

Contact us at contact@drolog.com.
