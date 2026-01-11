# Lab 5: Running the Analysis – Level 2 & Level 3 Stats

## Learning Objectives
By the end of this lab, you should be able to:
- Run Level 2 (L2) and Level 3 (L3) analyses using FEAT
- Understand and interpret the output from higher-level analyses

In Level 2 analyses, we average activation across runs for each participant. In Level 3, we average activation across participants.

We will be working with the `srndna-trustgame` dataset. Level 1 preprocessing has already been completed for six subjects (each with 5 runs).

---

## Step 1: Set Up Your Working Directory

### 1.1 Create a Folder
1. Open a terminal.
2. Create a folder for this lab:
```bash
mkdir ~/Lab_5
```

### 1.2 Download the Data
This lab uses a preprocessed dataset (~7 GB) hosted on OneDrive.

⚠️ Due to OneDrive’s limitations, **you must download this file manually via your Neurodesk browser**:


[Download Lab 5 Data from OneDrive](https://tuprd-my.sharepoint.com/:u:/g/personal/tug87422_temple_edu/ESth1ajeuLZIuGqLeXKtv7IBMtM0lwj7zvXogjTt5TVbCA?e=GaaTLT)
This is 7 GB compressed file so will take a while (or more) to download, so please do so ahead of time. 

Once downloaded, move the file into your working directory and unzip it:
```bash
mv ~/Downloads/__forNSCI8010_L2L3stats+PPI.zip ~/Lab_5/
cd ~/Lab_5
unzip __forNSCI8010_L2L3stats+PPI.zip
```

This will extract Level 1 FEAT directories for runs 1–5 for each subject.

---

## Step 2: Running Level 2 (Within-Subject) Analysis

### 2.1. Launch FEAT
In your *fsl* terminal, enter 
```bash
Feat &
```
In the FEAT window:
- Change from **First-level analysis** → **Higher-level analysis**
- Change from **Full analysis** → **Statistics only**

<img width="894" height="435" alt="image" src="https://github.com/user-attachments/assets/043b80c9-87eb-4225-bb80-f38e1bbd526f" />


### 2.2. Configure the Data Tab
- Output directory: `~/Lab_5/L2_OUTPUT`
- Number of inputs: **5** (one for each run)
- Click **Select FEAT directories**:
  - For each row, click the folder icon and navigate to the appropriate run-level `.feat` directory:
    - e.g., `~/Lab_5/__forNSCI8010_L2L3stats+PPI/sub-104/L1_task-trust_model-01_type-act_run-01_sm-6.feat`
    - Repeat for all five runs (for convenience, CTRL-SHIFT-A to select the path, copy-paste (CTRL-V) the first feat path, and change the run number).
![Selecting input FEAT directories L3](https://github.com/DVSneuro/2025-fmri-class/blob/main/images/lab5_image_04.png)


### 2.3. Configure the Stats Tab
- Set **Mixed effects** to: `Fixed Effects`
- Click **Full model setup**
  - EV1 Name: `Run_Average`
  - For each input, set the EV1 value to `1`
![image](https://github.com/user-attachments/assets/250ee0a1-3d9e-4838-9d7f-d0c0e3b5d434)

  - Set 2 contrasts, one positive (+1) one negative (-1)

<img width="342" height="343" alt="image" src="https://github.com/user-attachments/assets/75572b4b-9826-441c-b1d0-af4dc54e11c1" />


### 2.4. Post-Stats Tab
- Leave default settings
- Click **GO** to begin the analysis

---

## Step 3: Running Level 3 (Group-Level) Analysis

### 3.1. Launch FEAT Again
```bash
Feat &
```
In the FEAT window:
- Select **Higher-level analysis**
- Choose **Statistics only**

### 3.2. Configure the Data Tab
<img width="435" height="435" alt="image" src="https://github.com/user-attachments/assets/d4df0a49-66a2-4e37-a3c9-aa597614022d" />

- Output directory: `~/Lab_5/L3_Cope11`
- Set input type to: `3D cope images from FEAT directories`
- Number of inputs: **6** (one per subject)
- Click **Select FEAT directories**, then add the `cope11.feat` directory for each subject.


**Pro-tip:** You can use either kind of terminal to quickly list the paths:
```bash
ls -1d ~/Lab_5/__forNSCI8010_L2L3stats+PPI/sub-1*/L2*-act_sm-6.gfeat/cope11.feat/stats/cope1.nii.gz
```
Then copy the output, and in the FEAT GUI, click **Paste** at the bottom of the selection window.
![image](https://github.com/user-attachments/assets/f6fec011-7668-4211-8016-09c09039aaa3)

Use `Ctrl+V` to paste and press OK.

<img width="595" height="410" alt="image" src="https://github.com/user-attachments/assets/5068207e-5210-42c9-ad8c-34e745505acf" />


### 3.3. Configure the Stats Tab
- Mixed effects: `FLAME 1`
- Click **Full model setup**
  - EV1 Name: `Group_Average`
  - Set all EV1 values to `1`
- Under **Contrasts & F-tests**, set:
  - Number of contrasts: `2`
  - Contrast 1: Name = `positive`, value +1
  - Contrast 2: Name = `negative`, value -1


### 3.4. Post-Stats Tab
- Leave all settings as default
- Click **GO** to begin the analysis

---

## Lab 5 Questions
Answer the following questions in complete sentences.

**Q1.** In Level 2 analysis, why is the "Fixed Effects" option used instead of FLAME 1 when combining runs within a single subject? What assumptions does this method make about the data?

**Q2.** Describe the purpose of the `cope1.nii.gz` file used in Level 3 analysis. What does this file represent, and why is it used as input?

**Q3.** After running your Level 3 (group) analysis, you examined the thresholded z-stat maps. Describe how you determined whether the group-level effect was significant, and what brain regions (if any) showed activation for the contrast of interest.

**Q4.** Why is the use of standard space (e.g., `MNI152_T1_2mm_brain`) important in group-level analyses like Level 3? What problem would occur if this step were skipped?

**Q5.** Explain what the design matrix represents in the Level 3 model. How did you specify it, and what does each row/column correspond to in this context?
