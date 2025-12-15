Throughout this set of lab tutorials, we are operating on Neurodesk, a
neuroimaging computing platform that aims to make neuroimaging analyses
maximally reproducible and universally compatible:


The following instructions are based onthe assumption  that you have a Mac, Windows, or
Linux computer: you can work with the Neurodesk App; if you are using
any other system, please ignore steps 1-2, obtain a GitHub account, and
use [<u>the browser
version</u>](https://nam10.safelinks.protection.outlook.com/?url=https%3A%2F%2Fplay.neurodesk.org%2F&data=05%7C02%7Cshenghan.wang%40temple.edu%7C25cd484e6ecb4fa2fd8208de21a3a307%7C716e81efb52244738e3110bd02ccf6e5%7C0%7C0%7C638985183851618162%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=zEH3HvGLjX0DUOM90tfBL6zPXkoUeaYtnGJ%2FvxDG8ww%3D&reserved=0);
The rest of the setup starts to apply again from setup step 3.



# Hardware requirements

1.  Docker (as the container of the system and software): the host
    machine must support virtualization/containers and have enough
    free resources for both the host OS *and* the container environment.

2.  At least 5GB of free space for the NeuroDesk base image based on the app.

3.  Recommended for learning & moderate use: 4–8 cores, 16–32 GB RAM,
    SSD with ~50–100 GB free space.
    a.  8 GB RAM is doable but slow (tested). Be sure to exit out all other
        resource-hungry applications (e.g., browser) up memory to free if
        Your computer is reluctant in its capacity.
 

# Setup steps

1.  [<u>Download</u>](https://iishiishii.github.io/neurodesk.github.io/getting-started/local/neurodeskapp/)
    **NeuroDesk** and **Docker** installation packages for your system
    and install: restart your system if prompted. Installing docker
    requires administrator permission.

2.  Open Docker, wait for it to initialize, then open Neurodesk, click
    on "Open Local Neurodesk"

> <img src="./images/pre_lab/media/image1.png"
> style="width:6.5in;height:4.25208in"
> alt="A screenshot of a computer AI-generated content may be incorrect." />

3.  First time booting Neurodesk takes longer for the container to be
    populated - **be sure to stay connected to the internet during the
    process**. The latest progress will be displayed at the bottom of
    the sliding window. As long as new steps are completed every
    couple of minutes, it's working without a glitch. Wait for a
    while (up to 40 min).

 

<img src="./images/pre_lab/media/image2.png"
style="width:6.5in;height:4.10278in"
alt="A screenshot of a computer AI-generated content may be incorrect." />

4.  When it's properly initiated, click on the **Neurodesktop** icon

> <img src="./images/pre_lab/media/image3.png"
> style="width:6.5in;height:3.41597in"
> alt="A screenshot of a computer AI-generated content may be incorrect." />

5.  Click on either Desktop RDP or Desktop VNC, this should take you to
    the virtual machine interface.

> <img src="./images/pre_lab/media/image4.png"
> style="width:6.5in;height:4.89028in"
> alt="A computer screen shot of a computer screen AI-generated content may be incorrect." />
>
Operate on the VM just as you do with your average computer, the
functions you are going to use the most are highlighted from left
to the right: file system, browser (Firefox), terminal, and the most
recent version of FSL and FSLeyes

<img src="./images/pre_lab/media/image5.png"
style="width:6.5in;height:4.17153in"
alt="A screenshot of a computer AI-generated content may be incorrect." />


6.  Click on fsl 6.0.7.16; then a terminal window will pop up, the first
    time should still take longer (5-10 min).

<img src="./images/pre_lab/media/image6.png"
style="width:6.5in;height:4.24861in"
alt="A screenshot of a computer program AI-generated content may be incorrect." />

7.  After seeing "module fslXXX is installed", type in fsleyes or fsl in
    the terminal window you just set loaded fsl on to open either app.

8.  Finally, enter `pip install datalad` 
  -  If in your later downloading of OpenNeuro data, the terminal complains there is no        up-to-date git-annex as shown in the screenshot: enter the following code in the terminal, before `git-annex version` to confirm the installed version and repeat the datalad downloading command.

<img src="./images/pre_lab/media/image7.png"
 style="width:5in;height:0.42778in" />

```
wget https://downloads.kitenet.net/git-annex/linux/current/git-annex-standalone-amd64.tar.gz; 
tar -xzf git-annex-standalone-amd64.tar.gz;
export PATH=$PWD/git-annex.linux:$PATH 
