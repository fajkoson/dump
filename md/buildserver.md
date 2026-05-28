# Proposed Rack Build Server BOM

Estimated total: **1,013,143 Kč incl. VAT ≈ 48,425 USD**  
Exchange rate used: **1 USD = 20.922 Kč**

## 1. Component / Price Table

| Component Type | Qty | Component | Link | Availability | Unit Price incl. VAT | Line Total |
|---|---:|---|---|---|---:|---:|
| Rack barebone / case / motherboard / PSU | 1 | Supermicro AS-4125GS-TNRT2, 4U GPU A+ Server | [smicro.cz](https://smicro.cz/supermicro-as-4125gs-tnrt2) | On request | 222,746 Kč | 222,746 Kč |
| CPU | 2 | AMD EPYC 9335, 32C/64T, SP5, 210 W | [compos.cz](https://eshop.compos.cz/amd-cpu-epyc-9005-series-32-64t-model-9335-turin-3-4-4ghz-max-boost-128mb-210w-sp5-tray_d1286092.html) | Within 24 hours | 67,880 Kč | 135,760 Kč |
| RAM | 12 | Kingston 32 GB DDR5-5600 ECC Registered RDIMM (2Rx8) | [digitor.cz](https://www.digitor.cz/cs/304948-kingston-32gb-ddr5-5600mt-s-dimm-cl46-ecc-reg-2rx8-hynix-a-renesas) | Verified direct listing shows 2 pcs in stock; full 12 pcs likely needs split order or seller confirmation | 30,953 Kč | 371,436 Kč |
| Fast VM storage | 2 | Kingston DC3000ME 3.84 TB U.2 NVMe | [senetic.cz](https://www.senetic.cz/product/SEDC3000ME/3T8) | 50+ pcs | 47,478 Kč | 94,956 Kč |
| Snapshot / archive storage | 2 | Kingston DC600M 3.84 TB 2.5" SATA Enterprise SSD | [senetic.cz](https://www.senetic.cz/product/SEDC600M/3840G) | 21-30 pcs | 39,729 Kč | 79,458 Kč |
| CUDA test GPU | 1 | Gainward GeForce RTX 5060 Ti Python III 16G | [alza.cz](https://www.alza.cz/gainward-geforce-rtx-5060-ti-python-iii-16g-d12898929.htm?o=1) | In stock | 13,499 Kč | 13,499 Kč |
| UPS | 1 | APC Smart-UPS X 3000VA LCD NC, rack/tower | [tsbohemia.cz](https://www.tsbohemia.cz/apc-smart-ups-x-3000va-rack-tower-lcd-200-240v-with-network-card_d544259) | On order | 90,290 Kč | 90,290 Kč |
| Windows license | 2 | Microsoft Windows 11 Pro EN OEM | [tsbohemia.cz](https://www.tsbohemia.cz/ms-windows-11-pro-64-bit-cz-oem-dvd_d390385) | Stocked on several branches | 3,799 Kč | 7,598 Kč |
| Hypervisor | 1 | Proxmox VE Free | [proxmox.com](https://www.proxmox.com/en/proxmox-virtual-environment/overview) | Downloadable | 0 Kč | 0 Kč |

## Total

| Currency | Total |
|---|---:|
| CZK incl. VAT | **1,013,143 Kč** |
| USD estimate | **48,425 USD** |

---

## 2. Component Reasoning Table

| Component Type | Selected Component | Why This One |
|---|---|---|
| Rack chassis / motherboard / PSU | Supermicro AS-4125GS-TNRT2, 4U GPU A+ Server | This is the safest high-end rack choice because chassis, motherboard, PSU, risers, cooling, and GPU airflow are vendor-matched. Dual EPYC systems are cooling-sensitive, so a validated 4U server platform is lower risk than a generic rack case. |
| CPU | 2× AMD EPYC 9335 | Gives 2×32 physical cores, so each Windows 11 build VM can be mapped to one physical CPU socket. This is the correct platform for dual-socket AMD Zen 5. |
| RAM | 12× 32 GB DDR5-5600 ECC Registered RDIMM (384 GB total) | Uses proper server RDIMMs instead of workstation-style kits. The selected Kingston 2Rx8 module is based on a Czech direct listing that currently shows stock, but not enough pieces for the full build from one shop, so this part of the BOM should be treated as a real in-stock model choice rather than a guaranteed one-shop quantity. The goal is to keep most of the memory assigned to the Windows build VMs and leave only the amount the host actually benefits from. |
| Fast VM storage | 2× Kingston DC3000ME 3.84 TB U.2 NVMe | Use these as a ZFS mirror for Proxmox root, VM disks, and CI workspaces. Enterprise NVMe is preferred over consumer SSDs because sustained writes, endurance, and power-loss behavior matter for VM workloads. |
| Snapshot / archive storage | 2× Kingston DC600M 3.84 TB 2.5" SATA Enterprise SSD | The selected chassis uses 2.5" front bays, and for local snapshots you do not need the same capacity or performance class as the main VM storage. A mirrored pair of 3.84 TB enterprise SATA SSDs is a better fit here: it uses the chassis correctly, gives enough room for short local snapshot retention for two build VMs, and avoids paying for oversized NVMe archive storage you are unlikely to use fully. |
| GPU | RTX 5060 Ti 16 GB | Enough for CUDA build/test validation without wasting money on AI-training-class GPUs. 16 GB VRAM gives better test headroom than 8 GB cards. |
| UPS | APC Smart-UPS X 3000VA LCD NC | Dual EPYC + GPU + storage should not be protected by an undersized UPS. 3000 VA gives safer shutdown margin and avoids dirty VM/ZFS shutdowns during power loss. |
| Windows licenses | 2× Windows 11 Pro EN OEM | Budget assumes one Windows 11 Pro license per VM. Licensing should still be confirmed with a Microsoft reseller because remote/organizational virtualization can trigger Enterprise/VDA requirements. |
| Hypervisor | Proxmox VE Free | Fits the requested free hypervisor setup. CPU pinning, NUMA-aware VM layout, ZFS storage, snapshots, and PCIe GPU passthrough are the core reasons to use it here. |

---

## Proposed Storage Layout

| Pool | Hardware | Layout | Purpose |
|---|---|---|---|
| `rpool` | 2× Kingston DC3000ME | ZFS mirror, ~100-200 GB | Proxmox OS |
| `vm-fast` | Remaining NVMe capacity | ZFS mirror | Windows VM disks, CI workspaces, build temp data |
| `backup-snapshots` | 2× Kingston DC600M 3.84 TB 2.5" SATA Enterprise SSD | Mirror | Local snapshot target inside the build server |
| Off-host backup | Existing backup server | Proxmox Backup Server sync, ZFS send, or rsync | Real backup outside the build server |

Important caveat: reseller pages may describe different front-bay layouts for this platform. Confirm the exact backplane and whether your configuration exposes the 2× SATA bays as expected before ordering the snapshot SSD pair.

## Final Architecture

| VM | CPU Mapping | RAM | Storage | Notes |
|---|---|---:|---|---|
| Windows Build VM 1 | CPU socket 0 | 160 GB | `vm-fast` | Main build/test VM with higher RAM allocation for heavier builds |
| Windows Build VM 2 | CPU socket 1 | 160 GB | `vm-fast` | Parallel build/test VM with higher RAM allocation for heavier builds |
| CUDA testing | GPU passthrough to one VM | Shared by scheduling | `vm-fast` | Keep roughly 64 GB for Proxmox, ZFS, and host overhead; use one GPU unless both VMs need CUDA at the same time |
