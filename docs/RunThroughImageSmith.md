
# A Short Run Through Using ImageSmith on Open COSMOS

Here you will learn how to use ImageSmith to its maximum potential:

1. Easily create a new package.
2. Access the `$SNIC_TMP` folder.
3. Save deployment to the stable home folder.
4. Easy integration with Git.

You will not learn about Apptainer definition files or any other Apptainer specifics here.

## Prepare New Image Projects Quickly

ImageSmith has access to my [ImageBlueprint](https://github.com/stela2502/ImageBlueprint) package. The main idea behind this package is to quickly create an Apptainer project using:

<script>create_new_image_builder.sh <path_to>/<project_name></script>

The image will be prepared in the new folder. Out of the box, this image will support a Jupyter Lab web server, which acts as the main interaction point with the image once it is 'run'.

The `Makefile` is the main interaction point while building the new image, whereas the `<project_name>.def` file contains the definition for the new image.

The `Makefile` provides the following targets:

- **clean**   : Removes the sandbox folder and the SIF image file.
- **restart** : Rebuilds the sandbox from the definition file.
- **build**   : Builds the image from the sandbox folder.
- **direct**  : Builds the image from the definition file.
- **deploy**  : Prepares a module for COSMOS-Sens in a local path.

If you call `make` in the image's folder, you will chain the **clean**, **restart**, **build**, and **deploy** targets.

You need to modify the `generate_module.sh` script if you want the module to do something other than running the image.

Lastly, you can modify the definition file to adjust your Apptainer image to your needs. You are free to build whichever image you want.

## Building Images on Open COSMOS Needs Storage Space

Especially when you build images using a sandbox, the free storage on COSMOS runs out quickly.

But as you run the ImageSmith module on the compute nodes, you have access to the `$SNIC_TMP` folder. This folder is a large temporary directory that gets scrubbed after your process on the compute nodes is finished.

You can create your new project like this:

<script>create_new_image_builder.sh $SNIC_TMP/MyCoolProject</script>

The easiest way to interact with, for example, the definition file using ImageSmith is through the Jupyter Lab interface. Therefore, it is necessary to access the new files from the Jupyter Lab interface, and hence you should create a link from your (Jupyter) work directory to the `$SNIC_TMP`:

<script>ln -s $SNIC_TMP work</script>

Then you can open the `work/MyCoolProject.def` file using the Jupyter Lab interface. This makes it much easier to modify compared to the `vi` editor, which is also installed in ImageSmith.

Once your definition file contains what you want, you can easily deploy the new image to your home directory (if the image can be built) by stating:

<script>make -C work/MyCoolProject</script>

## Where Is My Module Now?

The module has been deployed to your home directory:

<script>~/sens5_shared/common/modules/<project_name>/1.0.lua</script>
<script>~/sens5_shared/common/software/<project_name>/1.0/<project_name>.1.0.sif</script>

The result of your work has been saved to your home folder, but your definition file is still only in the `$SNIC_TMP`. Instead of copying the definition file to your home folder, I recommend uploading it to Git. This way, you can easily access your work from outside Open COSMOS, too.

## How to Save Your Work

You could, of course, copy the whole building area to your home directory, but it is a lot of data and might not fit into your Lunarc allowance if you have no data project registered there.

Instead of copying the sandbox and image files, I recommend you create a new Git project and upload your definition files, scripts, and Makefile to either GitHub or any other Git server.

The Blueprint has already created a `.gitignore` in the path that will exclude both the sandbox and the image from being uploaded to Git.

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

<script>cd $SNIC_TMP/MyCoolProject</script>

Initialize a new Git repository:

<script>git init</script>

### Step 3: Add Your Files

Add the files you want to upload to the staging area:

<script>git add .</script>

### Step 4: Commit Your Changes

Commit your changes with a meaningful message:

<script>git commit -m "Initial commit with project files"</script>

### Step 5: Link to Your GitHub Repository

Copy the URL of your new GitHub repository (found on the repository page) and link it to your local repository:

<script>git remote add origin <repository_url></script>

Replace `<repository_url>` with the actual URL, for example:

<script>git remote add origin https://github.com/username/MyCoolProject.git</script>

### Step 6: Push Your Changes to GitHub

Finally, push your changes to the master branch of your GitHub repository:

<script>git push -u origin master</script>

Now your project is safely stored in your GitHub repository, and you can access it from anywhere!
