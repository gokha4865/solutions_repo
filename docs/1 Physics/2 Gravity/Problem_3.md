# Problem 3
# Trajectories of a Freely Released Payload Near Earth

## Introduction

Understanding the motion of objects under the influence of Earth's gravity is fundamental to space exploration. When a payload is released from a moving rocket near Earth, its subsequent trajectory is governed by its initial conditions (position and velocity at the time of release) and the gravitational force exerted by Earth. This problem explores the different types of trajectories possible and provides a computational tool to simulate and visualize these paths.

---

## Theoretical Background

The motion of the payload is primarily governed by **Newton's Law of Universal Gravitation**, which states that every particle in the universe attracts every other particle with a force that is directly proportional to the product of their masses and inversely proportional to the square of the distance between their centers. Mathematically, the force $F$ exerted by Earth (mass $M$) on the payload (mass $m$) is given by:

$$
F = -G \frac{Mm}{r^2} \hat{r}
$$

where:
- $G$ is the universal gravitational constant ($6.674 \times 10^{-11} \,{N·m}^2/\text{kg}^2$),
- $M$ is the mass of Earth ($5.972 \times 10^{24} \, \text{kg}$),
- $m$ is the mass of the payload,
- $r$ is the distance between the center of Earth and the payload,
- $\hat{r}$ is the unit vector pointing from the center of Earth to the payload.

From **Newton's second law of motion** ($F = ma$), the acceleration $a$ of the payload is:

$$
a = \frac{F}{m} = -G \frac{M}{r^2} \hat{r}
$$

This acceleration is always directed towards the center of Earth.

---

## Types of Trajectories

The type of trajectory the payload will follow depends on its **total mechanical energy**, which is the sum of its kinetic energy ($K$) and potential energy ($U$). The gravitational potential energy of the payload at a distance $r$ from the center of Earth is:

$$
U = -G \frac{Mm}{r}
$$

The kinetic energy of the payload with velocity $v$ is:

$$
K = \frac{1}{2} mv^2
$$

The total energy $E$ is:

$$
E = K + U = \frac{1}{2} mv^2 - G \frac{Mm}{r}
$$

The shape of the trajectory is determined by the sign of the total energy:

1. **Elliptical Trajectory ($E < 0$):**  
   If the total energy is negative, the payload is bound to Earth and will follow an elliptical orbit. A circle is a special case of an ellipse where the eccentricity is zero. This is typical for payloads that are intended to orbit Earth.

2. **Parabolic Trajectory ($E = 0$):**  
   If the total energy is exactly zero, the payload will follow a parabolic trajectory. This is the minimum energy required for the payload to escape Earth's gravitational pull and never return. The velocity at which this occurs at a given distance $r$ is the **escape velocity**:
   
   $$
   v_{\text{esc}} = \sqrt{\frac{2GM}{r}}
   $$

3. **Hyperbolic Trajectory ($E > 0$):**  
   If the total energy is positive, the payload has more than enough energy to escape Earth's gravity and will follow a hyperbolic trajectory.

---

## Numerical Simulation

To simulate the motion of the payload, we can use numerical methods to solve the equations of motion. We will use a simple **Euler method** for this demonstration, but more accurate methods like the **Runge-Kutta method** can be used for higher precision.

Let $\mathbf{r} = (x, y)$ be the position vector of the payload and $\mathbf{v} = (v_x, v_y)$ be its velocity vector in a 2D plane (we can extend this to 3D if needed). The acceleration vector $\mathbf{a} = (a_x, a_y)$ is given by:

$$
a_x = -GM \frac{x}{(x^2 + y^2)^{3/2}}
$$

$$
a_y = -GM \frac{y}{(x^2 + y^2)^{3/2}}
$$

The **Euler method** updates the position and velocity at each time step $\Delta t$ as follows:

$$
\mathbf{v}_{i+1} = \mathbf{v}_i + \mathbf{a}_i \Delta t
$$

$$
\mathbf{r}_{i+1} = \mathbf{r}_i + \mathbf{v}_i \Delta t
$$

---

This simulation allows us to visualize the trajectory of the payload under different initial conditions and understand the influence of Earth's gravity on its motion.

```python
import numpy as np
import matplotlib.pyplot as plt

# Constants
G = 6.674e-11  # Gravitational constant
M_earth = 5.972e24  # Mass of Earth
R_earth = 6371e3  # Radius of Earth

def simulate_trajectory(initial_position, initial_velocity, time_steps, dt):
    """
    Simulates the trajectory of a payload under Earth's gravity.

    Args:
        initial_position (np.array): Initial position vector [x, y] in meters.
        initial_velocity (np.array): Initial velocity vector [vx, vy] in m/s.
        time_steps (int): Number of time steps to simulate.
        dt (float): Time step size in seconds.

    Returns:
        tuple: Arrays of x and y positions of the payload over time.
    """
    position = np.array(initial_position, dtype=float)
    velocity = np.array(initial_velocity, dtype=float)
    history = [position.copy()]

    for _ in range(time_steps):
        r = np.linalg.norm(position)
        if r < R_earth:
            print("Payload crashed into Earth!")
            break

        # Calculate acceleration
        acceleration = -G * M_earth * position / (r**3)

        # Update velocity and position using Euler method
        velocity += acceleration * dt
        position += velocity * dt
        history.append(position.copy())

    history = np.array(history)
    return history[:, 0], history[:, 1]

if __name__ == "__main__":
    # Example initial conditions (altitude above Earth's surface)
    altitude = 500e3  # 500 km
    initial_distance = R_earth + altitude

    # Case 1: Circular Orbit (approximately)
    orbital_speed = np.sqrt(G * M_earth / initial_distance)
    initial_position_circular = [initial_distance, 0]
    initial_velocity_circular = [0, orbital_speed * 0.9]  # Slightly lower for elliptical
    time_steps_circular = 5000
    dt_circular = 10

    x_circular, y_circular = simulate_trajectory(initial_position_circular, initial_velocity_circular, time_steps_circular, dt_circular)

    # Case 2: Escape Trajectory
    escape_speed = np.sqrt(2 * G * M_earth / initial_distance)
    initial_position_escape = [initial_distance, 0]
    initial_velocity_escape = [0, escape_speed * 1.1]
    time_steps_escape = 5000
    dt_escape = 10

    x_escape, y_escape = simulate_trajectory(initial_position_escape, initial_velocity_escape, time_steps_escape, dt_escape)

    # Case 3: Reentry Trajectory
    initial_position_reentry = [initial_distance, 0]
    initial_velocity_reentry = [0, orbital_speed * 0.7]
    time_steps_reentry = 5000
    dt_reentry = 10

    x_reentry, y_reentry = simulate_trajectory(initial_position_reentry, initial_velocity_reentry, time_steps_reentry, dt_reentry)

    # Plotting
    plt.figure(figsize=(8, 8))
    plt.plot(x_circular, y_circular, label="Elliptical/Orbital Trajectory")
    plt.plot(x_escape, y_escape, label="Escape Trajectory")
    plt.plot(x_reentry, y_reentry, label="Reentry Trajectory")
    plt.scatter(0, 0, color='blue', s=500, label="Earth")
    earth_circle = plt.Circle((0, 0), R_earth, color='blue', fill=False)
    plt.gca().add_patch(earth_circle)
    plt.xlabel("X (m)")
    plt.ylabel("Y (m)")
    plt.title("Payload Trajectories Near Earth")
    plt.legend()
    plt.grid(True)
    plt.axis('equal')
    plt.show()
```


![alt text](Figure_31.png)

## Analysis and Discussion

The simulation demonstrates different types of trajectories based on the initial velocity of the payload.

### Orbital Insertion:
For a payload to be inserted into a stable orbit around Earth, its initial velocity at a certain altitude must be carefully chosen. The velocity should be high enough to counteract gravity and prevent the payload from falling back to Earth, but not so high that it escapes Earth's gravitational pull. The simulation with `initial_velocity_circular` shows an elliptical trajectory, which, with fine-tuning of the initial velocity, could become a circular orbit. Orbital insertion typically involves achieving a specific velocity vector at a desired altitude.

### Reentry:
A reentry trajectory is one where the payload is directed back towards Earth's atmosphere. This occurs when the payload's velocity is lower than the orbital velocity at its altitude, causing it to lose altitude over time due to gravity. The `initial_velocity_reentry` case illustrates this, where the payload's elliptical orbit intersects with Earth.

### Escape:
An escape trajectory is achieved when the payload's initial velocity is equal to or greater than the escape velocity at its initial position. In this case, the payload will have enough kinetic energy to overcome Earth's gravitational potential energy and move away from Earth indefinitely (following a parabolic or hyperbolic path). The `initial_velocity_escape` simulation shows the payload moving away from Earth.

---

## Graphical Representations

The Python script generates a plot showing the three different types of trajectories simulated:

- **Elliptical/Orbital Trajectory:** A closed path around Earth.
- **Escape Trajectory:** An open path where the payload moves away from Earth.
- **Reentry Trajectory:** A path that intersects with Earth.

The plot also includes a representation of Earth to scale.

---

## Further Considerations and Improvements

1. **More Accurate Numerical Methods:**  
   The Euler method is simple but can be inaccurate for long simulations or large time steps. Using methods like the **Runge-Kutta 4th order method** would provide more accurate results.

2. **3D Simulation:**  
   For more realistic scenarios, the simulation should be extended to three dimensions.

3. **Atmospheric Drag:**  
   In the lower altitudes, atmospheric drag plays a significant role in the trajectory of a payload, causing it to slow down and eventually burn up or land. This effect is not included in the current simulation.

4. **Perturbations:**  
   The gravitational forces of other celestial bodies (like the Moon and the Sun) can also affect the payload's trajectory over long periods. These perturbations are not considered here.

5. **Visualization:**  
   More sophisticated visualization tools can be used to create animations of the payload's motion.

---

## Conclusion

The trajectory of a payload released near Earth is highly dependent on its initial conditions, particularly its velocity. By understanding the relationship between the total energy and the type of trajectory, we can plan missions for orbital insertion, reentry, or escape. Numerical simulations provide a powerful tool for predicting and visualizing these trajectories, aiding in the design and execution of space missions. The provided Python script offers a basic framework for simulating these scenarios, which can be further refined to include more complex physical phenomena for increased accuracy.

