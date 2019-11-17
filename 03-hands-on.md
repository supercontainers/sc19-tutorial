/# Building and Running Containers

In the third hour we will build the preceding container from scratch.

Simply typing `singularity` will give you an summary of all the commands you can use. Typing `singularity help <command>` will give you more detailed information about running an individual command.

## The Singularity Image Format (SIF)

[SIF](https://github.com/sylabs/sif) is an open source implementation of the Singularity Container Image Format
that makes it easy to create complete and encapsulated container enviroments
stored in a single file.

![SIF Image](images/sif.png)

To build a SIF, you must use the `singularity build` command.  The `build` command installs an OS, sets up your container's environment and performs the actions stipulated on the definition file.  To use the `build` command, we need a **recipe file** (also called a definition file). A Singularity recipe file is a set of instructions telling Singularity what software to install in the container.

## Building a basic container

### Building from a container registry

The build command accepts a target as input and produces a container as output.

The target defines the method that build uses to create the container. It can be one of the following:

> URI beginning with library:// to build from the Container Library
> URI beginning with docker:// to build from Docker Hub
> URI beginning with shub:// to build from Singularity Hub
> path to a existing container on your local machine
> path to a directory to build from a sandbox
> path to a Singularity definition file

build can produce containers in two different formats that can be specified as follows.

> compressed read-only Singularity Image File (SIF) format suitable for production (default)
> writable (ch)root directory called a sandbox for interactive development ( --sandbox option)

Because build can accept an existing container as a target and create a container in either supported format you can convert existing containers from one format to another.

**Downloading an existing container from the Container Library**

You can use the build command to download a container from the Container Library.

```bash
singularity build lolcow.sif library://sylabs-jms/testing/lolcow
```

The first argument (lolcow.sif) specifies a path and name for your container. The second argument (library://sylabs-jms/testing/lolcow) gives the Container Library URI from which to download. By default the container will be converted to a compressed, read-only SIF. If you want your container in a writable format use the --sandbox option.

**Downloading an existing container from Docker Hub**

You can use build to download layers from Docker Hub and assemble them into Singularity containers.

```bash
singularity build nersc_ubuntu.sif docker://nersc/ubuntu-mpi:14.04
```

### Building from a recipe file

The Singularity source code contains several example definition files in the `/examples` subdirectory.  For more documentation about the singularity definition file you can go [here](https://sylabs.io/guides/3.4/user-guide/definition_files.html)

**Note:** You need to build containers on a file system where the sudo command can write files as root. This may not work in an HPC cluster setting if your home directory resides on a shared file server. If that's the case you may have to to `cd` to a local hard disk such as `/tmp`.

```bash
mkdir /sc19-tutorial
cd sc19-tutorial
vim ubuntu-mpi.def
```

Using your prefered IDE, build a file, should look something like this:

```bash
BootStrap: docker
From: nersc/ubuntu-mpi:14.04

%runscript
    echo "This is what happens when you run the container..."
```

See the [Singularity docs](https://www.sylabs.io/guides/3.4/user-guide/definition_files.html) for an explanation of each of these sections.

Singularity can build containers in several different file formats. The default is to build a [SIF](https://github.com/sylabs/sif) image. SIF is an open source implementation of the Singularity Container Image Format that makes it easy to create complete and encapsulated container enviroments stored in a single file.

Now let's use this recipe file as a starting point to build our `ubuntu_mpi.sif` container. Note that the build command requires `sudo` privileges, when used in combination with a recipe file.

```bash
sudo singularity build ubuntu-mpi.sif ubuntu-mpi.def
```

But if you want to shell into a container and tinker with it (like we will do here), you should build a sandbox (which is really just a file system under a foler directory).  This is great when you are still developing your container and don't yet know what should be included in the recipe file.

```bash
sudo singularity build --sandbox ubuntu_sandbox ubuntu-mpi.def
```

The `--sandbox` option in the command above tells Singularity that we want to build a special type of image (File system) for development purposes.

When your build finishes, you will have a basic Ubuntu + mpi installed container saved in a local directory called `ubuntu_mpi`.

### Using `shell` to explore and modify containers

Now let's enter our new container and look around.

```bash
singularity shell ubuntu_mpi.sif
```

Depending on the environment on your host system you may see your prompt change. Let's look at what OS is running inside the container.

```bash
Singularity ubuntu_mpi:~> cat /etc/os-release
NAME="Ubuntu"
VERSION="14.04.4 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.4 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

No matter what OS is running on your host, your container is running Ubuntu 14.04!

Let's try a few more commands:

```bash
$ singularity ubuntu_mpi:~> whoami
eduardo

$ singularity ubuntu_mpi:~> hostname
eduardo-laptop
```

Now let's compile a MPI bin and create a MPI ready container

Using your favorite text editor create a new file

> hellompi.def

```bash
BootStrap: docker
From: nersc/ubuntu-mpi:14.04

%post
    mkdir -p /sc19-tutorial
    cd /sc19-tutorial

    cat <<EOF >helloworld.c
    // Hello World MPI app
    #include <mpi.h>
    #include <stdio.h>

    int main(int argc, char** argv) {
        int size, rank;
        char buffer[1024];

        MPI_Init(&argc, &argv);

        MPI_Comm_size(MPI_COMM_WORLD, &size);
        MPI_Comm_rank(MPI_COMM_WORLD, &rank);

        gethostname(buffer, 1024);

        printf("hello from %d of %d on %s\n", rank, size, buffer);

        MPI_Barrier(MPI_COMM_WORLD);

        MPI_Finalize();
        return 0;
    }
EOF

   mpicc helloworld.c -o /sc19-tutorial/hello

%runscript
    /sc19-tutorial/hello $@
```

Let's rebuild the container with the new definition file.

```bash
sudo singularity build hellompi.sif hellompi.def
```

Note that we changed the name of the container.  By omitting the `--sandbox` option, we are building our container in the standard Singularity squashfs file format.  We are denoting the file format with the (optional) `.sif` extension (Singularity image format).  A squashfs file is compressed and immutable making it a good choice for a production environment.

Singularity stores a lot of [useful metadata](https://www.sylabs.io/guides/3.4/user-guide/environment_and_metadata.html).  For instance, if you want to see the recipe file that was used to create the container you can use the `inspect` command like so:

```bash
$ singularity inspect --deffile hellompi.sif
BootStrap: docker
From: nersc/ubuntu-mpi:14.04

%post
    mkdir -p /sc19-tutorial
    cd /sc19-tutorial

    cat <<EOF >helloworld.c
    // Hello World MPI app
    #include <mpi.h>
    #include <stdio.h>

    int main(int argc, char** argv) {
        int size, rank;
        char buffer[1024];

        MPI_Init(&argc, &argv);

        MPI_Comm_size(MPI_COMM_WORLD, &size);
        MPI_Comm_rank(MPI_COMM_WORLD, &rank);

        gethostname(buffer, 1024);

        printf("hello from %d of %d on %s\n", rank, size, buffer);

        MPI_Barrier(MPI_COMM_WORLD);

        MPI_Finalize();
        return 0;
    }
EOF

   mpicc helloworld.c -o /sc19-tutorial/hello

%runscript
    /sc19-tutorial/hello $@
```

Now let's run our container! 

```bash
singularity run hellompi.sif
hello from 0 of 1 on Eduardos-MacBook-Pro.local
```
