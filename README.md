# Linux-Kernel-Driven-Enumeration-Solution-cascaded-pcie-hierarchy

### System Top-Level PCIe Architecture

## System Architecture & Hardware Topology

The diagram below details the dual-host high-availability PCIe fabric engineered for this chassis. It highlights the symmetric routing paths from the Active and Standby Broadwell Root Complexes down through the cascading switch layers to the isolated daughter board modules.


 <img width="1408" height="768" alt="image" src="https://github.com/user-attachments/assets/dfa325d7-b422-413b-b960-020dfa54a73f" />

### 1. Active Host lspci -tv Output On the Active Host
### the kernel traverses the transparent primary switch down through the transparent Port 0 of each daughter board, directly enumerating the 5 target endpoints on their respective assigned bus offsets.

```text
 -[0000:00]-+-00.0  Intel Corporation Broadwell Host Bridge/Root Port 0
           \-01.0-[01-9F]--00.0-[02-9F]--+-01.0-[10-1F]--00.0-[11-1F]--+-00.0  Daughter Board PCIe Switch Port 0 (Active Ingress)
                                         |                             +-01.0  Daughter Board PCIe Switch Port 1 (NT-Port to Standby)
                                         |                             +-02.0  Endpoint Device A
                                         |                             +-03.0  Endpoint Device B
                                         |                             +-04.0  Endpoint Device C
                                         |                             +-05.0  NVMe Storage Element
                                         |                             \-06.0  Hardware Accelerator
                                         |
                                         +-02.0-[20-2F]--00.0-[21-2F]--+-00.0  Daughter Board PCIe Switch Port 0 (Active Ingress)
                                         |                             +-01.0  Daughter Board PCIe Switch Port 1 (NT-Port to Standby)
                                         |                             +-02.0  Endpoint Device A
                                         |                             ... [Devices B, C, NVMe, Accel]
                                         |
                                         ... [Slots 2 through 6 follow identical structural bus stepping]
                                         |
                                         \-08.0-[80-8F]--00.0-[81-8F]--+-00.0  Daughter Board PCIe Switch Port 0 (Active Ingress)
                                                                       +-01.0  Daughter Board PCIe Switch Port 1 (NT-Port to Standby)
                                                                       +-02.0  Endpoint Device A
                                                                       +-03.0  Endpoint Device B
                                                                       +-04.0  Endpoint Device C
                                                                       +-05.0  NVMe Storage Element
                                                                       \-06.0  Hardware Accelerator

```

### 2. Standby Host lspci -tv (The Hardware-Correct Isolated View)
### The Standby Host's primary motherboard switch carves out the exact same static slot bus allocations (10-1F, 20-2F, up to 80-8F). However, because the daughter board's Port 1 is an NTB, the configuration space terminates immediately at the slot bridge.The Standby Host sees only the NTB device itself sitting at function .0 of the primary allocated slot bus (e.g., 10:00.0). There is no child bus 11 and there are zero endp

```text
-[0000:00]-+-00.0  Intel Corporation Broadwell Host Bridge/Root Port 0
           \-01.0-[01-9F]--00.0-[02-9F]--+-01.0-[10-1F]--00.0  PLX/Microchip NTB Controller (Daughter Board 0, Port 1 Face)
                                         +-02.0-[20-2F]--00.0  PLX/Microchip NTB Controller (Daughter Board 1, Port 1 Face)
                                         +-03.0-[30-3F]--00.0  PLX/Microchip NTB Controller (Daughter Board 2, Port 1 Face)
                                         +-04.0-[40-4F]--00.0  PLX/Microchip NTB Controller (Daughter Board 3, Port 1 Face)
                                         +-05.0-[50-5F]--00.0  PLX/Microchip NTB Controller (Daughter Board 4, Port 1 Face)
                                         +-06.0-[60-6F]--00.0  PLX/Microchip NTB Controller (Daughter Board 5, Port 1 Face)
                                         +-07.0-[70-7F]--00.0  PLX/Microchip NTB Controller (Daughter Board 6, Port 1 Face)
                                         \-08.0-[80-8F]--00.0  PLX/Microchip NTB Controller (Daughter Board 7, Port 1 Face)

```
