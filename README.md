# FIFO Design (Synchronous & Asynchronous)

This repository contains **RTL implementations and verification testbenches** for both **Synchronous FIFO** and **Asynchronous FIFO** designs using **Verilog HDL**.

The focus of this project is:

* Correct **pointer-based FIFO design**
* Reliable **full and empty detection**
* Safe **clock domain crossing (CDC)** for asynchronous FIFOs
* Waveform-based functional verification

---

## Repository Structure

```text
FIFO/
├── sync_fifo/
│   ├── fifo.v
│   └── fifo_tb.v
├── async_fifo/
│   ├── asy_fifo.v
│   └── asy_fifo_tb.v
├── images/
│   ├── sync_fifo_waveform.png
│   └── async_fifo_waveform.png
└── README.md
```

---

## Synchronous FIFO

### Description

The **Synchronous FIFO** operates entirely within a **single clock domain**.
Both read and write operations are driven by the same clock, simplifying timing and control.

The design uses **binary read and write pointers** to index memory locations.
Data is never shifted inside memory; instead, pointers advance to maintain FIFO ordering.

---

### Module Interface

```verilog
module fifo #(parameter WIDTH = 8, DEPTH = 8)(
    input clk,
    input reset,
    input write,
    input read,
    input [WIDTH-1:0] data_in,
    output reg [WIDTH-1:0] data_out,
    output full,
    output empty
);
```

---

### Key Features

* Single clock domain
* Binary read and write pointers
* Full and empty flag generation
* Supports **simultaneous read and write**
* Pointer-based memory access (no data shifting)

---

### Design Notes

* FIFO depth is **8 entries**
* Pointer width is **3 bits**, matching depth (2³ = 8)
* One FIFO slot is intentionally left unused to avoid full/empty ambiguity
* Reset:

  * Clears pointers
  * Asserts `empty`
  * Deasserts `full`

---

### Waveform Verification

The waveform below demonstrates:

* Correct reset behavior
* Sequential writes and reads
* Proper `full` and `empty` flag transitions
* Data integrity and FIFO ordering

![Synchronous FIFO Waveform](./images/sync_fifo_waveform.png)

---

## Asynchronous FIFO

### Description

The **Asynchronous FIFO** supports **independent read and write clock domains**.

To ensure CDC safety, the design:

* Uses **binary pointers locally**
* Converts pointers to **Gray code** before synchronization
* Employs **double-flop synchronizers** across clock domains

This prevents metastability and enables reliable full/empty detection.

---

### Module Interface

```verilog
module asy_fifo #(parameter WIDTH = 8, DEPTH = 16)(
    input rd_clk,
    input wr_clk,
    input reset,
    input read,
    input write,
    input [WIDTH-1:0] data_in,
    output reg [WIDTH-1:0] data_out,
    output full,
    output empty
);
```

---

### Key Features

* Dual clock domains (`wr_clk`, `rd_clk`)
* Binary-to-Gray and Gray-to-binary conversion
* Double-flop pointer synchronizers
* Extra MSB for wrap-around detection
* Safe simultaneous read/write across domains

---

### Design Notes

* FIFO depth is **16 entries**
* Pointer width includes an **extra MSB** for full/empty distinction
* Full condition:

  * Detected by comparing inverted MSBs of synchronized pointers
* Empty condition:

  * Detected when synchronized pointers are equal
* Reset is assumed to be asserted to **both clock domains**

---

### Waveform Verification

The waveform below confirms:

* Correct data transfer across asynchronous clocks
* Safe CDC behavior
* Accurate `full` and `empty` signaling
* FIFO ordering preserved across clock boundaries

![Asynchronous FIFO Waveform](./images/async_fifo_waveform.png)

---

## Simulation & Verification Strategy

* Separate testbenches for synchronous and asynchronous FIFOs
* Independent clock generation for async FIFO
* Verification scenarios include:

  * Reset
  * Continuous write
  * Continuous read
  * Full condition
  * Empty condition
  * Simultaneous read/write

Waveforms confirm functional correctness in all tested scenarios.

---

## Design Assumptions & Limitations

* FIFO depth is a **power of 2**
* Pointer width is manually matched to FIFO depth
* Reset is assumed to be clean and synchronous to relevant clock domains
* Designs prioritize **correctness and clarity** over performance optimization

---

## Author

**Mohd Arhaan**
