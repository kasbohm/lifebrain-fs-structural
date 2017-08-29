# Docker image for Lifebrain structural processing
## Description
This app implements surface reconstruction using Freesurfer. It reconstructs the surface for each subject individually and then creates a study specific template. In case there are multiple sessions the Freesurfer longitudinal pipeline is used (creating subject specific templates) unless instructed to combine data across sessions. The current Freesurfer version is based on: freesurfer-Linux-centos6_x86_64-stable-pub-v6.0.0.tar.gz

The app requires data to be formatted according to the BIDS specification: 

http://bids.neuroimaging.io

https://www.nature.com/articles/sdata201644

In short, input data needs to have this folder structure: <data_dir>/sub<subject_id>/anat/<subject_id>_T1w.nii.gz

## Build instructions
	$ docker build -t lifebrain-fs-structural .

### Convert docker image to singularity for HPC processing

	$ docker run \
	-v /var/run/docker.sock:/var/run/docker.sock \
	-v ./singularity:/output \
	--privileged -t --rm \
	singularityware/docker2singularity \
	lifebrain-fs-structural

The (singularity) app is tested and working on the UiO Colossus Slurm cluster 
(note to self: need to force home dir with -H /path/) 

## Usage
The App has the following command line arguments:

    $ docker run -ti --rm bids/freesurfer --help
    usage: run.py [-h]
                  [--participant_label PARTICIPANT_LABEL [PARTICIPANT_LABEL ...]]
                  [--n_cpus N_CPUS]
                  [--stages {autorecon1,autorecon2,autorecon2-cp,autorecon2-wm,autorecon2-pial,autorecon3,autorecon-all,all}
                            [{autorecon1,autorecon2,autorecon2-cp,autorecon2-wm,autorecon2-pial,autorecon3,autorecon-all,all} ...]]
                  [--template_name TEMPLATE_NAME]
                  [--acquisition_label ACQUISITION_LABEL]
                  [--multiple_sessions {longitudinal,multiday}]
                  [--refine_pial {T2,FLAIR,None,T1only}]
                  [--hires_mode {auto,enable,disable}]
                  [--parcellations {aparc,aparc.a2009s} [{aparc,aparc.a2009s} ...]]
                  [--measurements {area,volume,thickness,thicknessstd,meancurv,gauscurv,foldind,curvind}
                                  [{area,volume,thickness,thicknessstd,meancurv,gauscurv,foldind,curvind} ...]]
                  [-v]
                  bids_dir output_dir {participant,group1,group2}
    FreeSurfer recon-all + custom template generation.

    positional arguments:
      bids_dir              The directory with the input dataset formatted
                            according to the BIDS standard.
      output_dir            The directory where the output files should be stored.
                            If you are running group level analysis this folder
                            should be prepopulated with the results of
                            the participant level analysis.
      {participant,group1,group2}
                            Level of the analysis that will be performed. Multiple
                            participant level analyses can be run independently
                            (in parallel) using the same output_dir. "goup1"
                            creates study specific group template. "group2 exports
                            group stats tables for cortical parcellation and
                            subcortical segmentation.

    optional arguments:
      -h, --help            show this help message and exit
      --participant_label PARTICIPANT_LABEL [PARTICIPANT_LABEL ...]
                            The label of the participant that should be analyzed.
                            The label corresponds to sub-<participant_label> from
                            the BIDS spec (so it does not include "sub-"). If this
                            parameter is not provided all subjects should be
                            analyzed. Multiple participants can be specified with
                            a space separated list.
      --n_cpus N_CPUS       Number of CPUs/cores available to use.
      --stages {autorecon1,autorecon2,autorecon2-cp,autorecon2-wm,autorecon2-pial,autorecon3,autorecon-all,all}
                            [{autorecon1,autorecon2,autorecon2-cp,autorecon2-wm,autorecon2-pial,autorecon3,autorecon-all,all} ...]
                            Autorecon stages to run.
      --template_name TEMPLATE_NAME
                            Name for the custom group level template generated for
                            this dataset
      --acquisition_label ACQUISITION_LABEL
                            If the dataset contains multiple T1 weighted images
                            from different acquisitions which one should be used?
                            Corresponds to "acq-<acquisition_label>"
      --multiple_sessions {longitudinal,multiday}
                            For datasets with multiday sessions where you do not
                            want to use the longitudinal pipeline, i.e., sessions
                            were back-to-back, set this to multiday, otherwise
                            sessions with T1w data will be considered independent
                            sessions for longitudinal analysis.
      --refine_pial {T2,FLAIR,None,T1only}
                            If the dataset contains 3D T2 or T2 FLAIR weighted
                            images (~1x1x1), these can be used to refine the pial
                            surface. If you want to ignore these, specify None or
                            T1only to base surfaces on the T1 alone.
      --hires_mode {auto,enable,disable}
                            Submilimiter (high resolution) processing. 'auto' -
                            use only if <1.0mm data detected, 'enable' - force on,
                            'disable' - force off
      --parcellations {aparc,aparc.a2009s} [{aparc,aparc.a2009s} ...]
                            Group2 option: cortical parcellation(s) to extract
                            stats from.
      --measurements {area,volume,thickness,thicknessstd,meancurv,gauscurv,foldind,curvind}
                            [{area,volume,thickness,thicknessstd,meancurv,gauscurv,foldind,curvind} ...]
                            Group2 option: cortical measurements to extract stats for.
      -v, --version         show program's version number and exit
Participant level

To run it in participant level mode (for one participant):

	docker run -ti --rm \
	-v /Users/filo/data/ds005:/bids_dataset:ro \
	-v /Users/filo/outputs:/outputs \
	bids/freesurfer \
	/bids_dataset /outputs participant --participant_label 01 
Group level

After doing this for all subjects (potentially in parallel) the group level analyses can be run.

To create a study specific template run:

	docker run -ti --rm \
	-v /Users/filo/data/ds005:/bids_dataset:ro \
	-v /Users/filo/outputs:/outputs \
	bids/freesurfer \
	/bids_dataset /outputs group1 
To export tables with aggregated measurements within regions of cortical parcellation and subcortical segementation run:

	docker run -ti --rm \
	-v /Users/filo/data/ds005:/bids_dataset:ro \
	-v /Users/filo/outputs:/outputs \
	bids/freesurfer \
	/bids_dataset /outputs group2 



## Changes from original BIDS-FreeSurfer app: 
### run.py: 
Modified defaults

removed required license argument
If you use this software outside of Lifebrain, please register for a license at https://surfer.nmr.mgh.harvard.edu/fswiki/License

### Dockerfile
Added mkdir /cluster /tsd /work /projects (UiO cluster specific needs)

Updated freesurfer license for lifebrain

## Acknowledgements
Repo cloned from https://github.com/BIDS-Apps/freesurfer. 

Thank you!

Freesurfer:
https://surfer.nmr.mgh.harvard.edu/fswiki/FreeSurferMethodsCitation
