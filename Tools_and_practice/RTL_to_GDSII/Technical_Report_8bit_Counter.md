# Technical Report: 8-bit Counter Digital Chip Design (RTL to GDSII)

**Author:** Hai Minh Truong  
**Date:** May 2026  
**Project:** Digital Chip Design Tutorial – 8-bit Counter  
**Tools:** OpenLane 2 + Sky130 PDK (130nm)  

---

## 1. Project Overview

### 1.1 Objective
The main objective of this project is to complete the full digital chip design flow of an **8-bit Counter** — from RTL coding to final GDSII layout — using open-source tools. This project serves as both a practical exercise and a foundation for understanding the complete ASIC design process.

### 1.2 Scope
- Design a functional 8-bit synchronous counter in SystemVerilog
- Perform simulation and verification
- Execute the full RTL-to-GDSII flow using OpenLane 2
- Analyze timing, power, and physical design results
- Generate a complete technical report documenting the process and results

### 1.3 Expected Outcomes
- Working GDSII file of the 8-bit counter
- Understanding of each stage in the ASIC design flow
- Analysis reports (timing, power, DRC, LVS)
- This technical report as documentation

---

## 2. Environment and Tools

| Category          | Tool / Technology              | Version / Details                  | Purpose                     |
|-------------------|--------------------------------|------------------------------------|-----------------------------|
| Operating System  | Windows + WSL2                 | Ubuntu 22.04 / 24.04               | Development environment     |
| PDK               | Sky130                         | Google/SkyWater 130nm              | Open-source process design kit |
| Framework         | OpenLane 2                     | v2.3.10                            | RTL-to-GDSII automation     |
| HDL               | SystemVerilog                  | -                                  | RTL design                  |
| Simulation        | Icarus Verilog / Verilator     | -                                  | Functional simulation       |
| Documentation     | Markdown + VS Code             | -                                  | Technical report            |

---

## 3. Design Flow Overview

This section provides a high-level overview of the complete RTL-to-GDSII design process used in this project. The goal is to give readers a clear picture of the tools and major stages before diving into detailed explanations in later sections.

The design follows the standard OpenLane 2 automated flow, which internally executes approximately 78 steps. For better understanding, these steps are grouped into **15 major stages** as shown below:

### 3.1 Detailed Design Flow Table

| No. | Stage                          | Main Tool(s)                  | Purpose / Scope                                                                 | Key Output                  | Status |
|-----|--------------------------------|-------------------------------|----------------------------------------------------------------------------------|-----------------------------|--------|
| 1   | RTL Design                     | SystemVerilog                 | Describe the 8-bit counter behavior using hardware description language         | RTL Code (.sv)              | Done |
| 2   | RTL Simulation                 | Icarus Verilog                | Verify logical correctness of the design before synthesis                        | Waveforms, Pass/Fail        | Done |
| 3   | Linting                        | Verilator                     | Check for coding style issues, unused signals, and potential bugs                | Lint report                 | Done |
| 4   | Synthesis                      | Yosys                         | Convert RTL into gate-level netlist using standard cells                         | Gate-level Netlist          | Done |
| 5   | Gate-Level Simulation          | Icarus + Cell Library         | Verify that synthesized netlist still behaves correctly                          | GL Simulation results       | Done |
| 6   | Floorplan                      | OpenROAD                      | Define chip area, core area, and IO placement                                    | Floorplan (.def)            | Done |
| 7   | Power Distribution Network     | OpenROAD                      | Create power and ground grid                                                     | PDN structure               | Done |
| 8   | Placement                      | OpenROAD                      | Place standard cells in the core area                                            | Placed design               | Done |
| 9   | Clock Tree Synthesis (CTS)     | OpenROAD                      | Build clock distribution network with low skew                                   | Clock tree                  | Done |
| 10  | Routing                        | OpenROAD                      | Connect all cells with metal wires                                               | Routed design            | Done |
| 11  | Post-PnR Simulation            | Icarus + SDF                  | Simulate with realistic delays after routing                                     | Post-layout waveforms       | Done |
| 12  | Static Timing Analysis         | OpenSTA                       | Check setup/hold timing violations                                               | Timing reports              | Done |
| 13  | Power Analysis                 | OpenROAD                      | Estimate power consumption (internal + switching + leakage)                      | Power report                | Done |
| 14  | Physical Verification          | Magic + Netgen                | Run DRC (Design Rule Check) and LVS (Layout vs Schematic)                        | Clean DRC/LVS report        | Done |
| 15  | GDSII Generation               | OpenLane                      | Export final layout file for fabrication                                         | GDSII file                  | Done |

### 3.2 Tool Summary by Stage

| Design Phase       | Tools Used                     | Category          |
|--------------------|--------------------------------|-------------------|
| RTL & Simulation   | SystemVerilog, Icarus, Verilator | Frontend         |
| Synthesis          | Yosys                          | Synthesis        |
| Physical Design    | OpenROAD (Floorplan to Routing) | Backend          |
| Timing & Power     | OpenSTA, OpenROAD              | Analysis         |
| Verification       | Magic, Netgen                  | Signoff          |
| Final Output       | OpenLane                       | GDSII Export     |

---

## 4. Project Milestones

| No. | Milestone                              | Status     | Target Date | Notes / Deliverables                     |
|-----|----------------------------------------|------------|-------------|------------------------------------------|
| 1   | Environment Setup & Tool Installation  | Done       | -           | OpenLane 2 + Sky130 working              |
| 2   | RTL Design & Basic Simulation          | Done       | -           | 8-bit counter module + testbench         |
| 3   | Full RTL-to-GDSII Flow Execution       | Done       | -           | Successful GDSII generation              |
| 4   | Timing & Power Analysis                | Done       | -           | Reports generated                        |
| 5   | Physical Verification (DRC/LVS)        | Done       | -           | All checks passed                        |
| 6   | Technical Report Completion            | In Progress| -           | This document                            |
| 7   | Final Review & Documentation           | Pending    | -           | Final version of report + GDSII          |

---

## 5. Detailed Execution & Analysis

This section provides in-depth explanations of each stage in the design flow.

### 5.1 Environment Setup

      ─ bash
      # Create virtual environment
      # Create virtual environment
      source ~/openlane-env/bin/activate
      # Install OpenLane with minimal dependencies
      pip install openlane --no-deps
      pip install cloup deprecated ioplace-parser lxml psutil rapidfuzz semver yamlcore httpx rich volare

      # Enable Sky130 PDK
      volare enable --pdk sky130 0fe599b2afb6708d281543108caf8310912f54af
      # Pull OpenLane Docker image
      docker pull ghcr.io/efabless/openlane2:2.3.10
      docker pull ghcr.io/efabless/openlane2:2.3.10


      ─ bash
      alias ol='docker run --rm -it \
        -v $(pwd):/work \
        -w /work \
        -v ~/.volare:/root/.volare \
        -v ~/.volare:/root/.volare \
        openlane'

    Explanation: Due to repeated build failures when installing OpenLane via pip (mainly caused by klayout and    libparse), the Docker-based approach was adopted. This method proved to be stable and reliable.

      ─ bash
      iverilog -g2012 -o counter_sim counter.sv tb_counter.sv
      vvp counter_sim
      # View waveform
      # View waveform

### 5.3 Linting

      ─ bash
      verilator --lint-only -Wall counter.sv

### 5.4 Synthesis

      ─ bash
      yosys

    Inside Yosys:

    Inside Yosys:
      read_verilog -sv counter.sv
      hierarchy -check -top counter
      proc
      opt
      opt
      memory
      techmap
      opt
      write_verilog -noattr counter_netlist.v
      stat
      exit

### 5.5 Gate-Level Simulation

─ bash
iverilog -g2012 -o gl_sim tb_counter.sv counter_netlist.v \
  /usr/share/pdk/sky130_fd_sc_hd/verilog/sky130_fd_sc_hd.v \
  /usr/share/pdk/sky130_fd_sc_hd/verilog/primitives.v
vvp gl_sim

### 5.6 OpenLane Configuration

─ bash
# Create config.json
cat > config.json << 'EOF'
{
    "DESIGN_NAME": "counter",
    "VERILOG_FILES": ["dir::counter.sv"],
    "CLOCK_PORT": "clk",
    "CLOCK_PERIOD": 10.0,
    "FP_CORE_UTIL": 50,
    "PL_TARGET_DENSITY_PCT": 60
}
EOF

### 5.7 – 5.16 OpenLane Physical Design Flow

The full physical design flow was executed using the Docker-based OpenLane command:

```bash
ol config.json
```

The flow completed all **78 stages** successfully.

#### Final Verification Results

- **Antenna Check**: Passed ✅
- **LVS Check**: Passed ✅
- **DRC Check**: Passed ✅

The design is considered manufacturable from a physical verification standpoint.

#### Final GDSII File Location

The final GDSII file is located at:

```
runs/RUN_2026-05-22_00-43-45/final/gds/counter.gds
```

---

## 6. Optimization & Improvements

*(This section will be added later)*

---

## 7. Challenges, Solutions & Lessons Learned

| Challenge | Description | Solution | Lesson Learned |
|---------|-------------|----------|----------------|
| Python installation failures | Repeated build errors with `klayout` and `libparse` | Switched to Docker-based OpenLane | Docker provides a much more stable environment for complex EDA tools |
| Docker authentication issues | GHCR denied access when pulling images | Generated Classic Personal Access Token with `read:packages` scope | GHCR requires token authentication, not regular password |
| WSL on D drive performance | Native compilation was unstable | Accepted limitations and used Docker | WSL performance is significantly better when the distro is on the system drive |
| Missing PDK files | Could not find Sky130 cells for simulation | Used `volare` to download and enable the PDK | The PDK must be explicitly enabled using `volare` |

---

## 8. Conclusion

The 8-bit counter was successfully designed from RTL to GDSII using open-source tools. Despite significant challenges during environment setup, the final design passed all physical verification checks (DRC, LVS, Antenna). The project demonstrated both the power and the current limitations of open-source ASIC design flows.

---

## Appendix

- References: Original tutorial by TS. Đặng Minh Tuấn
- Link: https://dangtuanvk.github.io/chip-design-tutorial/
- Tools Documentation: OpenLane 2, Sky130 PDK

---

**Document Status:** Full report updated with successful results and challenges  
**Last Updated:** May 2026  
**Next Action:** Add more analysis and lessons learned if needed