# Chapter 7: PLC Programming

## Overview
This is where theory meets practice. After understanding hardware, configuration, simulation, and data types, you're now ready to write actual PLC programs. This chapter covers the fundamental building blocks of PLC programming—from understanding how a PLC executes your code to writing your first working control logic.

## Learning Objectives
* **Program Execution:** Understand the PLC scan cycle and why execution order matters.
* **Program Structure:** Learn how programs are organized in TIA Portal (OBs, Networks, Rungs).
* **Ladder Logic:** Master the most common PLC programming language used in industry.
* **Basic Instructions:** Use contacts, coils, and logic instructions to build real control systems.

## Standards for Programming
* **Readability:** Write clean, well-commented code that other engineers can understand.
* **Modularity:** Break complex logic into smaller, manageable networks.
* **Safety First:** Always consider fail-safe states in your logic design.

---

## 1. The PLC Scan Cycle

The scan cycle is the heartbeat of every PLC. Understanding it is **critical** for writing correct logic.

### Scan Cycle Phases

|
 Phase 
|
 Description 
|
 Duration 
|
|
:---
|
:---
|
:---
|
|
**
1. Input Scan
**
|
 PLC reads all physical inputs and stores values in memory (Process Image Input - PII) 
|
 ~µs 
|
|
**
2. Program Execution
**
|
 PLC executes your logic from top to bottom, left to right 
|
 ~ms 
|
|
**
3. Output Scan
**
|
 PLC writes results from memory to physical outputs (Process Image Output - PIQ) 
|
 ~µs 
|
|
**
4. Housekeeping
**
|
 Internal diagnostics, communication, watchdog timer 
|
 ~µs 
|

### Scan Cycle Diagram

```
┌─────────────────────────────────────────────────────────┐
│                      SCAN CYCLE                         │
│                                                         │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐           │
│   │  READ    │   │ EXECUTE  │   │  WRITE   │           │
│   │  INPUTS  │──▶│  PROGRAM │──▶│  OUTPUTS │──┐        │
│   └──────────┘   └──────────┘   └──────────┘  │        │
│        ▲                                      │        │
│        └──────────────────────────────────────┘        │
│                    (Repeat)                            │
└─────────────────────────────────────────────────────────┘
```
⚠️ Important: Inputs are only read ONCE per scan. If an input changes mid-scan, the PLC won't see it until the next scan cycle.

Typical Scan Times
PLC Type	Typical Scan Time
S7-1200	1–10 ms
S7-1500	< 1 ms
S7-300/400	5–50 ms

## 2. Program Structure in TIA Portal
Organization Blocks (OBs)
Block Type	Description	When It Runs
OB1	Main Program Cycle	Every scan cycle
OB100	Startup	Once at PLC startup
OB82	Diagnostic Error	On diagnostic interrupt
OB121	Programming Error	On programming error
📌 For Beginners: Focus on OB1 – this is where 95% of your code will live.

Program Hierarchy
```
OB1 (Main Program)
├── Network 1: Motor Control
│   ├── Rung 1: Start Logic
│   └── Rung 2: Stop Logic
├── Network 2: Indicator Lamps
└── Network 3: Safety Interlocks
```

## Networks and Rungs
Term	Description
Network	A logical grouping of related rungs (with title and comment)
Rung	A single line of ladder logic (one complete circuit)
3. Programming Languages (IEC 61131-3)
TIA Portal supports multiple languages. Choose based on your application:

Language	Abbreviation	Best For
Ladder Diagram	LAD	Discrete logic, electricians, beginners
Function Block Diagram	FBD	Visual logic flow, analog processing
Structured Control Language	SCL	Math, loops, complex algorithms
Statement List	STL	Legacy systems (deprecated in S7-1500)
Graph	GRAPH	Sequential processes, state machines
📌 Recommendation: Start with Ladder Diagram (LAD) – it's the industry standard and easiest to troubleshoot.

4. Ladder Logic Fundamentals
Ladder logic mimics electrical relay circuits. Current flows from left (power rail) to right (neutral rail).

Basic Ladder Structure
```
|                                                          |
|    [ ]─────────────────────────────────────────( )       |
|   Input                                       Output     |
|                                                          |
|  (Power Rail)                              (Neutral Rail)|
```

Contacts (Inputs)
Symbol	Name	Description	Behavior
─[ ]─	Normally Open (NO)	Passes power when TRUE	Closed when input is ON
─[/]─	Normally Closed (NC)	Passes power when FALSE	Open when input is ON
Coils (Outputs)
Symbol	Name	Description	Behavior
─( )─	Output Coil	Standard output	ON when rung is TRUE
─(S)─	Set Coil	Latching ON	Stays ON until Reset
─(R)─	Reset Coil	Latching OFF	Turns OFF and stays OFF
5. Logic Instructions
AND Logic (Series Connection)
Both conditions must be TRUE for output to be TRUE.

```
|                                                          |
|    [Start_PB]────[Safety_OK]────────────────(Motor)      |
|                                                          |
```
Truth Table:

Start_PB	Safety_OK	Motor
0	0	0
0	1	0
1	0	0
1	1	1
OR Logic (Parallel Connection)
Either condition being TRUE makes output TRUE.

```|                                                          |
|    [Local_Start]─────┬───────────────────────(Motor)     |
|                      │                                   |
|    [Remote_Start]────┘                                   |
|                                                          |
```
Truth Table:

Local_Start	Remote_Start	Motor
0	0	0
0	1	1
1	0	1
1	1	1
NOT Logic (Negation)
Use NC contact to invert logic.

```
|                                                          |
|    [/Stop_PB]────────────────────────────────(Motor)     |
|                                                          |
Motor runs when Stop_PB is NOT pressed.
```
## 6. Essential Programming Patterns
Pattern 1: Start-Stop Circuit (Without Seal-In)
```
|                                                          |
|    [Start_PB]────[/Stop_PB]──────────────────(Motor)     |
|                                                          |
⚠️ Problem: Motor only runs while Start button is held down.

Pattern 2: Seal-In Circuit (Latching/Holding)

|                                                          |
|    [Start_PB]────┬────[/Stop_PB]─────────────(Motor)     |
|                  │                                       |
|    [Motor]───────┘                                       |
|                                                          |
```

## How It Works:

Press Start → Motor turns ON
Motor contact closes → Creates parallel path
Release Start → Motor stays ON (sealed in)
Press Stop → Opens NC contact → Motor turns OFF
Pattern 3: Using Set/Reset Coils
text
|  Network 1: Start Logic                                  |
|    [Start_PB]────[/Stop_PB]──────────────────(S Motor)   |
|                                                          |
|  Network 2: Stop Logic                                   |
|    [Stop_PB]─────────────────────────────────(R Motor)   |
|                                                          |
📌 Best Practice: Use seal-in circuits over Set/Reset coils – they're easier to troubleshoot.

## 7. TIA Portal Tag Table for Examples
Create these tags in your PLC Tag Table:

Name	Data Type	Address	Comment
Start_PB	Bool	%I0.0	Start Pushbutton (NO)
Stop_PB	Bool	%I0.1	Stop Pushbutton (NC wired as NO)
Local_Start	Bool	%I0.2	Local Panel Start
Remote_Start	Bool	%I0.3	Remote/HMI Start
Safety_OK	Bool	%I0.4	Safety Circuit Healthy
Motor	Bool	%Q0.0	Motor Contactor Output
Motor_Running	Bool	%Q0.1	Motor Running Indicator
Motor_Stopped	Bool	%Q0.2	Motor Stopped Indicator
8. Complete Programming Example: Motor Control
Requirements
Start motor with Start_PB
Stop motor with Stop_PB
Motor_Running lamp ON when motor is running
Motor_Stopped lamp ON when motor is stopped
Safety_OK must be TRUE for motor to run

Ladder Logic Solution
```
|  Network 1: Motor Control with Safety                    |
|  Title: Main Motor Start/Stop Logic                      |
|                                                          |
|    [Start_PB]────┬────[/Stop_PB]────[Safety_OK]──(Motor) |
|                  │                                       |
|    [Motor]───────┘                                       |
|                                                          |
|  Network 2: Motor Running Indicator                      |
|                                                          |
|    [Motor]───────────────────────────────(Motor_Running) |
|                                                          |
|  Network 3: Motor Stopped Indicator                      |
|                                                          |
|    [/Motor]──────────────────────────────(Motor_Stopped) |
|                                                          |
```

SCL Equivalent
```scl
// ==========================================================
// MOTOR CONTROL PROGRAM
// ==========================================================

REGION Network 1: Motor Control with Safety
    // Seal-in circuit with safety interlock
    IF ("Start_PB" OR "Motor") AND NOT "Stop_PB" AND "Safety_OK" THEN
        "Motor" := TRUE;
    ELSE
        "Motor" := FALSE;
    END_IF;
END_REGION

REGION Network 2: Indicator Lamps
    "Motor_Running" := "Motor";
    "Motor_Stopped" := NOT "Motor";
END_REGION
```

## 9. Common Beginner Mistakes
Mistake	Problem	Solution
Multiple coils for same output	Last coil wins, unpredictable behavior	Use only ONE coil per output
Forgetting NC for Stop button	Motor won't stop	Use NC contact [/Stop_PB]
No seal-in circuit	Motor runs only while button held	Add parallel contact
Ignoring scan cycle	Logic appears to not work	Check execution order
Using physical addresses	Hard to maintain and debug	Use symbolic tag names
⚠️ Critical Rule: Never use the same output coil ( ) more than once in your program!

## 10. Programming Best Practices
Code Organization
text
Network 1-10:    Safety Logic (ALWAYS FIRST)
Network 11-50:   Main Control Logic
Network 51-70:   Indicator Lamps / HMI
Network 71-99:   Diagnostics / Alarms

Commenting Standards
```
|  Network 15: Conveyor Motor Control                      |
|  Author: [Your Name]                                     |
|  Date: 2024-01-15                                        |
|  Description: Controls main conveyor based on            |
|               upstream sensor and safety status          |
```
## Naming Conventions
Type	Convention	Example
Inputs	[Device]_[Function]	Conveyor_StartPB
Outputs	[Device]_[Action]	Motor_Run
Internal	[System]_[State]	System_Ready
Timers	TMR_[Function]	TMR_StartDelay
Counters	CTR_[Function]	CTR_Parts

## 11. Exercises
Exercise 1: Basic Start-Stop
Create a program that:

Starts a motor with Start_PB
Stops the motor with Stop_PB
Include proper seal-in logic
Exercise 2: Two-Hand Safety
Create a program where:

Motor only runs when BOTH Left_PB AND Right_PB are pressed simultaneously
This is a common safety pattern for presses
Exercise 3: Selector Switch
Create a program with:

Auto_Mode selector switch
Manual_Start pushbutton
In Auto mode: Motor runs continuously
In Manual mode: Motor runs only while Manual_Start is pressed
Exercise 4: Priority Logic
Create a program where:

Stop_PB ALWAYS stops the motor
Stop has priority over Start (even if both pressed)

## 12. Quick Reference Card
Contact Types
TIA Portal	Symbol	Description
Normally Open	─[ ]─	TRUE when input = 1
Normally Closed	─[/]─	TRUE when input = 0
Positive Edge	─[P]─	TRUE for one scan on 0→1
Negative Edge	─[N]─	TRUE for one scan on 1→0
Coil Types
TIA Portal	Symbol	Description
Output	─( )─	Standard output
Set	─(S)─	Latch ON
Reset	─(R)─	Latch OFF
Set Dominant	─(SR)─	Set has priority
Reset Dominant	─(RS)─	Reset has priority
What's Next?
In Chapter 8: Timers and Counters, you'll learn how to add time delays and counting functions to your programs—essential for real-world automation.

Video Resources
🎥 YouTube: www.youtube.com/@oluwasedago
