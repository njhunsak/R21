---
title: MindBoggle
toc: true
weight: 2
---

## Installation

Build mindboggle:

```bash
docker build -t bids/mindboggle .;
```

Set the path on your host machine for the Docker container to access Mindboggle input and output directories:

```bash
HOST=/Volumes/wasatch/data  # path on host to access input/output
DOCK=/home/jovyan/work  # path to HOST from Docker container
docker run --rm -ti -v $HOST:$DOCK -p 5000 --entrypoint /bin/bash nipy/mindboggle
```

## Running Mindboggle

Set Freesurfer, ANTs, and Mindboggle paths:

```bash
DOCK=/home/jovyan/work
for i in $(ls $DOCK/analyses/R21/FreeSurferv6/); do
    echo "Processing participant $i"
    FREESURFER_SUBJECT=$DOCK/analyses/R21/FreeSurferv6/$i;
    ANTS_SUBJECT=$DOCK/analyses/R21/antsCT_OASIS30/$i;
    MINDBOGGLING=$DOCK/analyses/R21/mindboggling;
    MINDBOGGLED=$DOCK/analyses/R21/mindboggled;
    mindboggle \
    $FREESURFER_SUBJECT \
    --working $MINDBOGGLING \
    --out $MINDBOGGLED \
    --ants $ANTS_SUBJECT/BrainSegmentation.nii.gz \
    --roygbiv
done
```