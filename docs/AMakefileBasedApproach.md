# A Makefile-based approach to build and maintain an Apptainer image

My plan for this task is to create a command that streamlines the development of an Apptainer image, much like how ``cargo new <package_name> --lib`` sets up a new Rust library project.


The goal is to establish a structure where both the Apptainer build and deployment processes are automated and easily configurable. By doing so, you can quickly update the version or even the name of your package without needing to rewrite the build configuration each time. The main focus should be on creating a robust definition file, with the initial setup of the Lua model definition and other components being a one-time effort.

The ImageSmith has my implementation of that idea installed as ascript:

```bash
/opt/ImageSmith/create_new_image_builder.sh <new_directory_name>
```

This will create a folder and populate it with a Makefile, a default definition file and two scripts; ``shell.sh`` and ``run.sh``. 


## Makefile Options Explained

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
        Key command: ``rm -rf $(SANDBOX_DIR) $(IMAGE_NAME``


## How to Use This Package

After creating a new Apptainer image project, all scripts and the Makefile will be updated with your project name. To build a standard Bioinformatics Apptainer image, run:

```bash
make -C <new_directory_name>
```

This will create the sandbox, build the image, and stop at the deploy step. The deploy step is tailored to my development environment, and it will fail on other systems, such as when attempting to mount COSMOS-SENS on open COSMOS.

To test it locally, create a mock deploy directory in your home folder:
```bash
mkdir -p ~/common/modules 
mkdir ~/common/software
```

Then, update the Makefile by removing my personal path /sens5_shared/. If you wish to test your module on open COSMOS, modify the SERVER_DIR to point to your local path (e.g., $HOME/common/...). Otherwise, you can copy the image and Lua definition file to the server for deployment.

Finally, rerun the ``deploy`` step:
```bash
make deploy
```

If you have modified the SERVER_DIR in the Makefile you can now test your module:

```bash
module use ~/common/modules/
```

Afterwads you can 'run' your new Apptainer image using 

```bash
module load Singularity_Workshop/1.0
```

Please do not forget to run the image on the compute nodes again:
```text
#!/bin/bash
#SBATCH --ntasks-per-node 1
#SBATCH -N 1
#SBATCH -t 08:00:00
#SBATCH -A lu2024-7-5
#SBATCH -J start_Si
#SBATCH -o start_Si.%j.out
#SBATCH -e start_Si.%j.err

ml <your module name>/1.0

exit 0
```

In case you have kept the SERVER_DIR as it was you now can create the path's on COSMOS-SENS and copy the image and the Lua definition file to the respective positions on the server.

You can also start your apptainer image directly on open COSMOS using
```bash
apptainer run <your new package>_v1,0.sif
``` 
But do not forget to run that on the compute nodes, too.



Thank you for participating in this workshop! I hope you found it useful, and I appreciate your involvement.