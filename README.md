# QuPath Alignment of Serial Images (QuASI)

Originally code from https://github.com/MarkZaidi/QuASI

We made the user interface more interactive and robust to use.

List of QuPath scripts for alignment and stain deconvolution of whole-slide histology images (WSIs). Output of `Calculate-Transforms.groovy` is a list of affine transformations that can be used to transform WSIs or annotations. Output of `Apply-Transforms.groovy` is a multichannel ome.tiff containing the transformed image channels appended (and optionally deconvolved if brightfield) to the reference image.
![Alignments are only as good as the quality of the tissue](https://github.com/MarkZaidi/QuPath-Image-Alignment/blob/main/overlay_preview.png?raw=true)
Video tutorial found in https://youtu.be/EvvSsXExYOI?t=1

## Usage

### 00 Rename images in the project

All the files in the project have to fallow a few rules
- Name is composed of two parts separated by underscore `_`
- The part before the underscore is image id. It has to be same for all images
- The part after underscore identifies the staining. (In our example the staining of reference image is `DAB`)

Example:
- `IM1_DAB.vsi`
- `IM1_AEC.vsi`

### (skip) Calculate-Transforms.groovy (skip this part)
Use this script to generate transformation matrices for all images
- Load in all sets of images to be aligned into project. Rename file names such that the only underscore (_) in the image name separates the SlideID from stain. Example: N19-1107 30Gy M5_PANEL2.vsi. Make sure the SlideID is not contained within any other SlideID of an image set that is not intended to be aligned.
- Adjust the inputs specified under "Needed inputs", and run script (can run on any image, iterates over entire project)
- If script errors due to alignment failing to converge, set 'align_specific' to the SlideID of the image it failed on
- Set 'skip_image' to 1, rerun script to skip over the error-causing image
- Set 'skip_image' to 0, and either adjust 'AutoAlignPixelSize' or draw tissue annotations on all stains of images in list
- run script, verify all moving images contain a transform file located in the 'Affine' folder

### 01 Alignment - StoreMatrix automatic.groovy (optional)
Use this script if you want to pull a transformation matrix from the `Interactive Image Alignment` GUI.
You can [download a `.jar` file from the github](https://github.com/qupath/qupath-extension-align/releases) and then 
drag and this file into QuPath
- Open reference image in viewer (`DAB` in our example)
- Open the `Interactive Image Alignment` overlay, align an image
- While the overlay is still open, run the script and  set 'name' to the name of the current moving image in the dialog

### 02 Apply-Transforms.groovy
Use this script to apply the transform to WSIs, appending them together in one large multichannel ome.tiff, optionally including separated stains if brightfield (H&E or HDAB)
 - Run 'Calculate-Transforms.groovy' to generate the necessary transform (tform) matrices required.
 - Set 'refStain' to the same reference stain as used in 'Calculate-Transforms.groovy'
 - Adjust 'variables to set' depending on the degree of downsampling, whether to display in viewer, or write out as an ome.tiff
 - Run script only on images containing 'refStain'
 - Give yourself a pat on the back for actually reading the documentation :)
## To do:
- Bugs
  - when transforming an image set where one of the non-reference images has been rotated on import, ometiff gets somewhat corrupted during writing (tiles will fail during writing)
    - Verified as bug with QuPath by Pete: https://github.com/qupath/qupath/issues/641. Temp solution is to either make sure images are properly oriented prior to import, or perform manual alignment via `Alignment - StoreMatrix automatic.groovy`
- Features
  - Offer ability to select what channel(s) to align based on (implemented, might want to add option to specify different channels for different images)
  - Improve string matching of 'filteredMap' in `Apply-Transforms.groovy` so that if a SlideID is contained within another SlideID of a different image that is not to be aligned to, this won't confuse the script and attempt to append the different image sets together


