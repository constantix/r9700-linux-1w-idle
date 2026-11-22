# R9700 loaded-idle patches

These are the local Linux changes from the kernel that reported about 1 W idle SoC power on the R9700 while a model was resident.

Base for the VFCT and DF-Cstate patches: Linux master `05441d697f7f9a927264f4058cbbc6a03dfcac9d` (`7.4.0-rc1`).

The measured 1 W result came from the GFXOFF change. With that change removed, the same cards sat around 11–13 W with models resident. VFCT and DF-Cstate are separate and did not produce that result by themselves.

| Commit | What it changes | Result |
| --- | --- | --- |
| VFCT initial PCI bus | Select the VBIOS entry using the bus number captured when the device was discovered | Fixes wrong VBIOS selection after PCI bus renumbering |
| GFX12 compute-idle GFXOFF | Do not keep GFX12 from entering GFXOFF just because a compute context is idle | This is the change that produced about 1 W loaded idle |
| DF-Cstate allow after BACO | After a successful BACO runtime resume, allow DF C-state again on AMD device `1002:7551` | Diagnostic only. It is not the 1 W fix |
| PCIe resume retrain | If a link resumes slower than its Target Link Speed, request a retrain | Recovers a Gen 1 link. It is not the 1 W fix |

The GFXOFF behavior is already present in the base commit above, so that patch is the earlier change and is not meant to be applied again on top of this master snapshot. The other patches apply to that snapshot.

These readings are firmware-reported SoC power, not wall power.

## Why the retrain might be needed

On this board, Slot #2's CPU-to-switch link can wake from GPU runtime sleep at 2.5 GT/s x8 even though its Target Link Speed is still 32 GT/s x8. Linux is not capping the speed. The link is up, there is often no AER error, and the bandwidth interrupt only records the new speed. It does not retrain. A D3hot resume also does not take the D3cold power-up path that would retrain a bridge. The working idea is that hardware retrains itself on wake and can land at Gen 1 while software never asks again.

A manual Retrain Link write brought that port back to 32 GT/s. This patch does the same check from the kernel: after resume, if the link is up but slower than its target, retrain toward that target. It does not stop the drop. It recovers it. On the running kernel the request ran 100 times with no logged failure, and the link was back at 32 GT/s afterward. That shows recovery is being requested, not that the hardware cause is fixed.
