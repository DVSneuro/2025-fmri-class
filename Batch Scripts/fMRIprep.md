fMRIprep is a gold-standard pipeline for preprocessing. This documentation provides an introductory overview of the nature [for what purpose in the class?], what it takes in and what it generates, the basic commands and flags. For in depth study or reference, see [link](https://fmriprep.org/en/stable/usage.html#execution-and-the-bids-format). 


# What fMRIPrep really is
fMRIprep is an electic preprocessing pipeline. You are not running a self-contained program per se; you are configuring a workflow that calls dozens of neuroimaging tools based on your dataset structure.

- A **BIDS-aware** workflow engine (Nipype) that orchestrates:
  - FSL (BET, FLIRT, FNIRT)
  - FreeSurfer
  - ANTs
  - AFNI
- Runs inside a container (Docker / Singularity / Apptainer)
- Produces standardized derivatives, not stats


# Input requirements: BIDS conforming inputs and naming conventions
The first requirement is freesurfer license, so please download one we prepared from [here](https://github.com/DVSneuro/2025-fmri-class/blob/main/Utilities/fs_license.txt) and move it to your desktop.  
As for the inputs: 
```
### required
my_dataset/
├── dataset_description.json
├── sub-01/
│   ├── anat/
│   │   └── sub-01_T1w.nii.gz
│   └── func/
│       ├── sub-01_task-memory_run-01_bold.nii.gz #raw nifti
│       └── sub-01_task-memory_run-01_bold.json 
### optional or required in later processing steps
│       └── sub-01_task-memory_run-01_events.tsv  # task-based event file (not three-column format yet)
└── fmap/ # field map
    ├── sub-01_phasediff.nii.gz
    └── sub-01_magnitude1.nii.gz
```


# Minimum command
```
fmriprep
/raw_data_root \ # BIDS-conforming directory of your input 
/data_root/derivatives/fmriprep \ # where fmriprep output lies by BIDS convention
participant \ # suggesting performing subject-level analyses
--fs-license-file /home/jovyan/Desktop/fs_license.txt # freesurfer license is a must, even when explicit freesurfer steps can be skipped
```
This minimal input will allow you to perform fmriprep standard preprocessing steps on all the subjects in your `/raw_data_dir` and place all the default output in `/data_root/derivatives/fmriprep`. 

# Default workflow and outputs
1. Standard preprocessing workflow includes:
- Anatomical preprocessing (T1w)
  - Bias field correction (ANTs N4)
  - Skull-stripping (ANTs-based)
  - Tissue segmentation (GM/WM/CSF)
  - Spatial normalization:
  - Registers T1w → MNI152NLin2009cAsym template
  - FreeSurfer recon-all surfaces if FreeSurfer license is available (it’s required)
 
- Functional preprocessing (BOLD): for all functional runs found in your BIDS dataset:
  - Reference volume extraction (boldref)
  - Motion correction (MCFLIRT / realignment)
  - Susceptibility distortion correction (if fieldmaps exist)
  - Co-registration of BOLD → T1w
  - Spatial normalization to MNI152NLin2009cAsym
  - Resampling to standard space
  - Generation of brain masks in functional space
  - Confound extraction:
    - Motion parameters (trans_x, rot_y, etc.)
    - Framewise displacement
    - aCompCor / tCompCor (physiological noise or variabilities extracted using PCA)
    - Cosine drift regressors (equivalent of high-pass filter)
Note: fMRIprep **does not** do smoothing and leaves this step for the user's later discretion. 

2. Outputs generated: In derivatives/fmriprep/, for each subject, you will get output structured as:
```
derivatives/
 ├── fmriprep/
 │   ├── sub-01/
 │   │   ├── anat/
 │   │   ├── func/
 │   │   └── figures/
 │   └── logs/
```
with the following files in
- `anat/`
  - Skull-stripped T1w
  - Tissue probability maps
  - FreeSurfer outputs (if license)
- `func/`
  - Preprocessed BOLD in:
    - Native T1w space
    - MNI152NLin2009cAsym space
  - Brain masks
  - Confound time series (desc-confounds_timeseries.tsv)
- `logs/` with workflow details


# Useful flags
```
--participant_label 001 \ # dictating which subject your are dealing with; or $sub if you are using a wrapper script that passes on participant one at a time
--work-dir /home/jovyan/work \
--skip_bids_validation \ # if you are certain the data you work with is bid compliant, else fmriprep will either crash or worse do the wrong thing
--n_cpus 3 \ # controlling computing resource allocated to running the job. Alloting 4 will make fmriprep crash on an 8GB memory old computer, 1 on the same computer will get fmriprep going, but might be underutilizing it, so play around with this number given your own computational situation. 
--fs-no-reconall \ # disable the freesurfer surface reconstruction
--output-spaces MNI152NLin6Asym \ # change output space from the default
-fs-no-reconall \ #skip freesurfer surface reconstruction
--me-output-echos \ #multi-echo
--stop-on-first-crash \ # help debugging: Abort immediately when the first node crashes, preserve crash file (crash-*.txt), work directory state and print the first true error, not cascaded failures
```




