# A Makefile-based approach to build and maintain an Apptainer image

My plan was to create a command that streamlines the development of an Apptainer image, much like how ``cargo new <package_name> --lib`` sets up a new Rust library project.

The goal is to establish a structure where both the Apptainer build and deployment processes are automated and easily configurable. By doing so, you can quickly update the version or even the name of your package without needing to rewrite the build configuration each time. The main focus should be on creating a robust definition file, with the initial setup of the Lua model definition and other components being a one-time effort. The central configuration will be handeled by a Makefile.

## What is a Makefile?

A Makefile is a special file used by the ``make`` utility to automate the building and management of software projects. It defines a set of rules and dependencies for compiling code, linking files, and generating executables or other targets. Makefiles consist of targets, dependencies, and commands that describe how to build each part of the project. Typically, they are used in environments with many files, like C/C++ projects, where manual compilation would be cumbersome. We do not need to dive into the depth of that - just the targets will be used for the automatisation. I will not go into more detail later - the ImageSmith has my implementation of that idea installed as a script. Use it.

```bash
/opt/ImageSmith/create_new_image_builder.sh <path_to>/<project_name>
```

Running this will create a folder and populate it with a Makefile, a default definition file and three scripts; ``shell.sh``, ``run.sh`` and ``generate_module.sh``. 

The Makefile is the central organizer of this whole package. The ``generate_module.sh`` defines the structure of the Lua module with the module name and version comming from the Makefile. The two other scripts are mainly designed to interact with the image on your (local) development computer and are not touched in the deploy step.

## Makefile Targets Explained

   1. ``all``:
    This is the default target that runs when you simply type make. It sequentially executes the clean, restart, build, and deploy targets. This option is a convenient way to manage the entire process in one go.

   2. ``restart``:
    This target handles the sandbox environment, which is used to create the image. If the sandbox directory ``$(SANDBOX_DIR)`` already exists, it replaces it's contents; otherwise, it creates a new one. The sandbox is essentially a writable container that you can modify before building the final image.
        Key command: ``sudo apptainer build --sandbox $(SANDBOX_DIR) $(DEFINITION_FILE)``

   3. ``build``:
    This target builds the final Singularity image ``$(IMAGE_NAME)`` from the sandbox. The image is based on the sandbox. This is in theory not necessary if you update your definition file. I recommend to add all changes you apply to the sandbox to the definition file, too! But I do not always adhere to that philosophy...
        Key command: ``sudo apptainer build $(IMAGE_NAME) $(SANDBOX_DIR)``

   4. ``deploy``:
    This target copies the final image to the specified deployment directory ``$(DEPLOY_DIR)`` and creates the Lua defintion file based on the informations in the Makefile and the Lua outline in the ``generate_module.sh`` script. This is useful for moving the image to a location where it can be accessed and used by others.
        Key command: ``cp $(IMAGE_NAME) $(DEPLOY_DIR)``

   5. ``clean``:
    This target removes the sandbox directory and the image file. It's useful for cleaning up your workspace if you need to start fresh or if you're done with the current build.
        Key command: ``rm -rf $(SANDBOX_DIR) $(IMAGE_NAME)``


## How to Use This Package

The package path you have now uses the Bioinformatics minimal definition file to install R and Python and to make them accessible using the Jupyter lab server. You can, for example, simply replace it's contents with the defintion file you have prepared in the last step.


To build your own project now you can somply run:
```bash
make -C <path_to>/<project_name>
```
or 
```bash
cd <path_to>/<project_name>
make
```

The deploy step is currently tailored to my development environment where I mount the COSMOS-SENS shared folder ``/scale/gr01/shared`` in ``~/sens05_shared/``. When you also do that on your own computer you can deploy the image directly to COMSOS_SENS. On open COSMOS we can not mount COSMOS-SENS and therefore the deploy will create the local folder and place the module there.
This module will work 'as is' if you copy the ``software/<package_name>/<version>/*`` and ``module/<package_name>/<version>.lua`` files and folders to their respective COSMOS-SENS paths ``/scale/gr01/shared/common/``.

### Test the module on open COSMOS


The first test should be if you can load it directly using apptainer:
```bash
apptainer run <your new package>_v1.0.sif
``` 
But do not forget to run that on the compute nodes, too.

You can also test the function of the Lua module on open COSMOS, but for that you need to adjust the path the Lua module expects the image:

Open the created Lua file at ``~/sens05_shared/common/modules/<SANDBOX_DIR>/<VERSION>.lua``.
You need to change the line 
```text
local base = pathJoin("/scale/gr01/shared/common/software/<SANDBOX_DIR>/<VERSION>")
```
to 
```text
home=os.getenv( "HOME" )
local base = pathJoin(home,"/sens05_shared/common/software/<SANDBOX_DIR>/<VERSION>")
```
Keep the project specific ``<SANDBOX_DIR>/<VERSION>``.

Afterwards you can register your personal modules / software folder pair as modules like that:

```bash
module use ~/sens05_shared/common/modules/
```

Afterwads you can 'run' your new Apptainer image using 

```bash
module load <SANDBOX_DIR>/<VERSION>
```

Please do not forget to run the image on the compute nodes again:
```text
#!/bin/bash
#SBATCH --ntasks-per-node 1
#SBATCH -N 1
#SBATCH -t 01:00:00
#SBATCH -A lu2024-7-5
#SBATCH -J start_Si
#SBATCH -o start_Si.%j.out
#SBATCH -e start_Si.%j.err

ml <SANDBOX_DIR>/<VERSION>

exit 0
```

If you have modified the Lua definition file and now want to copy the image to COSMOS-SENS the easiest is to remove the ``~/sens01_shared/common`` folder and re-deploy the module:
```bash
rm -Rf ~/sens01_shared/common
make deploy
``` 

### Deploy on COSMOS-SENS

I recommend you to upload you image definitions to GitHub (do not upload the image and the sandbox by either exclude them in the ``.gitignore`` or calling ``make clean`` before ``git add --all .``.
This way you can easily clone that on your normal desktop, install apptainer there and build and deploy your image from your normal desktop to COSMOS-SENS.

If you can not directly deploy to COSMOS-SENS you can simply opy the ``software/<package_name>/<version>/*`` and ``module/<package_name>/<version>.lua`` files and folders to the COSMOS-SENS path ``/scale/gr01/shared/common/``.


Please document your images well, upload them to COSMOS-SENS and share them with us. I hope we all can easen our workloads by sharing!
While you are at it you can check out my SingSing/1.6 package that provides a single cell analysis environment with Seurat, scanpy and scvelo and other tools installed. 

Thank you for participating in this workshop! I hope you found it useful, and I appreciate your involvement.

Oh and please do not forget right now:

PLEASE REMOVE THE DATA YOU CREATED FROM THE SERVER!