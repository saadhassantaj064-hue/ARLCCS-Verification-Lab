## Task 2: Formal Expressions
 
| ID | Formula |
|----|---------|
| C1 | `Train_Present → ¬Barrier_Open` |
| C2 | `Train_Approaching → (Warning_On ∧ Alarm_On)` |
| C3 | `Train_Present → Barrier_Closed` |
| C4 | `Barrier_Opening → (Train_Cleared ∧ ¬Train_Present)` |
| C5 | `(Train_Approaching ∨ Train_Present) → Road_Red` |
| C6 | `Sensor_Fault → (Barrier_Closed ∧ Warning_On ∧ Operator_Alert)` |
 
A constraint `P → Q` is violated exactly when `P = TRUE` and `Q = FALSE`.
