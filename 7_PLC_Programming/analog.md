Analog Signal Processing in PLCs
Overview
Analog signals are continuous signals that represent real-world measurements like temperature, pressure, level, flow, and speed. Unlike digital signals (ON/OFF), analog signals have a range of values. Understanding how to read, scale, and use analog signals is essential for process control applications.

1. Understanding Analog Signals
Digital vs Analog Signals
Characteristic	Digital Signal	Analog Signal
Values	0 or 1 (ON/OFF)	Continuous range
Example	Pushbutton, Limit Switch	Temperature, Pressure
PLC Memory	Bool (1 bit)	Int/Real (16/32 bit)
Address	%I0.0, %Q0.0	%IW64, %QW80
Common Analog Signal Types
Signal Type	Range	Description	Common Use
0-10V	0 to 10 Volts	Voltage signal	Speed control, valve position
0-20mA	0 to 20 mA	Current signal	Older instruments
4-20mA	4 to 20 mA	Current signal (live zero)	Most industrial sensors
-10 to +10V	-10 to +10 Volts	Bipolar voltage	Bidirectional control
0-5V	0 to 5 Volts	Low voltage	Some sensors, legacy
Why 4-20mA is Industry Standard
text
┌─────────────────────────────────────────────────────────┐
│                    4-20mA SIGNAL                        │
│                                                         │
│  4mA ─────┬───────────────────────────────┬───── 20mA  │
│           │                               │             │
│        0% │◄─────── Process Range ───────►│ 100%       │
│     (Live Zero)                      (Full Scale)      │
│                                                         │
│  Advantages:                                            │
│  • 0mA = Wire broken (fault detection)                 │
│  • 4mA = True zero (live zero)                         │
│  • Less susceptible to electrical noise               │
│  • Can travel long distances                          │
└─────────────────────────────────────────────────────────┘
2. Analog Input Resolution
ADC (Analog to Digital Converter)
The PLC converts the continuous analog signal into a digital value using an ADC.

Resolution	Raw Value Range	Levels	Typical PLC
8-bit	0 to 255	256	Basic PLCs
12-bit	0 to 4095	4096	S7-1200
14-bit	0 to 16383	16384	S7-1200 HSC
16-bit	0 to 27648	27648	S7-1500, S7-300/400
Siemens S7-1200/1500 Raw Values
Signal Type	Raw Min	Raw Max	Notes
4-20mA	0	27648	Standard range
0-20mA	0	27648	Standard range
0-10V	0	27648	Standard range
-10 to +10V	-27648	27648	Bipolar
Underrange	-4864	-1	Below 4mA
Overrange	27649	32511	Above 20mA
Raw Value Diagram
text
       ◄──────────── Underrange ──────────►◄─────────── Normal Range ─────────►◄── Overrange ──►
       
       -4864                               0                                27648              32511
         │                                 │                                  │                  │
         ▼                                 ▼                                  ▼                  ▼
    ┌─────────────────────────────────────────────────────────────────────────────────────────────┐
    │    FAULT    │      < 4mA      │              4mA to 20mA                │    > 20mA        │
    │   (Wire     │    (Underrange) │              (Normal)                   │   (Overrange)    │
    │   Break)    │                 │                                         │                  │
    └─────────────────────────────────────────────────────────────────────────────────────────────┘
                                    │                                         │
                                  0%/4mA                                   100%/20mA
3. Analog Scaling Formula
The Scaling Equation
To convert raw ADC values to engineering units (EU), use linear interpolation:

text
                    (Raw - Raw_Min)
EU = EU_Min + ────────────────────── × (EU_Max - EU_Min)
                  (Raw_Max - Raw_Min)
Simplified Formula
text
         (Raw - Raw_Min) × (EU_Max - EU_Min)
EU = ───────────────────────────────────────── + EU_Min
              (Raw_Max - Raw_Min)
Example Calculation
Scenario: Temperature sensor 4-20mA representing 0°C to 100°C

Parameter	Value
Raw_Min	0
Raw_Max	27648
EU_Min	0.0 °C
EU_Max	100.0 °C
Current Raw Value	13824
Calculation:

text
         (13824 - 0) × (100.0 - 0.0)
Temp = ─────────────────────────────── + 0.0
              (27648 - 0)

         13824 × 100
Temp = ───────────────
           27648

Temp = 50.0 °C
4. Analog Scaling in Ladder Logic
Using NORM_X and SCALE_X Instructions (TIA Portal)
text
|  Network 1: Normalize Raw Value (0 to 1.0)               |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      NORM_X         │                               |
|  ──┤EN              ENO├──                               |
|    │                     │                               |
|  ──┤MIN (0)              │                               |
|    │                     │                               |
|  ──┤VALUE (Raw_Temp)     │                               |
|    │                     │                               |
|  ──┤MAX (27648)     OUT├──── Normalized_Value            |
|    │                     │                               |
|    └─────────────────────┘                               |
|                                                          |
|  Network 2: Scale to Engineering Units                   |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      SCALE_X        │                               |
|  ──┤EN              ENO├──                               |
|    │                     │                               |
|  ──┤MIN (0.0)            │                               |
|    │                     │                               |
|  ──┤VALUE (Norm_Value)   │                               |
|    │                     │                               |
|  ──┤MAX (100.0)     OUT├──── Temp_DegC                   |
|    │                     │                               |
|    └─────────────────────┘                               |
|                                                          |
Direct Calculation in Ladder (Using CALCULATE)
text
|  Network 1: Direct Scaling Formula                       |
|                                                          |
|    ┌─────────────────────────────────────────┐           |
|    │              CALCULATE                  │           |
|  ──┤EN                                  ENO├──           |
|    │                                         │           |
|    │  OUT = (IN1 - IN2) * (IN3 - IN4)       │           |
|    │        ─────────────────────────        │           |
|    │              (IN5 - IN6)                │           |
|    │        + IN4                            │           |
|    │                                         │           |
|  ──┤IN1: Raw_Temp (Analog Input)             │           |
|  ──┤IN2: 0 (Raw Min)                         │           |
|  ──┤IN3: 100.0 (EU Max)                      │           |
|  ──┤IN4: 0.0 (EU Min)                        │           |
|  ──┤IN5: 27648 (Raw Max)                     │           |
|  ──┤IN6: 0 (Raw Min)                    OUT├──Temp_DegC  |
|    │                                         │           |
|    └─────────────────────────────────────────┘           |
|                                                          |
5. Analog Scaling in SCL
Basic Scaling Function
scl
// ==========================================================
// ANALOG INPUT SCALING - BASIC
// ==========================================================

// Input: Raw analog value from PLC input
// Output: Scaled engineering value

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
Scaling Function Block (Reusable)
scl
// ==========================================================
// FUNCTION BLOCK: FB_AnalogScale
// ==========================================================
// Reusable scaling block for any analog input

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
            // Calculate even if underrange
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
            // Calculate even if overrange
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
Calling the Scaling Function Block
scl
// ==========================================================
// CALLING FB_AnalogScale IN MAIN PROGRAM (OB1)
// ==========================================================

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

// Instance for level sensor
"Level_Scale"(
    Raw_Value := "Raw_Level",           // %IW68
    Raw_Min := 0,
    Raw_Max := 27648,
    EU_Min := 0.0,
    EU_Max := 5000.0,                   // 0-5000 mm
    Enable_Limits := TRUE,
    Scaled_Value => "Level_mm",
    Underrange => "Level_Underrange",
    Overrange => "Level_Overrange",
    Valid => "Level_Valid"
);
6. Using TIA Portal Built-in NORM_X and SCALE_X
SCL with NORM_X and SCALE_X
scl
// ==========================================================
// USING BUILT-IN NORM_X AND SCALE_X INSTRUCTIONS
// ==========================================================

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
Complete Example with Error Handling
scl
// ==========================================================
// COMPLETE ANALOG SCALING WITH ERROR HANDLING
// ==========================================================

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
7. Analog Output Scaling (Reverse Scaling)
Output Scaling Formula
To convert engineering units back to raw values for analog output:

text
                    (EU - EU_Min)
Raw = Raw_Min + ────────────────────── × (Raw_Max - Raw_Min)
                  (EU_Max - EU_Min)
Ladder Logic for Analog Output
text
|  Network 1: Scale Engineering Units to Raw               |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      NORM_X         │                               |
|  ──┤EN              ENO├──                               |
|    │                     │                               |
|  ──┤MIN (0.0)            │   (EU Min)                    |
|    │                     │                               |
|  ──┤VALUE (Speed_Percent)│   (Desired speed 0-100%)      |
|    │                     │                               |
|  ──┤MAX (100.0)     OUT├──── Normalized_Speed            |
|    │                     │                               |
|    └─────────────────────┘                               |
|                                                          |
|  Network 2: Scale to Raw Output Value                    |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      SCALE_X        │                               |
|  ──┤EN              ENO├──                               |
|    │                     │                               |
|  ──┤MIN (0)              │   (Raw Min)                   |
|    │                     │                               |
|  ──┤VALUE (Norm_Speed)   │                               |
|    │                     │                               |
|  ──┤MAX (27648)     OUT├──── Raw_Speed_Out               |
|    │                     │                               |
|    └─────────────────────┘                               |
|                                                          |
|  Network 3: Write to Analog Output                       |
|                                                          |
|    ┌─────────────────────┐                               |
|  ──┤      MOVE           │                               |
|    │                     │                               |
|    │ IN: Raw_Speed_Out   │                               |
|    │                     │                               |
|    │ OUT: %QW80          │   (Analog Output Address)     |
|    │                     │                               |
|    └─────────────────────┘                               |
|                                                          |
SCL for Analog Output
scl
// ==========================================================
// ANALOG OUTPUT SCALING
// ==========================================================

// Input: Engineering value (e.g., 0-100% speed)
// Output: Raw value for analog output card

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
Analog Output Function Block
scl
// ==========================================================
// FUNCTION BLOCK: FB_AnalogOutput
// ==========================================================
// Reusable scaling block for analog outputs

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
            #Scaled := #Normalized * INT_TO_REAL(#Raw_Max - #Raw_Min) + INT_TO_REAL(#Raw_Min);
            
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
8. Complete Application Example: Temperature Control
Tag Table
Name	Data Type	Address	Comment
Raw_Temperature	Int	%IW64	Temperature sensor 4-20mA
Raw_Heater_Out	Int	%QW80	Heater control 0-10V
Temperature_DegC	Real	%MD100	Scaled temperature
Temp_Setpoint	Real	%MD104	Temperature setpoint
Heater_Percent	Real	%MD108	Heater output %
Temp_Valid	Bool	%M10.0	Temperature valid
Temp_Underrange	Bool	%M10.1	Temperature underrange
Temp_Overrange	Bool	%M10.2	Temperature overrange
Heater_Enable	Bool	%M10.3	Heater enable
System_Fault	Bool	%M10.4	System fault
Ladder Logic Implementation
text
|  Network 1: Scale Temperature Input                      |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      NORM_X         │                               |
|  ──┤EN              ENO├──                               |
|    │MIN: 0               │                               |
|    │VALUE: Raw_Temp      │                               |
|    │MAX: 27648      OUT├──── Norm_Temp                   |
|    └─────────────────────┘                               |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      SCALE_X        │                               |
|  ──┤EN              ENO├──                               |
|    │MIN: 0.0             │                               |
|    │VALUE: Norm_Temp     │                               |
|    │MAX: 100.0      OUT├──── Temperature_DegC            |
|    └─────────────────────┘                               |
|                                                          |
|  Network 2: Range Checking                               |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      CMP < (Int)    │                               |
|  ──┤IN1: Raw_Temperature │                               |
|    │IN2: 0          OUT├──── Temp_Underrange             |
|    └─────────────────────┘                               |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      CMP > (Int)    │                               |
|  ──┤IN1: Raw_Temperature │                               |
|    │IN2: 27648      OUT├──── Temp_Overrange              |
|    └─────────────────────┘                               |
|                                                          |
|  Network 3: Valid Signal                                 |
|                                                          |
|    [/Temp_Underrange]────[/Temp_Overrange]───(Temp_Valid)|
|                                                          |
|  Network 4: Simple On-Off Control                        |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      CMP < (Real)   │                               |
|  ──┤IN1: Temperature_DegC│                               |
|    │IN2: Temp_Setpoint   │                               |
|    │               OUT├──┬───────────────────(Heat_Req)  |
|    └─────────────────────┘│                              |
|                           │                              |
|    [Heater_Enable]────────┘                              |
|                                                          |
|  Network 5: Scale Heater Output                          |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      SEL (Select)   │                               |
|  ──┤G: Heat_Req          │                               |
|    │IN0: 0.0             │                               |
|    │IN1: 100.0      OUT├──── Heater_Percent              |
|    └─────────────────────┘                               |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      NORM_X         │                               |
|  ──┤EN              ENO├──                               |
|    │MIN: 0.0             │                               |
|    │VALUE: Heater_Percent│                               |
|    │MAX: 100.0      OUT├──── Norm_Heat                   |
|    └─────────────────────┘                               |
|                                                          |
|    ┌─────────────────────┐                               |
|    │      SCALE_X        │                               |
|  ──┤EN              ENO├──                               |
|    │MIN: 0               │                               |
|    │VALUE: Norm_Heat     │                               |
|    │MAX: 27648      OUT├──── Raw_Heater_Out              |
|    └─────────────────────┘                               |
|                                                          |
Complete SCL Implementation
scl
// ==========================================================
// TEMPERATURE CONTROL APPLICATION
// ==========================================================
// Reads 4-20mA temperature sensor
// Controls heater via 0-10V analog output

// ----------------------
// CONSTANTS
// ----------------------
#RAW_MIN := 0;
#RAW_MAX := 27648;
#TEMP_MIN := 0.0;       // 0°C
#TEMP_MAX := 100.0;     // 100°C
#HEATER_MIN := 0.0;     // 0%
#HEATER_MAX := 100.0;   // 100%

// ----------------------
// ANALOG INPUT SCALING
// ----------------------
// Read raw temperature input
#Raw_Temp := "Raw_Temperature";

// Check for underrange (wire break)
IF #Raw_Temp < #RAW_MIN THEN
    "Temp_Underrange" := TRUE;
    "Temp_Overrange" := FALSE;
    "Temp_Valid" := FALSE;
    "Temperature_DegC" := #TEMP_MIN;
    
// Check for overrange
ELSIF #Raw_Temp > #RAW_MAX THEN
    "Temp_Underrange" := FALSE;
    "Temp_Overrange" := TRUE;
    "Temp_Valid" := FALSE;
    "Temperature_DegC" := #TEMP_MAX;
    
// Normal range
ELSE
    "Temp_Underrange" := FALSE;
    "Temp_Overrange" := FALSE;
    "Temp_Valid" := TRUE;
    
    // Scale raw to engineering units
    #Norm_Temp := NORM_X(MIN := #RAW_MIN, VALUE := #Raw_Temp, MAX := #RAW_MAX);
    "Temperature_DegC" := SCALE_X(MIN := #TEMP_MIN, VALUE := #Norm_Temp, MAX := #TEMP_MAX);
END_IF;

// ----------------------
// TEMPERATURE CONTROL
// ----------------------
// Simple On-Off control with hysteresis
#Hysteresis := 2.0;     // 2°C deadband

IF "Heater_Enable" AND "Temp_Valid" THEN
    // Turn heater ON below setpoint
    IF "Temperature_DegC" < ("Temp_Setpoint" - #Hysteresis) THEN
        "Heater_Request" := TRUE;
        "Heater_Percent" := 100.0;
        
    // Turn heater OFF above setpoint
    ELSIF "Temperature_DegC" > ("Temp_Setpoint" + #Hysteresis) THEN
        "Heater_Request" := FALSE;
        "Heater_Percent" := 0.0;
    END_IF;
    // Maintain previous state within hysteresis band
    
ELSE
    // Heater disabled or invalid temperature
    "Heater_Request" := FALSE;
    "Heater_Percent" := 0.0;
END_IF;

// ----------------------
// ANALOG OUTPUT SCALING
// ----------------------
// Limit heater output
IF "Heater_Percent" < #HEATER_MIN THEN
    #Limited_Heater := #HEATER_MIN;
ELSIF "Heater_Percent" > #HEATER_MAX THEN
    #Limited_Heater := #HEATER_MAX;
ELSE
    #Limited_Heater := "Heater_Percent";
END_IF;

// Scale to raw output value
#Norm_Heater := NORM_X(MIN := #HEATER_MIN, VALUE := #Limited_Heater, MAX := #HEATER_MAX);
#Raw_Heater := SCALE_X(MIN := #RAW_MIN, VALUE := #Norm_Heater, MAX := #RAW_MAX);

// Write to analog output
"Raw_Heater_Out" := REAL_TO_INT(#Raw_Heater);

// ----------------------
// SYSTEM FAULT DETECTION
// ----------------------
"System_Fault" := "Temp_Underrange" OR "Temp_Overrange";

// ----------------------
// INDICATOR OUTPUTS
// ----------------------
"Heating_Lamp" := "Heater_Request";
"Fault_Lamp" := "System_Fault";
"Normal_Lamp" := NOT "System_Fault" AND "Heater_Enable";
9. Common Analog Scaling Scenarios
Scenario 1: Pressure Transmitter (0-10 Bar)
scl
// 4-20mA input representing 0-10 Bar
#Norm := NORM_X(MIN := 0, VALUE := "Raw_Pressure", MAX := 27648);
"Pressure_Bar" := SCALE_X(MIN := 0.0, VALUE := #Norm, MAX := 10.0);
Scenario 2: Level Transmitter (0-5000 mm)
scl
// 4-20mA input representing 0-5000 mm
#Norm := NORM_X(MIN := 0, VALUE := "Raw_Level", MAX := 27648);
"Level_mm" := SCALE_X(MIN := 0.0, VALUE := #Norm, MAX := 5000.0);
Scenario 3: Flow Meter (0-1000 L/min)
scl
// 4-20mA input representing 0-1000 L/min
#Norm := NORM_X(MIN := 0, VALUE := "Raw_Flow", MAX := 27648);
"Flow_LPM" := SCALE_X(MIN := 0.0, VALUE := #Norm, MAX := 1000.0);
Scenario 4: VFD Speed Reference (0-50 Hz)
scl
// Output 0-10V to VFD for 0-50 Hz
#Norm := NORM_X(MIN := 0.0, VALUE := "Speed_Hz_SP", MAX := 50.0);
#Raw := SCALE_X(MIN := 0, VALUE := #Norm, MAX := 27648);
"AO_VFD_Speed" := REAL_TO_INT(#Raw);
Scenario 5: Valve Position (0-100%)
scl
// Output 4-20mA to control valve 0-100%
#Norm := NORM_X(MIN := 0.0, VALUE := "Valve_Position_SP", MAX := 100.0);
#Raw := SCALE_X(MIN := 0, VALUE := #Norm, MAX := 27648);
"AO_Valve" := REAL_TO_INT(#Raw);
Scenario 6: Bipolar Input (-10 to +10 V representing -100 to +100)
scl
// Bipolar voltage input
#Norm := NORM_X(MIN := -27648, VALUE := "Raw_Bipolar", MAX := 27648);
"Bipolar_Value" := SCALE_X(MIN := -100.0, VALUE := #Norm, MAX := 100.0);
10. Quick Reference
Analog Input Scaling Steps
Read raw value from analog input (%IW)
Check for underrange/overrange
Normalize raw value to 0.0-1.0 using NORM_X
Scale normalized value to EU using SCALE_X
Use scaled value in your control logic
Analog Output Scaling Steps
Get engineering value from control logic
Limit value to valid range
Normalize EU value to 0.0-1.0 using NORM_X
Scale normalized value to raw using SCALE_X
Convert Real to Int and write to output (%QW)
Important Formulas
Direction	Formula
Input Scaling	EU = ((Raw - Raw_Min) / (Raw_Max - Raw_Min)) × (EU_Max - EU_Min) + EU_Min
Output Scaling	Raw = ((EU - EU_Min) / (EU_Max - EU_Min)) × (Raw_Max - Raw_Min) + Raw_Min
Standard Raw Values (Siemens)
Condition	Raw Value
0% / 4mA / 0V	0
100% / 20mA / 10V	27648
Underrange	< 0 (down to -4864)
Overrange	> 27648 (up to 32511)
