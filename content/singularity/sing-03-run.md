+++
title = "More on running containers"
slug = "sing-03-run"
weight = 3
+++

If you have not done so already, let's pull the latest Ubuntu container from Docker:

```sh
# cd ~/tmp
module load singularity
# salloc --cpus-per-task=1 --time=3:00:0 --mem-per-cpu=3600
singularity pull ubuntu.sif docker://ubuntu
```

We already saw some of these commands:

- `singularity shell ubuntu.sif` launches the container and opens an interactive shell inside it
- `singularity exec ubuntu.sif <command>` launches the container and runs a command inside it
- `singularity run ubuntu.sif` launches the container and executes the default runscript

Singularity matches users between the container and the host. For example, if you run a container that needs
to be root, you also need to be root outside the container.

## Running a single command

```sh
singularity exec ubuntu.sif ls /
singularity exec ubuntu.sif ls /; whoami
singularity exec ubuntu.sif bash -c "ls /; whoami"   # probably a safer way
singularity exec ubuntu.sif cat /etc/os-release
```

## Running a default script

We've already done this! If there is no default script, Singularity will give you the shell to type in your
commands.

## Starting a shell

We've already done this!

```sh
$ singularity shell ubuntu.sif
Singularity> whoami   # same username as in the host system
Singularity> groups   # same groups as on the host system
```

At startup Singularity simply copied the relevant user and group lines from the host system to files
`/etc/passwd` and `/etc/group` inside the container. Why do this? The container must ensure that you cannot
modify anything on the host system that you should not have permission to, i.e. you are restricted to the same
user permissions within the container as you are on the host system.

## Mounting external directories and copying environment

By default, Singularity containers are read-only, so you cannot write into its directories. However, from
inside the container you can organize read-write access to your directories on the host filesystem. The
command

```sh
singularity shell -B /home,/project,/scratch ubuntu.sif
```

will bind-mount `/home,/project,/scratch` inside the container so that these directories can be accessed for
both read and write, subject to your account's permissions, and then will run a shell. Inside the container:

```sh
pwd     # most likely your working directory
echo $USER
ls /home/$USER
ls /scratch/$USER
ls /project/def-sponsor00/$USER
```

You can mount host directories to specific paths inside the container, e.g.

```sh
singularity shell -B /project/def-sponsor00/${USER}:/project,$SCRATCH:/scratch ubuntu.sif
ls /project
ls /scratch
```

Note that by default Singularity typically mounts some of the host's directories. The flag `-C` will hide the
host's filesystems and environment variables, but then you need to explicitly bind-mount the needed paths (to
store results), e.g.

```sh
singularity shell -C -B /scratch ubuntu.sif   # inside see only /scratch
```

<!-- The reason is that it needs some space to store temporary files that get generated along the way, access some -->
<!-- host's system files, and also provide space in `/home` to store your data. -->

You can disable specific mounts, e.g. the following will start the container without a home directory:

```sh
singularity shell --no-mount home ubuntu.sif
cd     # no such file or directory
```

Alternatively, you can disable mounting `/home` with the `--no-home` flag. And you can disable multiple mounts
with something like `--no-mount tmp,sys,dev`.






In general, without `-C`, Singularity inherits all environment variables from the build/pull time. You can add
the `-e` flag to remove only the host's environment variables from your container, to start in a *cleaner*
environment:

```sh
$ singularity shell -e -B /home,/project,/scratch ubuntu.sif
Singularity> echo $USER
Singularity> $(whoami)
```

On the other hand, you can pass variables environment variables to your container by prefixing their names:

```sh
$ SINGULARITYENV_HI=hello singularity shell mpi.sif
Singularity> echo $HI
hello
```




Finally, you don't have to pass the same bind (`-B`) flags every time -- instead you can put them into a
variable (that can be stored in your `~/.bashrc` file):

```sh
export SINGULARITY_BIND="/home,/project/def-sponsor00/${USER}:/project,/scratch/${USER}:/scratch"
singularity shell ubuntu.sif
```

You can have more granular control (e.g. specifying read only) with the `--mount` flag -- for details see the
official
[Bind Paths and Mounts documentation](https://sylabs.io/guides/latest/user-guide/bind_paths_and_mounts.html).

> ## Key points
> 1. Your current directory and home directory are usually available by default in a container.
> 1. You have the same username and permissions in a container as on the host system.
> 1. You can specify additional host system directories to be available in the container.
> 1. It is a very good idea to use `-C` to hide the host's filesystems while mounting only few specific
>    directories.
