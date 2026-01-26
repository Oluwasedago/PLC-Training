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

## 2. Master Tag Table (TIA Portal V20 Optimized)
This table follows strict non-overlapping memory rules to ensure safe PLC operation.

| Name | Data Type | Logical Address | Range/End Address | Comment |
| :--- | :--- | :--- | :--- | :--- |
| Bool | Bool | M0.0 | M0.0 | 1 bit: True or False |
| Byte | Byte | MB1 | MB1 | 8 bits: 0-255 hex |
| SInt | SInt | MB2 | MB2 | 8-bit Signed (-128 to 127) |
| USInt | USInt | MB3 | MB3 | 8-bit Unsigned (0 to 255) |
| Char | Char | MB4 | MB4 | 8-bit Character (ASCII) |
| Word | Word | **MW6** | MB6 - MB7 | 16 bits: Bit string (Hex) |
| Int | Int | **MW8** | MB8 - MB9 | 16-bit Signed Integer |
| UInt | UInt | **MW10** | MB10 - MB11 | 16-bit Unsigned Integer |
| Date | Date | **MW12** | MB12 - MB13 | 16-bit Date format |
| DWord | DWord | **MD14** | MB14 - MB17 | 32-bit Bit string |
| DInt | DInt | **MD18** | MB18 - MB21 | 32-bit Signed Integer |
| UDInt | UDInt | **MD22** | MB22 - MB25 | 32-bit Unsigned Integer |
| Real | Real | **MD26** | MB26 - MB29 | 32-bit Floating Point |
| Time | Time | **MD30** | MB30 - MB33 | 32-bit Time (ms) |
| LReal | LReal | **MD34** | MB34 - MB41 | 64-bit Floating Point (8 bytes) |

---

## 3. SCL Stress Test: Overflow & Conversion
Use this snippet to observe how the PLC handles data limits and precision.

```scl
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
    
    // Using LReal for high precision
    #Tag_LReal := 123.456789012345;
END_REGION
```
## Resources
* **YouTube Guide:** [Siemens PLC Tags & Data Types Best Practices](https://www.youtube.com/@Oluwasedago)
