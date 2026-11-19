# UniXNU-msm8996-common

Shared platform support for Qualcomm MSM8996 (Snapdragon 820) SoCs. This
repository follows the [UniXNU SoC repository format](https://unixnu.com/porting/soc-support)
and is consumed by device repositories through `feeds.toml`.

Reference device: **Gree G0245D** (格力手机·色界, Snapdragon 820, 4 GB RAM,
64 GB UFS, Adreno 530) — device repository `UniXNU-g0245d`.

## Scope

| Path | Contents |
| --- | --- |
| `soc.toml` | SoC identity, hardware sources, reviewed component selection |
| `dts/msm8996.dtsi` | Shared CPU/GIC/bus/peripheral hardware description |
| `configs/msm8996.config` | Shared XNU kernel configuration fragment |
| `patches/xnu/series` | Ordered SoC patch series (currently empty) |
| `kits/` | Shared platform operations (planned) |

## Hardware summary

| Subsystem | Implementation | Notes |
| --- | --- | --- |
| CPU | 4x Kryo 280 (Cortex-A72 class) | 2+2 clusters, PSCI `smc` |
| Interrupts | GICv3, 3-cell specifiers | D 0x9bc0000 / R 0x9c00000 |
| Timer | ARMv8 architected timer | 19.2 MHz timebase |
| Storage | UFS 2.0 (ufshc@624000) | New `NativeUfs8996` component required |
| GPU | Adreno 530 (gpu@b00000) | New `Adreno530` component required |
| GPU SMMU | qcom,smmu-v2 (iommu@b40000) | arm,mmu-500 architecture |
| Display | MDP5 (mdp@901000) + DSI 28nm | New `Mdp5` component required |
| Serial | BLSP I2C/UART | New `BlspI2c` component required |
| USB | DWC3 (usb@6af8800) | Type-C data path |
| PCIe | 3 controllers (600000/608000/610000) | Wi-Fi sits on PCIe: QCA6174 via CNSS (G0245D board fact) |
| RPM/glink | rpm-msg-ram@68000 | Reuses QcomGlink/QcomSmem/QcomRpmh |
| Serial | BLSP I2C/UART | G0245D: BLSP2 UART2 @ 75b0000 (console), BLSP2 I2C @ 75ba000 (touch) |

Hardware values are transcribed from the mainline Linux `msm8996.dtsi`
(link pinned in `soc.toml`) and cross-checked against the G0245D stock device
tree (board.dtb from the QFIL package). Board-level enablement, RAM size,
reservations and peripheral wiring belong to the device repository; the
verified board overlay lives in `UniXNU-g0245d/dts/g0245d.dts`.

## Known gaps versus SM8550

1. **Interrupt specifier width** — mainline MSM8996 uses
   `#interrupt-cells = <3>`, SM8550 uses 4. `ArmCpu` and the ADT converter
   must be exercised against a 3-cell controller before enabling secondary
   CPUs.
2. **SMMU generation** — MSM8996 apps/adreno SMMUs are `qcom,smmu-v2`
   (arm,mmu-500). `ArmSmmu4K` matches mmu-500; verify the v2 context-bank
   layout.
3. **UFS 2.0** — `ShengUfs` targets UFS 3.x/4.x controllers and cannot be
   reused. A dedicated `NativeUfs8996` driver is planned.
4. **Display** — MDP5 is not DPU. Early bring-up uses the firmware
   framebuffer (GOP + `GopFramebuffer`) exactly like sheng's legacy display
   path; native MDP5 support is later work.
5. **GPU** — Adreno 530 is A5xx. `Adreno740`/`AdrenoMetal` are A7xx-only.
   GPU acceleration needs a new `Adreno530` + A5xx Metal translation stack.

## Status

Hardware bring-up is under way on the reference device. Firmware side (stage 1) is
closed: the Mu-Silicium platform builds, the BootShim container boots and the UniXNU
menu renders on the real panel. Two SoC-level facts were established on hardware and
are worth reading before touching timer or interrupt code:

- **`PcdArmArchTimer*IntrNum` takes GIC INTIDs, not the device-tree PPI index.** The
  8996 PPIs 13/14/11/10 are INTIDs 29/30/27/26. With the PPI values the timer handler
  is registered on SGIs and every `EVT_TIMER` consumer dies silently — a frozen BDS
  countdown and dead keypad debounce, from one defect. Details in
  `UniXNU-g0245d/docs/timer-ppi-finding.md`.
- The vendor `xbl.elf` itself carries the upstream `ArmGicDxe` and `TimerDxe`, so the
  standard ArmPkg timer path applies to this SoC; no Qualcomm private timer exists.

Storage, GPU, display-controller and BLSP drivers are still missing, so the device
remains on the firmware framebuffer. Track per-subsystem progress in the device
repository's `porting.toml`.

## License

BSD-2-Clause-Patent, matching UniXNU.
