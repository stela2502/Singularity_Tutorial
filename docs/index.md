# Bioinformatics focused Apptainer/Singularity Workshop 2024

## Introduction to Apptainer

Apptainer (formerly known as Singularity) is a powerful containerization tool tailored for scientific and high-performance computing (HPC) environments. Unlike other containerization platforms like Docker, Apptainer is designed with security in mind, allowing users to run containers without needing elevated privileges. This makes it an excellent choice for bioinformaticians working on shared or secure systems.

Containers encapsulate software environments, including libraries, dependencies, and the application itself, ensuring reproducibility and ease of deployment. This is especially valuable in bioinformatics, where software dependencies can be complex and challenging to manage.


Even so Apptainer is available for all major operating systems the installation on Windows and Mac is not as straight forward as on a (pure) Linux system. Therefore we will run this course from within an Apptainer image. The imagge can be installed on any Apptainer capable system and allows to build an image as a normal user.

## Building Apptainer images for COSMOS-SENSE on open COSMOS

For this workshop we will use the open COSMOS system to build our Apptainer images.
For that you need ([to download](https://www.cendio.com/thinlinc/download/)) Thinlink and log into open COSMOS at cosmos-dt.lunarc.lu.se.

The module can be loaded using 

* module load ImageSmith/1.0

But as we are responsible Bioinformaticians we will not start the image like that, but run it on the compute nodes instead:

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

Copy this into a file called ``RunImageSmith.sbatch`` and start the system with
*sbatch RunImageSmith.sbatch

But I am sure that is clear to all of you!

You will get the PID of the slurm job when the submission is finished. In the stdout of the process you will find the web link you can reach the image under:

*watch cat *<PID>*

```text
...

[C 2024-09-16 10:33:26.138 ServerApp]

    To access the server, open this file in a browser:
        file:///home/stefanl/.local/share/jupyter/runtime/jpserver-2698141-open.html
    Or copy and paste one of these URLs:
        http://cn009:9734/lab?token=<token>
        http://127.0.0.1:9734/lab?token=<token>
...
```

Use the web links - not the file!

In the Jupyter lab web page you can open a 'Terminal' - that is all we need to build an image.


## Desining your own Apptainer HPC Image

Right from the start you should think about which way you want to interact with the image:

    1. Command line tools only
    2. Interactive software
    3. A server started in the image

I assume most of you use interactive sessions based on R or Python for your daily work.
In that case I recommend to also install Jupyter lab as (as you see here) this interface gives you a lot of possibilities to interact with your software.

What we need in a minimal configuration is therefore: Python with Jupyter lab, R and the R-Jupyter integration.
The next question is which operating system do you need? And that is defined by the software tools you want to use.
As Linux is likely the systemn you need for Bioinformatics we will focus on Linux.

I have become horribly lacy - ask ChatGPT:

```text
Hi Chatty - can you give me a minimal Apptainer def file that build this minimal system:
What we need in a minimal configuration is therefore: Python with Jupyter lab, R and the R-Jupyter integration.
Base it on Unbutu latest - please.
```

I got:

```text

# Apptainer definition file for Python, JupyterLab, and R
BootStrap: docker
From: ubuntu:latest

%post
    # Update and install basic dependencies
    apt-get update && apt-get install -y \
        wget \
        curl \
        build-essential \
        libssl-dev \
        libcurl4-openssl-dev \
        libxml2-dev \
        software-properties-common \
        dirmngr \
        gnupg2 \
        locales

    # Set locale to UTF-8
    locale-gen en_US.UTF-8
    update-locale LANG=en_US.UTF-8

    # Install Python and JupyterLab
    apt-get install -y python3 python3-pip python3-dev
    pip3 install --upgrade pip
    pip3 install jupyterlab

    # Install R and dependencies
    apt-get install -y r-base r-base-dev

    # Install R-Jupyter integration (IRKernel)
    R -e "install.packages('IRkernel', repos='https://cloud.r-project.org/')"
    R -e "IRkernel::installspec(user = FALSE)"  # Register R kernel with Jupyter

    # Clean up
    apt-get clean
    rm -rf /var/lib/apt/lists/*

%environment
    # Set environment variables
    export LC_ALL=C.UTF-8
    export LANG=C.UTF-8
    export PYTHONNOUSERSITE="true"

%runscript
    # Default command to run when the container starts
    jupyter lab --no-browser --ip=0.0.0.0 --allow-root

%labels
    Maintainer ChatGPT
    Version 1.0

%help
    This Apptainer container includes:
    - Python with JupyterLab
    - R with Jupyter integration (IRKernel)
    - Based on Ubuntu latest

```

Let's go back to the ImageSmith - create a new directory:

```sh
mkdir mkdir Singularity_Workshop
cd Singularity_Workshop
vi Singullarity_Workshop.def
```

Highlight the definition file above and paste it into your vi editor using the middle mouse button.
Close the vim editor using ":x".

Great we have our forst definition of a Apptainer image. But before we build it - [let's look into the definition file, what does all of that do](./InDetailDefFile.md)?




## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.
