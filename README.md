
# nifti-typecheck

`nifti-typecheck` is a program that checks a .nii file to determine
what it is. This is particularly useful for NIfTI-2 files, which use
the same extension (.nii) as NIfTI-1, but are a 64-bit updated format
that can store non-volumetric data like surfaces. A key issue with
.nii is that compression of volumetric data is encouraged, but
compression of surface data is problematic due to lack of support in
tools. Thus, knowing what a .nii file "really" is can be useful in
file management.

Install location: `/dartfs/rc/lab/D/DBIC/DBIC/code/bin/nifti-typecheck`

# What it does
`nifti-typecheck` reads a NIfTI‑1 or NIfTI‑2 file image header and classifies the image as one of the following types:
 * volumetric – a regular 3‑D or 4‑D volume
 * surface – a surface‑based file (e.g., CIFTI “surface” variant)
 * cifti – a CIFTI file (identified primarily by intent code 30xx)
 * unknown – the type could not be determined

The classification is based on a combination of explicit intent codes and simple dimension‑based heuristics (e.g., 3D → volumetric, 4D → surface). 
See the Python source code for the exact rules.

# Command‑line options

usage: nifti-typecheck [-h] [-d] filepath

where filepath is the path to the file to inspect

optional arguments:
 * `-h`/`--help`: show help message and exit
 * `-d`/`--detail`: Print detailed information

# NIfTI History

NIfTI-1 (2004) was designed primarily for volumetric data (the
committee that designed it did include intent codes for meshes,
but then subsequently drafted a separate companion standard
called GIFTI for triangular surface meshes). It is an open
successor to the de facto standard ANALYZE 7.5 format intended to 
resolve issues like left-right coordinate system confusion, and
it consequently inherited the ANALYZE header's design choices, such as a
minimalist binary header (as of the final 1.1 (2005) an allowance for
custom "extensions" between the header block and data block was added;
see [NIfTI-1.1
FAQ](https://nifti.nimh.nih.gov/nifti-1/documentation/faq.html#Q21)).

The NIfTI-2 standard (2014) was created in large part to address the
limitation of using 16 bit integers for dimensions in NIfTI-1. The
`short dim[8]` in NIfTI-1 that limits to 32,767 per dimension was
changed to us 64-bit integers (which changed the header length from 348
to 540 bytes and shifted offsets for header fields).  When it was,
the Human Connectome Project (HCP), which had created the CIFTI format
to represent data across cortical surfaces (grayordinates) and
subcortical voxels simultaneously, chose NIfTI-2 as its physical
container (as of the final CIFTI-2).

CIFTI files are _supposed_ to use a compound extension
(e.g. ".dtseries.nii"), but if for any reason they end up with a
simple .nii extension, they are indistinguishable from NIfTI-1
volumetric images at the file-naming level. However, their structure
is quite different, typically with a <time x grayordinate> matrix and
an associated XML extension packed as a NIfTI extension (address
mapped access to this block is why compression breaks such files). For
volumetric data the differences are more limited, and the new standard
was intended to require minimal changes for existing header processing
programs and libraries.

Some tools output "bare" .nii files that actually have surface contents.
For example, FreeSurfer's `mri_surf2surf` and `mri_convert` will save
cortical thickness, curvature, or other surface scalar maps as .nii.
These will not necessarily break with compression like CIFTI, but
since they are not traditional volumetric images it can be helpful to
be able to find and identify them.

See:
 * NIfTI-1/1.1: https://nifti.nimh.nih.gov/nifti-1/
 * NIfTI-2: https://nifti.nimh.nih.gov/nifti-2/
 * CIFTI: https://www.nitrc.org/projects/cifti/
 * GIFTI: https://www.nitrc.org/projects/gifti/

# Caveats

It does not work on compressed images (e.g. .nii.gz); this is because
an important motivator for it was determining whether uncompressed
.nii files should be compressed (volumetric: yes, surface: no).

The tool relies on heuristics when explicit intent codes are
absent. Unusual dimension layouts or non‑standard intent codes could
lead to mis‑classification. Ideal usage is on a pool of images where
the general types included are known in advance. When heuristics
are used, the type classification ends with "?".

Only the most common intent codes are recognized (e.g., 3001 for
CIFTI, 1006/3001 for surface). Custom (or future) codes are treated as
“unknown”.

# Sample output
```
[~]$ cd /dartfs/rc/lab/D/DBIC/DBIC/archive/HCP/HCP1200/100307
[100307]$ nifti-typecheck MNINonLinear/100307.corrThickness.164k_fs_LR.dscalar.nii
cifti
```
Or, with `--detail`,
```
[100307]$ nifti-typecheck -d MNINonLinear/100307.corrThickness.164k_fs_LR.dscalar.nii
Classified image type: ImageType.CIFTI
cifti

Header Details:
  version: 2
  dimensions: (1, 1, 1, 1, 1, 298261)
  ndim: 6
  magic: n+2

  datatype: 16
  intent_code: 3006
  endian: little
```