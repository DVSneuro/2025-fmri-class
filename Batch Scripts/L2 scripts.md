Recall that L2 analysis is for averaging L1 output across runs for each subject. Similar to L1 analysis scripts, scripts for batch job L2 is split into the wrapper script `run_L2stats.sh` and the core analysis script `L2stats.sh`
The following is the line-by-line explanation of them: 


# Required Inputs:
- L2 template with relative input values using placeholder such as "INPUT1"
- existing L1 output following BIDS format
- subject list


# Script 1: run_L2stats.sh
Run_L2stats.sh loops over analysis types (e.g., act, PPI variants), grabs all subjects listed in sub_all.txt
and submits each subject to L2stats.sh.
```
#!/bin/bash

# ensure paths are correct irrespective from where user runs the script
scriptdir="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"  # directory where this script lives
maindir="$(dirname "$scriptdir")"  # parent directory, usually project root

# create log file to record what we did and when
logs=$maindir/logs  # logs directory
logfile=${logs}/rerunL2_date-`date +"%FT%H%M"`.log  # timestamped logfile

# loop over analysis types
for type in "act"; do # only "act" analysis active now, others commented out
  #for type in "nppi-dmn"; do # alternative PPI analysis

  for sub in `cat ${scriptdir}/sub_all.txt`; do  # loop over subjects listed in sub_all.txt
	  #for sub in 10964; do  # debug: run single subject

		# define L2 analysis script and max parallel jobs
  		SCRIPTNAME=${maindir}/code/L2stats.sh
  		NCORES=20  # max concurrent jobs

  		# throttle loop: wait if >= NCORES jobs are running
  		while [ $(ps -ef | grep -v grep | grep $SCRIPTNAME | wc -l) -ge $NCORES ]; do
    		sleep 1s
  		done

  		# launch subject-level L2 analysis in background
  		bash $SCRIPTNAME $sub $type $logfile &
  		sleep 1s  # small delay to avoid race conditions

	done

	echo "complete ${sub}"  # prints completion message for current type
done
```

# Script 2: L2stats.sh
L2stats.sh catches the subject and analysis type run_L2stats.sh sending over, and generates standardized input path, output directory and swapping those in the generic L2 analyses template before running it in FEAT. 
```
#!/bin/bash

# ensure paths are correct irrespective of where the user runs the script
scriptdir="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"  # directory of this script
maindir="$(dirname "$scriptdir")"  # parent directory, project root

# --- Inputs and common variables that are passed from run_L2stats.sh's `bash $SCRIPTNAME $sub $type $logfile &`---
sub=$1        # subject ID from Script 1
type=$2       # analysis type (e.g., act, PPI)
logfile=$3    # logfile path
task=sharedreward # task name (edit if needed)
sm=6          # smoothing kernel (edit if needed)


MAINOUTPUT=${maindir}/derivatives/fsl/sub-${sub}  # subject-specific output directory
NCOPES=34      # expected number of contrasts
model=1        # GLM model number

# --- choose L2 template based on analysis type ---
if [ "${type}" == "act" ]; then
	ITEMPLATE=${maindir}/templates/L2_task-${task}_model-${model}_type-act.fsf  # ACT template
	NCOPES=${NCOPES}  # keep original number of contrasts
else
	ITEMPLATE=${maindir}/templates/L2_task-${task}_model-${model}_type-nppi-dmn.fsf  # PPI template
	let NCOPES=${NCOPES}+1  # PPI has one extra contrast
fi

# L1 run-level FEAT inputs for this subject
INPUT1=${MAINOUTPUT}/L1_task-${task}_model-${model}_type-${type}_run-1_sm-${sm}.feat
INPUT2=${MAINOUTPUT}/L1_task-${task}_model-${model}_type-${type}_run-2_sm-${sm}.feat

# --- check for existing output ---
OUTPUT=${MAINOUTPUT}/L2_task-${task}_model-${model}_type-${type}_sm-${sm}  # final L2 output path
if [ -e ${OUTPUT}.gfeat/cope${NCOPES}.feat/cluster_mask_zstat1.nii.gz ]; then
	echo "skipping existing output"  # if analysis already complete (as suggested by the first zstats map), stop current job;
else
	rm -rf ${OUTPUT}.gfeat  # delete partial/broken output if it exists

	# generate subject-specific FSF template
	OTEMPLATE=${MAINOUTPUT}/L2_task-${task}_model-${model}_type-${type}.fsf
	sed -e 's@OUTPUT@'$OUTPUT'@g' \
	-e 's@INPUT1@'$INPUT1'@g' \
	-e 's@INPUT2@'$INPUT2'@g' \
	<$ITEMPLATE> $OTEMPLATE

	# run FEAT for this subject
	feat $OTEMPLATE

	# cleanup intermediate files to save disk space
	for cope in `seq ${NCOPES}`; do
		rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/stats/res4d.nii.gz
		rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/stats/corrections.nii.gz
		rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/stats/threshac1.nii.gz
		rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/filtered_func_data.nii.gz
		rm -rf ${OUTPUT}.gfeat/cope${cope}.feat/stats/var_filtered_func_data.nii.gz
	done

fi
```
