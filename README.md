
About `qayiq`

`qayiq` is a hands-on project developed to deepen understanding of Linux containers and the Linux kernel, leveraging various online guides. It explores core Linux kernel features essential for containerization:

`Namespaces`: Essential for isolating kernel objects into distinct sets accessible only by specific process trees. For instance, PID namespaces isolate processes, and network namespaces segregate network stacks.

`Seccomp`: Limits the system calls a process can execute, enhancing security by controlling syscall access.

`Capabilities`: Defines privileges for uid 0 (root), restricting the actions root can perform to improve security.

`Cgroups`: Manages resource allocation (memory, disk I/O, CPU time) among processes, ensuring efficient resource use.

These components collectively form the backbone of Linux containers, showcasing the isolation, resource control, and security mechanisms at play.

## Usage

`qayiq` can be used to run `bin/sh . ` from the `/` directory as `root` (-u 0) with the following command (optional `-v` for verbose output):

```bash
$ sudo ./bin/qayiq -u 0 -m / -c /bin/sh -a . [-v]

22:08:41 INFO  ./src/qayiq.c:96: initializing socket pair...
22:08:41 INFO  ./src/qayiq.c:103: setting socket flags...
22:08:41 INFO  ./src/qayiq.c:112: initializing container stack...
22:08:41 INFO  ./src/qayiq.c:120: initializing container...
22:08:41 INFO  ./src/qayiq.c:131: initializing cgroups...
22:08:41 INFO  ./src/cgroups.c:73: setting memory.max to 1G...
22:08:41 INFO  ./src/cgroups.c:73: setting cpu.weight to 256...
22:08:41 INFO  ./src/cgroups.c:73: setting pids.max to 64...
22:08:41 INFO  ./src/cgroups.c:73: setting cgroup.procs to 1458...
22:08:41 INFO  ./src/qayiq.c:139: configuring user namespace...
22:08:41 INFO  ./src/qayiq.c:147: waiting for container to exit...
22:08:41 INFO  ./src/container.c:43: ### QAYIQCONTAINER STARTING - type 'exit' to quit ###

# ls
bin         home                lib32       media       root        sys         vmlinuz
boot        initrd.img          lib64       mnt         run         tmp         vmlinuz.old
dev         initrd.img.old      libx32      opt         sbin        usr
etc         lib                 lost+found  proc        srv         var
# echo "i am a container"
i am a container
# exit

22:08:55 INFO  ./src/qayiq.c:153: freeing resources...
22:08:55 INFO  ./src/qayiq.c:168: Thank you for using qayiq
```

## Setup

`qayiq` requires a number of tools and libraries to be installed to build the project and for development.

```bash
# Install all required tooling and dependencies
$ sudo apt install -y make
$ make setup
```

### Dependencies

`qayiq` depends on the following "non-standard" libraries:

- `libseccomp`: used to set up seccomp filters
- `libcap`: used to set container capabilities
- `libcuni1`: used for testing with CUnit
- [argtable](http://argtable.org/): used to parse command line arguments
- [rxi/log.c](https://github.com/rxi/log.c): used for logging

`qayiq` uses a number of LLVM-18-based tools for development, linting, formatting, debugging and Valgrind to check for memory leaks.

## Build

The included `Makefile` provides a few targets to build `qayiq`.
The variable `debug=1` can be set to run any of the targets in "debug" mode, which builds the project with debug symbols and without optimizations (especially useful for the debugger and valgrind).

```bash
# Build qayiq (executable is in bin/)
# The default target also runs, "make lint" and "make format" to lint and format the code
$ make


# Build qayiq with debug flags
$ make debug=1
```

## Development
`qayiq` is developed using [Visual Studio Code](https://code.visualstudio.com/) and [GitHub Codespaces](https://github.com/codespaces). The repository contains all the necessary configuration files to use these tools effectively.
`qayiq` relies on low-level Linux features, so it must be run on a Linux system. [GitHub Codespaces](https://github.com/codespaces) acts weird at times when tweaking low-level container settings: I found [getutm.app](https://getutm.app) to work well with [Debian](http://debian.org) on my Mac when in doubt.

The included `Makefile` provides a few targets useful for development:

```bash
# Run tests
$ make test

# Run linter
$ make lint

# Run formatter
$ make format

# Run valgrind
$ make check

# Clean the build
$ make clean
```

Furthermore, the project includes a [Visual Studio Code](https://code.visualstudio.com/) configuration in `.vscode/` that can be used to run the built-in debugger (at this moment it is "disabled" since `qayiq` should be run as `root` and [CodeLLDB](https://github.com/vadimcn/codelldb) does not have that option).

## Structure

The project is structured as follows:

```txt
├── .devcontainer       configuration for GitHub Codespaces
├── .github             configuration GitHub Actions and other GitHub features
├── .vscode             configuration for Visual Studio Code
├── bin                 the executable (created by make)
├── build               intermediate build files e.g. *.o (created by make)
├── include             header files
├── lib                 third-party libraries
├── scripts             scripts for setup and other tasks
├── src                 C source files
│   ├── qayiq.c         (main) Entry point for the CLI
│   └── *.c
├── tests               contains tests
├── .clang-format       configuration for the formatter
├── .clang-tidy         configuration for the linter
├── .gitignore
├── LICENSE
├── Makefile
└── README.md
```

## Testing and documentation

At the moment, the project does not contain any automated tests or tools to document the code.
In the future, suitable tools for automated testing and documentation might be added.

## Limitations

1. `qayiq` has been tested on Debian 12, running the Linux kernel at version 6.1.0, with user namespaces and cgroupsv2 enabled.

2. `qayiq` does not handle network namespaces, so the container cannot access the network. Networking can roughly be setup as follows:

   - create a new network namespace
   - create a virtual ethernet pair
   - move one end of the pair to the new network namespace
   - assign an IP address to the interface in the new network namespace
   - setup routing and NAT

    In C this is usually done via the `rtnetlink` interface. Furthermore, network usage can be limited with the `net_prio` cgroup controller.

## Improvements

- Investigate further, document and refactor: user and mount and cgroup namespaces, syscalls and capabilities
- The functions in `cgroups.c`, `mount.c`, `sec.c`, `user.c` are specific to `qayiq` and should be made more generic
- CMake and Conan are industry standards, so they should be used eventually instead of Make and the current build system. Unfortunately, CMake and Conan also add a lot of complexity which is not needed at this time.

## Credits

Some of the resources that have been used to develop `qayiq` are:

- [Linux kernel documentation](https://www.kernel.org/doc/html/latest/index.html)
- [user_namespaces docs](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
- [cgroup_namespaces docs](https://man7.org/linux/man-pages/man7/cgroup_namespaces.7.html)
- [mount_namespaces docs](https://man7.org/linux/man-pages/man7/mount_namespaces.7.html)
- [Linux containers in 500 lines of code](https://blog.lizzie.io/linux-containers-in-500-loc.html#fn.6)
- [Containers from scratch](https://medium.com/inside-sumup/containers-from-scratch-part-1-b719effd1e0a)
- [The current adoption status of cgroup v2 in containers](https://medium.com/nttlabs/cgroup-v2-596d035be4d7)
- [Docker under the Hood](https://medium.com/devops-dudes/docker-under-the-hood-0-naming-components-and-runtime-9a89cfbbe783)
- [A deep dive into Linux namespaces](https://ifeanyi.co/posts/linux-namespaces-part-1/)

## QA

- **What does "qayiq" mean?** It's Uzbek word for "a boat".
