# Bioinformatics focused Apptainer/Singularity Workshop 2024

## Introduction to Apptainer

Apptainer (formerly known as Singularity) is a powerful containerization tool tailored for scientific and high-performance computing (HPC) environments. Unlike other containerization platforms like Docker, Apptainer is designed with security in mind, allowing users to run containers without needing elevated privileges. This makes it an excellent choice for bioinformaticians working on shared or secure systems.

Containers encapsulate software environments, including libraries, dependencies, and the application itself, ensuring reproducibility and ease of deployment. This is especially valuable in bioinformatics, where software dependencies can be complex and challenging to manage.

Although Apptainer is available for all major operating systems, the installation process on Windows and Mac is not as straightforward as on a pure Linux system. Therefore, we will run this course using an Apptainer image. This image can be installed on any system that supports Apptainer and allows you to build an image as a regular user.

## Building Apptainer images for COSMOS-SENSE on open COSMOS

For this workshop, we will use the open COSMOS system to build our Apptainer images.
To do this, you'll need to [download](https://www.cendio.com/thinlinc/download/) Thinlink and log into open COSMOS at cosmos-dt.lunarc.lu.se.

Building an apptainer image does need superuser rights or at least some elevated rights on the system.
In other words you can not build an Apptainer image on an HPC platform - even if the platform does support Singulatrity/Apptainer.
To fix this we deveoped an Apptainer image that has apptainer installed inside: [ImageSmith](https://github.com/stela2502/ImageSmith).

This image is based on Apline linux as it produces quite slim images and it is installed as a module on COSMOS.
You can load this module with the following command:

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

Here’s how you can obtain a minimal Apptainer definition file for this setup:

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

Let's go back to the ImageSmith to build an image based on this defintion:

1. Create a new directory

Jupyter lab allows you to create a folder using the graphical iterface, but you could also use the Terminal:
 
```sh
mkdir mkdir Singularity_Workshop
cd Singularity_Workshop
```

If you rather want to use the graphical interface: in the upper left corner of the Jupyter lab is an icon list - the third icon in the second row creates a new folder.

2. Create the definition file

To create the definition file you can cd into the created folder or use Jupyter lab to navigate into that folder. On the command line only the vi editor is installed:

```bash
vi Singularity_Workshop.def
```
Paste the copied text with the middle mouse button and save & close vi session with ``:x``.

Using the Jupyter lab interface you can also create a new file in the folder using the large plus sign benethe the create folder icon. You can the select Other -> Text file IN the main window and copy the text into the new file; save it as "Singularity_Workshop.def".

3. Build a Apptainer sandbox

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

You should keep the %environment and the %runscript as this will allow you to (1) install your packages as the install script will install them pater on nad (2) test if the manually moduified sandbox can start the jupyter lab as expected in the end.

From that mminaiml def file you can build a MINIMAL sandbox and install your packages manuall - hammering out the issues of your %post section in one go:
```bash
apptainer build --sandbox Singularity_Workshop MINAIMAL.def
apptainer shell --writable Singularity_Workshop
```

Or - assuming that the Singularity_Workshop sandbox was built correctly we now can use that sandbox to install the packages that we need for our daily work:

```bash
apptainer shell --writable Singularity_Workshop
```

Install all you need and do not forget to add all working install steps to the def file, too.
This way all later builds will already have your modifications in it!

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




