# Overview
Parallel to L1 and L2 scripts set, these two scripts for L3 (group-level) analyses consists of 
- `run_L3stats.sh` that loops over all contrasts and all analysis types and act as a parallel job launcher, calling L3stats.sh for each contrast.
- `L3stats.sh` performs Level 3 FEAT analyses for one contrast (cope/contrast of parameter estimates) and one model.

# Input required 
1. An input template where all the L1/L2 input paths (relative path accommodating contrast information), GLM matrix with covariates or ones, and L3 contrast are defined. 
2. The L2/L1 outputs are located in BIDS compliant directory and consistent with the input paths in the input template 

# run_L3stats.sh: passing contrast nature (number and name) and analysis type to `L3stats. sh`
```
#!/bin/bash

# This script loops over all contrasts and calls L3stats.sh for each one.
# It acts as a parallel job launcher for group-level analyses.

# --- robust path handling ---
scriptdir="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
maindir="$(dirname "$scriptdir")"

## optional: create log file
#logs=$maindir/logs
#logfile=${logs}/rerunL3_date-`date +"%FT%H%M"`.log

# --- loop over analysis types ---
for analysis in act; do 	#for analysis in nppi-dmn; do
	analysistype=type-${analysis}  # e.g., "type-act"

	# --- loop over contrasts for the chosen analysis ---
	# Each string: "<copenum> <copename>"
	for copeinfo in "1 C_pun" "2 C_rew" "3 F_pun" "4 F_rew" "5 S_pun" "6 S_rew" "7 C_neu" "8 F_neu" "9 S_neu" \
	                 "10 rew-pun" "11 F-S" "12 F-C" "13 FS-C" "14 rew-pun_F-S" "15 rew-pun_S-C" \
	                 "16 rew-pun_F-C" "17 rew_F-S" "18 rew_S-C" "19 rew_F-C" "20 rew-neu_F-S" \
	                 "21 rew-neu_S-C" "22 reu-neu_F-C" "23 F-SC" "24 rew_F-SC" "25 pun_F-SC" \
	                 "26 rew_pun_F-SC" "27 F_dec" "28 S_dec" "29 C_dec" "30 Face-NonFace" \
	                 "31 pun_F-S" "32 pun_F-C" "33 dec_F-S" "34 dec_F-C"; do

		# split string into two variables
		set -- $copeinfo
		copenum=$1
		copename=$2

		# skip contrast if it does not exist
		if [ "${analysistype}" == "type-act" ] && [ "${copeinfo}" == "23 phys" ]; then
			echo "skipping phys for activation since it does not exist..."
			continue
		fi

		# --- control parallel execution ---
		NCORES=5  # max number of concurrent L3 jobs
		SCRIPTNAME=${maindir}/code/L3stats.sh

		# throttle loop: wait until less than NCORES are running
		while [ $(ps -ef | grep -v grep | grep $SCRIPTNAME | wc -l) -ge $NCORES ]; do
			sleep 1s
		done

		# launch L3stats.sh in background
		bash $SCRIPTNAME $copenum $copename $analysistype &  # optional: $logfile
		sleep 1s  # small delay
		echo "complete ${copeinfo} ${analysistype}"  # status message
	done
done
```
 
# L3stats.sh: th
```
#!/bin/bash

# This script performs Level 3 (group-level) statistics in FSL.
# It handles three types of analyses: 
#   1) Two groups
#   2) Two groups with covariates
#   3) Single group average
# It can also optionally run permutation-based stats (randomise).

# --- ensure robust path handling ---
scriptdir="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"  # directory of this script
maindir="$(dirname "$scriptdir")"  # parent directory (project root)

# --- study-specific inputs ---
task=sharedreward  # task name

N=93  # total number of subjects
MODEL=XX  #how you name your template in the model place for the specific analysis
# --- input arguments ---
copenum=$1                 # contrast number
copenum_thresh_randomise=999 # threshold to decide which contrasts to run randomise on
copename=$2                # contrast name
REPLACEME=$3               # the analysistype in run_L3stats, e.g. "type-act"
logfile=$4                 # logfile path

# --- output folder ---
MAINOUTPUT=${maindir}/derivatives/fsl/L3_task-${task}_model-${MODEL}_type-act_n${N}

mkdir -p $MAINOUTPUT  # create output directory if it doesn't exist

# --- One group analysis --- the structure of the script is almost identical for two group analysis, most changes need to take place in the template.
cnum_pad=`zeropad ${copenum} 2`  # zero-padded contrast number
OUTPUT=${MAINOUTPUT}/L3_task-${task}_${REPLACEME}_cnum-${cnum_pad}_cname-${copename}_onegroup

# --- check if output exists ---
if [ -e ${OUTPUT}.gfeat/cope1.feat/cluster_mask_zstat1.nii.gz ]; then
	# FEAT already exists, can optionally run permutation stats
	cd ${OUTPUT}.gfeat/cope1.feat
	if [ ! -e randomise_tfce_corrp_tstat2.nii.gz ] && [ $copenum -ge $copenum_thresh_randomise ]; then
		# run randomise (permutation-based inference)
		randomise -i filtered_func_data.nii.gz \
		          -o randomise \
		          -d design.mat \
		          -t design.con \
		          -m mask.nii.gz \
		          -T -c 2.6 -n 10000
	fi

else 
	# FEAT does not exist or is incomplete -> rerun
	#echo "running: ${OUTPUT}" >> $logfile
	rm -rf ${OUTPUT}.gfeat  # remove partial output

	# --- create contrast-specific FSF template ---
	ITEMPLATE=${maindir}/templates/L3_template_task-sharedreward_model-${MODEL}_n${N}.fsf
	OTEMPLATE=${MAINOUTPUT}/L3_task-${task}_${REPLACEME}_copenum-${copenum}.fsf

	# replace placeholders in the template
	sed -e 's@OUTPUT@'$OUTPUT'@g' \
	    -e 's@COPENUM@'$copenum'@g' \
	    -e 's@REPLACEME@'$REPLACEME'@g' \
	    -e 's@BASEDIR@'$maindir'@g' \
	    <$ITEMPLATE> $OTEMPLATE

	# run L3 FEAT
	feat $OTEMPLATE

	# --- cleanup intermediate files ---
	rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/stats/res4d.nii.gz
	rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/stats/corrections.nii.gz
	rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/stats/threshac1.nii.gz
	#rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/filtered_func_data.nii.gz  # optional
	rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/var_filtered_func_data.nii.gz
fi
```

# Summary of both scripts
**run_L3stats.sh** loops over all contrasts and analysis types, launches multiple L3stats.sh jobs in parallel and throttling to avoid overloading CPU/memory, while **L3stats.sh** runs Level 3 FEAT analysis for one contrast and one analysis type. These two scripts together implement a safe, restartable, and scalable Level 3 analysis pipeline in FSL.
