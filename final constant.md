## Task 3: Violations
 
| ID | Violation values | What went wrong | How we know it is violated |
|----|------------------|-----------------|----------------------------|
| C1 | `Train_Present = TRUE`, `Barrier_Open = TRUE` | The barrier stayed open while the train was on the crossing. | The "if" part is true but `¬Barrier_Open` is false. |
| C2 | `Train_Approaching = TRUE`, `Warning_On = TRUE`, `Alarm_On = FALSE` | The light is on but the alarm did not sound. | The rule needs both, and `Alarm_On` is false. |
| C3 | `Train_Present = TRUE`, `Barrier_Closed = FALSE` | The barrier jammed halfway as the train arrived. | Train is present but the barrier is not fully closed. |
| C4 | `Barrier_Opening = TRUE`, `Train_Present = TRUE`, `Train_Cleared = FALSE` | The barrier started opening while rear carriages were still on the crossing. | Opening started without confirmed clearance. |
| C5 | `Train_Approaching = TRUE`, `Road_Red = FALSE` | The road signal stayed green as the train approached. | Train is approaching but the signal is not red. |
| C6 | `Sensor_Fault = TRUE`, `Barrier_Closed = FALSE`, `Warning_On = FALSE`, `Operator_Alert = FALSE` | A sensor failed and the system did nothing. | Fault is true but none of the safe-mode outputs are on. |
 
