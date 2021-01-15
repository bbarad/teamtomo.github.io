# Introduction

In this tutorial, we will reprocess a five tilt-series subset of 
[*EMPIAR-10164*](https://www.ebi.ac.uk/pdbe/emdb/empiar/entry/10164/), 
a cryo-ET dataset of HIV-1 virus like particles (VLPs) from the 
[Briggs group](https://www2.mrc-lmb.cam.ac.uk/groups/briggs/).

Data will be processed *ab initio*, highlighting many important concepts and 
exposing you to powerful tools which will help with your own data processing needs.

```{note}
This subset was used to benchmark both 
[NovaCTF](https://www.sciencedirect.com/science/article/pii/S1047847717301272)
and 
[Warp](https://www.nature.com/articles/s41592-019-0580-y).
```


This processing will be carried out combing the 
[Warp-RELION-M](http://www.warpem.com/warp/?page_id=827) 
pipeline with tools from 
[Dynamo](https://wiki.dynamo.biozentrum.unibas.ch/w/index.php/Main_Page).
Combination of these tools yields a powerful, 
flexible framework for *ab initio* and geometrical approaches to subtomogram averaging,
benefitting from multi-particle refinement in [M](http://www.warpem.com/warp/?page_id=827).

From these five tilt-series this procedure can produce a 3.4 Å reconstruction 
of the HIV-1 CA-SP1 hexamer.

## A note about scripting
As the software ecosystem for cryo-ET matures, some scripting is often required to achieve your goals. 
This tutorial is provided with all necessary 
[scripts](https://github.com/open-subtomo/open-subtomo/tree/master/guides/EMPIAR-10164/scripts).

For illustrative purposes, the contents of these scripts will appear as `code blocks` in the tutorial when used. 

```matlab
% this is a comment
disp('Hello World!')
```

Reading these blocks is intended to serve as a gentle introduction to MATLAB scripting for working with
the powerful tools Dynamo has to offer. 

For general purpose scripting, we personally prefer `Python` for its wealth of open source
[data science infrastructure](https://www.scipy.org/) 
and 
[educational resources](https://www.youtube.com/watch?v=5rNu16O3YNE&t=4103s).

## Requirements and Setup
```{tabbed} Hardware
- A reasonably modern CPU. 
- At least 32 GB RAM.
- At least 1 NVIDIA GPU, with at least 8 GB of dedicated memory.
- At least 10 GB of free disk space.

More RAM may be required for particle extraction at the smallest pixel sizes, we used a machine with 128GB.
To keep read/write times low, we recommend a Solid State Drive (SSD).
```


```{tabbed} Software
- Access to both Windows and Linux operating systems
- [MATLAB](https://fr.mathworks.com/products/matlab.html) r2019a or later
- [IMOD](https://bio3d.colorado.edu/imod/) 
- [Warp](www.warpem.com/) v1.0.9 or later
- [Relion](https://www3.mrc-lmb.cam.ac.uk/relion/index.php?title=Main_Page) 3.0.8 or later
- [Dynamo](https://wiki.dynamo.biozentrum.unibas.ch/w/index.php/Main_Page) 1.1.478 or later
- [dynamo2m](https://github.com/alisterburt/dynamo2m)
- [autoalign_dynamo](https://github.com/alisterburt/autoalign_dynamo)
- [scripts](https://github.com/open-subtomo/open-subtomo/tree/master/guides/EMPIAR-10164/scripts) provided with this guide
```

## Downloading the data
The five tilt series subset of EMPIAR-10164 used for this guide should first be downloaded from 
[EMPIAR](https://www.ebi.ac.uk/pdbe/emdb/empiar/entry/10164/). 

For this purpose, we provide a simple shell script 
([`download.sh`](https://github.com/open-subtomo/open-subtomo/blob/master/guides/EMPIAR-10164/scripts/download.sh)).

`download.sh` organises files according to the directory structure expected by the rest of the guide.

````{tabbed} shell script
```bash
echo "Downloading the raw data. This may take a couple of hours!"
echo
echo
for i in 01 03 43 45 54;
do
    echo "===================================================="
    echo "================= Downloading TS_${i} ================"
    wget --show-progress -m -q -nd -P ./mdoc ftp://ftp.ebi.ac.uk/empiar/world_availability/10164/data/mdoc-files/TS_${i}.mrc.mdoc;
    wget --show-progress -m -q -nd -P ./frames ftp://ftp.ebi.ac.uk/empiar/world_availability/10164/data/frames/TS_${i}_*.mrc;
done
```
````

````{tabbed} directory structure
```{code block} bash
.
├── frames
│   ├── TS_01_000_0.0.mrc
│   ├── TS_01_001_3.0.mrc
│   └── ...
└── mdoc
    ├── TS_01.mrc.mdoc
    ├── TS_02.mrc.mdoc
    └── ...
```
````

````{tabbed} file extensions
Files with the `.mrc` extension are multi-frame micrographs for each tilt angle in a tilt series. 

They are in the [MRC2014](https://www.ccpem.ac.uk/mrc_format/mrc2014.php) file format.

Files with the `.mdoc` extension are [SerialEM](https://bio3d.colorado.edu/SerialEM/) metadata files.

They contain metadata about images in a tilt-series.
````
**Downloading the data may take a few hours. Start this early!**

## Documentation links
If you are looking for a more in-depth explanation of a parameter, setting or tool, you are encouraged to check out the documentation of any software packages we used.

- **Warp**: [Warp user guide](http://www.warpem.com/warp/?page_id=51)
- **Dynamo**: [Dynamo wiki](https://www.wiki.dynamo.biozentrum.unibas.ch/w/index.php/)
- **Relion**: [RELION wiki](https://www3.mrc-lmb.cam.ac.uk/relion/index.php/Main_Page)
