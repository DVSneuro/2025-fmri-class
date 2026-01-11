In the earlier lab material, we operated FSL GUI alone on individual runs, subjects and contrasts in L1, L2, and L3 analyses. However, for data structured and named in BIDS format, using batch scripts allows us to save this manual labor.
This set of documents walks you through these scripts for each level of analyses using FSL line by line. 

# Overview
Conceptually, each level of analysis has two scripts
- run_LXstats.sh = job scheduler/ dispatcher
- LXstats.sh = FEAT engine for single-run in L1/single-subject in L2/single-contrast in L3
  
The focus of the current document, **run_L1stats.sh** and **L1stats**, this pair of bash scripts together implements a parallelized Level-1 GLM pipeline in FSL, supporting three analysis types:

- Activation GLM (standard task regressors)
- Seed-based PPI
- Network-based PPI (nPPI)

They are designed to:
1. Loop over subjects × runs × tasks × PPI types
2. Run analyses in MNI space on fMRIPrep outputs
3. Avoid rerunning completed analyses
4. Control compute load manually (job throttling)
5. Use template .fsf files and sed to programmatically generate FEAT designs
6. Enforce activation → PPI dependency
7. Fix FEAT’s registration assumptions to avoid re-registration

# Required Inputs
To run L1 activation analyses, you need to have
1. The list of subject numbers that you can use to locate the imaging file in BIDS structure that you are performing L1 analyses on
2. fMRIprep output for each subject's run 
3. confound files for each subject's run
4. Three-column event file for each subject's run
5. L1 template with the information job or subject-specific information replaced by placeholders for 

To run L1 PPI/network PPI analyses, you need to 
1. first complete L1 activation analyses.
2. having resliced gradient network mask and resliced and binarized ROI masks. 

# The wrapper script: run_L1stats.sh

```bash
#!/bin/bash
# Dispatcher script for Level-1 FEAT analyses.
# This script loops over tasks, PPI modes, subjects, and runs,
# and launches L1stats.sh jobs in parallel while limiting concurrency.

# --------------------------------------------------
# Robustly determine the directory where this script lives,
# regardless of where it is called from.
# --------------------------------------------------
scriptdir="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"

# Define project base directory as the parent of the script directory
basedir="$(dirname "$scriptdir")"

# Number of functional runs per subject
nruns=2

# --------------------------------------------------
# Loop over tasks
# --------------------------------------------------
for task in sharedreward; do

    # --------------------------------------------------
    # Loop over PPI modes
    #   0   = activation analysis
    #   str = seed-based or network-based PPI
    # Putting 0 first ensures activation runs before PPI
    # --------------------------------------------------
    for ppi in 0; do
   # for ppi in "dmn" "VS"; do


        # --------------------------------------------------
        # Loop over subjects (read from text file): sub_alll.txt contains a column of subject number
        # --------------------------------------------------
        for sub in `cat ${basedir}/code/sub_all.txt`; do
        

            # --------------------------------------------------
            # Loop over runs
            # --------------------------------------------------
            for run in `seq $nruns`; do
            # for run in 1; do

                # --------------------------------------------------
                # Job throttling: limit number of concurrent L1stats.sh jobs
                # --------------------------------------------------
                SCRIPTNAME=${basedir}/code/L1stats.sh
                NCORES=4   # maximum number of parallel jobs allowed

                # While the number of running L1stats.sh processes
                # is greater than or equal to NCORES, wait
                while [ $(ps -ef | grep -v grep | grep $SCRIPTNAME | wc -l) -ge $NCORES ]; do
                    sleep 5s
                done

                # --------------------------------------------------
                # Launch Level-1 analysis in background
                # Arguments:
                #   1 = subject ID
                #   2 = run number
                #   3 = ppi mode
                #   4 = task name
                # --------------------------------------------------
                bash $SCRIPTNAME $sub $run $ppi $task &

                # Small delay to avoid overwhelming the system
                sleep 1s

                # Logging
                echo "complete ${sub} ${run}"
            done
        done
    done
done
```

# The main script: L1stats.sh 
```
#!/usr/bin/env bash
# --------------------------------------------------
# Level-1 FEAT analysis script
#
# This script runs a single first-level GLM in FSL
# for one subject, one run, and one model type.
#
# Supported analyses:
#   1) Activation GLM
#   2) Seed-based PPI
#   3) Network-based PPI (nPPI)
#
# IMPORTANT:
#   Activation analyses must be run before PPI analyses.
# --------------------------------------------------

# --------------------------------------------------
# Robust path resolution
# --------------------------------------------------
scriptdir="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
maindir="$(dirname "$scriptdir")"

# Root directory containing fMRIPrep outputs
rf1datadir=/ZPOOL/data/projects/rf1-sra-data

# --------------------------------------------------
# Study-specific parameters
# --------------------------------------------------
# Spatial smoothing kernel (mm FWHM)
# Must match the value expected in the FEAT .fsf templates
sm=6

# --------------------------------------------------
# Input arguments (passed from run_L1stats.sh)
# --------------------------------------------------
sub=$1       # subject ID
run=$2       # run number
ppi=$3       # 0 = activation, otherwise seed/network label
TASK=$4      # task name

# --------------------------------------------------
# Output directory for subject-level FSL derivatives
# --------------------------------------------------
MAINOUTPUT=${maindir}/derivatives/fsl/sub-${sub}
mkdir -p $MAINOUTPUT

# --------------------------------------------------
# Functional data input
# Preprocessed by fMRIPrep and already in MNI space
# --------------------------------------------------
DATA=${rf1datadir}/derivatives/fmriprep/sub-${sub}/func/sub-${sub}_task-${TASK}_run-${run}_part-mag_space-MNI152NLin6Asym_desc-preproc_bold.nii.gz

# Number of time points (needed by FEAT)
NVOLUMES=`fslnvols $DATA`

# --------------------------------------------------
# Confounds file (formatted for FEAT)
# --------------------------------------------------
CONFOUNDEVS=${rf1datadir}/derivatives/fsl/confounds/sub-${sub}/sub-${sub}_task-${TASK}_run-${run}_part-mag_desc-fslConfounds.tsv

# Abort if confounds are missing
if [ ! -e $CONFOUNDEVS ]; then
    echo "missing confounds: $CONFOUNDEVS " >> ${maindir}/re-runL1.log
    exit
fi

# --------------------------------------------------
# Event (EV) directory
# --------------------------------------------------
EVDIR=${maindir}/derivatives/fsl/EVfiles/sub-${sub}/${TASK}/run-${run}

# --------------------------------------------------
# Handle missed-trial EVs
# If the EV exists, use a real EV shape;
# otherwise use a dummy shape to preserve design structure
# --------------------------------------------------
EV_MISSED_TRIAL=${EVDIR}_decision-missed.txt
if [ -e $EV_MISSED_TRIAL ]; then
    SHAPE_MISSED_TRIAL=3
else
    SHAPE_MISSED_TRIAL=10
fi

# ==================================================
# NETWORK-BASED PPI (nPPI)
# ==================================================
if [ "$ppi" == "ecn" -o  "$ppi" == "dmn" ]; then

    # --------------------------------------------------
    # Output directory for nPPI analysis
    # --------------------------------------------------
    OUTPUT=${MAINOUTPUT}/L1_task-${TASK}_model-1_type-nppi-${ppi}_run-${run}_sm-${sm}

    # Skip analysis if output already exists
    if [ -e ${OUTPUT}.feat/cluster_mask_zstat1.nii.gz ]; then
        exit
    else
        echo "missing feat output 1: $OUTPUT " >> ${maindir}/re-runL1.log
        rm -rf ${OUTPUT}.feat
    fi

    # --------------------------------------------------
    # Require activation FEAT output to exist
    # Used for brain masking during network time-series extraction
    # --------------------------------------------------
    MASK=${MAINOUTPUT}/L1_task-${TASK}_model-1_type-act_run-${run}_sm-${sm}.feat/mask
    if [ ! -e ${MASK}.nii.gz ]; then
        echo "cannot run nPPI because you're missing $MASK"
        exit
    fi

    # --------------------------------------------------
    # Extract time series from canonical network masks
    # --------------------------------------------------
    for net in `seq 0 9`; do
        NET=${maindir}/masks/networkmasks/nan_rPNAS_2mm_net000${net}.nii.gz
        TSFILE=${MAINOUTPUT}/ts_task-${TASK}_net000${net}_nppi-${ppi}_run-${run}.txt

        # Extract demeaned network time series within activation mask with 
        fsl_glm -i $DATA -d $NET -o $TSFILE --demean -m $MASK

        # Dynamically create variables INPUT0 ... INPUT9 and assign value with TSFILEs
        eval INPUT${net}=$TSFILE
    done

    # --------------------------------------------------
    # Define main and control networks
    # --------------------------------------------------
    DMN=$INPUT3
    ECN=$INPUT7

    if [ "$ppi" == "dmn" ]; then
        MAINNET=$DMN
        OTHERNET=$ECN
    else
        MAINNET=$ECN
        OTHERNET=$DMN
    fi

    # --------------------------------------------------
    # Generate FEAT design from template
    # --------------------------------------------------

    # locate the general input L1 template
    ITEMPLATE=${maindir}/templates/L1_task-${TASK}_model-1_type-nppi.fsf
    # create an empty L1 output template (the file actually executed by FEAT in each iteration of L1stats.sh)
    OTEMPLATE=${MAINOUTPUT}/L1_task-${TASK}_model-1_nppi-${ppi}_run-${run}.fsf

    # swap the placeholder in the input template with specific information defined earlier in the scripts,
    # e.g., string "DATA" will be replaced by the value of $DATA in this script, and write it in the output template.
    sed -e 's@OUTPUT@'$OUTPUT'@g' \
        -e 's@DATA@'$DATA'@g' \
        -e 's@EVDIR@'$EVDIR'@g' \
        -e 's@EV_MISSED_TRIAL@'$EV_MISSED_TRIAL'@g' \
        -e 's@SHAPE_MISSED_TRIAL@'$SHAPE_MISSED_TRIAL'@g' \
        -e 's@CONFOUNDEVS@'$CONFOUNDEVS'@g' \
        -e 's@MAINNET@'$MAINNET'@g' \
        -e 's@OTHERNET@'$OTHERNET'@g' \
        -e 's@INPUT0@'$INPUT0'@g' \
        -e 's@INPUT1@'$INPUT1'@g' \
        -e 's@INPUT2@'$INPUT2'@g' \
        -e 's@INPUT4@'$INPUT4'@g' \
        -e 's@INPUT5@'$INPUT5'@g' \
        -e 's@INPUT6@'$INPUT6'@g' \
        -e 's@INPUT8@'$INPUT8'@g' \
        -e 's@INPUT9@'$INPUT9'@g' \
        -e 's@NVOLUMES@'$NVOLUMES'@g' \
        <$ITEMPLATE> $OTEMPLATE

    feat $OTEMPLATE

# ==================================================
# ACTIVATION or SEED-BASED PPI
# ==================================================
else

    # --------------------------------------------------
    # Determine analysis type
    # --------------------------------------------------
    if [ "$ppi" == "0" ]; then
        TYPE=act
        OUTPUT=${MAINOUTPUT}/L1_task-${TASK}_model-1_type-${TYPE}_run-${run}_sm-${sm}
    else
        TYPE=ppi
        OUTPUT=${MAINOUTPUT}/L1_task-${TASK}_model-0_type-${TYPE}_seed-${ppi}_run-${run}_sm-${sm}
    fi

    # Skip completed analyses
    if [ -e ${OUTPUT}.feat/cluster_mask_zstat1.nii.gz ]; then
        exit
    else
        echo "missing feat output 2: $OUTPUT " >> ${maindir}/re-runL1.log
        rm -rf ${OUTPUT}.feat
    fi

    # --------------------------------------------------
    # Create FEAT design
    # --------------------------------------------------
    ITEMPLATE=${maindir}/templates/L1_task-${TASK}_model-1_type-${TYPE}.fsf
    OTEMPLATE=${MAINOUTPUT}/L1_sub-${sub}_task-${TASK}_model-1_type-${TYPE}_run-${run}.fsf

    # Activation GLM: swap the placeholder in the input template with specific information defined earlier in the scripts, e.g., string "DATA" will be replaced by the value of $DATA in this script, and write it in the output template. 
    if [ "$ppi" == "0" ]; then
        sed -e 's@OUTPUT@'$OUTPUT'@g' \
            -e 's@DATA@'$DATA'@g' \
            -e 's@EVDIR@'$EVDIR'@g' \
            -e 's@EV_MISSED_TRIAL@'$EV_MISSED_TRIAL'@g' \
            -e 's@SHAPE_MISSED_TRIAL@'$SHAPE_MISSED_TRIAL'@g' \
            -e 's@SMOOTH@'$sm'@g' \
            -e 's@CONFOUNDEVS@'$CONFOUNDEVS'@g' \
            -e 's@NVOLUMES@'$NVOLUMES'@g' \
            <$ITEMPLATE> $OTEMPLATE

    # Seed-based PPI
    else
        PHYS=${MAINOUTPUT}/ts_task-${TASK}_mask-${ppi}_run-${run}.txt
        MASK=${maindir}/masks/seed-${ppi}.nii.gz

        # Extract physiological regressor (one column of eigenvariate across time) based on preprocessed data and ROI mask, and write into PHYS regressor file
        fslmeants -i $DATA -o $PHYS -m $MASK --eig

        # swapping placeholders in input template with specific information
        sed -e 's@OUTPUT@'$OUTPUT'@g' \
            -e 's@DATA@'$DATA'@g' \
            -e 's@EVDIR@'$EVDIR'@g' \
            -e 's@EV_MISSED_TRIAL@'$EV_MISSED_TRIAL'@g' \
            -e 's@SHAPE_MISSED_TRIAL@'$SHAPE_MISSED_TRIAL'@g' \
            -e 's@PHYS@'$PHYS'@g' \
            -e 's@SMOOTH@'$sm'@g' \
            -e 's@CONFOUNDEVS@'$CONFOUNDEVS'@g' \
            -e 's@NVOLUMES@'$NVOLUMES'@g' \
            <$ITEMPLATE> $OTEMPLATE
    fi

   # running the L1 job based on the specified output template
    feat $OTEMPLATE
fi

# ==================================================
# Fix FEAT registration (data already in MNI space)
# ==================================================
mkdir -p ${OUTPUT}.feat/reg
ln -s $FSLDIR/etc/flirtsch/ident.mat ${OUTPUT}.feat/reg/example_func2standard.mat
ln -s $FSLDIR/etc/flirtsch/ident.mat ${OUTPUT}.feat/reg/standard2example_func.mat
ln -s ${OUTPUT}.feat/mean_func.nii.gz ${OUTPUT}.feat/reg/standard.nii.gz

# ==================================================
# Clean up large intermediate files
# ==================================================
rm -rf ${OUTPUT}.feat/stats/res4d.nii.gz
rm -rf ${OUTPUT}.feat/stats/corrections.nii.gz
rm -rf ${OUTPUT}.feat/stats/threshac1.nii.gz
rm -rf ${OUTPUT}.feat/filtered_func_data.nii.gz
```
