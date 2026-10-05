# Automated Railway Level-Crossing Control System (ARLCCS)
### Software Verification Lab: Constraints, Formalization, Violations

## 0. Signal Vocabulary

| Symbol | Meaning |
|---|---|
| `Train_Approaching` | Sensor reports a train in the approach zone |
| `Train_Present` | Train is occupying the crossing |
| `Train_Cleared` | Train has completely left the crossing (confirmed by the exit sensor) |
| `Barrier_Closed` | Barrier is fully down |
| `Barrier_Open` | Barrier is fully up |
| `Barrier_Opening` | Barrier is moving upward (open command issued) |
| `Barrier_Lowering` | Barrier is moving downward |
| `Barrier_Fault` | Barrier failure detected |
| `Warning_On` | Warning lights flashing |
| `Alarm_On` | Audible alarm sounding |
| `Road_Red` | Road traffic signal is red |
| `Sensor_Fault` | A train-detection sensor has failed |
| `Sensor_Conflict` | Sensors give contradictory readings |
| `Comm_Loss` | Communication with sensors, control center or train lost |
| `Emergency` | Emergency condition declared |
| `Operator_Alert` | Control-center operator notified |
| `Train_Stop_Signal` | Stop signal sent to the approaching train |
| `Train_Proceed` | Proceed (clear) signal given to the train |
| `Warning_Time` | Seconds the warning has been active |
| `T_min` | Minimum warning time before the barrier lowers (e.g. 5 s) |

---

## Task 1: Identify Constraints (12)

| ID | Constraint (simple English) | Why it is necessary |
|---|---|---|
| **C1** | The barrier must not be open while a train is present in the crossing. | Road vehicles could enter the crossing and collide with the train. |
| **C2** | When a train is approaching, the warning lights and alarm must be on. | Drivers and pedestrians need advance notice to stop or clear the crossing. |
| **C3** | While a train is present, the barrier must be fully closed. | Prevents any road entry during passage (stronger than C1: a half-moving barrier is also unsafe). |
| **C4** | The barrier may start opening only after the train is confirmed to have completely cleared the crossing. | Opening on a partial or assumed clearance (e.g. the train's tail is still on the crossing) can cause a collision. |
| **C5** | The road traffic signal must be red whenever a train is approaching or present. | Gives road users a second, redundant stop indication alongside the barrier. |
| **C6** | If a sensor fails, the system must go to the safe state: barrier closed, warnings on, operator alerted. | A failed sensor can no longer prove "no train", so the system must assume danger. |
| **C7** | If a barrier failure is detected, the operator must be alerted and a stop signal sent to the train. | A crossing that cannot be closed is unsafe; the train must be stopped before it arrives. |
| **C8** | If communication is lost, the system must go to the safe state and alert the operator. | Loss of communication means the system can no longer trust its picture of the track. |
| **C9** | If sensor readings conflict, the system must treat it as the worst case (train possibly present): barrier closed and operator alerted. | Incorrect readings must never lead to an "all clear" decision. |
| **C10** | In an emergency, the barrier must be closed, the alarm on, and the train stop signal sent. | Emergency conditions require the most restrictive safe configuration. |
| **C11** | The train may be given a proceed signal only if the barrier is fully closed and not faulty. | The train must never be told it can pass an unprotected crossing. |
| **C12** | The barrier must not start lowering until the warning has been active for at least `T_min` seconds. | Gives vehicles time to clear the crossing and prevents trapping them. |

---

ault) and still shows green. | `P = (T ∨ F) = T`, `Q = Road_Red = F`. |
| **C6** | `Sensor_Fault = T`, `Barrier_Closed = F`, `Warning_On = F`, `Operator_Alert = F` | The sensor fails silent, the system carries on as "no train", and nobody is told. This is a fail-**unsafe** design. | `P = T`; `Q = F ∧ F ∧ F = F`. Detected by the safety-monitoring unit comparing the fault flag with the output state. |
| **C7** | `Barrier_Fault = T`, `Operator_Alert = T`, `Train_Stop_Signal = F` | The operator was alerted, but no stop signal was sent to the approaching train, which will reach an unprotected crossing. | `P = T`; `Q = T ∧ F = F`. |
| **C8** | `Comm_Loss = T`, `Barrier_Closed = F` (barrier stays at its last known "open" state), `Warning_On = F` | The link to the sensors times out and the controller keeps using its stale last value ("no train"). | `P = T`; `Q = F ∧ F ∧ ? = F`. Detected via a heartbeat timeout combined with the barrier state. |
| **C9** | `Sensor_Conflict = T` (entry sensor = train, exit sensor = "cleared"), `Barrier_Closed = F`, `Operator_Alert = F` | The system "voted" for the sensor that says clear and opened the barrier, ignoring the contradiction. | `P = T`; `Q = F ∧ F = F`. Conflict is detected by comparing the redundant sensors. |
| **C10** | `Emergency = T`, `Barrier_Closed = T`, `Alarm_On = T`, `Train_Stop_Signal = F` | The emergency button closed the barrier and sounded the alarm, but the stop command to the train was dropped, so the train keeps coming. | `P = T`; `Q = T ∧ T ∧ F = F`. |
| **C11** | `Train_Proceed = T`, `Barrier_Closed = F` | The train is given a green aspect before the barrier has finished lowering (race condition between the signalling and barrier tasks). | `P = T`; `Q = Barrier_Closed ∧ ¬Barrier_Fault = F ∧ ? = F`. |
| **C12** | `Barrier_Lowering = T`, `Warning_Time = 1 s`, `T_min = 5 s` | The barrier started coming down 1 s after the lights started, trapping a car on the crossing. | `P = T`; `Q = (1 ≥ 5) = F`. Requires a timer, so it is checked with a **timed** monitor. |

---



**Design insight.** Constraints C6 to C10 all force a *fail-safe* state (barrier closed, warnings on, operator alerted). The safe default for a level crossing is "assume a train is coming", because an unnecessary closure is an inconvenience, while an unnecessary opening can be fatal.
