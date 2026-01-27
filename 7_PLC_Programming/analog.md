================================================================================
                    PLC ANALOG SIGNAL PROCESSING
                    Complete Training Document
================================================================================

================================================================================
TABLE OF CONTENTS
================================================================================

1. Understanding Analog Signals
2. Analog Input Resolution
3. Analog Scaling Formula
4. Analog Scaling in Ladder Logic
5. Analog Scaling in SCL
6. Using TIA Portal NORM_X and SCALE_X
7. Analog Output Scaling
8. Complete Application Example
9. Common Analog Scaling Scenarios
10. Quick Reference

================================================================================
1. UNDERSTANDING ANALOG SIGNALS
================================================================================

OVERVIEW
--------
Analog signals are continuous signals that represent real-world measurements 
like temperature, pressure, level, flow, and speed. Unlike digital signals 
(ON/OFF), analog signals have a range of values.

DIGITAL VS ANALOG SIGNALS
-------------------------
+------------------+----------------------+----------------------+
| Characteristic   | Digital Signal       | Analog Signal        |
+------------------+----------------------+----------------------+
| Values           | 0 or 1 (ON/OFF)      | Continuous range     |
| Example          | Pushbutton, Switch   | Temperature, Pressure|
| PLC Memory       | Bool (1 bit)         | Int/Real (16/32 bit) |
| Address          | %I0.0, %Q0.0         | %IW64, %QW80         |
+------------------+----------------------+----------------------+

COMMON ANALOG SIGNAL TYPES
--------------------------
+-------------+------------------+------------------------+------------------+
| Signal Type | Range            | Description            | Common Use       |
+-------------+------------------+------------------------+------------------+
| 0-10V       | 0 to 10 Volts    | Voltage signal         | Speed, valve     |
| 0-20mA      | 0 to 20 mA       | Current signal         | Older instruments|
| 4-20mA      | 4 to 20 mA       | Current (live zero)    | Most sensors     |
| -10 to +10V | -10 to +10 Volts | Bipolar voltage        | Bidirectional    |
| 0-5V        | 0 to 5 Volts     | Low voltage            | Legacy systems   |
+-------------+------------------+------------------------+------------------+

WHY 4-20mA IS INDUSTRY STANDARD
-------------------------------

    4mA ─────────────────────────────────────── 20mA
     │                                           │
   0% │◄──────── Process Range ─────────────────►│ 100%
(Live Zero)                                 (Full Scale)

Advantages:
• 0mA = Wire broken (fault detection)
• 4mA = True zero (live zero)
• Less susceptible to electrical noise
• Can travel long distances

================================================================================
2. ANALOG INPUT RESOLUTION
================================================================================

ADC (ANALOG TO DIGITAL CONVERTER)
---------------------------------
The PLC converts the continuous analog signal into a digital value.

+------------+------------------+--------+------------------+
| Resolution | Raw Value Range  | Levels | Typical PLC      |
+------------+------------------+--------+------------------+
| 8-bit      | 0 to 255         | 256    | Basic PLCs       |
| 12-bit     | 0 to 4095        | 4096   | S7-1200          |
| 14-bit     | 0 to 16383       | 16384  | S7-1200 HSC      |
| 16-bit     | 0 to 27648       | 27648  | S7-1500, S7-300  |
+------------+------------------+--------+------------------+

SIEMENS S7-1200/1500 RAW VALUES
-------------------------------
+-------------+-----------+-----------+------------------+
| Signal Type | Raw Min   | Raw Max   | Notes            |
+-------------+-----------+-----------+------------------+
| 4-20mA      | 0         | 27648     | Standard range   |
| 0-20mA      | 0         | 27648     | Standard range   |
| 0-10V       | 0         | 27648     | Standard range   |
| -10 to +10V | -27648    | 27648     | Bipolar          |
| Underrange  | -4864     | -1        | Below 4mA        |
| Overrange   | 27649     | 32511     | Above 20mA       |
+-------------+-----------+-----------+------------------+

RAW VALUE DIAGRAM
-----------------

◄─── Underrange ───►◄────────── Normal Range ──────────►◄── Overrange ──►

-4864                0                               27648              32511
  │                  │                                 │                  │
  ▼                  ▼                                 ▼                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│   FAULT   │  < 4mA    │         4mA to 20mA          │    > 20mA       │
│  (Wire    │(Underrange│           (Normal)            │  (Overrange)    │
│  Break)   │           │                               │                 │
└─────────────────────────────────────────────────────────────────────────┘
                        │                               │
                      0%/4mA                        100%/20mA

================================================================================
3. ANALOG SCALING FORMULA
================================================================================

THE SCALING EQUATION
--------------------
To convert raw ADC values to engineering units (EU):

                      (Raw - Raw_Min)
    EU = EU_Min + ──────────────────── × (EU_Max - EU_Min)
                    (Raw_Max - Raw_Min)

SIMPLIFIED FORMULA
------------------

           (Raw - Raw_Min) × (EU_Max - EU_Min)
    EU = ────────────────────────────────────── + EU_Min
                  (Raw_Max - Raw_Min)

EXAMPLE CALCULATION
-------------------
Scenario: Temperature sensor 4-20mA representing 0°C to 100°C

+-------------------+-------------+
| Parameter         | Value       |
+-------------------+-------------+
| Raw_Min           | 0           |
| Raw_Max           | 27648       |
| EU_Min            | 0.0 °C      |
| EU_Max            | 100.0 °C    |
| Current Raw Value | 13824       |
+-------------------+-------------+

Calculation:

           (13824 - 0) × (100.0 - 0.0)
    Temp = ──────────────────────────── + 0.0
                  (27648 - 0)

           13824 × 100
    Temp = ────────────
              27648

    Temp = 50.0 °C

================================================================================
4. ANALOG SCALING IN LADDER LOGIC
================================================================================

USING NORM_X AND SCALE_X INSTRUCTIONS (TIA PORTAL)
--------------------------------------------------

Network 1: Normalize Raw Value (0 to 1.0)
─────────────────────────────────────────
    ┌─────────────────────┐
    │      NORM_X         │
  ──┤EN              ENO├──
    │                     │
  ──┤MIN (0)              │
    │                     │
  ──┤VALUE (Raw_Temp)     │
    │                     │
  ──┤MAX (27648)     OUT├──── Normalized_Value
    │                     │
    └─────────────────────┘

Network 2: Scale to Engineering Units
─────────────────────────────────────
    ┌─────────────────────┐
    │      SCALE_X        │
  ──┤EN              ENO├──
    │                     │
  ──┤MIN (0.0)            │
    │                     │
  ──┤VALUE (Norm_Value)   │
    │                     │
  ──┤MAX (100.0)     OUT├──── Temp_DegC
    │                     │
    └─────────────────────┘

================================================================================
5. ANALOG SCALING IN SCL
================================================================================

BASIC SCALING FUNCTION
----------------------

// ==========================================================
// ANALOG INPUT SCALING - BASIC
// ==========================================================

// Define scaling parameters
#Raw_Min := 0;
#Raw_Max := 27648;
#EU_Min := 0.0;
#EU_Max := 100.0;

// Read raw analog input
#Raw_Value := "IW64";  // Analog input address

// Scaling formula
#Scaled_Value := ((INT_TO_REAL(#Raw_Value) - #Raw_Min) * (#EU_Max - #EU_Min)) 
                 / (#Raw_Max - #Raw_Min) + #EU_Min;

// Assign to output tag
"Temperature_DegC" := #Scaled_Value;

--------------------------------------------------------------------------------

SCALING FUNCTION BLOCK (REUSABLE)
---------------------------------

// ==========================================================
// FUNCTION BLOCK: FB_AnalogScale
// ==========================================================

FUNCTION_BLOCK "FB_AnalogScale"

VAR_INPUT
    Raw_Value : Int;        // Raw ADC value from analog input
    Raw_Min : Int;          // Minimum raw value (typically 0)
    Raw_Max : Int;          // Maximum raw value (typically 27648)
    EU_Min : Real;          // Minimum engineering units
    EU_Max : Real;          // Maximum engineering units
    Enable_Limits : Bool;   // Enable high/low limiting
END_VAR

VAR_OUTPUT
    Scaled_Value : Real;    // Scaled output in engineering units
    Underrange : Bool;      // Signal below range (wire fault)
    Overrange : Bool;       // Signal above range
    Valid : Bool;           // Signal is valid
END_VAR

VAR
    Raw_Real : Real;        // Raw value as Real for calculation
END_VAR

BEGIN
    // Convert raw integer to real for calculation
    #Raw_Real := INT_TO_REAL(#Raw_Value);
    
    // Check for underrange (below 4mA / wire break)
    IF #Raw_Value < #Raw_Min THEN
        #Underrange := TRUE;
        #Overrange := FALSE;
        #Valid := FALSE;
        
        IF #Enable_Limits THEN
            #Scaled_Value := #EU_Min;
        ELSE
            #Scaled_Value := ((#Raw_Real - INT_TO_REAL(#Raw_Min)) * (#EU_Max - #EU_Min)) 
                             / (INT_TO_REAL(#Raw_Max) - INT_TO_REAL(#Raw_Min)) + #EU_Min;
        END_IF;
        
    // Check for overrange (above 20mA)
    ELSIF #Raw_Value > #Raw_Max THEN
        #Underrange := FALSE;
        #Overrange := TRUE;
        #Valid := FALSE;
        
        IF #Enable_Limits THEN
            #Scaled_Value := #EU_Max;
        ELSE
            #Scaled_Value := ((#Raw_Real - INT_TO_REAL(#Raw_Min)) * (#EU_Max - #EU_Min)) 
                             / (INT_TO_REAL(#Raw_Max) - INT_TO_REAL(#Raw_Min)) + #EU_Min;
        END_IF;
        
    // Normal range
    ELSE
        #Underrange := FALSE;
        #Overrange := FALSE;
        #Valid := TRUE;
        
        // Standard scaling formula
        #Scaled_Value := ((#Raw_Real - INT_TO_REAL(#Raw_Min)) * (#EU_Max - #EU_Min)) 
                         / (INT_TO_REAL(#Raw_Max) - INT_TO_REAL(#Raw_Min)) + #EU_Min;
    END_IF;
    
END_FUNCTION_BLOCK

--------------------------------------------------------------------------------

CALLING THE SCALING FUNCTION BLOCK
----------------------------------

// Instance for temperature sensor
"Temperature_Scale"(
    Raw_Value := "Raw_Temperature",     // %IW64
    Raw_Min := 0,
    Raw_Max := 27648,
    EU_Min := 0.0,
    EU_Max := 100.0,
    Enable_Limits := TRUE,
    Scaled_Value => "Temperature_DegC",
    Underrange => "Temp_Underrange",
    Overrange => "Temp_Overrange",
    Valid => "Temp_Valid"
);

// Instance for pressure sensor
"Pressure_Scale"(
    Raw_Value := "Raw_Pressure",        // %IW66
    Raw_Min := 0,
    Raw_Max := 27648,
    EU_Min := 0.0,
    EU_Max := 10.0,                     // 0-10 Bar
    Enable_Limits := TRUE,
    Scaled_Value => "Pressure_Bar",
    Underrange => "Press_Underrange",
    Overrange => "Press_Overrange",
    Valid => "Press_Valid"
);

================================================================================
6. USING TIA PORTAL BUILT-IN NORM_X AND SCALE_X
================================================================================

SCL WITH NORM_X AND SCALE_X
---------------------------

// Step 1: Normalize raw value to 0.0 - 1.0 range
"Normalized_Temp" := NORM_X(
    MIN := 0,
    VALUE := "Raw_Temperature",
    MAX := 27648
);

// Step 2: Scale normalized value to engineering units
"Temperature_DegC" := SCALE_X(
    MIN := 0.0,
    VALUE := "Normalized_Temp",
    MAX := 100.0
);

--------------------------------------------------------------------------------

COMPLETE EXAMPLE WITH ERROR HANDLING
------------------------------------

// Constants
#RAW_MIN := 0;
#RAW_MAX := 27648;
#UNDERRANGE_LIMIT := -4864;
#OVERRANGE_LIMIT := 32511;

// Read raw analog input
#Raw_Temp := "Raw_Temperature";     // %IW64

// Error checking
IF #Raw_Temp < #UNDERRANGE_LIMIT THEN
    // Wire break or severe underrange
    "Temp_Fault" := TRUE;
    "Temp_Underrange" := TRUE;
    "Temp_Overrange" := FALSE;
    "Temp_Valid" := FALSE;
    "Temperature_DegC" := 0.0;
    
ELSIF #Raw_Temp < #RAW_MIN THEN
    // Underrange (below 4mA)
    "Temp_Fault" := FALSE;
    "Temp_Underrange" := TRUE;
    "Temp_Overrange" := FALSE;
    "Temp_Valid" := FALSE;
    "Temperature_DegC" := 0.0;
    
ELSIF #Raw_Temp > #OVERRANGE_LIMIT THEN
    // Severe overrange
    "Temp_Fault" := TRUE;
    "Temp_Underrange" := FALSE;
    "Temp_Overrange" := TRUE;
    "Temp_Valid" := FALSE;
    "Temperature_DegC" := 100.0;
    
ELSIF #Raw_Temp > #RAW_MAX THEN
    // Overrange (above 20mA)
    "Temp_Fault" := FALSE;
    "Temp_Underrange" := FALSE;
    "Temp_Overrange" := TRUE;
    "Temp_Valid" := FALSE;
    "Temperature_DegC" := 100.0;
    
ELSE
    // Normal range - perform scaling
    "Temp_Fault" := FALSE;
    "Temp_Underrange" := FALSE;
    "Temp_Overrange" := FALSE;
    "Temp_Valid" := TRUE;
    
    // Two-step scaling using NORM_X and SCALE_X
    #Normalized := NORM_X(MIN := #RAW_MIN, VALUE := #Raw_Temp, MAX := #RAW_MAX);
    "Temperature_DegC" := SCALE_X(MIN := 0.0, VALUE := #Normalized, MAX := 100.0);
    
END_IF;

================================================================================
7. ANALOG OUTPUT SCALING (REVERSE SCALING)
================================================================================

OUTPUT SCALING FORMULA
----------------------
To convert engineering units back to raw values for analog output:

                      (EU - EU_Min)
    Raw = Raw_Min + ────────────────── × (Raw_Max - Raw_Min)
                     (EU_Max - EU_Min)

--------------------------------------------------------------------------------

LADDER LOGIC FOR ANALOG OUTPUT
------------------------------

Network 1: Scale Engineering Units to Raw
─────────────────────────────────────────
    ┌─────────────────────┐
    │      NORM_X         │
  ──┤EN              ENO├──
    │                     │
  ──┤MIN (0.0)            │   (EU Min)
    │                     │
  ──┤VALUE (Speed_Percent)│   (Desired speed 0-100%)
    │                     │
  ──┤MAX (100.0)     OUT├──── Normalized_Speed
    │                     │
    └─────────────────────┘

Network 2: Scale to Raw Output Value
────────────────────────────────────
    ┌─────────────────────┐
    │      SCALE_X        │
  ──┤EN              ENO├──
    │                     │
  ──┤MIN (0)              │   (Raw Min)
    │                     │
  ──┤VALUE (Norm_Speed)   │
    │                     │
  ──┤MAX (27648)     OUT├──── Raw_Speed_Out
    │                     │
    └─────────────────────┘

Network 3: Write to Analog Output
─────────────────────────────────
    ┌─────────────────────┐
  ──┤      MOVE           │
    │                     │
    │ IN: Raw_Speed_Out   │
    │                     │
    │ OUT: %QW80          │   (Analog Output Address)
    │                     │
    └─────────────────────┘

--------------------------------------------------------------------------------

SCL FOR ANALOG OUTPUT
---------------------

// Limit input to valid range
IF "Speed_Setpoint" < 0.0 THEN
    #Limited_Speed := 0.0;
ELSIF "Speed_Setpoint" > 100.0 THEN
    #Limited_Speed := 100.0;
ELSE
    #Limited_Speed := "Speed_Setpoint";
END_IF;

// Normalize EU to 0-1 range
#Normalized := NORM_X(MIN := 0.0, VALUE := #Limited_Speed, MAX := 100.0);

// Scale to raw output range
#Raw_Output := SCALE_X(MIN := 0, VALUE := #Normalized, MAX := 27648);

// Write to analog output
"QW80" := REAL_TO_INT(#Raw_Output);

--------------------------------------------------------------------------------

ANALOG OUTPUT FUNCTION BLOCK
----------------------------

FUNCTION_BLOCK "FB_AnalogOutput"

VAR_INPUT
    EU_Value : Real;        // Engineering units value
    EU_Min : Real;          // Minimum engineering units
    EU_Max : Real;          // Maximum engineering units
    Raw_Min : Int;          // Minimum raw value (typically 0)
    Raw_Max : Int;          // Maximum raw value (typically 27648)
    Enable : Bool;          // Enable output
END_VAR

VAR_OUTPUT
    Raw_Output : Int;       // Raw value for analog output
    Limited : Bool;         // Value was limited
    Fault : Bool;           // Calculation fault
END_VAR

VAR
    Limited_Value : Real;
    Normalized : Real;
    Scaled : Real;
END_VAR

BEGIN
    IF #Enable THEN
        // Limit input to valid range
        IF #EU_Value < #EU_Min THEN
            #Limited_Value := #EU_Min;
            #Limited := TRUE;
        ELSIF #EU_Value > #EU_Max THEN
            #Limited_Value := #EU_Max;
            #Limited := TRUE;
        ELSE
            #Limited_Value := #EU_Value;
            #Limited := FALSE;
        END_IF;
        
        // Check for divide by zero
        IF (#EU_Max - #EU_Min) = 0.0 THEN
            #Fault := TRUE;
            #Raw_Output := #Raw_Min;
        ELSE
            #Fault := FALSE;
            
            // Normalize to 0-1
            #Normalized := (#Limited_Value - #EU_Min) / (#EU_Max - #EU_Min);
            
            // Scale to raw range
            #Scaled := #Normalized * INT_TO_REAL(#Raw_Max - #Raw_Min) 
                       + INT_TO_REAL(#Raw_Min);
            
            // Convert to integer
            #Raw_Output := REAL_TO_INT(#Scaled);
        END_IF;
    ELSE
        // Output disabled - set to minimum
        #Raw_Output := #Raw_Min;
        #Limited := FALSE;
        #Fault := FALSE;
    END_IF;
    
END_FUNCTION_BLOCK

================================================================================
8. COMPLETE APPLICATION EXAMPLE: TEMPERATURE CONTROL
================================================================================

TAG TABLE
---------
+-------------------+-----------+---------+-------------------------+
| Name              | Data Type | Address | Comment<span class="ml-2" /><span class="inline-block w-3 h-3 rounded-full bg-neutral-a12 align-middle mb-[0.1rem]" />
