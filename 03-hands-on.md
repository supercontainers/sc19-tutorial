# Building and Running Containers

In the second hour we will build the preceding container from scratch.

Simply typing `singularity` will give you an summary of all the commands you can use. Typing `singularity help <command>` will give you more detailed information about running an individual command.

## Building a basic container

To build a singularity container, you must use the `build` command.  The `build` command installs an OS, sets up your container's environment and installs the apps you need.  To use the `build` command, we need a **recipe file** (also called a definition file). A Singularity recipe file is a set of instructions telling Singularity what software to install in the container.

The Singularity source code contains several example definition files in the `/examples` subdirectory.  Let's copy the ubuntu example to our home directory and inspect it.

**Note:** You need to build containers on a file system where the sudo command can write files as root. This may not work in an HPC cluster setting if your home directory resides on a shared file server. If that's the case you may have to to `cd` to a local hard disk such as `/tmp`.

```bash
mkdir ../app
cd app
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

Now let's use this recipe file as a starting point to build our `ubuntu_mpi.sif` container. Note that the build command requires `sudo` privileges, when used in combination with a recipe file.

```bash
sudo singularity build --sandbox ubuntu_sandbox ubuntu-mpi.def
```

The `--sandbox` option in the command above tells Singularity that we want to build a special type of image (File system) for development purposes.

Singularity can build containers in several different file formats. The default is to build a [SIF](https://github.com/sylabs/sif) image. SIF is an open source implementation of the Singularity Container Image Format that makes it easy to create complete and encapsulated container enviroments stored in a single file.

But if you want to shell into a container and tinker with it (like we will do here), you should build a sandbox (which is really just a file system under a foler directory).  This is great when you are still developing your container and don't yet know what should be included in the recipe file.

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
    mkdir -p /app
    cd /app

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

   mpicc helloworld.c -o /app/hello

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
    mkdir -p /app
    cd /app

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

   mpicc helloworld.c -o /app/hello
```

Now let's run our container! 

```bash
singularity exec hellompi.sif /app/hello
hello from 0 of 1 on Eduardos-MacBook-Pro.local
```
