[helpdesk]: ../../helpdesk/index.md

# Spack

[Spack](https://spack.readthedocs.io/en/latest/) is a package manager that
automates the download-build-install process for HPC software. It is especially
useful for building and maintaining installations of many different versions of
the same software. It also comes with a virtual environment feature that is
useful when developing software.

LUMI provides a module to load a pre-configured Spack instance: `module load
spack`. When you load this module, you will use a Spack instance configured to
compile software with the Cray programming environment. The software will be
installed in a location determined by you in `$SPACK_USER_PREFIX`. This Spack
instance is "chained" to the upstream one in `/appl/lumi/spack`, which means
that you can build new packages on top of the already installed ones by the
LUMI User Support Team (similar to how our EasyBuild setup works).

!!! important "The software installed with Spack in /appl/lumi/spack/ is provided as is."
    It may not have received any testing after installation! We also build the
    software there in a more fool-proof way with slightly less optimizations:
    Zen2 architecture instead of Zen3, OpenBLAS instead of Cray LibSci, and
    Netlib Scalapack. This may have a small impact on performance, but is
    usually fine.

## Using Spack on LUMI

**To install software with Spack**, perform the following steps. In this
example, we will install [kokkos](https://kokkos.org/about/), a C++ parallel
programming framework, into a hypothetical project storage folder
`/project/project_465000XYZ/spack`. We want to configure this package with AMD
GPU support and activate extra array bounds checking for debugging.

1. Initialize Spack.

    ```bash
    $ export SPACK_USER_PREFIX=/project/project_465000XYZ/spack 
    $ module load spack/22.08-2
    ```

    We recommend that you set `$SPACK_USER_PREFIX` in e.g. your `.bash_profile`
    file to avoid having to set it every time you want to use Spack.

2. Check the information Spack has about the package and especially the
   configuration options:

    ```bash
    $ spack info kokkos
    ```

    From reading the package information, it becomes clear that the install
    command should be:

    ```bash
    $ spack install kokkos+rocm+debug_bounds_check amdgpu_target=gfx90a %gcc@11.2.0
    ```

    The flag `+rocm` activates GPU support, and `+debug_bounds_check` adds the
    array bounds checking. We also need to specify the type of GPU:
    `amdgpu_target==gfx90a` (note the double equal signs which has the special meaning of propagating the GPU target to all dependencies). In this case, we give no explicit specification of a compiler, which means that Spack will choose gcc 11.2.0 for us when compiling.
    
3. Before installing, it is good practice to check the dependencies that Spack
   will install. Sometimes this can be many, many, packages! Running this
   command can take some time (up to a few minutes):

    ```console
    $ spack spec -I kokkos+rocm+debug_bounds_check amdgpu_target==gfx90a
        Input spec
    --------------------------------
    -   kokkos+debug_bounds_check+rocm amdgpu_target==gfx90a

    Concretized
    --------------------------------
    -   kokkos@3.7.00%gcc@11.2.0~aggressive_vectorization~compiler_warnings~cuda~cuda_constexpr~cuda_lambda~cuda_ldg_intrinsic~cuda_relocatable_device_code~cuda_uvm~debug+debug_bounds_check~debug_dualview_modify_check~deprecated_code~examples~explicit_instantiation~hpx~hpx_async_dispatch~hwloc~ipo~memkind~numactl~openmp~openmptarget~pic+profiling~profiling_load_print~pthread~qthread+rocm+serial+shared~sycl~tests~tuning~wrapper amdgpu_target=gfx90a build_system=cmake build_type=RelWithDebInfo intel_gpu_arch=none std=14 arch=linux-sles15-zen2
    [^]      ^cmake@3.24.3%gcc@11.2.0~doc+ncurses+ownlibs~qt build_system=generic build_type=Release arch=linux-sles15-zen2
    [^]          ^ncurses@6.3%gcc@11.2.0~symlinks+termlib abi=none build_system=autotools arch=linux-sles15-zen2
    [^]              ^pkgconf@1.8.0%gcc@11.2.0 build_system=autotools arch=linux-sles15-zen2
    [^]          ^openssl@1.1.0i-fips%gcc@11.2.0~docs~shared build_system=generic certs=mozilla arch=linux-sles15-zen2
    [^]      ^hip@5.2.3%gcc@11.2.0~ipo build_system=cmake build_type=Release patches=7ed1232 arch=linux-sles15-zen2
    [^]          ^comgr@5.2.3%gcc@11.2.0~ipo build_system=cmake build_type=Release arch=linux-sles15-zen2
    [^]              ^rocm-cmake@5.2.3%gcc@11.2.0~ipo build_system=cmake build_type=Release arch=linux-sles15-zen2
    [^]          ^glx@1.4%gcc@11.2.0 build_system=bundle arch=linux-sles15-zen2
    [^]              ^mesa@22.1.2%gcc@11.2.0+glx+llvm+opengl~opengles+osmesa~strip build_system=meson buildtype=release default_library=shared patches=ee737d1 arch=linux-sles15-zen2
    ...

    ```

    The packages that are already installed in your own Spack instance will
    have a `[+]` in the first column, and the packages that Spack found
    installed upstream will have `[^]`. A `-` means Spack did not find the
    package and will build it. In this case, all dependencies are already
    installed so building a new kokkos package will be fast.

4. When you're satisfied with what Spack plans to do, install it:

    ```console
    $ spack install kokkos+rocm+debug_bounds_check amdgpu_target==gfx90a
    [+] /appl/lumi/spack/22.08/0.19.0/opt/spack/pkgconf-1.8.0-vvar5d2
    [+] /usr (external openssl-1.1.0i-fips-kjnvpofxvream6prmex4f2t73xrrwuzu)
    [+] /appl/lumi/spack/22.08/0.19.0/opt/spack/libiconv-1.16-kcfqnf5
    [+] /appl/lumi/spack/22.08/0.19.0/opt/spack/libmd-1.0.4-hihpx7d
    [+] /appl/lumi/spack/22.08/0.19.0/opt/spack/xz-5.2.7-3x3rucn
    [+] /appl/lumi/spack/22.08/0.19.0/opt/spack/zlib-1.2.13-m4ntfr4
    ...
    ==> Installing kokkos-3.7.00-nioell5q7dkrdfu6meerk3o33gzlb7tj
    ==> kokkos exists in binary cache but with different hash
    ==> No binary for kokkos-3.7.00-nioell5q7dkrdfu6meerk3o33gzlb7tj found: installing from source
    ==> Fetching https://mirror.spack.io/_source-cache/archive/62/62e3f9f51c798998f6493ed36463f66e49723966286ef70a9dcba329b8443040.tar.gz
    ==> No patches needed for kokkos
    ==> kokkos: Executing phase: 'cmake'
    ==> kokkos: Executing phase: 'build'
    ==> kokkos: Executing phase: 'install'
    ==> kokkos: Successfully installed kokkos-3.7.00-nioell5q7dkrdfu6meerk3o33gzlb7tj
      Stage: 0.59s.  Cmake: 33.02s.  Build: 28.89s.  Install: 7.03s.  Total: 1m 18.93s
    [+] /project/project_465000XYZ/spack/22.08/0.19.0/kokkos-3.7.00-nioell5
    ```

    The final line shows where the software is installed on disk. A module will
    also be generated automatically and added to your `$MODULEPATH`. The
    modules are generated with a short hash code (5 characters "nioel" here) to
    prevent naming collisions.

    ```bash
    $ module load kokkos/3.7.00-gcc-nioel
    ```

## What to do when a Spack install fails

1. **Check if the error displayed suggests an easy solution.** If there is an
   error, Spack will point you to an installation log for the particular
   package. In the same directory, the full build directory can also be found
   in `/tmp`. Inspecting the output logs from configure or cmake can sometimes
   be fruitful.

    Some failures can be avoided by:

    - building a different version of the packages
    - building with a different compiler (try `%gcc` or `%cce`)
    - disabling a variant of the package
    - modifying which dependencies are used to build the target package (see
      [Specs and
      Dependencies](https://spack.readthedocs.io/en/latest/basic_usage.html#specs-dependencies)
      in the official Spack documentation)

    In some cases, changes have to made to the `package.py` file of a package.
    Unfortunately, this is not straightforward as the package repository is
    located in `/appl/lumi`, which is read-only. In such cases, you have to
    clone to your own Spack instance and configure it using our configuration
    files.

2. **Seek help:** you can check the 
    [official Spack documentation](https://spack.readthedocs.io), open a ticket 
    at the [LUMI Helpdesk](https://www.lumi-supercomputer.eu/user-support/need-help/) 
    or ask the Spack community via the [Spack Slack](http://spackpm.slack.com), 
    the Spack Slack community can be very helpful.

## Description of the different Spack modules

* Module `spack/22.08`: This is Spack release version 0.18.1 based on the Cray Programming Environment 22.08. The ROCM packages are external and comes from the HPE provided ROCM 5.0.2 in `/opt/rocm` (which is rather old).
* Module `spack/22.08-2`: This is Spack release version 0.19.0 based on the Cray Programming Environment 22.08. The ROCM packages are built from source by Spack and corresponds to ROCM release version 5.2.3. Testing has indicated that it should be possible to run ROCM 5.2.3-based software using the older drivers from ROCM 5.1.3, which is installed on the LUMI-G compute nodes.


## Spack on LUMI (advanced)

This section further explains the Spack setup on LUMI.

The upstream Spack instances maintained by the [User Support Team][helpdesk]
are located in subdirectories of `/appl/lumi/spack`, numbered according to the
Cray Programming Environment release version, and Spack release version. For
example:

```text
/appl/lumi/spack/22.08/0.18.1/
```

This is a Spack instance based on Spack release 0.18.1 configured to use the
compilers and MPI libraries from the Cray Programming Environment release
22.08. In general, we will only install one version of Spack per programming
environment. **These instances are not meant to be used directly by users** as
they are configured to install packages centrally, i.e. `spack install` will
fail with permission errors, but you can copy the configuration files from
there if you want to make your own Spack instance.

Instead, for users, there are "fake" chained spack instances installed
alongside these which are configured with install packages in a user-specified
location. These instances have names which end in `-user`, e.g.:

```text
/appl/lumi/spack/22.08/0.18.1-user/
```

This is what the `spack` modules use, and what you should use as user on the
system. If you check the `etc/spack/config.yaml` inside, you will see that they
set the `install_tree` property to a value provided by `$SPACK_USER_PREFIX`.

```text
config:
    build_jobs: 32
    install_path_scheme: '{name}-{version}-{hash:7}'
    install_tree: $SPACK_USER_PREFIX/22.08/0.18.1
    source_cache: $SPACK_USER_PREFIX/22.08/0.18.1/cache
    misc_cache:   $SPACK_USER_PREFIX/22.08/0.18.1/cache
    install_missing_compilers: false
    suppress_gpg_warnings: true
```

A similar trick is used in the `modules.yaml` file to install modules in
`$SPACK_USER_PREFIX/22.08/0.18.1/modules/tcl`.

```text
modules:
  default:
    roots:
      tcl: $SPACK_USER_PREFIX/22.08/0.18.1/modules/tcl
    enable:
      - tcl
    tcl:
      hash_length: 5
      projections:
        all: '{name}/{version}-{compiler.name}'
```

If you want to use Spack directly (without `module load spack`), you can source
the Spack initialization scripts as usual. They can be found on disk in e.g.:

```bash
$ source /appl/lumi/spack/22.08/0.18.1-user/share/spack/setup-env.sh
```

You just need to make sure that `$SPACK_USER_PREFIX` is set and that the
`cache` and `modules/tcl` subdirectories exist within that directory.

## Further reading

- [Spack documentation](https://spack.readthedocs.io/en/latest/index.html)
- [Spack tutorial](https://spack.readthedocs.io/en/latest/tutorial.html)
- [Spack Source code](https://github.com/spack/spack) (especially the package definitions)
- [Spack Slack](https://slack.spack.io/)
