# ⏱ Real-Time Scheduling Simulator: CUS / TBS

##  Project Summary
This project implements two real-time scheduling servers — **CUS (Constant Utilization Server)** and **TBS (Total Bandwidth Server)** — to handle **periodic and aperiodic tasks**.  
It simulates task arrivals, ready-queue management, EDF-based execution, deadline handling, and produces statistics such as missed jobs, response time, and utilization.

---

## 1) Task Model & Input

### Task Types
- **PeriodTask**: Periodic tasks, releasing jobs at fixed intervals.  
- **AperiodTask**: One-time tasks, served by CUS/TBS.  

### Common Fields
- `Arrive_time`: Arrival time of the job  
- `Period`: Period of task (for periodic tasks)  
- `Execution_time (C)`: Required computation time (WCET)  
- `Absolute_deadline (D)`: Deadline (release + period, or assigned by server)  
- `TID`: Task ID  

---

## 2) Core Data Structures
- **Ready Queue**: Implemented as a `deque` (linked list).  
  Sorted by **EDF** (Earliest Deadline First).  
- **Clock**: Discrete simulation clock.  
- **Statistics**:  
  - `MissPJobNumber` – missed periodic jobs  
  - `FinishedPJobNumber` – finished periodic jobs  
  - `FinishedAJobNumber` – finished aperiodic jobs  
  - `Average_Response_Time` – mean response time of aperiodics  

---

## 3) Simulation Workflow

1. **Read Task File** → Load periodic and aperiodic tasks.  
2. **Initialize Ready Queue** → Store available jobs.  
3. **Main Loop (`while clock < MaxSimTime`)**:  
   1. Check deadlines → mark/remove missed jobs.  
   2. Release new periodic jobs.  
   3. Handle new aperiodic arrivals → assign deadlines (CUS/TBS).  
   4. Sort ready queue by **EDF priority**.  
   5. Execute the job at head for 1 time unit.  
   6. Update statistics and system clock.  

---

## 4) CUS (Constant Utilization Server)

### Goal
Guarantee a **constant utilization bound** for aperiodic jobs while keeping periodic jobs schedulable.

### Deadline Assignment
When an aperiodic job `J_i` (exec time `C_i`) arrives at time `t`:  
\[
D_i = \max(t, D_{i-1}) + \frac{C_i}{U_s}
\]  
where `U_s` is the server utilization.  

### Pseudocode
```text
cus_deadline = 0
while clock < MaxSimTime:
    remove_missed_periodic_jobs()
    release_periodic_jobs(clock)
    for each arriving aperiodic:
        cus_deadline = max(clock, cus_deadline) + C_i / U_s
        job.D = cus_deadline
        push_ready(job)

    sort_ready_by_EDF()
    run_one_time_unit_and_update()
    clock += 1
