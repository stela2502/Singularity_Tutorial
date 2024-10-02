# A Makefile-based approach to build and maintain an Apptainer image

My plan was to create a command that streamlines the development of an Apptainer image, much like how ``cargo new <package_name> --lib`` sets up a new Rust library project.

The goal is to establish a structure where both the Apptainer build and deployment processes are automated and easily configurable. By doing so, you can quickly update the version or even the name of your package without needing to rewrite the build configuration each time. The main focus should be on creating a robust definition file, with the initial setup of the Lua model definition and other components being a one-time effort. The central configuration will be handeled by a Makefile.

The image will be prepared in the new folder. Out of the box, this image will support a Jupyter Lab web server, which acts as the main interaction point with the image once it is 'run'.

The `Makefile` is the main interaction point while building the new image, whereas the `<project_name>.def` file contains the definition for the new image.


## What is a Makefile?

A Makefile is a special file used by the ``make`` utility to automate the building and management of software projects. It defines a set of rules and dependencies for compiling code, linking files, and generating executables or other targets. Makefiles consist of targets, dependencies, and commands that describe how to build each part of the project. Typically, they are used in environments with many files, like C/C++ projects, where manual compilation would be cumbersome. We do not need to dive into the depth of that - just the targets will be used for the automatisation. I will not go into more detail later.

## Quickly create a Apptainer image from a Blueprint

The ImageSmith has my implementation of the automatization ([ImageBlueprint](https://github.com/stela2502/ImageBlueprint)) installed as a script:

```bash
create_new_image_builder.sh <path_to>/<project_name>
```

Running this will create a folder and populate it with a Makefile, a default definition file and three scripts; ``shell.sh``, ``run.sh`` and ``generate_module.sh``. 

The Makefile is the central organizer of this whole package. The ``generate_module.sh`` defines the structure of the Lua module with the module name and version comming from the Makefile. The two other scripts are mainly designed to interact with the image on your (local) development computer and are not touched in the deploy step.

### Makefile Targets Explained

   1. ``all``:
    This is the default target that runs when you simply type make. It sequentially executes the clean, restart, build, and deploy targets. This option is a convenient way to manage the entire process in one go.

   2. ``restart``:
    This target handles the sandbox environment, which is used to create the image. If the sandbox directory ``$(SANDBOX_DIR)`` already exists, it replaces it's contents; otherwise, it creates a new one. The sandbox is essentially a writable container that you can modify before building the final image.
        Key command: ``sudo apptainer build --sandbox $(SANDBOX_DIR) $(DEFINITION_FILE)``

   3. ``build``:
    This target builds the final Singularity image ``$(IMAGE_NAME)`` from the sandbox. The image is based on the sandbox. This is in theory not necessary if you update your definition file. I recommend to add all changes you apply to the sandbox to the definition file, too! But I do not always adhere to that philosophy.
        Key command: ``sudo apptainer build $(IMAGE_NAME) $(SANDBOX_DIR)``
   4. ``direct``:
    This target skipps the restart and builds the image directly from the defintion file.
        Key command: ``sudo apptainer build $(IMAGE_NAME) $(DEFINITION_FILE)``

   5. ``deploy``:
    This target copies the final image to the specified deployment directory ``$(DEPLOY_DIR)`` and creates the Lua defintion file based on the informations in the Makefile and the Lua outline in the ``generate_module.sh`` script. This is useful for moving the image to a location where it can be accessed and used by others.
        Key command: ``cp $(IMAGE_NAME) $(DEPLOY_DIR)``

   6. ``clean``:
    This target removes the sandbox directory and the image file. It's useful for cleaning up your workspace if you need to start fresh or if you're done with the current build.
        Key command: ``rm -rf $(SANDBOX_DIR) $(IMAGE_NAME)``


If you call `make` in the image's folder, you will chain the **clean**, **restart**, **build**, and **deploy** targets.

You need to modify the `generate_module.sh` script if you want the module to do something other than running the image.

Lastly, you can modify the definition file to adjust your Apptainer image to your needs. You are free to build whichever image you want.


## Building Images on Open COSMOS Needs Storage Space

Especially when you build images using a sandbox, the free storage on COSMOS runs out quickly.

But as you run the ImageSmith module on the compute nodes, you have access to the `$SNIC_TMP` folder. This folder is a large temporary directory that gets scrubbed after your process on the compute nodes is finished.

You can create your new project like this:

```bash
create_new_image_builder.sh $SNIC_TMP/MyCoolProject
```

The easiest way to interact with, for example, the definition file using ImageSmith is through the Jupyter Lab interface. Therefore, it is necessary to access the new files from the Jupyter Lab interface, and hence you should create a link from your (Jupyter) work directory to the `$SNIC_TMP`:

```bash
ln -s $SNIC_TMP work
```

Then you can open the `work/MyCoolProject.def` file using the Jupyter Lab interface. This makes it much easier to modify compared to the `vi` editor, which is also installed in ImageSmith.

Once your definition file contains what you want, you can easily deploy the new image to your home directory (if the image can be built) by stating:

```bash
make -C work/MyCoolProject
```

## Where Is My Module Now?

The module has been deployed to your home directory:

```bash
~/sens05_shared/common/modules/<project_name>/1.0.lua
~/sens05_shared/common/software/<project_name>/1.0/<project_name>.1.0.sif
```

The result of your work has been saved to your home folder, but your definition file is still only in the `$SNIC_TMP`. Instead of copying the definition file to your home folder, I recommend uploading it to Git. This way, you can easily access your work from outside Open COSMOS, too.

## How to Save Your Work

You could, of course, copy the whole building area to your home directory, but it is a lot of data and might not fit into your Lunarc allowance if you have no data project registered there.

Instead of copying the sandbox and image files, I recommend you create a new Git project and upload your definition files, scripts, and Makefile to either GitHub or any other Git server. The sandbox can be regenerated from the image and the image is already saved to your home folder.

The Blueprint includes a `.gitignore` in the path that will exclude both the sandbox and the image from being uploaded to Git.

## A Tiny GitHub Tutorial

Here’s a quick guide on how to create a new GitHub repository and upload your project:

### Step 1: Create a GitHub Repository

1. Go to [GitHub](https://github.com/) and log in (or sign up if you don’t have an account).
2. Click the **+** icon in the upper right corner and select **New repository**.
3. Fill in the repository name (e.g., `MyCoolProject`) and a description (optional).
4. Choose whether you want the repository to be **Public** or **Private**.
5. Click **Create repository**.

### Step 2: Initialize Your Local Git Repository

Open a terminal and navigate to your project directory:

```bash
cd $SNIC_TMP/MyCoolProject
```

Initialize a new Git repository:

```bash
git init
```

### Step 3: Add Your Files

Add the files you want to upload to the staging area:

```bash
git add --all .
```

### Step 4: Commit Your Changes

Commit your changes with a meaningful message:

```bash
git commit -m "Initial commit with project files"
```

### Step 5: Link to Your GitHub Repository

Copy the URL of your new GitHub repository (found on the repository page) and link it to your local repository:

```bash
git remote add origin <repository_url>
```

Replace `<repository_url>` with the actual URL, for example:

```bash
git remote add origin https://github.com/username/MyCoolProject.git
```

### Step 6: Push Your Changes to GitHub

Finally, push your changes to the master branch of your GitHub repository:

```bash
git branch -M main
git push -u origin main
```

Now your project is safely stored in your GitHub repository, and you can access it from anywhere!

## Test the module on open COSMOS

Now you have your image saved both in your home path as well as on Github.
Hence we can now look into the function of our image we need to start it:


The first test should be if you can load it directly using apptainer and the helper scripts we have:

```bash
# shell into the sandbox
$SNIC_TMP/MyCoolProject/shell.sh
# run the local image
$SNIC_TMP/MyCoolProject/run.sh
``` 

You can also test the function of the Lua module on open COSMOS, but for that you need to adjust the path the Lua module expects the image:

Open the created Lua file at ``~/sens05_shared/common/modules/<SANDBOX_DIR>/<VERSION>.lua``.
You need to comment the line 
```text
-- local base = pathJoin("/scale/gr01/shared/common/software/<SANDBOX_DIR>/<VERSION>")
```
and add these two lines defining the private location as module source path.
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
rm -Rf ~/sens05_shared/common
make deploy
``` 

### Deploy on COSMOS-SENS

We have already uploaded the defintion to GitHub.
Therfore you can easily clone that on your normal desktop, install apptainer there and build and deploy your image from your normal desktop to COSMOS-SENS.

If you want to use the image prepared on open COSMOS I recommend you to tar your deploy data structure (all in `~/sens05_share/common/`) and copy that to COSMOS-SENS using e.g. `scp`. Cou can soimply untar that in `/scale/gr01/shared/common/`. But make sure you only copy the module you want.

Please document your images well, upload them to COSMOS-SENS and share them with us. I hope we all can easen our workloads by sharing!
While you are at it you can check out my SingSingCell/1.6 package that provides a single cell analysis environment with Seurat, scanpy, scvelo and other tools installed ([on github](https://github.com/stela2502/singularityImages) this is pre ImageSmith). 

Please remove your `~/sens05_shared` folder once you are finished - you have saved all your work on github already.

Thank you for participating in this workshop! I hope you found it useful, and I appreciate your involvement.

