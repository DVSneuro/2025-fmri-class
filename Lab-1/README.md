# Lab 1: Intro to Linux, MRI Basics, and FSL

## Learning Objectives
By the end of this lab, you should be able to:

- Navigate the terminal in Linux (Ubuntu)
- Open anatomical and functional MRI images using FSL tools
- Extract basic information about MRI images (e.g., voxel size, dimensions)
- Understand the difference between T1- and T2-weighted scans

---

## Part 1: Exploring the Terminal

### 1.1 Open a terminal window
You’ll use the terminal to navigate and run FSL commands.

### 1.2 Navigate to your data directory
```bash
cd ~/ds003745/
```

### 1.3 Check the contents of a subject folder
```bash
cd sub-104/anat
ls -l
```

### 1.4 What is the file size of the anatomical scan?
```bash
ls -lh sub-104_T1w.nii.gz
```

---

## Part 2: Viewing Images in FSL

### 2.1 Open the anatomical image
```bash
fslview_deprecated sub-104_T1w.nii.gz &
```

- Click around the brain to see voxel values.
- Use the histogram tool under **Tools → Image Histogram**.

### 2.2 Adjust image contrast and brightness
Use the GUI window sliders or "Auto" button to improve visibility.

---

## Part 3: Exploring fMRI (Functional) Images

### 3.1 Navigate to the functional image directory
```bash
cd ~/ds003745/sub-104/func
```

### 3.2 Open a functional image
```bash
fslview_deprecated sub-104_task-trust_run-01_bold.nii.gz &
```

- Scroll through volumes using the movie controller.
- Note the difference in contrast compared to the anatomical image.

---

## Part 4: T1 vs T2 Contrast

### 4.1 Open both images simultaneously
```bash
fslview_deprecated ../anat/sub-104_T1w.nii.gz sub-104_task-trust_run-01_bold.nii.gz &
```

- Compare image brightness and tissue types.
- Recall:
  - T1: **white matter appears bright**
  - T2: **CSF appears bright**

---

## Part 5: Estimating Brain Volume

### 5.1 Use `fslstats` to calculate brain volume
```bash
fslstats ../anat/sub-104_T1w.nii.gz -V
```
- Output: `[# of voxels] [voxel volume]`
- Multiply to get total brain volume in mm³.

---

## Summary Questions

**Q1.** What’s the difference between a T1-weighted and a T2-weighted scan? Which one did you view today?

**Q2.** What command did you use to view a histogram of image intensities?

**Q3.** What command did you use to calculate brain volume, and what was the approximate value?

**Q4.** How do you know if the image contrast is T1 or T2? What visual cues do you use?

---

## End of Lab 1