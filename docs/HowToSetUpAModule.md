# How to set up an Apptainer image as a COSMOS module

The module system on COSMOS is used to manage software packages. It allows users to dynamically load and unload specific software environments without affecting the entire system. Modules are defined through `Lua` scripts, which specify environment variables, dependencies, and version control for different software. 

A valid way to learn about how to load an image in the slurm/module system is to look at the defintion of a working image, e.g. our ImageSmith/1.0 Lua defintion. First, we need to find where that definition file is stored on the server:

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
```lua
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

The file on COSMOS looks different TO WHAT? - use the one on COSMOS instead. The difference to the one here is that in the official file the bind path is set up to run on either open COSMOS, COSMOS-SENS or your local development machine without the need to adjust the Apptainer bind setting.


## Explanation of Lua Module Definition

This Lua module definition file provides a wrapper for loading and unloading an Apptainer/Singularity container.

### Breakdown of the Script

<p style="text-indent: -1em; padding-left: 1em;">
1. <b>Help Message</b>: The help message for this module can be accessed when users load the module. It provides a description of what the Singularity image contains, specifically a Python JupyterLab interface with both Python and R support.
</p>
```lua
help([[This module is an example Singularity Image providing  a 'naked' Python Jupyter Lab interface to both Python and R ]])
```

<p style="text-indent: -1em; padding-left: 1em;">
2. <b>Local Variables</b>: Two local variables are defined in the module. The first variable, `version`, defines the version of the image as `1.0`, while the second variable, `base`, specifies the path to the directory where the Singularity image (`ImageSmith_v1.0.sif`) is stored:
</p>
```lua
local version = "1.0"
local base = pathJoin("/sw/pkg/ImageSmith/1.0")
```

<p style="text-indent: -1em; padding-left: 1em;">
3. <b>Singularity Command Execution on Load:</b>
   The core of the script is the command executed when the module is loaded. It runs the Singularity container (`ImageSmith_v1.0.sif`) with the `singularity run --cleanenv` command, which strips the environment variables to avoid conflicts.
</p>
```lua
execute{cmd="singularity run -B /projects,/local --cleanenv ".. base.. "/ImageSmith_v".. version ..".sif",modeA={"load"}}
```
<p style="text-indent: -1em; padding-left: 2em;">
   <b>--cleanenv</b>: Ensures that the environment variables are cleaned up, avoiding potential conflicts with the host environment. 
</p>
<p style="text-indent: -1em; padding-left: 2em;">
   <b>-B /projects,/local</b> does mount two folders from the external environment (COSMOS) into the image (/projects  and /local)
</p>
<p  style="padding-left: 1em;">
   The `modeA={"load"}` ensures this command is executed only when the module is loaded.
</p>


<p style="text-indent: -1em; padding-left: 1em;">
4. <b>Whatis Information:</b> The `whatis` entries define **metadata** for the module, giving details about the module's meta information:
</p>   
```lua
whatis("Name         : ImageSmith singularity image")
whatis("Version      : ImageSmith 1.0")
whatis("Category     : Image")
whatis("Description  : Singularity image providing Python and R and a jupyter lab as default entry point ")
whatis("Installed on : 11/09/2024 ")
whatis("Modified on  : --- ")
```

<p style="text-indent: -1em; padding-left: 1em;">
5. <b>Family Definition:</b> The `family("images")` line is used to group this module under a "family" of modules called **"images"**. This is part of the environment module system to avoid conflicting module loads, ensuring that users can only load one image from the "images" family at a time:
</p>
```lua
family("images")
```
<p style="text-indent: -1em; padding-left: 1em;">
6. <b>(Commented) Module Path Prepending:</b>  This section is commented out, but it seems like it was intended to <b>dynamically modify the module path</b> to include an Anaconda environment related to this image:
</p>
```lua
--local mroot = os.getenv("MODULEPATH_ROOT")
--local mdir = pathJoin(mroot,"Compiler/anaconda",version)
--prepend_path("MODULEPATH",mdir)
```
<p style="padding-left: 1em;">
   If activated, this section would prepend a new path to `MODULEPATH`, which is where additional module files (e.g., for compilers or virtual environments) could reside. This part might be useful if you want to modify the user's `$PATH`.
</p>

### Summary

This `Lua` module definition file:

- Provides a Singularity image (`ImageSmith_v1.0.sif`) that runs **Python, R, and JupyterLab**.
- When the module is loaded, it runs the Singularity container using `singularity run --cleanenv`, making it a "clean" environment.
- Provides metadata about the module (like name, version, and description) for user reference.
- Allows for easy extension by dynamically changing the module path (though currently commented out).
  
In essence, it automates loading and executing the Singularity container in an HPC or module-based environment for seamless use of the apptainer system.

## How do we create a module on COSMOS?

The example module was installed by a system administrator. Only they can install into those folders.

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

**But this folder is not mounted on the open COSMOS.** And we do not have a common folder on open COSMOS either.
Therefore - to test your image - we need a private module folder:

```bash
mkdir -p ~/sen05_shared/common/software
mkdir -p ~/sen05_shared/common/modules
```

Please use this location - it will be used later on too.

To register a module you need to create a folder with the same name (the package's name) in both the ``modules`` and ``software`` folders.
Software will be installed into a ``software/<package_name>/<version>/`` directory whereas the Lua definition of the software will reside in the file ``modules/<package_name>/<version>.lua``.

As a starter you only need to replace the module description and the paths in the `Lua` description file we got from the ImageSmith Apptainer module:

```text

help([[This module is an example Singularity Image prowiding
       a 'naked' Python Jupyter Lab interface to both Python and R ]])          
                                                            
local version = 1.0
local home=os.getenv( "HOME" )
local base = pathJoin(home,"/sens05_shared/common/software/Singularity_Workshop/1.0") 
                                                                                
                                               
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
whatis("Installed by : Me")                                             
                                                                                
family("images")                                                                

-- these I normally let in here for help if I need something else                   
-- Change Module Path                                                           
--local mroot = os.getenv("MODULEPATH_ROOT")                                    
--local mdir = pathJoin(mroot,"Compiler/anaconda",version)                      
--prepend_path("MODULEPATH",mdir)                                               
-- 

```

Create this file in ``~/sens05_shared/shared/common/modules/Singularity_Workshop/1.0.lua`` and copy the image file you have created in the last step to ``/scale/gr01/shared/common/software/Singularity_Workshop/1.0/``.

You now all have a module path that you can activate using 

```bash
module use ~/sens05_shared/shared/common/modules
```

You can start your module using the common command
```bash
module load  Singularity_Workshop/1.0
```

# Module Names are Unique

On COSMOS-SENS we have a common folder where you can 'publish' your modules for all users of COSMOS-SENS's **sens-gr01** group.
```bash
/scale/gr01/shared/common/modules
```

By default all of you can create modules there. But you cannot modify modules of other users. In other words, you will need to rename your module. I have simplified the creation and maintainance of Apptainer modules by transitioning from executing commands in the terminal to using [a Makefile based approach](AMakefileBasedApproach.md).
