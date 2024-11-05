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

```Dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
```

Or

```Dockerfile
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```

Or

```Dockerfile
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

The FROM instruction initializes a new build stage and sets the base image for subsequent instructions. As such, a valid Dockerfile must start with a FROM instruction. The image can be any valid image.

- `ARG` is the only instruction that may precede FROM in the Dockerfile.

- `FROM` can appear multiple times within a single Dockerfile to ***create multiple images*** or <u>use one build stage as a dependency for another</u>. Simply make a note of the last image ID output by the commit before each new `FROM` instruction. Each `FROM` instruction clears any state created by previous instructions.

- Optionally a name can be given to a new **build stage** by adding `AS name` to the `FROM` instruction. The name can be used in subsequent `FROM <name>`, `COPY --from=<name>`, and `RUN --mount=type=bind,from=<name>` instructions to refer to the image built in this stage.

- The `tag` or `digest` values are optional. If you omit either of them, the builder assumes a `latest` tag by default. The ***builder returns an error*** if it can't find the `tag` value.

### Understand how ARG and FROM interact

`FROM` instructions support variables that are declared by any `ARG` instructions that occur before the first `FROM`.

```Dockerfile
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

An `ARG` declared before a `FROM` is outside of a build stage, so it can't be used in any instruction after a `FROM`. To use the default value of an `ARG` declared before the first `FROM` use an `ARG` instruction without a value inside of a build stage:

```Dockerfile
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

### best practices with `FROM`

Whenever possible, use current official images as the basis for your images. 
Docker recommends the Alpine image as it is tightly controlled and small in size,
while still being a full Linux distribution.

## `WORKDIR`

The `WORKDIR` instruction sets the working directory for any 
- `RUN`, 
- `CMD`, 
- `ENTRYPOINT`, 
- `COPY` and 
- `ADD` 

instructions that follow it in the Dockerfile.
If the `WORKDIR` doesn't exist, it ***will be created even if it's not used*** in any subsequent Dockerfile instruction.

The `WORKDIR` instruction can be used multiple times in a Dockerfile.
If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction.
For example:

```Dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

The output of the final `pwd` command in this Dockerfile would be `/a/b/c`.

The `WORKDIR` instruction can resolve environment variables previously set using `ENV`.
You can only use environment variables explicitly set in the Dockerfile.
For example:

```Dockerfile
ENV DIRPATH=/path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```

The output of the final `pwd` command in this Dockerfile would be `/path/$DIRNAME`.

If not specified, the default working directory is `/`. 
In practice, if you aren't building a Dockerfile from scratch (`FROM scratch`), 
the `WORKDIR` may likely be set by the base image you're using.

Therefore, to avoid ***unintended operations in unknown directories***, it's best practice to set your `WORKDIR` explicitly.

---

For ***clarity and reliability***, you should always use absolute paths for your `WORKDIR`. 
Also, you should use `WORKDIR` instead of proliferating instructions like `RUN cd … && do-something`,
which are ***hard to read, troubleshoot, and maintain***.

## `COPY`

### What is a build context?

The build context is ***the <u>set of files</u> that your build can access***. The positional argument that you pass to the build command specifies the context that you want to use for the build:

```
docker build [OPTIONS] PATH | URL | -
                       ^^^^^^^^^^^^^^
```

### What is a multi-stage build?

With multi-stage builds, you use multiple `FROM` statements in your Dockerfile. Each `FROM` instruction can use a different base, and each of them begins a new stage of the build. You can ***<u>selectively</u> copy artifacts from one stage to another, leaving behind*** everything you don't want in the final image.

The following Dockerfile has two separate stages: one for building a binary, and another where the binary gets copied from the first stage into the next stage.

```Dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.23
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```

You only need the single Dockerfile. No need for a separate build script. Just run `docker build`.

```
docker build -t hello .
```

The end result is a tiny production image with nothing but the binary inside. None of the build tools required to build the application are included in the resulting image.

How does it work? The second `FROM` instruction starts a new build stage with the `scratch` image as its base. The `COPY --from=0` line copies just the built artifact from the previous stage into this new stage. The Go SDK and any intermediate artifacts are left behind, and not saved in the final image.

### best practices

`ADD` and `COPY` are ***functionally similar***. `COPY` supports basic copying of files into the container, from ***the build context*** or from ***a stage in a multi-stage build***. `ADD` supports features for ***fetching files from remote*** HTTPS and Git URLs, and ***extracting tar files automatically*** when adding files from the build context.

You'll mostly want to use `COPY` for copying files from one stage to another in a multi-stage build. If you need to add files from the build context to the container temporarily to execute a `RUN` instruction, you can often substitute the `COPY` instruction with a bind mount instead. For example, to temporarily add a `requirements.txt` file for a `RUN pip install` instruction:

```Dockerfile
RUN --mount=type=bind,source=requirements.txt,target=/tmp/requirements.txt \
    pip install --requirement /tmp/requirements.txt
```

Bind ***mounts are more efficient*** than `COPY` for including files from the build context in the container. Note that bind-mounted files are only added temporarily for a single `RUN` instruction, and ***don't persist in the final image***. If you need to include files from the build context in the final image, use `COPY`.

## `ADD`

https://docs.docker.com/reference/dockerfile/#add

`ADD` has two forms. The latter form is required for paths containing whitespace.

```Dockerfile
ADD [OPTIONS] <src> ... <dest>
ADD [OPTIONS] ["<src>", ... "<dest>"]
```

The `ADD` instruction copies new files or directories from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

Files and directories can be copied from 
- the build context [???],
- a remote URL, or
- a Git repository.

The `ADD` and `COPY` instructions are functionally similar, but serve slightly different purposes.

### Source

You can specify multiple source files or directories with `ADD`. The **last argument must always be the destination**. For example, to add two files, `file1.txt` and `file2.txt`, <u>from the build context</u> to `/usr/src/things/` in the build container:

```Dockerfile
ADD file1.txt file2.txt /usr/src/things/
```

If you specify multiple source files, either ***directly or using a wildcard***, then the destination must be a directory (must end with a slash `/`).

To add files from a remote location, you can specify a URL or the address of a Git repository as the source. For example:

```Dockerfile
ADD https://example.com/archive.zip /usr/src/things/
ADD git@github.com:user/repo.git /usr/src/things/
```

BuildKit ***detects the type*** of `<src>` and processes it accordingly.

- If `<src>` is a local file or directory, the contents of the directory are ***<u>copied to</u>*** the specified destination.
- If `<src>` is a local tar archive, it is ***<u>decompressed and extracted to</u>*** the specified destination.
- If `<src>` is a URL, the contents of the URL are ***<u>downloaded and placed at</u>*** the specified destination.
- If `<src>` is a Git repository, the repository is ***<u>cloned to</u>*** the specified destination.

#### Adding files from the build context

Any relative or local path that doesn't begin with a 

- `http://`,
- `https://`, or
- `git@`

protocol prefix is considered a local file path. The local file path is ***relative to the build context***. For example, if the build context is the current directory, `ADD file.txt /` adds the file at `./file.txt` to the root of the filesystem in the build container.

When adding source files from the build context, their paths are interpreted as relative to the root of the context. If you specify a relative path leading outside of the build context, such as `ADD ../something /something`, **parent directory paths are stripped out** automatically. The effective source path in this example becomes `ADD something /something`.

If the source is a directory, the contents of the directory are copied, including filesystem metadata. <u>The directory itself isn't copied, only its contents</u>. If it contains subdirectories, these are also copied, and merged with any existing directories at the destination. Any conflicts are resolved in favor of the content being added, on a file-by-file basis, except if you're trying to copy a directory onto an existing file, in which case an error is raised.

If the source is a file, the file and its metadata are copied to the destination. File permissions are preserved.

If the source is a file and a directory with the same name exists at the destination, an error is raised.

If you pass a Dockerfile through stdin to the build (`docker build - < Dockerfile`), there is no build context. In this case, you can only use the `ADD` instruction to copy remote files. You can also pass a tar archive through stdin: (`docker build - < archive.tar`), the Dockerfile at the root of the archive and the rest of the archive will be used as the context of the build.

### best practices 

The `ADD` instruction is best for when you need to download a remote artifact as part of your build. `ADD` is better than manually adding files using something like `wget` and `tar`, because it ensures a more precise build cache. `ADD` also has built-in support for checksum validation of the remote resources, and a protocol for parsing branches, tags, and subdirectories from Git URLs.

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

```Dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

The `EXPOSE` instruction informs Docker that the **container** listens on the specified network ports **at runtime**. You can specify whether the port listens on TCP or UDP, and the <u>default is TCP</u> if you don't specify a protocol.

The `EXPOSE` instruction ***doesn't actually publish*** the port. It functions as a type of **documentation between the person who builds the image and the person who runs the container**, about which ports are intended to be published. To publish the port when running the container, use the `-p` flag on `docker run` to publish and map one or more ports, or the `-P` flag to publish all exposed ports and map them to <u>high-order</u> [???] ports.

By default, `EXPOSE` assumes TCP. You can also specify UDP:

```Dockerfile
EXPOSE 80/udp
```

To expose on both TCP and UDP, include two lines:

```Dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

In this case, if you use `-P` with `docker run`, the port will be exposed once for TCP and once for UDP. Remember that `-P` uses an ephemeral high-ordered host port on the host, so TCP and UDP doesn't use the same port.

Regardless of the EXPOSE settings, you can <u>override</u> them at runtime by using the `-p` flag. For example

```Dockerfile
docker run -p 80:80/tcp -p 80:80/udp ...
```

---

The `EXPOSE` instruction indicates the **ports on which a <u>container</u> listens for connections**. Consequently, you should use the common, traditional port for your application. For example, an image containing the Apache web server would use `EXPOSE 80`, while an image containing MongoDB would use `EXPOSE 27017` and so on.

For external access, your users can execute `docker run` with a flag indicating how to map the specified port to the port of their choice. For **container linking**, Docker provides environment variables for the path from the recipient container back to the source [container](for example, `MYSQL_PORT_3306_TCP`).

## `USER`

```Dockerfile
USER <user>[:<group>]
```

or 

```Dockerfile
USER <UID>[:<GID>]
```

The `USER` instruction sets the user name (or UID) and optionally the user group (or GID)
to use as the default user and group for the **remainder of the current <u>stage</u>**.
The specified user is used for `RUN` instructions and at runtime,
runs the relevant `ENTRYPOINT` and `CMD` commands.

---

If a service can run without privileges, use `USER` to change to a non-root user.
Start by creating the user and group in the Dockerfile with something like the following example:

```Dockerfile
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
```

#### Note 1
Consider an explicit UID/GID.
Users and groups in an image are assigned a non-deterministic UID/GID 
in that the "next" UID/GID is assigned regardless of image rebuilds.
So, if it's critical, you should assign an explicit UID/GID.

#### Note 2

Due to an unresolved bug in the Go archive/tar package's handling of sparse files,
attempting to create a user with a significantly large UID inside a Docker container
can lead to disk exhaustion because `/var/log/faillog` in the container layer is filled with NULL (\0) characters.
A workaround is to pass the `--no-log-init` flag to useradd.
The Debian/Ubuntu `adduser` wrapper does not support this flag.

---

Avoid installing or using `sudo` as it has unpredictable TTY and signal-forwarding behavior that can cause problems.

Lastly, to reduce layers and complexity, avoid switching `USER` back and forth frequently.

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