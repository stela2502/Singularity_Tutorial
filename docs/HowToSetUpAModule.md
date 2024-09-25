# How to set up an Apptainer image as a COSMOS-SENSE module

A valid way to learn about how to load an image in the slurm / module system is to look at the defintion of a working image, e.g. the ImageSmith/1.0 Lua defintion. First we need to find where the definition file is stored on the server:

```bash 
module show ImageSmith/1.0
```
```text
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   /sw/easybuild_milan/modules/all/Core/ImageSmith/1.0.lua:
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
help([[This module is an example Singularity Image prowiding  
       a 'naked' Python Jupyter Lab interface to both Python and R ]])
execute{cmd="singularity run --cleanenv /sw/pkg/ImageSmith/1.0/ImageSmith_v1.0.sif", modeA={"load"}}
whatis("Name         : ImageSmith singularity image")
whatis("Version      : ImageSmith 1.0")
whatis("Category     : Image")
whatis("Description  : Singularity image providing Python and R and a jupyter lab as default entry point ")
whatis("Installed on : 11/09/2024 ")
whatis("Modified on  : --- ")
family("images")
```

And then look into the Lua definition file:
```bash
cat /sw/easybuild_milan/modules/all/Core/ImageSmith/1.0.lua
```

```text
help([[This module is an example Singularity Image prowiding  
       a 'naked' Python Jupyter Lab interface to both Python and R ]])

local version = "1.0"
local base = pathJoin("/sw/pkg/ImageSmith/1.0")


-- this happens at load
execute{cmd="singularity run --cleanenv ".. base.. "/ImageSmith_v".. version ..".sif",modeA={"load"}}

-- this happens at unload
-- could also do "conda deactivate; " but that should be part of independent VE module

-- execute{cmd="exit",modeA={"load"}}

whatis("Name         : ImageSmith singularity image")
whatis("Version      : ImageSmith 1.0")
whatis("Category     : Image")
whatis("Description  : Singularity image providing Python and R and a jupyter lab as default entry point ")
whatis("Installed on : 11/09/2024 ")
whatis("Modified on  : --- ")

family("images")

-- Change Module Path
--local mroot = os.getenv("MODULEPATH_ROOT")
--local mdir = pathJoin(mroot,"Compiler/anaconda",version)
--prepend_path("MODULEPATH",mdir)
--
```

## Explanation of Lua Module Definition

This Lua module definition file provides a wrapper for loading and unloading a Singularity container.

### Breakdown of the Script

1. **Help Message:**
   ```
   help([[This module is an example Singularity Image providing  
          a 'naked' Python Jupyter Lab interface to both Python and R ]])
   ```
   - This section defines a **help message** that users can access when they load the module. It provides a description of what the Singularity image contains: a Python JupyterLab interface with Python and R support.
   This already is a good example of what you should NOT DO. Change your help string to something correct and useful! 

2. **Local Variables:**
   ```
   local version = "1.0"
   local base = pathJoin("/sw/pkg/ImageSmith/1.0")
   ```
   - Two local variables are defined:
     - `version`: Defines the version of the image (`1.0`).
     - `base`: Defines the path to the directory where the Singularity image (`ImageSmith_v1.0.sif`) is stored, i.e., `/sw/pkg/ImageSmith/1.0`.

3. **Singularity Command Execution on Load:**
   ```
   execute{cmd="singularity run --cleanenv ".. base.. "/ImageSmith_v".. version ..".sif",modeA={"load"}}
   ```
   - This is the core of the script.
   - When the module is loaded, the `singularity run --cleanenv` command is executed. It runs the Singularity container (`ImageSmith_v1.0.sif`), stripping the environment variables to avoid conflicts (with the `--cleanenv` flag).
   - The `modeA={"load"}` ensures this command is only executed **when the module is loaded**.

4. **Unloading Section (Commented Out):**
   ```
   -- execute{cmd="exit",modeA={"load"}}
   ```
   - This section is currently commented out. It could potentially handle the **unload** process when the module is unloaded. Right now, it's left blank, but it could be used for actions like cleaning up the environment.
   - The comment mentions that "conda deactivate" could be run on unload, but this stems from the definition file I copied starting with Lua definition files. It would not make sense in this context.

5. **Whatis Information:**
   ```
   whatis("Name         : ImageSmith singularity image")
   whatis("Version      : ImageSmith 1.0")
   whatis("Category     : Image")
   whatis("Description  : Singularity image providing Python and R and a jupyter lab as default entry point ")
   whatis("Installed on : 11/09/2024 ")
   whatis("Modified on  : --- ")
   ```
   - These `whatis` entries define **metadata** for the module, giving details about the module's:
     - Name
     - Version
     - Category
     - Description (JupyterLab with Python and R)
     - Installation and modification dates
   - This information is useful for users to see a quick summary when using `module show` or similar commands.

6. **Family Definition:**
   ```
   family("images")
   ```
   - The `family("images")` line is used to group this module under a "family" of modules called **"images"**. This is part of the environment module system to avoid conflicting module loads, ensuring that users can only load one image from the "images" family at a time.

7. **(Commented) Module Path Prepending:**
   ```
   --local mroot = os.getenv("MODULEPATH_ROOT")
   --local mdir = pathJoin(mroot,"Compiler/anaconda",version)
   --prepend_path("MODULEPATH",mdir)
   ```
   - This section is commented out, but it seems like it was intended to **dynamically modify the module path** to include an Anaconda environment related to this image.
   - If activated, this section would prepend a new path to `MODULEPATH`, which is where additional module files (e.g., for compilers or virtual environments) could reside. THis part might be useful if you e.g. want to modity the users ``$PATH```.

### Summary

This Lua module definition file:
- Provides a Singularity image (`ImageSmith_v1.0.sif`) that runs **Python, R, and JupyterLab**.
- When the module is loaded, it runs the Singularity container using `singularity run --cleanenv`, making it a "clean" environment.
- Provides metadata about the module (like name, version, and description) for user reference.
- Allows for easy extension by dynamically changing the module path (though currently commented out).
  
In essence, it automates loading and executing the Singularity container in an HPC or module-based environment for seamless use of the apptainer system.

But this is a module installed by a system administrator. Only them can install into these folders.

We have a shared folder in COSMOS-SENS ``/scale/gr01/shared/common/`` where we have set up a modules and software folder.
The modules installed there can be loaded into the module system by issuing:

```bash
module use /scale/gr01/shared/common/modules
```

Afterwards all modules that were created by us are available using the normal 

```bash
module spider module_name
module load module_name/version
```

To register a module you need to create the same folder in both the ``modules`` and ``software`` folder.
Software will be installed into a ``software/<package_name>/<version>/`` directory whereas the Lua definition of the software will reside in the file ``modules/<package_name>/<version>.lua``.

I am no master of modules and can only help you a litlle here.

We only need to replace the Module description and the paths in the Lua description file we got from the ImageSmith Apptainer module:

```text

help([[This module is an example Singularity Image prowiding
       a 'naked' Python Jupyter Lab interface to both Python and R ]])          
                                                            
local version = 1.0                                                      
local base = pathJoin("/scale/gr01/shared/common/software/Singularity_Workshop/1.0") 
                                                                                
                                               
-- this happens at load                                                        
execute{cmd="singularity run -B/scale,/sw ".. base.. "/Singularity_Workshop_v1.0.sif", modeA={"load"} }
                                                            
                                                                                
-- this happens at unload   
-- nothing here

whatis("Name         : Singularity_Workshop singularity image")                        
whatis("Version      : Singularity_Workshop 1.0)                                 
whatis("Category     : Image")                                                  
whatis("Description  : Singularity image providing Python and R and a jupyter lab")
whatis("Installed on : 17.09.2024 ")                                   
whatis("Modified on  : --- ")                                                   
whatis("Installed by : Stefan Lang")                                             
                                                                                
family("images")                                                                

-- these I normally let in here for help if I need something else                   
-- Change Module Path                                                           
--local mroot = os.getenv("MODULEPATH_ROOT")                                    
--local mdir = pathJoin(mroot,"Compiler/anaconda",version)                      
--prepend_path("MODULEPATH",mdir)                                               
-- 

```
This file needs to go to ``/scale/gr01/shared/common/modules/Singularity_Workshop/1.0.lua`` and the image
needs to be copied to ``/scale/gr01/shared/common/software/Singularity_Workshop/1.0/Singularity_Workshop_v1.0.sif``.

Of cause the module name `Singularity_Workshop`  will not be available for all of you due to user access rights restrictions on our shared folders. We all need to use different folders in the modules and software folder.

In other words, you will need to rename your module. I often forget to change the module name or version in one or more places (in an apptainer call or anywhere in the Lua module). And therfore I have simplified this whole process: I have transitioned from executing commands in the terminal to using[a Makefile based approach](AMakefileBasedApproach.md).
