# Tutorial data for neuroimaging

This repo is to generate smaller but still useful example datasets.

## What needs doing

My general plan was to experiment with:

* Taking mean of FMRI 4D series
* dipy_median_otsu to make brain mask
* smooth by e.g. 16mm and threshold to widen brain mask
* apply brain mask

Then review the file size after gzip compression.

Git uses `zlib` == `gzip` compression.

The [Git default compression
level](https://git-scm.com/docs/git-config#Documentation/git-config.txt-corecompression)
is `-1`, corresponding to the Zlib default compression level.

At the moment the [Zlib default compression
level](http://www.zlib.net/manual.html) is 6.

In any case, looking at the size of a default `gzip` on the relevant files
will be close or the same as the compression that Git will achieve.

At the moment, of the data files in `psych-214-fall-2016` repository, we have the following sizes from `gzip`:

```
 31M Mar  1 11:34 ds107_sub012_t1r2.nii.gz
 23M Mar  1 11:34 ds114_sub009_t2r1.nii.gz
 21M Mar  1 11:34 y_ds107_sub012_highres.nii.gz
 15M Mar  1 11:34 mni_icbm152_t1_tal_nlin_asym_09a.nii.gz
8.7M Mar  1 11:34 ds114_sub009_highres.nii.gz
6.1M Mar  1 11:34 ds107_sub012_highres.nii.gz
5.3M Mar  1 11:34 ds107_sub001_highres.nii.gz
528K Mar  1 11:34 ds114_sub009_highres_brain_222.nii.gz
474K Mar  1 11:34 mni_icbm152_t1_tal_nlin_asym_09a_masked_222.nii.gz
145K Mar  1 11:34 secret_rotated_volume.nii.gz
141K Mar  1 11:34 rotated_volume.nii.gz
103K Mar  1 11:34 ds114_sub009_highres_brain_mask.nii.gz
 93K Mar  1 11:34 mni_icbm152_t1_tal_nlin_asym_09a_mask.nii.gz
```

## Sketch

Stuff I just did in IPython:

```python
import numpy as np
import nibabel as nib

img = nib.load('ds114_sub009_t2r1.nii')
# No scaling for this image.
print(img.header.slope, img.header.inter)

# Make mean image.
data = np.array(img.dataobj).mean(axis=-1)
mean_data = np.array(img.dataobj).mean(axis=-1)
nib.save(nib.Nifti1Image(mean_data, None, img.header), 'mean_ds114_func.nii')

# Make mask with dipy_median_otsu --save_masked mean_ds114_func.nii
mask = nib.load('dwi_masked.nii.gz')
mask_data = np.array(mask.dataobj)
masked_data = data * mask_data[..., None]
nib.save(
    nib.Nifti1Image(masked_data, None, img.header),
    'masked_ds114_sub009_t2r1.nii')

# Maybe inspect in https://github.com/rordenlab/MRIcroGL
```
