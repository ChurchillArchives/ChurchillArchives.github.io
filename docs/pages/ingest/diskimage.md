---
title: Disk imaging
layout: default
nav_order: 5
parent: Ingest
---
At present, we have a workflow for taking disk (flux) images of floppy disks, and are developing one for working with other material. We currently have the capacity to image 3-inch, 3.5-inch and 5.25-inch floppy disks, and aim to be able to image 8-inch floppy disks in the near future.

## Floppy disks

We use a [greaseweazle](https://github.com/keirf/greaseweazle) controller with original floppy drives to create flux images of floppy disks in our collections, and then derive disk images from the flux images. We prioritise capturing that initial flux image as a preservation action, on the understanding that taking a low level copy of the physical magnetic media is the most important step.

We capture the flux streams in the .hfe format. This necessitates choosing a bitrate for the disk to be copied (generally 250 kbit/s for double density disks and 500 kbit/s for high density disks), and we do not set any other parameters (such as track or side information) at this point, as they can be set when deriving a disk image from the flux copy.

An example command:

```
gw read DDdisk.hfe::bitrate=250
```

For a double density disk, where we have aliased 'gw' to the full path of the greaseweazle.exe.

We then analyse the flux image with [HxC floppy emulator](https://hxc2001.com/download/floppy_drive_emulator/), both to check if the disk copied well (to determine if there was data there at all, whether we need to clean the drive heads and re-image, etc), and also to determine the specifications of the disk (track/sector numbers, number of sides, etc), which we then use to derive the disk image from the flux copy.

Generally we aim to use the greaseweazle software 'convert' command to derive the disk image from the flux copy, but there are occasions when this is not appropriate. Notably, the current version of the greaseweazle software cannot convert to a .DSK file suitable for Amstrad CPC, and so we use an alternative here (conveniently, the HxC floppy emulator does this). 

## Other material

We are still developing a workflow here, but our general intentions are to take disk images from within Windows where possible, using a utility such as [IsoBuster](https://www.isobuster.com/). We recognise that there will be some items for which this will not be suitable, but given that we only intend to disk image a [very small proportion of non-floppy material](https://churchillarchives.github.io/pages/ingest.html), we accept that we will need to deal with them on a case-by-case basis.