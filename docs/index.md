# Bioinformatics focused Apptainer/Singularity Workshop 2024

## Introduction to Apptainer

Apptainer (formerly known as Singularity) is a powerful containerization tool tailored for scientific and high-performance computing (HPC) environments. Unlike other containerization platforms like Docker, Apptainer is designed with security in mind, allowing users to run containers without needing elevated privileges. This makes it an excellent choice for bioinformaticians working on shared or secure systems.

Containers encapsulate software environments, including libraries, dependencies, and the application itself, ensuring reproducibility and ease of deployment. This is especially valuable in bioinformatics, where software dependencies can be complex and challenging to manage.

## Why use Apptainer/Sigularity

1. **Containers are safe from external changes.** Remember when R 4.4 was deemed unsafe and LU automatically updated R without warning and destroyed your previous workflow? If your setup was Apptainer based then it would not have been affected.

2. **Reproducibility** The results from your workflow can sometimes change by simply updating a package in the workflow. To make results really reproducilble it makes sense to have an image of your workflow that cannot change. This would be useful for future labmates when you leave the lab, but still need to use your exact workflows.

3. **Installing own software on COSMOS-SENS**. LUNARC are happy to install software, but sometimes some python packages are a pain to install. Apptainer provides a way to bundle the tools and packages you need for your analysis in one place and install them on you HPC system without the help of anybody while using the same software both on your development machine as well as on the HPC platform.

***Although Apptainer is available for all major operating systems, the installation process on Windows and Mac is not as straightforward as on a pure Linux system***. Therefore, we will run this course using an Apptainer image. This image can be installed on any system that supports Apptainer and allows you to build an image as a regular user - even in an HPC environment. ***In essence, we are going to use an image to make an image***.


***Apptainer Images are Processor Architecture specific.*** Apples new M chips are arm64 based and therefore images build on Mac's are also arm64 based. This makes them incompatible with COSMOS which is amd64 based. Sorry.

## Building Apptainer images on your developmental machine

Building an Apptainer image from start to finish can be summarized in three main steps:

**1. Create a defintion file**
Save this text in a definition file like "MyPackage.def".
    
```text
# Which operating system should be used as a basis for this image
bootstrap: docker
From: alpine:latest

%post
    # Update the package index and install essential packages
    # The commands here are the same as you would use inside the system to install ppackaged if you are root
    apk update

    # Install build tools and libraries required for Python and R
    # You must not have comments after the '\' in the following lines!
    # E.g. ChatGPT likes to add them.
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

**2. Build the image**

```bash
apptainer build MyPackage.sif MyPackage.def
```

**3. Start the image**

```bash
apptainer run MyPackage.sif
```

This gives you a barebones Linux image with a few essential things installed. In order to install your own software you need to make a sandbox first:

```bash
apptainer build --sandbox MyPackage MyPackage.def
```

You can 'shell' into the created `MyPackage` folder:

```bash
apptainer shell --writable MyPackage
```

Where you can now install your own software using the usual methods. After this you can test it, and then build a .sif image, but this is something that we will cover later.


While it is ***technically possible*** for you to install Apptainer on your own non-Linux computer, doing so would introduce unnecessary complexity to this workshop. Installing Apptainer requires administrative (super user) rights, which can be a challenge if your computer is managed by your institution, such as LU. Additionally, each operating system (Linux, Windows, macOS) has its own specific installation steps, and supporting all platforms would take us beyond the focus of this workshop.

Instead, we will be use **open COSMOS** to build our images. While you may not have super user rights there either, Apptainer is already installed, and we've developed a method to use an existing Apptainer image to build new ones. This allows us to work efficiently within the system provided, without needing to deal with the complexities of local installations.


## Building Apptainer images for COSMOS-SENS on open COSMOS

For this workshop, we will use the open COSMOS system to build our Apptainer images.
To get started, you'll need to [download ThinLink](https://www.cendio.com/thinlinc/download/) and login to open COSMOS at ``cosmos-dt.lunarc.lu.se``.

Building an apptainer image needs superuser rights, or at least some elevated rights on the system.
This means that on most HPC platforms (even if they support Singularity/Apptainer), you won't be able to build an image directly due to restricted permissions.


To overcome this limitation, Stefan has developed an Apptainer image called [ImageSmith](https://github.com/stela2502/ImageSmith), which has Apptainer installed inside it. *So yes, we're going to generate images from an image!* This allows you to build Apptainer images without needing superuser rights. The ImageSmith image is based on Alpine Linux, chosen for its ability to create slim, efficient images. It has already been installed as a module on COSMOS which you can load with the following command:

```bash
module load ImageSmith/1.0
```

However, as responsible bioinformaticians (hopefully), we will not run the image directly on our frontend. Instead, we will execute it on a compute node:


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
watch cat *<PID>*
```
In the output, you will find a web link to access the image. Look for lines similar to:
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

Use one of these URLs to access the image in your browser.
In the Jupyter lab web page, you can open a 'Terminal' - that is all we need to build an image.


# Designing Your Own Apptainer HPC Image

When designing your Apptainer HPC image, consider how you want to interact with it:

1. **A Server Started in the Image**  
   You may want your image to launch a server upon startup, facilitating remote access and interaction.
   We will not cover that explicitly here and use the Jupyter lab server instead.

2. **Command Line Tools Only**  
   For those who ONLY rely on command line tools, I recommend you create a script that handles the apptainer loading (including all binds and other apptainer options) and make that script available as a command if you load the Lua module. We will not cover this version here, but you can look into my [Rustody_image github Repo](https://github.com/stela2502/Rustody_image) if you are interested.

3. **Interactive Software**  
   If you primarily use interactive sessions based on R or Python, I recommend installing JupyterLab. This provides numerous possibilities for interacting with your software like Terminal access, console access to R and Python, and also Jupter notebooks for R and Python.
 

## Minimal Configuration

For a minimal configuration, we need the following:

- Python with JupyterLab
- R
- R-Jupyter integration

This allows for interaction with command line tools as well as interactive usage of Python and R packages.

## Additional Considerations

Unfortunately due to Apptainer internals and the user rights on COSMOS we are restricted to build the same system as the ImageSmith is based on. So we need to stick with alpine:latest for now.


## Apptainer Definition File Structure

An Apptainer definition file is a key component for creating containers with Apptainer. This file defines the environment, software, and configuration necessary for your container. Here’s a brief overview of its structure:

<p style="text-indent: -1em; padding-left: 1em;">
1. <b>Header Section</b>: This includes the `Bootstrap` and `From` directives,
    which specify the method and base image for the container.
</p>
```text
Bootstrap: docker
From: alpine:latest
```

<p style="text-indent: -1em; padding-left: 1em;">
2. <b>Post Section</b>: This section contains commands that are executed in the container environment after it is created. 
    It can include installation commands and other configuration steps.
</p>
```text
%post
    apt-get update && apt-get install -y python3
```
<p style="text-indent: -1em; padding-left: 1em;">
3. <b>Environment Section</b>b>: This section allows you to set environment 
    variables within the container.
</p>
```text
%environment
    export PATH=/usr/local/bin:$PATH
```

<p style="text-indent: -1em; padding-left: 1em;">
4. <b>Run Section</b>: This section specifies commands that should be executed when the container is run.
</p>
```text
%run
    python3 myscript.py
```

<p style="text-indent: -1em; padding-left: 1em;">
5. <b>Files Section</b>: Use this section to include files into the container.
</p>
```text
%files
    ./myscript.py /usr/local/bin/myscript.py
```

<p style="text-indent: -1em; padding-left: 1em;">
6. <b>Label Section</b>: This optional section allows you to add metadata to your container.
</p>
```text
%labels
    Author: Your Name
    Version: 1.0
```


## Example

Here’s how you can obtain a minimal Apptainer definition file for our minimal Bioinformatics setup using ChatGPT:

```text
Can you give me a minimal Apptainer def file that builds this minimal system:
What we need in a minimal configuration is therefore: Python with JupyterLab, R, and the R-Jupyter integration.
Base it on Apline latest - please. 
```

The response is normally lacking quite a bit, but you get the overall structure of the def file for free.
If you are critical and look for small things like ``From alpine:latest`` you can force ChatGPT to improve. If nothing else it is a first start. It is helpful to use the free GPT-4o for that.


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

    # Allow pip to modify system-wide packages (ChatGPT forgets that ALWAYS)
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

You see that some of Apptainers capabilities have not been used here: %files and %labels have all been omitted.
But for the time being this is enough - let's go back to the ImageSmith to build an image based on this definition:

### Create a new directory

Jupyter lab allows you to create a folder using the graphical iterface, but you could also use the Terminal:
     
```sh
mkdir Singularity_Workshop
cd Singularity_Workshop
```

If you would rather use the graphical interface- in the upper left corner of the Jupyter lab is an icon list - the third icon creates a new folder.

### Create the definition file

To create the definition file  only the vi editor is installed on the command line:

```bash
vi Singularity_Workshop.def
```
Paste the copied text with the middle mouse button and save & close vi session with pressing `Esc` and then writing ``:x``.

As the vi tool is special (annoying), it is recommended you use the Jupyter lab interface instead: You can create a new file in the folder using the large plus sign beneth the create folder icon. You can then select Other -> Text file in the main window and copy the text into the new file; save it as "Singularity_Workshop.def".


### Build an Apptainer sandbox

The sandbox is an intermediate writable folder representation of your final Apptainer image. The `%post` section is where your software is installed, and sometimes the installation of software can fail (for example, if a dependency is unmet). The sandbox is a good way to fix the installation of software before the image is built from the sandbox. It's also a good way to quickly install new packages if the rest of the image should stay the same as before.

To build a sandbox for manual modification we can run this command:
```bash
apptainer build --sandbox Singularity_Workshop Singularity_Workshop.def
```

*If* this breaks with your own version of the def file, create a new Minimal.def where all lines of the `%post` have been removed:
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

You should keep the `%environment` and the `%runscript` as this will allow you to 1) install your Python packages into the system directory mimiking the install process and 2) test if the manually modified sandbox can start the jupyter lab as expected in the end.

1. From that minimal def file you can build a MINIMAL sandbox and install your packages manually - hammering out the issues of your `%post` section in one go:
```bash
apptainer build --sandbox Singularity_Workshop MINIMAL.def
apptainer shell --writable Singularity_Workshop
```
2. After you have installed everything, including the jupyter lab server, you can test it with:
```bash
apptainer run Singularity_Workshop
```

**Install the software you need, and do not forget to add all working install steps to the def file too**. This way all later builds will already have your modifications in it!

### Build the Apptainer image

Finally we build the working image with:
```bash
apptainer build Singularity_Workshop.sif Singularity_Workshop
```

Or after you have added all manual steps into the def file:
```bash
apptainer build Singularity_Workshop.sif Singularity_Workshop.def
```

That is the bare bones of image creation. You can now copy this image to COSMOS-SENS or any machine with Apptainer/Singularity installed and use the image and software within. We can now extend this idea further and ponder, [how we can put this image into a module that can be called during a job submission](HowToSetUpAModule.md)?



