# R9700 loaded-idle patches

These are the local Linux changes from the kernel that reported about 1 W idle SoC power on the R9700 while a model was resident.

Base for the VFCT and DF-Cstate patches: Linux master `05441d697f7f9a927264f4058cbbc6a03dfcac9d` (`7.4.0-rc1`).

The measured 1 W result came from the GFXOFF change. With that change removed, the same cards sat around 11–13 W with models resident. VFCT and DF-Cstate are separate and did not produce that result by themselves.

| Commit | What it changes | Result |
| --- | --- | --- |
| VFCT initial PCI bus | Select the VBIOS entry using the bus number captured when the device was discovered | Fixes wrong VBIOS selection after PCI bus renumbering |
| GFX12 compute-idle GFXOFF | Do not keep GFX12 from entering GFXOFF just because a compute context is idle | This is the change that produced about 1 W loaded idle |
| DF-Cstate allow after BACO | After a successful BACO runtime resume, allow DF C-state again on AMD device `1002:7551` | Diagnostic only. It is not the 1 W fix |

The GFXOFF behavior is already present in the base commit above, so that patch is the earlier change and is not meant to be applied again on top of this master snapshot. The other two patches apply to that snapshot.

These readings are firmware-reported SoC power, not wall power. The PCIe resume-retrain change is not included: it recovers a downtrained link and is not part of the 1 W result.
