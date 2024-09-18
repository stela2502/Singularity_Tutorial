# Bioinformatics focused Apptainer/Singularity Workshop 2024

## Introduction to Apptainer

Apptainer (formerly known as Singularity) is a powerful containerization tool tailored for scientific and high-performance computing (HPC) environments. Unlike other containerization platforms like Docker, Apptainer is designed with security in mind, allowing users to run containers without needing elevated privileges. This makes it an excellent choice for bioinformaticians working on shared or secure systems.

Containers encapsulate software environments, including libraries, dependencies, and the application itself, ensuring reproducibility and ease of deployment. This is especially valuable in bioinformatics, where software dependencies can be complex and challenging to manage.

Although Apptainer is available for all major operating systems, the installation process on Windows and Mac is not as straightforward as on a pure Linux system. Therefore, we will run this course using an Apptainer image. This image can be installed on any system that supports Apptainer and allows you to build an image as a regular user.

## Building Apptainer images for COSMOS-SENSE on open COSMOS

For this workshop, we will use the open COSMOS system to build our Apptainer images.
To do this, you'll need to [download](https://www.cendio.com/thinlinc/download/) Thinlink and log into open COSMOS at cosmos-dt.lunarc.lu.se.

You can load the necessary module with the following command:

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


## Desining your own Apptainer HPC Image

Right from the start you should consider how you want to interact with the image:

    1. Command line tools only
    2. Interactive software
    3. A server started in the image

If you primarily use interactive sessions based on R or Python for your daily work,
I recommend installing Jupyter lab as well. This interface provides numerous possibilities for interacting with your software.

For a minimal configuration, we need the following:

- Python with JupyterLab
- R
- R-Jupyter integration

This allows for interaction with command line tools as well as interactive usage of Python and R packages.

Unfortunately due to Apptainer internals we are restricted to build the same system as the ImageSmith is based on. So we need to stick with alpine:latest for now.

Here’s how you can obtain a minimal Apptainer definition file for this setup:

```text
Hi Chatty - can you give me a minimal Apptainer def file that builds this minimal system:
What we need in a minimal configuration is therefore: Python with JupyterLab, R, and the R-Jupyter integration.
Base it on Apline latest - please.
```

You will probably not get there from the start, but if you are critical and look for small things like ``From alpine:latest`` you can force Chatty to improve. If nothing else it is a first start. It is helpful to use the free GPT-4o for that.


My **modified** ChatGPT output:
```text

# This is an Apptainer definition file for creating a container
# based on Alpine Linux with Python, JupyterLab, R, and R-Jupyter integration.

Bootstrap: docker
From: alpine:latest  # Use the latest Alpine Linux image as the base

%help
    This container provides a minimal environment with:
    - Python (with JupyterLab)
    - R
    - R-Jupyter integration
    Based on Alpine Linux.

%post
    # Update the package index and install essential packages
    apk update

    # Install build tools and libraries required for Python and R
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

Let's go back to the ImageSmith - create a new directory:
 
```sh
mkdir mkdir Singularity_Workshop
cd Singularity_Workshop
```

Jupyter lab also has an option to create a new folder: in the upper left corner the third icon in the second row.

To create the definition file you can cd into the created folder or use Jupyter lab to navigate into that folder. On the command line only the vi editor is installed:

```bash
vi Singularity_Workshop.def
```
Paste the copied text with the middle mouse button and close a and save the vi session with ``:x``.

Using the Jupyter lab interface you can also create a new file in the folder using the large plus sign benethe the create folder icon. You can the select Other -> Text file copy in the text and save as  "Singularity_Workshop.def".


Great we have our first definition of a Apptainer image. Let's build that:
```bash
apptainer build --sandbox Singularity_Workshop Singularity_Workshop.def
```

That is the bare bone of image creation.

But [how do we add this Apptainer image as a COSMOS-SENSE module](HowToSetUpAModule.md)?




