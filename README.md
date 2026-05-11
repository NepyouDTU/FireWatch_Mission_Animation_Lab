# Overall theory used and assumptions built upon

This model estimates the total electrical power needed for flight by adding together the main power loads on the drone:

$P_\text{total} = P_\text{hover}(1-\text{lift gain}) + P_\text{parasitic} + P_\text{sensor}$

The estimated flight time is then calculated from the available battery energy divided by the total power consumption, assuming a perfect system:

$t_\text{flight} = \frac{E_\text{battery}}{P_\text{total}}$

Battery energy is converted directly from watt-hours to joules, ignoring losses during conversion:

$E_\text{battery} = \text{Wh} \times 3600$

## Hover power and lift gain

The term $P_\text{hover}(1-\text{lift gain})$ represents the effective power required to keep the drone airborne.

$P_\text{hover}$ is the baseline power needed to generate enough thrust to balance the drone’s weight in hover. The factor $(1-\text{lift gain})$ accounts for the fact that forward motion can reduce the amount of rotor power needed. For example, if aerodynamic lift or improved airflow provides a lift gain of $20\%$, then only $80\%$ of the hover power is required:

$P_\text{effective hover} = P_\text{hover}(1-0.20)$

This assumption makes sense because many flying systems become more efficient in forward flight than in pure hover. This is especially the case for drones, where forward momentum can reduce the need for the rotors to work in disturbed air.

## Parasitic power

The parasitic power is modeled as:

$P_\text{parasitic} = 150\left(\frac{v}{12}\right)^3$

where $v$ is the flight speed.

This cubic relationship is a common approximation for aerodynamic drag power. Drag force increases approximately with the square of speed:

$F_\text{drag} \propto v^2$

This is a simplified version of the general formula for parasitic drag: $D = \frac{1}{2}\rho v^2 C_D A$

Power is force multiplied by velocity:

$P = Fv$

so the power required to overcome drag scales approximately as:

$P_\text{drag} \propto v^2 \cdot v = v^3$

The constant $150$ means that at the reference speed $v = 12$, the parasitic power is $150\text{ W}$:

$P_\text{parasitic} = 150\left(\frac{12}{12}\right)^3 = 150\text{ W}$

Using $\left(\frac{v}{12}\right)^3$ makes the model easy to scale relative to this known reference point. If the speed doubles, parasitic power increases by a factor of $2^3 = 8$, which reflects how quickly aerodynamic losses grow at higher speeds.

## Sensor power

The sensor term $P_\text{sensor}$ represents the power consumed by onboard electronics, such as cameras, sensors, processors, or communication hardware.

This is added directly because it is an additional electrical load on the battery:

$P_\text{total} = P_\text{flight} + P_\text{electronics}$

Unlike parasitic drag, sensor power is assumed to be approximately constant with flight speed.

We made it variable, so it is easy to simulate the usage of different types of cameras, for example standard, thermal, and event cameras.

```python
# Quantitative Analysis for DJI M350 RTK Performance

def calculate_mission_impact(battery_wh, hover_p, lift_gain, speed_ms, sensor_p):
    """
    Calculates drone performance based on raw electrical and aerodynamic inputs.
    """
    # 1. Aerodynamic Power Model
    # Induced Power: Power to stay airborne, reduced by translational lift gain
    p_induced_moving = hover_p * (1 - lift_gain)
    
    # Parasitic Power: Calculated based on the cube of speed
    # Reference: 150W is an estimated drag for this drone class at 12 m/s
    p_parasitic = 150.0 * (speed_ms / 12.0)**3
    
    # Total System Power Draw
    total_p = p_induced_moving + p_parasitic + sensor_p
    
    # 2. Performance Metrics
    flight_time_mins = (battery_wh / total_p) * 60
    distance_km = (speed_ms * 3.6) * (flight_time_mins / 60)
    
    # Area Mapped: assuming 150 m swath width for wildfire thermal mapping
    area_km2 = distance_km * 0.150 
    
    return {
        "Total Power (W)": round(total_p, 2),
        "Flight Time (min)": round(flight_time_mins, 2),
        "Distance (km)": round(distance_km, 2),
        "Area (km2)": round(area_km2, 3)
    }
