# Chapter 8: Introduction to Timers and Counters

## Overview
Time and Quantity are the two most common variables in production. This chapter covers the IEC timers and counters used in Siemens S7 environments.

## Learning Objectives
* **Timer Operations:**
    * **TON:** On-Delay (Wait before acting).
    * **TOF:** Off-Delay (Keep acting for a duration after the signal is gone).
    * **TP:** Pulse (Generate a fixed-length signal).
* **Counter Operations:** Incremental (CTU) and Decremental (CTD) logic for batching.

## Technical Detail
We focus on **IEC Timers** rather than legacy S5 Timers. IEC timers are more flexible as they use the `TIME` data type and can be embedded within Function Blocks as multi-instances.

## Resources
* **YouTube Guide:** [Implementing Timers and Counters](https://www.youtube.com/@Oluwasedago)
