Throughout this set of lab tutorial, we are operating on Neurodesk, a
neuroimaging computing platform that aims to make neuroimaging analyses
maximally reproducible and universally compatible:

 

 

 

 

The following instructions is based on that you have a Mac, Windows or
Linux computer; if you are using any other system, please ignore steps
1-2, obtain a Github account, and use [<u>the browser
version</u>](https://nam10.safelinks.protection.outlook.com/?url=https%3A%2F%2Fplay.neurodesk.org%2F&data=05%7C02%7Cshenghan.wang%40temple.edu%7C25cd484e6ecb4fa2fd8208de21a3a307%7C716e81efb52244738e3110bd02ccf6e5%7C0%7C0%7C638985183851618162%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=zEH3HvGLjX0DUOM90tfBL6zPXkoUeaYtnGJ%2FvxDG8ww%3D&reserved=0);
the rest of the set-up start to apply again from step 3.

 

 

Mac, Windows or Linux computer can work with the Neurodesk App

 

Hardware requirements

1.  Docker (as the container of the system and software): the host
    machine must support virtualization / containers and have enough
    free resources for both the host OS *and* the container environment.

2.  At least 5GB free space for NeuroDesk base image based on the app.

3.  Recommended for learning & moderate use: 4–8 cores, 16–32 GB RAM,
    SSD with ~50–100 GB free space.

    1.  8 GB RAM doable but slow (tested). Be sure to exit out all other
        resource hungry applications (e.g. browser) up memory to free if
        your computer is reluctant.

>  

 

Setup steps

1.  [<u>Download</u>](https://iishiishii.github.io/neurodesk.github.io/getting-started/local/neurodeskapp/)
    installation NeuroDesk and Docker packages for your system and
    install : restart if prompted, installing docker requires
    administrator permission

2.  Open Docker, wait for it to initialize, then open Neurodesk, click
    on "Open Local Neurodesk"

> <img src="./images/pre_lab/media/image1.png"
> style="width:6.5in;height:4.24861in"
> alt="A screenshot of a computer AI-generated content may be incorrect." />

1.  First time booting Neurodesk takes longer for the container to be
    populated - **be sure to stay connected to internet during the
    process**. The latest progress will be displayed at the bottom of
    the sliding window. As long as new steps completed is popping up
    once a couple of minutes it's working without a glitch. Wait for a
    bit for up to 40 min.

 

<img src="./images/pre_lab/media/image2.png"
style="width:6.5in;height:4.10139in"
alt="A screen shot of a computer AI-generated content may be incorrect." />

1.  When it's properly initiated, click on the Neurodesktop icon

> <img src="./images/pre_lab/media/image3.png"
> style="width:6.5in;height:3.41528in"
> alt="A screenshot of a computer AI-generated content may be incorrect." />

1.  Click on either Desktop RDP or Desktop VNC, this should take you to
    the virtual machine interface.

> <img src="./images/pre_lab/media/image4.png"
> style="width:6.5in;height:4.88958in"
> alt="A computer screen shot of a computer screen AI-generated content may be incorrect." />
>
> Operate on the VM just as you do with your average computer, the
> function you are going to interact the most are highlighted from left
> to right: file system, browser, terminal, and the most recent version
> of fsl and fsleyes

<img src="./images/pre_lab/media/image5.png"
style="width:6.5in;height:4.16736in"
alt="A screenshot of a computer AI-generated content may be incorrect." />

 

1.  Click on fsl 6.0.7.16; then a terminal window will pop up, the first
    time should still takes longer (5-10 min).

<img src="./images/pre_lab/media/image6.png"
style="width:6.5in;height:4.25625in"
alt="A screenshot of a computer screen AI-generated content may be incorrect." />

 

1.  After seeing "module fslXXX is installed", type in fsleyes and fsl
    in the terminal window you just set loaded fsl on(5min for booting
    this is not good)

2.  Finally enter \`\`\`Pip install datalad\`\`\` (note: WIP: some
    version of Neurodesk (the oen for windows?) have pip pre-installed
    some don't; I am still trying to figure out how to resolve it on
    mac) to install the tool to upload and download the

 
