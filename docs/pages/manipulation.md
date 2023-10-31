---
title: Manipulation
layout: default
nav_order: 7
description: "Audio, video, and image file format conversion"
---
# Manipulation: file format conversion

This page details:
* deriving access copies from  high resolution analogue materials that we have had digitised
* deriving access copies of DVD video and CD-DA audio following our ingest procedures

## Image conversion

When we have high resolution images in our collections, such as the TIFF images from in-house digitisation, we should generate a low resolution access copy for research use. We make such access copies in JPEG format, and use [ImageMagick](https://imagemagick.org/) for mass conversion. In our experience, using this without providing any settings other than the input and output files generates acceptable file sizes for our typical use case, e.g. deriving c. 5MB JPEG files from 150-200 MB TIFF images. 

We will typically run this on a whole folder of images at once through Windows PowerShell, targetting the present working directory:

```
gci -filter *.tif | ForEach-Object { & MagickConvert $_.FullName ".\$($_.BaseName).jpg"}
```
Where we have aliased 'MagickConvert' to the full path of the convert.exe from ImageMagick.

## Video conversion

### Single file copies

When we have high resolution video in our collections, such as those copied from analogue film, we should generate a low resolution access copy for research use. We make such access copies in mp4 format, and use [ffmpeg](https://www.ffmpeg.org/) for conversion.

By default, we use the following settings during conversion:
* -c:v libx264
* -movflags faststart
* -pix_fmt yuv420p
* -vf yadif
* -c:a aac
* -hide_banner

```
ffmpeg -i "$in" -c:v libx264 -movflags faststart -pix_fmt yuv420p -vf yadif -c:a aac "$out" -hide_banner
```
Where we have aliased 'ffmpeg' to the path of the ffmpeg.exe file, '$in' to the path of the input file, and '$out' to the path of the desired output file (which will be a .mp4).

### DVDs - concatenation 

When applying our logical copying procedure to DVDs, we will typically be left with multiple .VOB files per disk that need to be concatenated back together to avoid cuts that would not be present when viewing the original in a DVD player. 

There may be one or more tracks (separate videos/episodes) on a disk, and we want to group the files for each of these separately. These can be identified by filename, where filenames beginning 'VTS_01_' are from the first track, 'VTS_02_' from the second, and so on. Typically we will concatenate from VTS_XX_01 onwards, ignoring files ending _0.VOB, which typically store material for use on menus. 

Other than the input being changed to a 'concat' variant, we use the same procedure as for single files, for example for combining 4 such files for the first track in the present working directory:

```
ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB|VTS_01_4.VOB" -c:v libx264 -movflags faststart -pix_fmt yuv420p -vf yadif '..\..\CombinedVideo.mp4'
```
Where we have aliased 'ffmpeg' to the full path of the ffmpeg.exe file, and we send the output (named here 'CombinedVideo.mp4' down two levels, outside of what is assumed to be a VIDEO_TS folder.

## Audio conversion

### Single file copies

For single files we use [ffmpeg](https://www.ffmpeg.org/) for conversion. In our experience, using this without providing any settings other than the input and output files generates acceptable file sizes for our typical use case. We output in either MP3 or M4A.

```
ffmpeg -i "$in" "$out"
```
Where we have aliased 'ffmpeg' to the path of the ffmpeg.exe file, '$in' to the path of the input file, and '$out' to the path of the desired output file (which will be a .mp3 or .m4a).

### CD-DA - breaking up WAV files according to a CUE file

Our ingest procedure for CD-DA audio generates a single WAV audio file along with a CUE file with details of how the tracks are divided.

We use [Audacity](https://www.audacityteam.org/) to convert the files, and we use [J. Scott Smith's 'cue2lbl' tool](http://grimblefritz.com/audacity/cue2lbl.php) tool to convert the CUE files into Audacity labels.

We open the WAV file and import the labels (Import -> Labels) to Audacity, then export as individual files (Export -> Export Multiple). We use all default options (export as MP3, split files based on labels, name files using label/track name). 
