# Docker image for Lifebrain structural processing
Changes from original BIDS-app: 
Changed defaults in run.py

Dockerfile:
Added mkdir /cluster /tsd (UiO cluster specific needs)
Updated freesurfer license

## Build instructions
docker build -t lifebrain-fs-structural .

## Convert docker image to singularity for processing on HPC. 

docker run \
-v /var/run/docker.sock:/var/run/docker.sock \
-v ./singularity:/output \
--privileged -t --rm \
singularityware/docker2singularity \
lifebrain-fs-structural

Tested on UiO Slurm cluster. 
