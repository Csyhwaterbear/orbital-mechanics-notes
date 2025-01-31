# Impulsive Maneuvers

In this chapter, we will consider general maneuvers that can change the orbit of a spacecraft. All orbital changes require application of an external force to modify one or more of the orbital elements. This external force most commonly comes from an on-board propulsion system, although it can also come from gravitational interactions with other celestial bodies (so-called slingshot maneuvers).

In this chapter, we are going to focus on maneuvers using on-board propulsion systems. In this case, the force is called **thrust**. In general, we will not be concerned with the source of the thrust, except insofar as it determines the type of maneuver.

We will consider two types of maneuvers:

1. Impulsive maneuvers
2. Non-impulsive maneuvers

In the case of an **impulsive maneuver**, the application of the force is very short relative to the time scale of the coasting period of the maneuver. Therefore, we can assume that the velocity vector changes instantaneously, either magnitude, direction, or both. This also means that the position of the spacecraft does not change during the maneuver.

The advantage of this approach is that we can neglect the additional force term in the equations of motion and treat the transfer orbit exactly the same as we did in previous chapters. The disadvantage is that the magnitude of the thrust must be rather large to accomplish the required energy change in a short time interval.

In the case of a **non-impulsive maneuver**, by contrast, the force is applied over a significant portion of the transfer. As a result, the magnitude of the thrust can be much lower to accomplish the same energy change. The disadvantage is that we must solve the equations of motion including the thrust term, resulting in a more complicated system of equations to solve.

## Change of Velocity

We will now focus on impulsive maneuvers. For impulsive maneuvers, the relevant metric we are interested in is the change of velocity of the spacecraft, $\Delta \vector{v}$. Notice that the velocity change is a vector—we can change the magnitude of the velocity (a pumping maneuver) or the direction of the velocity (a cranking maneuver).

The on-board propulsion system is nearly always a reaction engine, operating on Newton's Third Law. That is, the propulsion system accelerates some mass to a high velocity and ejects it away from the spacecraft. The resulting momentum transfer causes a reaction force on the spacecraft, changing its momentum.

The magnitude of the velocity change is related to the consumed propellent by the ideal rocket equation:

:::{math}
:label: eq:rocket-thrust-equation
\frac{\Delta m}{m} = 1 - e^{\frac{-\Delta v}{I_{sp}g_0}}
:::

where $\Delta m$ is the mass of the consumed propellent, $\Delta v$ is the magnitude of $\Delta \vector{v}$, $g_0$ is the standard sea-level gravitational acceleration, and $I_{sp}$ is the [**specific impulse**](https://en.wikipedia.org/wiki/Specific_impulse).

### Specific Impulse

Specific impulse is used as a performance metric for reaction engines. It is defined as the thrust generated by the propulsion device divided by the sea-level weight consumption rate of the fuel:

:::{math}
:label: eq:specific-impulse
I_{sp} = \frac{\text{Thrust}}{\text{Sea-level weight rate of fuel consumption}}
:::

Therefore, $I_{sp}$ has units of seconds. Higher values of $I_{sp}$ are better, because it means we are either getting more thrust per unit fuel consumed per unit time (the top gets bigger and the bottom is constant) or we are getting the same thrust at a lower fuel consumption rate (the top is constant and the bottom gets bigger).

Although $I_{sp}$ is a measure of the efficiency of the engine in converting fuel chemical energy into thrust, it is not the only performance metric of interest for rocket engines. In practice, engines with high specific impulse tend to have very low thrust. This makes them poorly suited for applications that require a large acceleration, such lift-off from the surface of the earth.

## Total Velocity Change

As suggested by Eq. {eq}`eq:rocket-thrust-equation`, the required propellant mass when using an engine with a given $I_{sp}$ is exponentially proportional to the change of velocity. The $\Delta v$ in Eq. {eq}`eq:rocket-thrust-equation` must be the magnitude of the *total* velocity change for the maneuver.

Some maneuvers require multiple impulse events to accomplish the goal. The total $\Delta v$ is the sum of the absolute value of the change in velocity required for each impulse event. This means that speeding up or slowing down or changing the direction of the spacecraft all required some $\Delta v$ and some propellant to accomplish.

For a single impulse, the change in velocity is given by the vector difference between the velocity before the impulse and after the impulse:

:::{math}
:label: eq:single-impulse-delta-v
\Delta \vector{v} = \vector{v}_2 - \vector{v}_1
:::

where $\vector{v}_2$ is the velocity after the impulse and $\vector{v}_1$ is the velocity before the impulse.

If multiple impulses are required to achieve a particular orbital change, then the magnitude of $\Delta \vector{v}$ from each impulse, computed by Eq. {eq}`eq:single-impulse-delta-v`, must be added together for use in Eq. {eq}`eq:rocket-thrust-equation`.

Due to the direct relationship between the required velocity change to perform a maneuver and the required propellant, maneuvers are often quoted by their $\Delta v$. All other things being equal, smaller $\Delta v$ is better.
