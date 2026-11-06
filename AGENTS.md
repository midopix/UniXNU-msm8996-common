# SoC agent guide (msm8996)

This repository owns shared MSM8996 platform resources: CPU topology,
controller descriptions, SoC configuration fragments and SoC patches.

## Read first

| Task | Required reading |
| --- | --- |
| Every port | [UniXNU porting guide](https://unixnu.com/porting/), [SoC support](https://unixnu.com/porting/soc-support) |
| Hardware | [Device tree](https://unixnu.com/porting/device-tree), [matching](https://unixnu.com/drivers/device-tree-matching), this repository's `dts/msm8996.dtsi` |
| Components | [Component catalog](https://unixnu.com/reference/components), UniXNU `components.toml` |
| Kernel | [Source and patches](https://unixnu.com/xnu/source-and-patches), [4 KiB pages](https://unixnu.com/xnu/four-k-page-tables) |

## Work rules

- Shared controller resources belong here; board wiring, enablement and
  policy belong in the device repository (`UniXNU-g0245d`).
- Every hardware value must cite its source (mainline DTSI, vendor source
  or measured state) in the commit message and in `soc.toml`.
- Do not select `Adreno740`, `AdrenoMetal`, `ShengUfs` or other
  SM8550-specific components here: their hardware scope does not cover
  MSM8996.
- `GICv3` on MSM8996 declares `#interrupt-cells = <3>`. Before enabling
  secondary CPUs, verify `ArmCpu` and the ADT converter against 3-cell
  specifiers and record the result in the device `porting.toml`.

## Verification

```sh
# From the UniXNU checkout, after selecting the device:
./unixnu status --json
./unixnu tree
dtc -I dts -O dtb -i ../UniXNU-msm8996-common/dts -o /tmp/g0245d.dtb ../UniXNU-g0245d/dts/g0245d.dts
```

Record findings in the device repository's `porting.toml` (cpu, memory,
storage, display areas), not here.
