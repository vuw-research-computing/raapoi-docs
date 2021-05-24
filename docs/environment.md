# Enviroment Setup

## Preparing your environment

Rāpoi has an extensive library of applications and software available. There are numerous programming languages and libraries (R, Julia, Python, lua, OpenMPI, blas, etc) as well as dozens of applications (Matlab, Gaussian, etc).  We also keep older versions of software to ensure compatibility.

Because of this, Rāpoi developers use a tool called *lmod* to allow a user to load a specific version of an application, language or library and start using it for their work. The _module_ command will show you what software is available to load, and will add the software to your environment for immediate use. To show all software available to load type the following:

  `module avail`

You will see a long list of available modules to load, including a path, eg _lua/5.3.5_
However, instead of searching through a long list, if you know you want to use lua, you can find the path with the keyword subcommand:

  `module keyword lua`

If you want to know more about a particular module you can use the _whatis_ or
_show_ subcommand.  Some modules have this available, for instance:

```
harrelwe@raapoi-master:~$ module show trimmomatic/20190304
----------------------------------------------------------------------
   /home/software/tools/modulefiles/trimmomatic/20190304:
----------------------------------------------------------------------
load("java/jdk/1.8.0_121")
setenv("TM_HOME","/home/software/apps/trimmomatic/20190304/bin")
```

```
  module whatis R/CRAN/3.5

  R/CRAN/3.5          : Adds the R library path to the pre-built CRAN modules
```

## Adding or loading software

Once you have found the module path you can load the software:

  `module load lua/5.3.5`

After the module loads you can type `srun --pty lua` at a prompt, or add it to the path of your lua script (the RC team recommends using /usr/bin/env instead of an absolute path).

Showing/listing the module environment modifications
You can discover what the module will load into your environment you can run
module show, for example here is what R adds:

```
module show R/3.5.1
--------------------------------------------------
   /home/software/tools/modulefiles/R/3.5.1:
--------------------------------------------------

whatis("Adds the R language path to your environment ")
prepend_path("PATH","/home/software/apps/R/3.5.1/bin")
```

## Listing loaded modules

To see what modules you have loaded into your environment you can run the command:

`module list`  

By default you will have the config module loaded (please do not unload that module).  For example, here are the modules I have loaded in my environment when I wrote this section:

```
module list

Currently Loaded Modules:
  1) config   2) tassel/3   3) python/3.7.0   4) python/modules/3.7

```
