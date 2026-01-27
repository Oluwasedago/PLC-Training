PLC Programming Examples: Ladder Logic & SCL
1. Start-Stop Motor Control (Basic)
Ladder Logic
text
|  Network 1: Motor Start-Stop with Seal-In                |
|                                                          |
|    [Start_PB]────┬────[/Stop_PB]─────────────(Motor)     |
|                  │                                       |
|    [Motor]───────┘                                       |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 1: START-STOP MOTOR CONTROL
// ==========================================================

// Seal-in circuit logic
IF ("Start_PB" OR "Motor") AND NOT "Stop_PB" THEN
    "Motor" := TRUE;
ELSE
    "Motor" := FALSE;
END_IF;
Tag Table
Name	Data Type	Address	Comment
Start_PB	Bool	%I0.0	Start Pushbutton (NO)
Stop_PB	Bool	%I0.1	Stop Pushbutton (NC)
Motor	Bool	%Q0.0	Motor Output
2. Motor Control with Safety Interlock
Ladder Logic
text
|  Network 1: Motor Control with Safety                    |
|                                                          |
|    [Start_PB]────┬────[/Stop_PB]────[Safety_OK]──(Motor) |
|                  │                                       |
|    [Motor]───────┘                                       |
|                                                          |
|  Network 2: Safety Fault Indicator                       |
|                                                          |
|    [/Safety_OK]──────────────────────────(Safety_Fault)  |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 2: MOTOR CONTROL WITH SAFETY INTERLOCK
// ==========================================================

// Motor can only run if safety circuit is OK
IF ("Start_PB" OR "Motor") AND NOT "Stop_PB" AND "Safety_OK" THEN
    "Motor" := TRUE;
ELSE
    "Motor" := FALSE;
END_IF;

// Safety fault indicator
"Safety_Fault" := NOT "Safety_OK";
Tag Table
Name	Data Type	Address	Comment
Start_PB	Bool	%I0.0	Start Pushbutton
Stop_PB	Bool	%I0.1	Stop Pushbutton
Safety_OK	Bool	%I0.2	Safety Circuit Status
Motor	Bool	%Q0.0	Motor Output
Safety_Fault	Bool	%Q0.1	Safety Fault Lamp
3. Two-Hand Safety Control
Ladder Logic
text
|  Network 1: Two-Hand Safety (Both buttons required)      |
|                                                          |
|    [Left_PB]────[Right_PB]───────────────────(Press_Run) |
|                                                          |
|  Network 2: Warning if only one hand detected            |
|                                                          |
|    [Left_PB]────[/Right_PB]──┬───────────────(Warning)   |
|                              │                           |
|    [/Left_PB]───[Right_PB]───┘                           |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 3: TWO-HAND SAFETY CONTROL
// ==========================================================

// Press only runs when BOTH hands are on buttons
"Press_Run" := "Left_PB" AND "Right_PB";

// Warning if only one hand is detected (XOR logic)
"Warning" := ("Left_PB" AND NOT "Right_PB") OR (NOT "Left_PB" AND "Right_PB");
Tag Table
Name	Data Type	Address	Comment
Left_PB	Bool	%I0.0	Left Hand Button
Right_PB	Bool	%I0.1	Right Hand Button
Press_Run	Bool	%Q0.0	Press Operate Output
Warning	Bool	%Q0.1	Single Hand Warning
4. Forward-Reverse Motor Control
Ladder Logic
text
|  Network 1: Forward Control                              |
|                                                          |
|    [Fwd_PB]────┬────[/Stop_PB]────[/Reverse]──(Forward)  |
|                │                                         |
|    [Forward]───┘                                         |
|                                                          |
|  Network 2: Reverse Control                              |
|                                                          |
|    [Rev_PB]────┬────[/Stop_PB]────[/Forward]──(Reverse)  |
|                │                                         |
|    [Reverse]───┘                                         |
|                                                          |
|  Network 3: Motor Running Indicator                      |
|                                                          |
|    [Forward]────┬────────────────────────(Motor_Running) |
|                 │                                        |
|    [Reverse]────┘                                        |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 4: FORWARD-REVERSE MOTOR CONTROL
// ==========================================================

// Forward control with interlock (cannot run if Reverse is ON)
IF ("Fwd_PB" OR "Forward") AND NOT "Stop_PB" AND NOT "Reverse" THEN
    "Forward" := TRUE;
ELSE
    "Forward" := FALSE;
END_IF;

// Reverse control with interlock (cannot run if Forward is ON)
IF ("Rev_PB" OR "Reverse") AND NOT "Stop_PB" AND NOT "Forward" THEN
    "Reverse" := TRUE;
ELSE
    "Reverse" := FALSE;
END_IF;

// Motor running indicator
"Motor_Running" := "Forward" OR "Reverse";
Tag Table
Name	Data Type	Address	Comment
Fwd_PB	Bool	%I0.0	Forward Pushbutton
Rev_PB	Bool	%I0.1	Reverse Pushbutton
Stop_PB	Bool	%I0.2	Stop Pushbutton
Forward	Bool	%Q0.0	Forward Contactor
Reverse	Bool	%Q0.1	Reverse Contactor
Motor_Running	Bool	%Q0.2	Motor Running Lamp
5. Auto/Manual Mode Selection
Ladder Logic
text
|  Network 1: Mode Selection                               |
|                                                          |
|    [Auto_Sel]────────────────────────────────(Auto_Mode) |
|                                                          |
|    [/Auto_Sel]──────────────────────────────(Manual_Mode)|
|                                                          |
|  Network 2: Auto Mode Operation                          |
|                                                          |
|    [Auto_Mode]────[Sensor]───────────────────(Motor)     |
|                                                          |
|  Network 3: Manual Mode Operation                        |
|                                                          |
|    [Manual_Mode]────[Manual_PB]────┬─────────(Motor)     |
|                                    │                     |
|                     [Motor]────────┘                     |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 5: AUTO/MANUAL MODE SELECTION
// ==========================================================

// Mode determination
"Auto_Mode" := "Auto_Sel";
"Manual_Mode" := NOT "Auto_Sel";

// Motor control based on mode
IF "Auto_Mode" THEN
    // Auto mode: Motor follows sensor
    "Motor" := "Sensor";
ELSIF "Manual_Mode" THEN
    // Manual mode: Seal-in circuit
    IF "Manual_PB" OR "Motor" THEN
        "Motor" := TRUE;
    ELSE
        "Motor" := FALSE;
    END_IF;
ELSE
    "Motor" := FALSE;
END_IF;
Tag Table
Name	Data Type	Address	Comment
Auto_Sel	Bool	%I0.0	Auto/Manual Selector
Sensor	Bool	%I0.1	Process Sensor
Manual_PB	Bool	%I0.2	Manual Start Button
Auto_Mode	Bool	%M0.0	Auto Mode Active
Manual_Mode	Bool	%M0.1	Manual Mode Active
Motor	Bool	%Q0.0	Motor Output
6. Conveyor System with Multiple Sensors
Ladder Logic
text
|  Network 1: Conveyor Start-Stop                          |
|                                                          |
|    [Start_PB]────┬────[/Stop_PB]────[/Jam_Det]─(Conv_Run)|
|                  │                                       |
|    [Conv_Run]────┘                                       |
|                                                          |
|  Network 2: Entry Sensor Detection                       |
|                                                          |
|    [Conv_Run]────[Entry_Sens]────────────(Part_Detected) |
|                                                          |
|  Network 3: Exit Sensor Detection                        |
|                                                          |
|    [Conv_Run]────[Exit_Sens]─────────────(Part_Complete) |
|                                                          |
|  Network 4: Jam Detection                                |
|                                                          |
|    [Entry_Sens]────[/Exit_Sens]────[TMR_Done]──(Jam_Det) |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 6: CONVEYOR SYSTEM WITH SENSORS
// ==========================================================

// Conveyor start-stop with jam interlock
IF ("Start_PB" OR "Conv_Run") AND NOT "Stop_PB" AND NOT "Jam_Det" THEN
    "Conv_Run" := TRUE;
ELSE
    "Conv_Run" := FALSE;
END_IF;

// Part detection at entry
"Part_Detected" := "Conv_Run" AND "Entry_Sens";

// Part completion at exit
"Part_Complete" := "Conv_Run" AND "Exit_Sens";

// Jam detection timer logic
IF "Entry_Sens" AND NOT "Exit_Sens" THEN
    "Jam_Timer"(IN := TRUE, PT := T#10S);
ELSE
    "Jam_Timer"(IN := FALSE, PT := T#10S);
END_IF;

"Jam_Det" := "Jam_Timer".Q;
Tag Table
Name	Data Type	Address	Comment
Start_PB	Bool	%I0.0	Start Pushbutton
Stop_PB	Bool	%I0.1	Stop Pushbutton
Entry_Sens	Bool	%I0.2	Entry Sensor
Exit_Sens	Bool	%I0.3	Exit Sensor
Conv_Run	Bool	%Q0.0	Conveyor Motor
Part_Detected	Bool	%Q0.1	Part Detected Lamp
Part_Complete	Bool	%Q0.2	Part Complete Lamp
Jam_Det	Bool	%M0.0	Jam Detected Flag
Jam_Timer	TON	-	Jam Detection Timer
7. Traffic Light Sequence
Ladder Logic
text
|  Network 1: System Enable                                |
|                                                          |
|    [Enable_SW]────[/Fault]───────────────(System_Active) |
|                                                          |
|  Network 2: Green Light                                  |
|                                                          |
|    [System_Active]────[State_Green]──────────(Green_Out) |
|                                                          |
|  Network 3: Yellow Light                                 |
|                                                          |
|    [System_Active]────[State_Yellow]────────(Yellow_Out) |
|                                                          |
|  Network 4: Red Light                                    |
|                                                          |
|    [System_Active]────[State_Red]────────────(Red_Out)   |
|                                                          |
|  Network 5: Flash Mode (Fault or Night)                  |
|                                                          |
|    [/System_Active]────[Flash_Pulse]────────(Yellow_Out) |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 7: TRAFFIC LIGHT SEQUENCE
// ==========================================================

// State machine variables
// State 0 = Green, State 1 = Yellow, State 2 = Red

// System enable check
"System_Active" := "Enable_SW" AND NOT "Fault";

IF "System_Active" THEN
    
    // State machine timer
    "State_Timer"(IN := TRUE, PT := "State_Time");
    
    IF "State_Timer".Q THEN
        // Reset timer and advance state
        "State_Timer"(IN := FALSE, PT := "State_Time");
        
        CASE "Current_State" OF
            0: // Green -> Yellow
                "Current_State" := 1;
                "State_Time" := T#3S;
            1: // Yellow -> Red
                "Current_State" := 2;
                "State_Time" := T#10S;
            2: // Red -> Green
                "Current_State" := 0;
                "State_Time" := T#10S;
        END_CASE;
    END_IF;
    
    // Output assignment based on state
    "Green_Out" := ("Current_State" = 0);
    "Yellow_Out" := ("Current_State" = 1);
    "Red_Out" := ("Current_State" = 2);
    
ELSE
    // Flash mode - yellow flashing
    "Flash_Timer"(IN := TRUE, PT := T#500MS);
    IF "Flash_Timer".Q THEN
        "Flash_Timer"(IN := FALSE, PT := T#500MS);
        "Flash_Pulse" := NOT "Flash_Pulse";
    END_IF;
    
    "Green_Out" := FALSE;
    "Yellow_Out" := "Flash_Pulse";
    "Red_Out" := FALSE;
END_IF;
Tag Table
Name	Data Type	Address	Comment
Enable_SW	Bool	%I0.0	System Enable Switch
Fault	Bool	%I0.1	Fault Input
Green_Out	Bool	%Q0.0	Green Light Output
Yellow_Out	Bool	%Q0.1	Yellow Light Output
Red_Out	Bool	%Q0.2	Red Light Output
System_Active	Bool	%M0.0	System Active Flag
Current_State	Int	%MW10	Current State Number
Flash_Pulse	Bool	%M0.1	Flash Pulse Bit
State_Timer	TON	-	State Duration Timer
Flash_Timer	TON	-	Flash Rate Timer
State_Time	Time	%MD20	Current State Duration
8. Tank Level Control
Ladder Logic
text
|  Network 1: Low Level - Start Fill Pump                  |
|                                                          |
|    [/Level_High]────[Level_Low]────┬─────(Fill_Pump)     |
|                                    │                     |
|    [Fill_Pump]────[/Level_High]────┘                     |
|                                                          |
|  Network 2: High Level - Start Drain Valve               |
|                                                          |
|    [Level_High]────[/Level_Low]────┬────(Drain_Valve)    |
|                                    │                     |
|    [Drain_Valve]────[/Level_Low]───┘                     |
|                                                          |
|  Network 3: Level Indicators                             |
|                                                          |
|    [Level_Low]────[/Level_High]──────────────(Low_Lamp)  |
|                                                          |
|    [Level_High]────[/Level_Low]─────────────(High_Lamp)  |
|                                                          |
|    [/Level_Low]────[/Level_High]───────────(Normal_Lamp) |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 8: TANK LEVEL CONTROL
// ==========================================================

// Fill pump control - Start on low, stop on high
IF "Level_Low" AND NOT "Level_High" THEN
    "Fill_Pump" := TRUE;
ELSIF "Level_High" THEN
    "Fill_Pump" := FALSE;
END_IF;

// Drain valve control - Start on high, stop on low
IF "Level_High" AND NOT "Level_Low" THEN
    "Drain_Valve" := TRUE;
ELSIF "Level_Low" THEN
    "Drain_Valve" := FALSE;
END_IF;

// Level indicator lamps
"Low_Lamp" := "Level_Low" AND NOT "Level_High";
"High_Lamp" := "Level_High" AND NOT "Level_Low";
"Normal_Lamp" := NOT "Level_Low" AND NOT "Level_High";
Tag Table
Name	Data Type	Address	Comment
Level_Low	Bool	%I0.0	Low Level Sensor
Level_High	Bool	%I0.1	High Level Sensor
Fill_Pump	Bool	%Q0.0	Fill Pump Output
Drain_Valve	Bool	%Q0.1	Drain Valve Output
Low_Lamp	Bool	%Q0.2	Low Level Indicator
High_Lamp	Bool	%Q0.3	High Level Indicator
Normal_Lamp	Bool	%Q0.4	Normal Level Indicator
9. Star-Delta Motor Starter
Ladder Logic
text
|  Network 1: Main Contactor Control                       |
|                                                          |
|    [Start_PB]────┬────[/Stop_PB]────[/OL_Trip]──(Main_K) |
|                  │                                       |
|    [Main_K]──────┘                                       |
|                                                          |
|  Network 2: Star Contactor (Initial Start)               |
|                                                          |
|    [Main_K]────[/Delta_K]────[/Star_TMR.Q]───(Star_K)    |
|                                                          |
|  Network 3: Star Timer                                   |
|                                                          |
|    [Star_K]────────────────────────────[TON Star_TMR]    |
|                                         PT: T#5S         |
|                                                          |
|  Network 4: Delta Contactor (After Timer)                |
|                                                          |
|    [Main_K]────[Star_TMR.Q]────[/Star_K]─────(Delta_K)   |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 9: STAR-DELTA MOTOR STARTER
// ==========================================================

// Main contactor control
IF ("Start_PB" OR "Main_K") AND NOT "Stop_PB" AND NOT "OL_Trip" THEN
    "Main_K" := TRUE;
ELSE
    "Main_K" := FALSE;
    "Star_K" := FALSE;
    "Delta_K" := FALSE;
END_IF;

// Star-Delta transition timer
"Star_TMR"(IN := "Star_K", PT := T#5S);

// Star contactor - Active at start, before timer completes
IF "Main_K" AND NOT "Delta_K" AND NOT "Star_TMR".Q THEN
    "Star_K" := TRUE;
ELSE
    "Star_K" := FALSE;
END_IF;

// Delta contactor - Active after timer completes
IF "Main_K" AND "Star_TMR".Q AND NOT "Star_K" THEN
    "Delta_K" := TRUE;
END_IF;

// Running indicator
"Motor_Running" := "Main_K" AND ("Star_K" OR "Delta_K");

// Star indicator
"Star_Ind" := "Star_K";

// Delta indicator
"Delta_Ind" := "Delta_K";
Tag Table
Name	Data Type	Address	Comment
Start_PB	Bool	%I0.0	Start Pushbutton
Stop_PB	Bool	%I0.1	Stop Pushbutton
OL_Trip	Bool	%I0.2	Overload Trip
Main_K	Bool	%Q0.0	Main Contactor
Star_K	Bool	%Q0.1	Star Contactor
Delta_K	Bool	%Q0.2	Delta Contactor
Motor_Running	Bool	%Q0.3	Running Indicator
Star_Ind	Bool	%Q0.4	Star Mode Indicator
Delta_Ind	Bool	%Q0.5	Delta Mode Indicator
Star_TMR	TON	-	Star-Delta Timer
10. Batch Counter with Reset
Ladder Logic
text
|  Network 1: Count Enable                                 |
|                                                          |
|    [Run_Mode]────[/Batch_Complete]───────(Count_Enable)  |
|                                                          |
|  Network 2: Part Counter                                 |
|                                                          |
|    [Count_Enable]────[Part_Sensor]───────[CTU Counter]   |
|                                           CU: Part_Sens  |
|                                           R: Reset_PB    |
|                                           PV: Batch_Size |
|                                                          |
|  Network 3: Batch Complete                               |
|                                                          |
|    [Counter.Q]───────────────────────(Batch_Complete)    |
|                                                          |
|  Network 4: Reset Logic                                  |
|                                                          |
|    [Reset_PB]────────────────────────────────(Reset_Out) |
|                                                          |
SCL Code
scl
// ==========================================================
// EXAMPLE 10: BATCH COUNTER WITH RESET
// ==========================================================

// Count enable logic
"Count_Enable" := "Run_Mode" AND NOT "Batch_Complete";

// Edge detection for part sensor
"Part_Sensor_Edge" := "Part_Sensor" AND NOT "Part_Sensor_Prev";
"Part_Sensor_Prev" := "Part_Sensor";

// Counter logic
IF "Reset_PB" THEN
    "Part_Count" := 0;
    "Batch_Complete" := FALSE;
ELSIF "Count_Enable" AND "Part_Sensor_Edge" THEN
    "Part_Count" := "Part_Count" + 1;
END_IF;

// Batch complete check
IF "Part_Count" >= "Batch_Size" THEN
    "Batch_Complete" := TRUE;
END_IF;

// Output indicators
"Count_Display" := "Part_Count";
"Complete_Lamp" := "Batch_Complete";
"Running_Lamp" := "Count_Enable";
Tag Table
Name	Data Type	Address	Comment
Run_Mode	Bool	%I0.0	Run Mode Selector
Part_Sensor	Bool	%I0.1	Part Detection Sensor
Reset_PB	Bool	%I0.2	Reset Pushbutton
Count_Enable	Bool	%M0.0	Count Enable Flag
Batch_Complete	Bool	%M0.1	Batch Complete Flag
Part_Sensor_Edge	Bool	%M0.2	Part Sensor Edge
Part_Sensor_Prev	Bool	%M0.3	Part Sensor Previous
Part_Count	Int	%MW10	Current Part Count
Batch_Size	Int	%MW12	Target Batch Size
Complete_Lamp	Bool	%Q0.0	Batch Complete Lamp
Running_Lamp	Bool	%Q0.1	System Running Lamp
Count_Display	Int	%QW10	Count Display Output
Quick Reference: Common Instructions
Ladder Logic Symbols
Symbol	Name	Description
─[ ]─	NO Contact	TRUE when input = 1
─[/]─	NC Contact	TRUE when input = 0
─( )─	Output Coil	Energize output
─(S)─	Set Coil	Latch ON
─(R)─	Reset Coil	Latch OFF
─[P]─	Positive Edge	One scan on rising edge
─[N]─	Negative Edge	One scan on falling edge
SCL Operators
Operator	Description	Example
AND	Logical AND	A AND B
OR	Logical OR	A OR B
NOT	Logical NOT	NOT A
XOR	Exclusive OR	A XOR B
:=	Assignment	Output := Input
=	Comparison	IF A = B THEN
<>	Not Equal	IF A <> B THEN
Timer Types
Type	Description	Use Case
TON	On-Delay	Delay before turning ON
TOF	Off-Delay	Delay before turning OFF
TP	Pulse	Fixed pulse duration
Counter Types
Type	Description	Use Case
CTU	Count Up	Count parts, events
CTD	Count Down	Remaining count
CTUD	Up/Down	Bidirectional counting
