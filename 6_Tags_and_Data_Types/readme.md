# Chapter 6: PLC Tags and Data Types

## Overview
Data management separates a "coder" from an "engineer." This chapter covers how variables are stored in memory and why selecting the correct data type is essential for math accuracy.

## Learning Objectives
* **Binary Math:** How Bits, Bytes, Words (16-bit), and DWords (32-bit) are structured.
* **Data Types:** Difference between Integers (INT), Real (Floating Point), and Time.
* **Tag Scoping:** Global Tags (PLC Tags) vs. Local Tags (Block Interface).

## Standards for Tags
* **Naming Conventions:** Use descriptive names (e.g., `Motor_Conveyor_Start`) instead of generic addresses (`M0.0`).
* **UDTs:** Introduction to User-Defined Data Types for structuring complex data.

---

## 1. Valid M Memory Address Prefixes

| Prefix | Size | Supported Data Types |
| :--- | :--- | :--- |
| %M | 1 bit | Bool |
| %MB | 1 byte (8 bits) | Byte, SInt, USInt, Char |
| %MW | 2 bytes (16 bits) | Word, Int, UInt, Date |
| %MD | 4 bytes (32 bits) | DWord, DInt, UDInt, Real, Time |

> ⚠️ **Important:** There is **NO `%ML` prefix** in TIA Portal. 64-bit and complex data types cannot be stored in M memory.

---

## 2. Data Type Storage Locations

| Storage Location | Supported Data Types |
| :--- | :--- |
| **PLC Tag Table (M Memory)** | Bool, Byte, SInt, USInt, Char, Word, Int, UInt, Date, DWord, DInt, UDInt, Real, Time |
| **Data Blocks (DB) Only** | LReal, String, WString, Arrays, Structs, UDTs |

---

## 3. Master Tag Table (TIA Portal V20 Optimized)
This table follows strict non-overlapping memory rules to ensure safe PLC operation.

| Name | Data Type | Logical Address | Range/End Address | Comment |
| :--- | :--- | :--- | :--- | :--- |
| Bool | Bool | %M0.0 | M0.0 | 1 bit: True or False |
| Byte | Byte | %MB1 | MB1 | 8 bits: 0-255 hex |
| SInt | SInt | %MB2 | MB2 | 8-bit Signed (-128 to 127) |
| USInt | USInt | %MB3 | MB3 | 8-bit Unsigned (0 to 255) |
| Char | Char | %MB4 | MB4 | 8-bit Character (ASCII) |
| Word | Word | %MW6 | MB6 - MB7 | 16 bits: Bit string (Hex) |
| Int | Int | %MW8 | MB8 - MB9 | 16-bit Signed Integer |
| UInt | UInt | %MW10 | MB10 - MB11 | 16-bit Unsigned Integer |
| Date | Date | %MW12 | MB12 - MB13 | 16-bit Date format |
| DWord | DWord | %MD14 | MB14 - MB17 | 32-bit Bit string |
| DInt | DInt | %MD18 | MB18 - MB21 | 32-bit Signed Integer |
| UDInt | UDInt | %MD22 | MB22 - MB25 | 32-bit Unsigned Integer |
| Real | Real | %MD26 | MB26 - MB29 | 32-bit Floating Point |
| Time | Time | %MD30 | MB30 - MB33 | 32-bit Time (ms) |

> ⚠️ **Note:** LReal (64-bit), String, and other complex data types must be stored in a **Data Block (DB)**, not in the PLC Tag Table.

---

## 4. Data Block for Complex Types

For data types that cannot be stored in M memory, create a **Global Data Block (DB)**:

### Example DB Structure:
| Name | Data Type | Comment |
| :--- | :--- | :--- |
| MyLReal | LReal | 64-bit Floating Point |
| MyString | String[50] | String with max 50 characters |
| MyArray | Array[0..9] of Int | Array of 10 integers |

### Accessing DB Variables:
```scl
"DataBlockName".MyLReal := 123.456789012345;
"DataBlockName".MyString := 'Hello World';
```

## 5. SCL Stress Test: Overflow & Conversion
Use this snippet to observe how the PLC handles data limits and precision.

```sscl
// ==========================================================
// DATA TYPE STRESS TEST
// ==========================================================

REGION 1. INTEGER OVERFLOW
    // Max Signed Int is 32767. Adding 1 causes a wrap-around.
    #Tag_Int := 32767;
    #Tag_Int := #Tag_Int + 1; // Result: -32768
END_REGION

REGION 2. PRECISION HANDLING
    // Converting Real to Int causes rounding.
    #Tag_Int := REAL_TO_INT(123.75); // Result: 124
    
    // Using LReal for high precision (must be in a DB)
    "MyDataBlock".Tag_LReal := 123.456789012345;
END_REGION
```
