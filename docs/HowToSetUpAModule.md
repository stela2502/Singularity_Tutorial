# How to set up an Apptainer image as a COSMOS-SENSE module

We have a shared folder in COSMOS-SENS 
``/scale/gr01/shared/common/``
where we have set up a modules and software folder.
The modules installed there can be loaded into the module system by issuing:

```bash
module use /scale/gr01/shared/common/modules
```

Afterwards all modules that were created by us are available using the normal 

```bash
module spider module_name
module load module_name/version
```

To register a module you need to create the same folder in both the modules and software folder.
Software will be installed into a ``software/<version>/`` directory whereas the Lua definition of the software will reside in the file ``modules/<version>.lua``.

I am no master of modules and can only help you a litlle here.

A typical module definition file can look like that:

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

This module Lua file already specifies where the image should be placed:
```text
local base = pathJoin("/scale/gr01/shared/common/software/Singularity_Workshop/1.0") 
execute{cmd="singularity run -B/scale,/sw ".. base.. "/Singularity_Workshop_v1.0.sif", modeA={"load"} }
```
We need to copy the image to ``/scale/gr01/shared/common/software/Singularity_Workshop/1.0/Singularity_Workshop_v1.0.sif``.

Of cause **this module name** will not be available for all of you due to user access rights restrictions on our shared folders.

In other words, you will need to rename your module. I often forget to change the module name or version in one or more places (in an apptainer call or anywhere in the Lua module). And therfore I have simplified this whole process: I have transitioned from executing commands in the terminal to using[a Makefile based approach](AMakefileBasedApproach.md).
