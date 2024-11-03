A Dockerfile is a text-based document that's used to create a container image. It provides instructions to the image builder on
- the commands to run,
- files to copy,
- startup command,
- etc.

---

a Dockerfile typically follows these steps:
1. Determine your base image
2. Install application dependencies
3. Copy in any relevant source code and/or binaries
4. Configure the final image

## Creating the Dockerfile

0. Create a file named `Dockerfile` in the same folder as the file `package.json`.

1. In the Dockerfile, define your base image by adding the following line:

```Dockerfile
FROM node:20-alpine
```

2. Now, define the working directory by using the `WORKDIR` instruction. This will specify where future commands will run and the directory files will be copied inside the container image.

```Dockerfile
WORKDIR /app
```

3. Copy all of the files from your project on your machine into the container image by using the COPY instruction:

```Dockerfile
COPY . .
```

4. Install the app's dependencies by using the yarn CLI and package manager. To do so, run a command using the RUN instruction:

```Dockerfile
RUN yarn install --production
```

5. Finally, specify the default command to run by using the CMD instruction:

```Dockerfile
CMD ["node", "./src/index.js"]
```

And with that, you should have the following Dockerfile:

```Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "./src/index.js"]
```

## Common instructions

Some of the most common instructions in a Dockerfile include:

`FROM <image>` - this specifies the **base** image that the build will **extend**.

`WORKDIR <path>` - this instruction specifies the **"working directory"** or the **path** in the image **where files will be copied** and **commands will be executed**.

`COPY <host-path> <image-path>` - this instruction tells the builder to copy files from the host and put them into the container image.

`RUN <command>` - this instruction tells the builder to run the specified command.

`ENV <name> <value>` - this instruction sets an environment variable that a **running container will use**.

`EXPOSE <port-number>` - this instruction sets configuration on the image that indicates ***a port the <u>image</u> would like to expose***.

`USER <user-or-uid>` - this instruction **sets the default user** for all <u>subsequent</u> instructions.

`CMD ["<command>", "<arg1>"]` - this instruction sets the **default command** a <u>container</u> using this image will run.

# Best Practices

## `FROM`

Whenever possible, use current official images as the basis for your images. 
Docker recommends the Alpine image as it is tightly controlled and small in size,
while still being a full Linux distribution.

## `WORKDIR`

For ***clarity and reliability***, you should always use absolute paths for your `WORKDIR`. 
Also, you should use `WORKDIR` instead of proliferating instructions like `RUN cd … && do-something`,
which are ***hard to read, troubleshoot, and maintain***.

## `COPY`

## `RUN`

Split long or complex `RUN` statements on multiple lines separated with backslashes 
to make your Dockerfile more readable, understandable, and maintainable.

For example, you can chain commands with the `&&` operator,
and use use escape characters to break long commands into multiple lines.

```Dockerfile
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo
```

You can also use **here documents** to run multiple commands without chaining them with a pipeline operator:

```Dockerfile
RUN <<EOF
apt-get update
apt-get install -y \
    package-bar \
    package-baz \
    package-foo
EOF
```

### `apt-get`

One common use case for `RUN` instructions in Debian-based images is to install software using `apt-get`. 
Because `apt-get` installs packages, the `RUN apt-get` command has several counter-intuitive behaviors to look out for.

Always combine `RUN apt-get update` with `apt-get install` in the same `RUN` statement. For example:

```Dockerfile
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo
```

Using `apt-get update` alone in a RUN statement causes **caching issues** and subsequent `apt-get install` instructions to fail.
For example, this issue will occur in the following Dockerfile:

```Dockerfile
# syntax=docker/dockerfile:1

FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl
```

***After*** building the image, all layers are in the **Docker cache**.
Suppose you later modify `apt-get install` by adding an extra package [`nginx`] as shown in the following Dockerfile:

```Dockerfile
# syntax=docker/dockerfile:1

FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

Docker sees the initial and modified instructions as identical and <u>reuses the cache from previous steps</u>.
As a result the `apt-get update` isn't executed because **the build uses the cached version**.
Because the `apt-get update` isn't run, your build can potentially get an outdated version of the curl and nginx packages.

Using `RUN apt-get update && apt-get install -y` **ensures your Dockerfile installs the latest** package versions
with no further coding or manual intervention. 
This technique is known as <u>***cache busting***</u>. 
You can also achieve cache busting by **specifying a package version**. 
This is known as <u>***version pinning***</u>. For example:

```Dockerfile
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo=1.3.*
```

Version pinning forces the build to retrieve a particular version ***regardless of what's in the cache***.
This technique can also reduce failures due to <u>**unanticipated changes in required packages**</u>.

Below is a well-formed RUN instruction that demonstrates all the `apt-get` recommendations:

```Dockerfile
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
    && rm -rf /var/lib/apt/lists/*
```

The `s3cmd` argument specifies a version `1.1.*`.
If the image previously used an older version,
***specifying the new one causes a cache bust*** of `apt-get update` and ensures the installation of the new version. 

***Listing packages on each line can also prevent mistakes in package duplication.***

In addition, when you clean up the apt cache by removing `/var/lib/apt/lists` it ***reduces the image size***,
since the apt cache isn't stored in a layer. 
Since the `RUN` statement starts with `apt-get update`, the package cache is always refreshed prior to `apt-get install`.

Official Debian and Ubuntu images automatically run `apt-get clean`, so explicit invocation is not required.

### Using pipes

Some `RUN` commands depend on the ability to pipe the output of one command into another, 
using the pipe character (`|`), as in the following example:

```Dockerfile
RUN wget -O - https://some.site | wc -l > /number
```

Docker executes these commands using the `/bin/sh -c` interpreter,
which only evaluates the exit code of the last operation in the pipe to determine success.
In the example above, this build step succeeds and produces a new image so long as the `wc -l` command succeeds,
even if the `wget` command fails.

If you want the command to fail due to an error at any stage in the pipe,
prepend `set -o pipefail &&` to ensure that an unexpected error prevents the build from inadvertently succeeding. 
For example:

```Dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

<u>***Note***</u>: 
Not all shells support the `-o pipefail` option.
In cases such as the dash [bash?] shell on Debian-based images, 
consider using the exec form of RUN to explicitly choose a shell that does support the `pipefail` option. For example:

```Dockerfile
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```

## `ENV`

Environment variable persistence can cause unexpected side effects. 
For example, setting 
`ENV DEBIAN_FRONTEND=noninteractive`
changes the behavior of `apt-get`, and may confuse users of your image.

If an environment variable is only needed during build, and not in the final image, consider setting a value for a single command instead:

```Dockerfile
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y ...
```

Or using `ARG`, which is not persisted in the final image:

```Dockerfile
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y ...
```

## `EXPOSE`

## `USER`

## `CMD`

The `CMD` instruction should be used to run the software contained in your image, along with any arguments. `CMD` should almost always be used in the form of `CMD ["executable", "param1", "param2"]`. Thus, if the image is for a service, such as Apache and Rails, you would run something like `CMD ["apache2","-DFOREGROUND"]`. Indeed, this form of the instruction is recommended for any <u>service-based image</u>.

In most other cases, `CMD` should be given an interactive shell, such as bash, python and perl. For example, `CMD ["perl", "-de0"]`, `CMD ["python"]`, or `CMD ["php", "-a"]`. Using this form means that when you execute something like `docker run -it python`, you’ll ***get dropped into a usable shell, ready*** to go. `CMD` should rarely be used in the manner of `CMD ["param", "param"]` in conjunction with `ENTRYPOINT`, unless you and your expected users are already quite familiar with how `ENTRYPOINT` works.

(An `ENTRYPOINT` allows you to configure a container that will run as an executable.)

## `LABEL`

## `ONBUILD`

An `ONBUILD` command executes after the current Dockerfile build completes. 
`ONBUILD` <u>executes **in any child** image derived `FROM` the current image</u>. 
Think of the `ONBUILD` command as ***an instruction that the parent Dockerfile gives to the child Dockerfile***.

A Docker build executes `ONBUILD` commands **before any command** in a child Dockerfile.

`ONBUILD` is useful for images that are going to be built `FROM` a given image. 
For example, you would use `ONBUILD` for a language stack image that builds arbitrary user software 
written in that language within the Dockerfile, as you can see in Ruby's `ONBUILD` variants.

Images built with `ONBUILD` should get a separate tag. For example, `ruby:1.9-onbuild` or `ruby:2.0-onbuild`.

Be careful when putting `ADD` or `COPY` in `ONBUILD`. 
The onbuild image fails catastrophically if the new build's context is missing the resource being added. 
Adding a separate tag, as recommended above, helps mitigate this by allowing the Dockerfile author to make a choice.