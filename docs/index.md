# Bioinformatics focused Apptainer/Singularity Workshop 2024

## Introduction to Apptainer

Apptainer (formerly known as Singularity) is a powerful containerization tool tailored for scientific and high-performance computing (HPC) environments. Unlike other containerization platforms like Docker, Apptainer is designed with security in mind, allowing users to run containers without needing elevated privileges. This makes it an excellent choice for bioinformaticians working on shared or secure systems.

Containers encapsulate software environments, including libraries, dependencies, and the application itself, ensuring reproducibility and ease of deployment. This is especially valuable in bioinformatics, where software dependencies can be complex and challenging to manage.

Remember when R 4.4 was deamed unsave and automatically updated - destroying your previous workflow? If you had your Apptainer based software then you could still use the 'unsave' R 4.4 workflow.
And in addition Apptainer provides a way to bundle the tools and packages you need for your analysis in one place, install them on you HPC system without the help of anybody else and use the same software both on your development machine as well as on the HPC system.

Although Apptainer is available for all major operating systems, the installation process on Windows and Mac is not as straightforward as on a pure Linux system. Therefore, we will run this course using an Apptainer image. This image can be installed on any system that supports Apptainer and allows you to build an image as a regular user - even in an HPC environment.

## Building Apptainer images on your developmental machine


Building an Apptainer image from start to finish can be summarized in three main steps:

1. Create a defintion file

    Save this text in a definition file like "MyPackage.def".
```text
# Which operating system should be used as a bassis for this image
bootstrap: docker
From: alpine:latest

%post
    # Update the package index and install essential packages
    # The commands here are the same as you would use inside the system to install ppackaged if you are root
    apk update

    # Install build tools and libraries required for Python and R
    # You must not have comments after the '\' in the following lines!
    # E.g. ChattGPT likes to add them.
    apk add --no-cache bash \
        build-base \
        zeromq-dev \
        libffi-dev \
        musl-dev \
        openblas-dev \
        python3 \
        py3-pip \
        python3-dev \
        py3-setuptools \
        py3-wheel 

    # Allow pip to modify system-wide packages
    export PIP_BREAK_SYSTEM_PACKAGES=1

    # Install JupyterLab using pip
    pip3 install --no-cache-dir jupyterlab

%environment
    # Ensure /usr/local/bin is in the PATH so JupyterLab can be found
    export PATH="/usr/local/bin:$PATH"
    # if you want to install more python packages in the sandbox:
    export PIP_BREAK_SYSTEM_PACKAGES=1

%runscript
    # By default, run JupyterLab when the container starts
    jupyter lab --port 9734 --ip=0.0.0.0 --allow-root --no-browser
```
2. Build the image
```bash
apptainer build MyPackage.sif MyPackage.def
```
3. Start the image
```bash
apptainer run MyPackage.sif
```

While it is technically possible for you to install Apptainer on your own computer, doing so would introduce unnecessary complexity to the course. Installing Apptainer requires administrative (super user) rights, which can be a challenge if your computer is managed by your institution, such as LU. Additionally, each operating system (Linux, Windows, macOS) has its own specific installation steps, and supporting all platforms would take us beyond the focus of this workshop.

Instead, we will be using the open COSMOS system to build the images. While you may not have super user rights there either, Apptainer is already installed, and we've developed a method to use an existing Apptainer image to build new ones. This allows us to work efficiently within the system provided, without needing to deal with the complexities of local installations.


## Building Apptainer images for COSMOS-SENSE on open COSMOS

For this workshop, we will use the open COSMOS system to build our Apptainer images.
To get started, you'll need to [download ThinLink](https://www.cendio.com/thinlinc/download/) and log into open COSMOS at ``cosmos-dt.lunarc.lu.se``.

Building an apptainer image does need superuser rights or at least some elevated rights on the system.
IThis means that on most HPC platforms, even if they support Singularity/Apptainer, you won't be able to build an image directly due to restricted permissions.


To overcome this limitation, we developed an Apptainer image called [ImageSmith](https://github.com/stela2502/ImageSmith), which has Apptainer installed inside it. This allows you to build Apptainer images without needing superuser rights.

The ImageSmith image is based on Alpine Linux, chosen for its ability to create slim, efficient images. It has already been installed as a module on COSMOS. You can load the module with the following command:

```bash
module load ImageSmith/1.0
```

However, as responsible bioinformaticians, we will not run the image directly on our frontend systems. Instead, we will execute it on the compute nodes:


To do this, copy the following into a file named ``RunImageSmith.sbatch``:
```text
#!/bin/bash
#SBATCH --ntasks-per-node 1
#SBATCH -N 1
#SBATCH -t 08:00:00
#SBATCH -A lu2024-7-5
#SBATCH -J start_Si
#SBATCH -o start_Si.%j.out
#SBATCH -e start_Si.%j.err

ml ImageSmith/1.0

exit 0
```

Then start the image with:
```bash
sbatch RunImageSmith.sbatch
```

After submission, you will receive the PID of the SLURM job. To view the process output, you can use:

```bash
watch cat <PID>
```
In the output, you will find a web link to access the image. Look for lines similar to:
```bash
...

[C 2024-09-16 10:33:26.138 ServerApp]

    To access the server, open this file in a browser:
        file:///home/stefanl/.local/share/jupyter/runtime/jpserver-2698141-open.html
    Or copy and paste one of these URLs:
        http://cn009:9734/lab?token=<token>
        http://127.0.0.1:9734/lab?token=<token>
...
```

Use one of these URLs to access the image in your browser.
In the Jupyter lab web page, you can open a 'Terminal' - that is all we need to build an image.


# Designing Your Own Apptainer HPC Image

When designing your Apptainer HPC image, consider how you want to interact with it:

1. **A Server Started in the Image**  
   You may want your image to launch a server upon startup, facilitating remote access and interaction.
   We will not cover that esplicitly here.

2. **Command Line Tools Only**  
   For those who ONLY rely on command line tools I recommand to create a script that handles the apptainer loading (incluiding all binds and other apptainer options) and make that script available as a command if you load the Lua module. We will not cover this version here, but you can look into my [Rustody_image github Repo](https://github.com/stela2502/Rustody_image) if you are interested.

3. **Interactive Software**  
   If you primarily use interactive sessions based on R or Python, I recommend installing JupyterLab. This provides numerous possibilities for interacting with your software like Terminal access, console access to R and Python, and also Jupter notebooks for R and Python.
 

## Minimal Configuration

For a minimal configuration, we need the following:

- Python with JupyterLab
- R
- R-Jupyter integration

This allows for interaction with command line tools as well as interactive usage of Python and R packages.

## Additional Considerations

Unfortunately due to Apptainer internals we are restricted to build the same system as the ImageSmith is based on. So we need to stick with alpine:latest for now.

## Example

Here’s how you can obtain a minimal Apptainer definition file for this setup using ChatGPT:

```text
Hi Chatty - can you give me a minimal Apptainer def file that builds this minimal system:
What we need in a minimal configuration is therefore: Python with JupyterLab, R, and the R-Jupyter integration.
Base it on Apline latest - please. 
```

The response is normally lacking quite a bit, but you get the overall structure of the def file for free.
If you are critical and look for small things like ``From alpine:latest`` you can force Chatty to improve. If nothing else it is a first start. It is helpful to use the free GPT-4o for that.


My **modified** ChatGPT output:
```text

# This is an Apptainer definition file for creating a container
# based on Alpine Linux with Python, JupyterLab, R, and R-Jupyter integration.

Bootstrap: docker
From: alpine:latest  # Use the latest Alpine Linux image as the base

%post
    # Update the package index and install essential packages
    apk update

    # Install build tools and libraries required for Python and R
    # You must not have comments after the '\' in the following lines!
    apk add --no-cache bash \
        build-base \
        zeromq-dev \
        libffi-dev \
        musl-dev \
        openblas-dev \
        R \
        R-dev \
        R-doc \
        python3 \
        py3-pip \
        python3-dev \
        py3-setuptools \
        py3-wheel 

    # Allow pip to modify system-wide packages
    export PIP_BREAK_SYSTEM_PACKAGES=1

    # Install JupyterLab using pip
    pip3 install --no-cache-dir jupyterlab

    # Install R packages for Jupyter integration
    R -e "install.packages(c('IRkernel', 'IRdisplay'), repos='https://cloud.r-project.org/')"

    # Set up IRkernel to make R available as a Jupyter kernel
    R -e "IRkernel::installspec(user = FALSE)"

%environment
    # Ensure /usr/local/bin is in the PATH so JupyterLab can be found
    export PATH="/usr/local/bin:$PATH"
    # if you want to install more python packages in the sandbox:
    export PIP_BREAK_SYSTEM_PACKAGES=1

%runscript
    # By default, run JupyterLab when the container starts
    jupyter lab --port 9734 --ip=0.0.0.0 --allow-root --no-browser

```

Let's go back to the ImageSmith to build an image based on this definition:

### Create a new directory

Jupyter lab allows you to create a folder using the graphical iterface, but you could also use the Terminal:
     
```sh
mkdir mkdir Singularity_Workshop
cd Singularity_Workshop
```

If you rather want to use the graphical interface: in the upper left corner of the Jupyter lab is an icon list - the third icon creates a new folder.

### Create the definition file

To create the definition file  only the vi editor is installed on the command line:

```bash
vi Singularity_Workshop.def
```
Paste the copied text with the middle mouse button and save & close vi session with ``:x``.

I recommend you to use the Jupyter lab interface instead: You can create a new file in the folder using the large plus sign beneth the create folder icon. You can then select Other -> Text file in the main window and copy the text into the new file; save it as "Singularity_Workshop.def".

### Build an Apptainer sandbox

The sandbox is an intermediate writable folder representation of the final Apptainer image.
It is very useful to debug a failing %post section, but also to quickly install new packages if the rest of the image should stay the same.

To build that sandbox for manual modification we can run this command:
```bash
apptainer build --sandbox Singularity_Workshop Singularity_Workshop.def
```

If this breaks with your own version of the def file I recommand to create a new Minimal.def text file with all lines of the %post removed:
```text
Bootstrap: docker
From: alpine:latest  # Use the latest Alpine Linux image as the base

%post

%environment
    # Ensure /usr/local/bin is in the PATH so JupyterLab can be found
    export PATH="/usr/local/bin:$PATH"
    # if you want to install more python packages in the sandbox:
    export PIP_BREAK_SYSTEM_PACKAGES=1

 %runscript
    # By default, run JupyterLab when the container starts
    jupyter lab --port 9734 --ip=0.0.0.0 --allow-root --no-browser
```

You should keep the %environment and the %runscript as this will allow you to (1) install your Python packages into the system directory mimiking the install process and (2) test if the manually modified sandbox can start the jupyter lab as expected in the end.

1. From that minimal def file you can build a MINIMAL sandbox and install your packages manually - hammering out the issues of your %post section in one go:
```bash
apptainer build --sandbox Singularity_Workshop MINAIMAL.def
apptainer shell --writable Singularity_Workshop
```
2. You can test your run script using this folder
```bash
apptainer run Singularity_Workshop
```

The sandbox will also help you to install your other packages. I do not assume you are used to alpine linux - or?
Install the software you need and do not forget to add all working install steps to the def file, too.
This way all later builds will already have your modifications in it!

### Build the Apptainer image

Finally we build the image with:
```bash
apptainer build Singularity_Workshop.sif Singularity_Workshop
```

Or after you have added all manual steps into the def file:
```bash
apptainer build Singularity_Workshop.sif Singularity_Workshop.def
```

That is the bare bone of image creation.

But [how do we add this Apptainer image as a COSMOS-SENSE module](HowToSetUpAModule.md)?




