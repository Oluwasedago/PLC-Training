# 📚 Industrial Automation Glossary

This glossary provides definitions for common terms, acronyms, and concepts used throughout the PLC Training course.

---

## A
* **AI / AO (Analog Input / Output):** Continuous signals used for variables like temperature (4-20mA) or motor speed (0-10V).
* **ALM (Automation License Manager):** The Siemens utility used to manage and activate TIA Portal software licenses.

## B
* **BOOL (Boolean):** The simplest data type representing a single bit (True/False, 1/0).

## C
* **CPU (Central Processing Unit):** The "brain" of the PLC that executes the user program and manages hardware communication.
* **CTU / CTD:** Count Up / Count Down. Instructions used to track the number of events or items.

## D
* **DB (Data Block):** A memory area used to store program data. Global DBs are accessible by all blocks; Instance DBs are tied to a specific Function Block.
* **DI / DQ (Digital Input / Digital Output):** Discrete signals. Inputs are usually sensors/buttons; Outputs (Digital Quantities) are usually relays or solenoids.

## F
* **FC (Function):** A logic block that performs a specific task but does not have its own dedicated memory (stateless).
* **FB (Function Block):** A logic block that uses an "Instance Data Block" to remember its state from one scan cycle to the next.

## G
* **GSD / GSDML:** General Station Description. A file provided by manufacturers to allow the PLC to communicate with 3rd-party hardware via PROFIBUS or PROFINET.

## H
* **HMI (Human Machine Interface):** A digital screen or dashboard that allows operators to interact with the PLC and monitor the process.

## I
* **IEC 61131-3:** The international standard for programmable controller programming languages (LAD, SCL, FBD, etc.).
* **IP Address:** The unique numerical identifier for a device on a PROFINET/Ethernet network.

## L
* **LAD (Ladder Logic):** A graphical programming language that resembles electrical relay diagrams.

## M
* **MLFB:** The German acronym for the "Article Number" or part number of a Siemens component (e.g., 6ES7...).
* **Modbus:** A common serial communication protocol used to link industrial electronic devices.

## O
* **OB (Organization Block):** The interface between the PLC Operating System and the User Program. OB1 is the main cyclic loop.

## P
* **PROFINET:** An industry-standard communication protocol based on Ethernet, used for high-speed data exchange between PLCs and I/O.
* **PLCSIM:** The Siemens software tool used to simulate PLC hardware on a computer.

## S
* **Scan Cycle:** The repetitive process where the PLC reads inputs, executes logic, and writes outputs.
* **SCL (Structured Control Language):** A high-level text-based programming language (similar to Pascal) used in TIA Portal.

## T
* **TIA Portal:** Totally Integrated Automation Portal. The engineering software suite by Siemens.
* **TON / TOF:** Timer On-Delay / Timer Off-Delay. Instructions used to manage time-based logic.

## U
* **UDT (User-Defined Data Type):** A custom-defined structure that allows you to group different data types into a single named variable.

---
*Created for the PLC Training Program by Oluwasedago.*
