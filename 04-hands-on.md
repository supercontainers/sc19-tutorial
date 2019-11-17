# Sylabs cloud

In this sections we will build learn how to push our `hellompi.sif` image into the sylabs library, then to pull it from a different host, and how can we add trust by signing our image with a PGP protocol and store my public keys in the cloud.

Also we will learn how to use the sylabs cloud builder to build our image without root privileges.

## Remote Login

first make sure you have a Sylabs cloud token - get one [here](https://cloud.sylabs.io/home). Save it to ~/.singularity/sylabs-token

List and Login to Remotes

To list existing remote endpoints, run this:

```bash
singularity remote list

NAME           URI              GLOBAL
[SylabsCloud]  cloud.sylabs.io  YES
```

The [...] brackets around the name SylabsCloud show that this is the current default remote endpoint.

To login to a remote, for the first time or if your token expires or was revoked:

```bash
singularity remote login

# Login to another remote endpoint
singularity remote login <remote_name>

# example...
$ singularity remote login SylabsCloud
singularity remote login SylabsCloud
INFO:    Authenticating with remote: SylabsCloud
Generate an API Key at https://cloud.sylabs.io/auth/tokens, and paste here:
API Key:
INFO:    API Key Verified!

# or with the token as a string
$ singularity remote login --tokenfile <login-token-string> SylabsCloud
INFO:    Authenticating with remote: SylabsCloud
Generate an API Key at https://cloud.sylabs.io/auth/tokens, and paste here:
API Key:
INFO:    API Key Verified!
```

## Keystore

On 2018 Sylabs launched the a suite of cloud services, a comfortable home for SIF based containers.

but first we need to go to [Sylabs Cloud](https://cloud.sylabs.io/library) and create a login user, then create a token file and store it under `$HOME/.singularity/sylabs-token`, as default. (you can always store it in a prefered path and use the `-t/--tokenfile` flag to redirect to a new path, or set the token as an env var `SYLABS-TOKEN=A_VERY_LONG_TOKEN`)

The Sylabs [Keystore](https://cloud.sylabs.io/keystore) offers a way to easily search, visualize and share SIF signing keys, so that you can verify images downloaded from the Container Library, and allow others to verify images you create.

Firsy we need to create a set of keys!

The 'keys' command  allows you to manage local OpenPGP key stores by creating a new store and new keys pairs. You can also list available keys from the default store. Finally, the keys command offers subcommands to communicate with an HKP key server to fetch and upload public keys.

```bash
eduardo@linux> singularity keys list
Public key listing (/home/eduardo/.singularity/sypgp/pgp-public):

```

Here we can see that I currently don't have any keys on my host.

```bash
eduardo@linux> singularity keys newpair
Enter your name (e.g., John Doe) : eduardo arango
Enter your email address (e.g., john.doe@example.com) : eduardo@sylabs.io
Enter optional comment (e.g., development keys) : oss all the things
Generating Entity and OpenPGP Key Pair... Done
Enter encryption passphrase :
eduardo@linux> singularity keys list
Public key listing (/home/eduardo/.singularity/sypgp/pgp-public):

0) U: eduardo arango (oss all the things) <eduardo@sylabs.io>
   C: 2019-01-21 12:14:12 -0500 -05
   F: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
   L: 4096
   --------
```

Now we have a keypair, now let's use this keys to sign our cow, so we can tell our cow from the herd, with the `singularity sign` command.

The sign command allows a user to create a cryptographic signature on either a single data object or a list of data objects within the same SIF group. By default without parameters, the command searches for the primary partition and creates a verification block that is then added to the SIF container file.

```bash
eduardo@linux> singularity sign hellompi.sif
Signing image: hellompi.sif
Enter key passphrase:
Signature created and applied to hellompi.sif
```

Now we are going to use the `verify` comand. The verify command allows a user to verify cryptographic signatures on SIF container files. There may be multiple signatures for data objects and multiple data objects signed. By default the command searches for the primary partition signature. If found, a list of all verification blocks applied on the primary partition is gathered so that data integrity (hashing) and signature verification is done for all those blocks.

```bash
eduardo@linux> singularity verify hellompi.sif
Verifying image: hellompi.sif
Data integrity checked, authentic and signed by:
     eduardo arango (oss all the things) <eduardo@sylabs.io>, KeyID XXXXXXXXXXXXXXXX
```

That's my container image!

But now... how can I verify my image if I take it out from my trusted PC, and take it to a friends PC, I need to also bring my key pairs!, with `singularity keys push` , Upload an OpenPGP public key to a key server.

```bash
eduardo@linux> singularity keys push XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
public key `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX' pushed to server successfully
```

Now I can go to the [Keystore](https://cloud.sylabs.io/keystore) and check that my key pair is there, and so I can pull it from a different host (with my Sylabs login credentials) and verify my image.

## The container library

Available as a cloud service, or for on-prem deployment, the sif Library help users to manage, store and share containers. The cloud service portion will offer common Linux distributions, programming languages and AI frameworks, which will be updated regularly. A clear web interface and simple command-line syntax will let you search across containers and `singularity pull` them down to your system.

Now that we have an account at sylabs cloud, and the token file set, let's push our SIF into the clouds!

> push

The Singularity push command allows you to upload your sif image to a library of your choosing.

```bash
eduardo@linux> singularity push hellompi.sif library://sylabsed/examples/hellompi:latest
INFO:    Now uploading hellompi.sif to the library
 139.66 MiB / 139.66 MiB [=================================================================================================================================================================] 100.00% 2.75 MiB/s 50s
INFO:    Setting tag latest
WARNING: latest replaces an existing tag
```

The warning is due the user `syalbsed` (aka me) already had a hellompi tagged as latest, so it must be replaced to the "new" image.

> pull

The 'pull' command allows you to download or build a container from a given URI.  Supported URIs include:

- library: Pull an image from the currently configured library
     library://[user[collection/[container[:tag]]]]
- docker: Pull an image from Docker Hub
     docker://user/image:tag
- shub: Pull an image from Singularity Hub to CWD
     shub://user/image:tag

This tutorial focuses on the Sylabs Cloud library, so we will use the library URI to retrieve our image.

My `hellompi` image is at <https://cloud.sylabs.io/library/_container/5b9e91c694feb900016ea40b> , you can go there to download the image from a UI environment or via CLI

```bash
# Pull with Singularity
$ singularity pull <name.sif> library://sylabsed/examples/hellompi:latest
# Pull by unique ID (reproducible even if tags change)
$ singularity pull  <name.sif> library://sylabsed/examples/hellompi:sha256.699eccab2e5c31043f540a9d5fbd3c8dc105e7355bbb7b855697aa223f5b71d0
```

**note** if you don't set a name `<name.sif>` the command `singularity pull` will by default set the image name based on the image name and tag name, in this case will be `hellompi_latest.sif`

Now you know how to `push/pull` your SIF images! now let's make sure we are pulling the same image, our just add a security step to our workflow

## The remote builder

So, what if I don't have root privileges on my host?

For this concern, Sylabs has developed a Remote Build Service, that can be leveraged from the command line by using the --remote flag:

```bash
eduardo@linux> singularity build --remote hellompi.sif hellompi.def
searching for available build agent.........INFO:    Starting build...
Getting image source signatures
Copying blob sha256:dca7be20e546564ad2c985dae3c8b0a259454f5637e98b59a3ca6509432ccd01
 40.80 MiB / 40.80 MiB  1s
Copying blob sha256:40bca54f5968c2bdb0d8516e6c2ca4d8f181326a06ff6efee8b4f5e1a36826b8
 816 B / 816 B  0s
Copying blob sha256:61464f23390e7d30cddfd10a22f27ae6f8f69cc4c1662af2c775f9d657266016
 515 B / 515 B  0s
Copying blob sha256:d99f0bcd5dc8b557254a1a18c6b78866b9bf460ab1bf2c73cc6aca210408dc67
 854 B / 854 B  0s
Copying blob sha256:120db6f90955814bab93a8ca1f19cbcad473fc22833f52f4d29d066135fd10b6
 163 B / 163 B  0s
Copying config sha256:473d4d9cf99523631d35a7645b7a5f276db55ec97da2f10ebf915e14b3c80552
 2.62 KiB / 2.62 KiB  0s
Writing manifest to image destination
Storing signatures
INFO:    Creating SIF file...
INFO:    Build complete: /tmp/image-564730361
INFO:    Now uploading /tmp/image-564730361 to the library
 36.42 MiB / 36.42 MiB  100.00% 37.88 MiB/s 0s
INFO:    Setting tag latest
 36.42 MiB / 36.42 MiB [===================================================================================================================================================================] 100.00% 2.36 MiB/s 15s
```

Now you have you image, and is also stored in the cloud library, in case you need to re-used it for the future.

### Resources

- [Library](https://cloud.sylabs.io/library)
- [Remote builder](https://cloud.sylabs.io/builder)
- [Keystore](https://cloud.sylabs.io/keystore)

> Labnote

- [Sylabs container library manage secure containers](https://www.sylabs.io/2018/05/sylabs-container-library-manage-secure-containers/)
