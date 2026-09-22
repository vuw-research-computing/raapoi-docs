# Old Module System

The old module system used module files stored under:

```bash
/home/software/tools/modulefiles
```

Many older packages in this system may no longer work. Prefer the current module system described in [Environment Setup](environment.md).

## Identifying old modules

When you run `module avail`, the old modules appear under a heading similar to:

```bash
-------------------------- /home/software/tools/modulefiles --------------------------
```

Older module names are often lower case, such as `python`, `r`, and `openmpi`, while current modules commonly use names such as `Python`, `R`, and `OpenMPI`.

## Accounts created before March 2022

Accounts created before March 2022 may not automatically load the current module system. To enable it alongside the old system, add the following line to your `.bashrc` file. Users of zsh can make the equivalent change in `.zshrc`.

First back up and edit your `.bashrc`:

```bash
cd ~
cp .bashrc .bashrc_backup
nano .bashrc
```

Find the line:

```bash
module use -a /home/software/tools/modulefiles
```

Add this line after it:

```bash
module use /home/software/tools/eb_modulefiles/all/Core
```

Log out and back in, then test the current module system:

```bash
module load foss/2020a
```

If you need to restore the previous configuration:

```bash
cp .bashrc_backup .bashrc
```


