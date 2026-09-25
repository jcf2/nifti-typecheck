
# nifti-typecheck

`nifti-typecheck` is a program that checks a .nii file to determine what it is. This is particularly useful
for NIfTI-2 files, which use the same extension (.nii) as NIfTI-1, but are a 64-bit updated format that can
store non-volumetric data like surfaces. A key issue with .nii is that compression of volumetric data is
encouraged, but compression of surface data is problematic due to lack of support in tools. Thus, knowing
what a .nii file "really" is can be useful in file management.

Install location: `/dartfs/rc/lab/D/DBIC/DBIC/code/bin/nifti-typecheck`

# What it does
`nifti-typecheck` reads a NIfTI‑1 or NIfTI‑2 file image header and classifies the image as one of the following types:
 * volumetric – a regular 3‑D or 4‑D volume
 * surface – a surface‑based file (e.g., CIFTI “surface” variant)
 * cifti – a CIFTI file (identified primarily by intent code 3001)
 * unknown – the type could not be determined

The classification is based on a combination of explicit intent codes and simple dimension‑based heuristics (e.g., 3D → volumetric, 4D → surface). 
See the Python source code for the exact rules.

# Command‑line options
 * `nifti-typecheck <nifti_file_path>` – run the program on the supplied file.
 * `nifti-typecheck <nifti_file_path> -d` – run in debug mode. The program will print the full header dictionary (dimensions, intent code, datatype, endian, etc.) in addition to the image type.

# Caveats
It does not work on compressed images (e.g. .nii.gz); this is because an important motivator for it was determining whether uncompressed .nii files should be compressed (volumetric: yes, surface: no).

The tool relies on heuristics when explicit intent codes are absent. Unusual dimension layouts or non‑standard intent codes could lead to mis‑classification. Ideal usage is on a pool of images where the general types included are known in advance.

Only the most common intent codes are recognised (e.g., 3001 for CIFTI, 1006/3001 for surface). Custom (or future) codes are treated as “unknown”.

# Sample output
```
$ nifti-typecheck /dartfs/rc/lab/D/DBIC/DBIC/archive/HCP/HCP1200/100307/MNINonLinear/100307.corrThickness.164k_fs_LR.dscalar.nii
cifti
```
Or, with detail option,
```
$ nifti-typecheck mybrain.nii -d
volumetric
Header Details:
  version: 1
  dimensions: (91, 109, 91)
  ndim: 3
  magic: n+1
  datatype: 4
  intent_code: 0
  endian: little
```
