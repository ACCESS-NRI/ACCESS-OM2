# Developers Guide

This guide steps through the process to develop the ACCESS-OM2 model using [`spack`](spack.io), the build from source package manager that is used to build and deploy the model.  Spack generally automatically downloads package sources to a cache, compiles and installs them.  Spack also supports model development using a modified environment where the components that are being actively developed are compiled in a local source directory, and the rest built in the normal spack manner.

This guide is to assist developers who wish to modify one or more of the model components of ACCESS-OM2 and compile the modified code. Typically there is a development cycle, where the code is modified, compiled, tested and further modified based on testing.

It is assumed the modifications and compilation will be done on the [NCI HPC system `gadi`](https://opus.nci.org.au/display/Help/Gadi+User+Guide).

Much of this is adapted from the [spack Developer Workflows Tutorial](https://spack-tutorial.readthedocs.io/en/latest/tutorial_developer_workflows.html).

## Initial set-up 

It is necessary to make local copies of the following repositories using `git clone`: 
- [`spack`](https://github.com/ACCESS-NRI/spack.git) - the build from source package manager
- [`spack-config`](https://github.com/ACCESS-NRI/spack-config.git) - a `spack` configuration for `gadi`]
- [`spack-packages`](https://github.com/ACCESS-NRI/spack-packages.git) - ACCESS-NRI package repository which defines a number of the packages used by ACCESS-OM2

```bash
git clone -c feature.manyFiles=true https://github.com/ACCESS-NRI/spack.git --branch releases/v0.21 --single-branch --depth=1
git clone https://github.com/ACCESS-NRI/spack-packages.git --branch main
git clone https://github.com/ACCESS-NRI/spack-config.git --branch main
```

Then link all the `spack-config` settings to your local `spack` instance:
```bash
ln -s -r -v spack-config/v0.21/gadi/* spack/etc/spack/
```

> [!NOTE]
>
> The steps in this section need only be done once, unless the cloned repositories need to be updated to a newer version.
>
> This guide is adapted from [instructions on the ACCESS-Hive Forum](https://forum.access-hive.org.au/t/how-to-build-access-om2-on-gadi/1545)
>
> `spack` is cloned from the ACCESS-NRI fork as on occasion there are some fixes that are back-ported by ACCESS-NRI from more recent versions of `spack`.

## Enable spack

For every new login or new shell it is necessary to add your local copy of `spack` to your shell, as well as some other settings. 
```bash
. spack-config/spack-enable.bash
```
> [!NOTE]
> There is a space between the `.` and the path, as we are not treating `spack-enable.bash` as an executable

## Development environment

Spack [has environments](https://spack.readthedocs.io/en/latest/environments.html), which is similar to many other related implementations such as conda environments and python virtual environments.

Environments create an isolated operating environment within which `spack` can only see and access packages that are added to the environment. In this repository, the model to be built is defined using the `spack.yaml` environment file.

### Source code

There are two options for local source directory location: 

1. Use a path to an existing source code repository
2. Let spack clone the code for you and place it in the environment directory

If option 1 is preferred the source code for the component(s) to be modified has to be available on the filesystem, e.g. using `git clone`, before the next steps. For more details, see the [Notes](#notes) section below.

> [!Note]
> The choice of where the source code for an environment should reside depends on use case and personal prefrence. If an existing source code repository location is used in more than one environment it requires the code repository be kept in sync with the purpose of the development environment, e.g. two separate feature branches utilising two different development environments would require the developer to check out the correct branch when switching between development environments. It would also preclude building both environments simultaneously.

### Creating an environment

These instructions use [independent environments](https://spack-tutorial.readthedocs.io/en/latest/tutorial_environments.html#creating-an-independent-environment) rather than managed environments. The [main difference](https://spack-tutorial.readthedocs.io/en/latest/tutorial_environments.html#managed-versus-independent-environments) is that managed environments live within the spack directory structure, independent environments are outside it, and must be referenced directly by their path. 

First step is choose a location for your development environment directory: it will only contain text files and possibly source code repositories. All compiled packages will be put in the directories defined in `spack-config` (defined with [`install_tree`](https://spack.readthedocs.io/en/latest/config_yaml.html#install-tree-root) in [`config.yaml`](https://github.com/ACCESS-NRI/spack-config/blob/main/common/config.yaml#L4)).

Choose a name that suits the aim of the work. Make a directory with that name, `cd` into that directory then type the command `spack env create -d .`. This creates a `spack` independent environment in the current directory.

### Modifying an environment

Once the environment is created type `spack env activate .` from within the environment directory to activate the environment. 

Add packages that are required, and use `spack develop` to indicate which packages will be modified and recompiled. 

By default `spack` will clone the source code automatically, but it is possible to use an existing source code repository directory if that is preferred.

It is necessary after `spack develop` to call `spack concretize -f` to force spack to update the concretization so that it picks up the changes to the packages that are being developed.

## Compiling Modified Code

Now the source code can be modified, and then compiled by invoking `spack install`

## Examples

The instructions above are best understood by example.

### Developing a single model component

In this example the `mom5` component will be modified but all other components and dependencies remain the same. 

This example also uses the [`ACCESS-OM2` `spack.yaml` environment file](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/main/spack.yaml) as an argument to the [`spack env create` command](https://spack.readthedocs.io/en/latest/command_index.html#spack-env-create). This makes the development environment the same as that for the released ACCESS-OM2 model that is referenced in the local copy of the repo. So it is necessary to clone [this repository](https://github.com/ACCESS-NRI/ACCESS-OM2/).

1. Create an environment with a name that reflects the purpose, activate, and set the `mom5` package to be a development environment: 
```bash
$ git clone https://github.com/ACCESS-NRI/ACCESS-OM2.git
$ mkdir mom5-dev
$ cd mom5-dev
$ spack env create -d . ../ACCESS-OM2/spack.yaml
==> Created environment in /g/data/.../envs/mom5-dev
==> You can activate this environment with:
==>   spack env activate /g/data/.../envs/mom5-dev
$ spack env activate .
$ spack env status
==> Using spack.yaml in current directory: /g/data/.../envs/mom5-dev
$ spack develop mom5@git.master
==> Configuring spec mom5@git.master=2023.11.09 for development at path mom5
$ spack add mom5@git.2023.11.09
==> Adding mom5@git.2023.11.09=2023.11.09 to environment /g/data/.../dev_instructions/envs/mom5-dev
```

2. Concretize the updated environment. This will update what spack thinks is required to build the defined specs, in this case `access-om2` and `mom5`. Note that the `mom5` spec has a `dev_path` argument that points to the location of the sources it will use to build the package.

```bash
$ spack concretize -f
==> Concretized access-om2@git.2024.03.0=2024.03.0
 -   5nk4sjp  access-om2@git.2024.03.0=2024.03.0%intel@19.0.5.281~deterministic build_system=bundle arch=linux-rocky8-x86_64
[^]  v3zncpq      ^cice5@git.2023.10.19=2023.10.19%intel@19.0.5.281~deterministic~optimisation_report build_system=makefile arch=linux-rocky8-x86_64
[^]  aretozi          ^datetime-fortran@1.7.0%intel@19.0.5.281 build_system=autotools patches=80b9577 arch=linux-rocky8-x86_64
[^]  i3inxza          ^netcdf-c@4.7.4%intel@19.0.5.281~blosc~byterange~dap~fsync~hdf4~jna+mpi~nczarr_zip+optimize~parallel-netcdf+pic+shared~szip~zstd build_system=autotools arch=linux-rocky8-x86_64
[^]  a6mpuk7              ^hdf5@1.14.1-2%intel@19.0.5.281~cxx~fortran+hl~ipo~java~map+mpi+shared~szip~threadsafe+tools api=default build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[^]  arj4bfz                  ^zlib@1.2.13%intel@19.0.5.281+optimize+pic+shared build_system=makefile arch=linux-rocky8-x86_64
[^]  mnx4ggh          ^netcdf-fortran@4.5.2%intel@19.0.5.281~doc+pic+shared build_system=autotools patches=b050dbd arch=linux-rocky8-x86_64
[^]  sh2anmt          ^oasis3-mct@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report build_system=makefile arch=linux-rocky8-x86_64
[e]  4jwtg3n          ^openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~java~legacylaunchers~lustre~memchecker~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64
[^]  nwolfzy          ^parallelio@2.5.2%intel@19.0.5.281+fortran~ipo~logging+mpi~pnetcdf~shared~timing build_system=cmake build_type=Release generator=make patches=55a6d7a arch=linux-rocky8-x86_64
[^]  ltfg7jc      ^libaccessom2@git.2023.10.26=2023.10.26%intel@19.0.5.281~deterministic~ipo~optimisation_report build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[e]  y2zohex          ^cmake@3.24.2%intel@19.0.5.281~doc+ncurses+ownlibs~qt build_system=generic build_type=Release arch=linux-rocky8-x86_64
[^]  ugorlm6          ^gmake@4.4.1%intel@19.0.5.281~guile build_system=autotools arch=linux-rocky8-x86_64
[^]  nyxvikk          ^json-fortran@8.3.0%intel@19.0.5.281~ipo build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[^]  uyq7tvp          ^pkgconf@1.9.5%intel@19.0.5.281 build_system=autotools arch=linux-rocky8-x86_64
 -   lisulnp      ^mom5@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report+restart_repro build_system=makefile dev_path=/g/data/.../dev_instructions/envs/mom5-dev/mom5 type=ACCESS-OM arch=linux-rocky8-x86_64

==> Concretized mom5@git.2023.11.09=2023.11.09
 -   lisulnp  mom5@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report+restart_repro build_system=makefile dev_path=/g/data/.../dev_instructions/envs/mom5-dev/mom5 type=ACCESS-OM arch=linux-rocky8-x86_64
[^]  aretozi      ^datetime-fortran@1.7.0%intel@19.0.5.281 build_system=autotools patches=80b9577 arch=linux-rocky8-x86_64
[^]  ltfg7jc      ^libaccessom2@git.2023.10.26=2023.10.26%intel@19.0.5.281~deterministic~ipo~optimisation_report build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[e]  y2zohex          ^cmake@3.24.2%intel@19.0.5.281~doc+ncurses+ownlibs~qt build_system=generic build_type=Release arch=linux-rocky8-x86_64
[^]  ugorlm6          ^gmake@4.4.1%intel@19.0.5.281~guile build_system=autotools arch=linux-rocky8-x86_64
[^]  nyxvikk          ^json-fortran@8.3.0%intel@19.0.5.281~ipo build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[^]  uyq7tvp          ^pkgconf@1.9.5%intel@19.0.5.281 build_system=autotools arch=linux-rocky8-x86_64
[^]  i3inxza      ^netcdf-c@4.7.4%intel@19.0.5.281~blosc~byterange~dap~fsync~hdf4~jna+mpi~nczarr_zip+optimize~parallel-netcdf+pic+shared~szip~zstd build_system=autotools arch=linux-rocky8-x86_64
[^]  a6mpuk7          ^hdf5@1.14.1-2%intel@19.0.5.281~cxx~fortran+hl~ipo~java~map+mpi+shared~szip~threadsafe+tools api=default build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[^]  arj4bfz              ^zlib@1.2.13%intel@19.0.5.281+optimize+pic+shared build_system=makefile arch=linux-rocky8-x86_64
[^]  mnx4ggh      ^netcdf-fortran@4.5.2%intel@19.0.5.281~doc+pic+shared build_system=autotools patches=b050dbd arch=linux-rocky8-x86_64
[^]  sh2anmt      ^oasis3-mct@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report build_system=makefile arch=linux-rocky8-x86_64
[e]  4jwtg3n      ^openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~java~legacylaunchers~lustre~memchecker~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64

==> Updating view at /g/data/.../dev_instructions/envs/mom5-dev/.spack-env/view
==> Warning: Skipping external package: openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~java~legacylaunchers~lustre~memchecker~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64/4jwtg3n
```

3. Test compilation works without changes. Before modifications it is a good idea to test the component will build as-is:

```bash
$ spack install
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/datetime-fortran-1.7.0-aretozixwsdz42owabgzeiuxooyxrspg
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/json-fortran-8.3.0-nyxvikkez6fu64xtbeqqz2a4wngikfgi
==> openmpi@4.0.2 : has external module in ['openmpi/4.0.2']
[+] /apps/openmpi/4.0.2 (external openmpi-4.0.2-4jwtg3nt7p3zsod4rvn3i3jyyop3arpe)
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/pkgconf-1.9.5-uyq7tvpnuuo3gosnplkvuxktaeeipuqk
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/zlib-1.2.13-arj4bfz33zf63gvfionhorrrveci6kb4
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/hdf5-1.14.1-2-a6mpuk7tw5e4pyukxwalooc7jfldalcf
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-c-4.7.4-i3inxzaihefr3rqljoovhyevwai6bsff
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-fortran-4.5.2-mnx4gghyb5gngsl3fgosud6k72yu2u6a
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/oasis3-mct-git.2023.11.09=2023.11.09-sh2anmtjrygzixya2fjc3p7cg4o5wc5h
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/parallelio-2.5.2-nwolfzy2xxb4wub65gmvlm3edhuolmlh
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/libaccessom2-git.2023.10.26=2023.10.26-ltfg7jcn6t4cefotvj3kjnyu5nru26xo
==> Installing mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk
==> No binary for mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk found: installing from source
==> No patches needed for mom5
==> mom5: Executing phase: 'edit'
==> mom5: Executing phase: 'build'
==> mom5: Executing phase: 'install'
==> mom5: Successfully installed mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk
  Stage: 0.00s.  Edit: 0.08s.  Build: 5m 26.48s.  Install: 0.16s.  Post-install: 0.47s.  Total: 5m 29.30s
[+] /g/data/.../dev_instructions/release/linux-rocky8-x86_64/intel-19.0.5.281/mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/cice5-git.2023.10.19=2023.10.19-v3zncpqjj2gyseudbwiudolcjq3k3leo
==> Installing access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62
==> No binary for access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62 found: installing from source
==> No patches needed for access-om2
==> access-om2: Executing phase: 'install'
==> access-om2: Successfully installed access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62
  Stage: 0.01s.  Install: 0.00s.  Post-install: 0.12s.  Total: 0.86s
[+] /g/data/.../dev_instructions/release/linux-rocky8-x86_64/intel-19.0.5.281/access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62
==> Updating view at /g/data/.../dev_instructions/envs/mom5-dev/.spack-env/view
==> Warning: Skipping external package: openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~java~legacylaunchers~lustre~memchecker~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64/4jwtg3n
```

4. Modify sources as required. For illustrative purposes a `module use` has been commented out to force an error in compilation:

```diff
$ git -C mom5 diff
diff --git a/src/accessom_coupler/ocean_solo.F90 b/src/accessom_coupler/ocean_solo.F90
index 838759c..b26dc2c 100644
--- a/src/accessom_coupler/ocean_solo.F90
+++ b/src/accessom_coupler/ocean_solo.F90
@@ -78,7 +78,7 @@ program main
 ! </NAMELIST>
 !
 
-  use constants_mod,            only: constants_init, SECONDS_PER_HOUR
+!  use constants_mod,            only: constants_init, SECONDS_PER_HOUR
   use data_override_mod,        only: data_override_init, data_override
   use diag_manager_mod,         only: diag_manager_init, diag_manager_end
   use field_manager_mod,        only: field_manager_init
```

5.  Rebuild modified `mom5` using `spack install`

```bash
$ spack install
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/datetime-fortran-1.7.0-aretozixwsdz42owabgzeiuxooyxrspg
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/json-fortran-8.3.0-nyxvikkez6fu64xtbeqqz2a4wngikfgi
==> openmpi@4.0.2 : has external module in ['openmpi/4.0.2']
[+] /apps/openmpi/4.0.2 (external openmpi-4.0.2-4jwtg3nt7p3zsod4rvn3i3jyyop3arpe)
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/pkgconf-1.9.5-uyq7tvpnuuo3gosnplkvuxktaeeipuqk
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/zlib-1.2.13-arj4bfz33zf63gvfionhorrrveci6kb4
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/hdf5-1.14.1-2-a6mpuk7tw5e4pyukxwalooc7jfldalcf
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-c-4.7.4-i3inxzaihefr3rqljoovhyevwai6bsff
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-fortran-4.5.2-mnx4gghyb5gngsl3fgosud6k72yu2u6a
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/parallelio-2.5.2-nwolfzy2xxb4wub65gmvlm3edhuolmlh
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/oasis3-mct-git.2023.11.09=2023.11.09-sh2anmtjrygzixya2fjc3p7cg4o5wc5h
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/libaccessom2-git.2023.10.26=2023.10.26-ltfg7jcn6t4cefotvj3kjnyu5nru26xo
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/cice5-git.2023.10.19=2023.10.19-v3zncpqjj2gyseudbwiudolcjq3k3leo
==> Installing mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk
==> No binary for mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk found: installing from source
==> No patches needed for mom5
==> mom5: Executing phase: 'edit'
==> mom5: Executing phase: 'build'
==> Error: ProcessError: Command exited with status 1:
    './MOM_compile.csh' '--type' 'ACCESS-OM' '--platform' 'spack' '--no_environ' '--no_version'

2 errors found in build log:
     155    make: Nothing to be done for 'all'.
     156    ...................................................................................................................................................................... Makefile is ready.
     157    make: Nothing to be done for 'all'.
     158    ..... Makefile is ready.
     159    mpifort -Duse_netCDF -Duse_libMPI -DACCESS_OM  -fpp -Wp,-w -I/g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/oasis3-mct-git.2023.11.09=2023.11.09-sh2anmtjrygzixya2fjc3p7c
            g4o5wc5h/include/psmile.MPI1 -I/g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/oasis3-mct-git.2023.11.09=2023.11.09-sh2anmtjrygzixya2fjc3p7cg4o5wc5h/include -I/g/data/vk8
            3/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/libaccessom2-git.2023.10.26=2023.10.26-ltfg7jcn6t4cefotvj3kjnyu5nru26xo/include -I/g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-fortran-4.5.2-mnx4gghyb5gngsl3fgosud6k72yu2u6a/include -fno-alias -safe-cray-ptr -fpe0 -ftz -assume byterecl -i4 -r8 -nowarn -check noarg_temp_created -assume nobuffered_io -convert big_endian -grecord-gcc-switches -align all -fp-model precise -fp-model source -align all -g3 -O2 -xCORE-AVX2 -debug all -check none -traceback -I/g/data/.../dev_instructio
            ns/envs/mom5-dev/mom5/exec/spack/lib_FMS -I/g/data/.../dev_instructions/envs/mom5-dev/mom5/exec/spack/ACCESS-OM/lib_ocean -c        /g/data/.../dev_instructions/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90
     160    ifort: command line warning #10121: overriding '-march=pentium4' with '-xCORE-AVX2'
  >> 161    /g/data/.../dev_instructions/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90(365): error #6404: This name does not have a type, and must have an explicit type.   [SECONDS_PER_HOUR]
     162        sfix_seconds = sfix_hours * SECONDS_PER_HOUR
     163    --------------------------------^
     164    compilation aborted for /g/data/.../dev_instructions/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90 (code 1)
  >> 165    make: *** [Makefile:20: ocean_solo.o] Error 1
     166    Make failed to create the ACCESS-OM executable
     167    ==> mom5: Executing phase: 'install'
     168    ==> [2024-04-29-15:33:46.059929] Installing exec/spack/ACCESS-OM/fms_ACCESS-OM.x to /g/data/.../dev_instructions/release/linux-rocky8-x86_64/intel-19.0.5.281/mom5-git.2023.11.09=2023.11.09
            -lisulnpbvmllenqb443lnuybhs3sopwk/bin
     169    ==> [2024-04-29-15:33:46.204205] Installing bin/mppnccombine.spack to /g/data/.../dev_instructions/release/linux-rocky8-x86_64/intel-19.0.5.281/mom5-git.2023.11.09=2023.11.09-lisulnpbvmlle
            nqb443lnuybhs3sopwk/bin

See build log for details:
  /g/data/.../dev_instructions/envs/mom5-dev/mom5/spack-build-out.txt

==> Warning: Skipping build of access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62 since mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk failed
==> Error: access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62: Package was not installed
==> Error: mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk: Package was not installed
==> Warning: Module file /g/data/.../dev_instructions/release/modules/linux-rocky8-x86_64/access-om2/2024.03.0 exists and will not be overwritten
==> Warning: Module file /g/data/.../dev_instructions/release/modules/linux-rocky8-x86_64/mom5/2023.11.09 exists and will not be overwritten
==> Error: Installation request failed.  Refer to reported errors for failing package(s).
```

6. Diagnose build error and rebuild

As expected the change resulted in an error and the build was not successful.

`spack` lets the user know where the build output and errors are viewable (`mom5/spack-build-out.txt`), but in this case the error is obvious:

```bash
  >> 161    /g/data/.../dev_instructions/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90(365): error #6404: This name does not have a type, and must have an explicit type.   [SECONDS_PER_HOUR]
     162        sfix_seconds = sfix_hours * SECONDS_PER_HOUR
     163    --------------------------------^
     164    compilation aborted for /g/data/.../dev_instructions/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90 (code 1)
```

As a result of the intentional error introduced above. 

Reverting that change and `spack install` worked as expected

```bash
$ git -C mom5 stash
Saved working directory and index state WIP on (no branch): baaf7ed Merge pull request #11 from ACCESS-NRI/mkmf-escape-fix
$ spack install
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/datetime-fortran-1.7.0-aretozixwsdz42owabgzeiuxooyxrspg
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/json-fortran-8.3.0-nyxvikkez6fu64xtbeqqz2a4wngikfgi
==> openmpi@4.0.2 : has external module in ['openmpi/4.0.2']
[+] /apps/openmpi/4.0.2 (external openmpi-4.0.2-4jwtg3nt7p3zsod4rvn3i3jyyop3arpe)
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/pkgconf-1.9.5-uyq7tvpnuuo3gosnplkvuxktaeeipuqk
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/zlib-1.2.13-arj4bfz33zf63gvfionhorrrveci6kb4
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/hdf5-1.14.1-2-a6mpuk7tw5e4pyukxwalooc7jfldalcf
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-c-4.7.4-i3inxzaihefr3rqljoovhyevwai6bsff
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-fortran-4.5.2-mnx4gghyb5gngsl3fgosud6k72yu2u6a
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/oasis3-mct-git.2023.11.09=2023.11.09-sh2anmtjrygzixya2fjc3p7cg4o5wc5h
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/parallelio-2.5.2-nwolfzy2xxb4wub65gmvlm3edhuolmlh
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/libaccessom2-git.2023.10.26=2023.10.26-ltfg7jcn6t4cefotvj3kjnyu5nru26xo
==> Installing mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk
==> No binary for mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk found: installing from source
==> No patches needed for mom5
==> mom5: Executing phase: 'edit'
==> mom5: Executing phase: 'build'
==> mom5: Executing phase: 'install'
==> mom5: Successfully installed mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk
  Stage: 0.00s.  Edit: 0.08s.  Build: 12.39s.  Install: 0.09s.  Post-install: 0.36s.  Total: 14.66s
[+] /g/data/.../dev_instructions/release/linux-rocky8-x86_64/intel-19.0.5.281/mom5-git.2023.11.09=2023.11.09-lisulnpbvmllenqb443lnuybhs3sopwk
[+] /g/data/vk83/apps/spack/0.20/release/linux-rocky8-x86_64/intel-19.0.5.281/cice5-git.2023.10.19=2023.10.19-v3zncpqjj2gyseudbwiudolcjq3k3leo
==> Installing access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62
==> No binary for access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62 found: installing from source
==> No patches needed for access-om2
==> access-om2: Executing phase: 'install'
==> access-om2: Successfully installed access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62
  Stage: 0.00s.  Install: 0.00s.  Post-install: 0.15s.  Total: 1.07s
[+] /g/data/.../dev_instructions/release/linux-rocky8-x86_64/intel-19.0.5.281/access-om2-git.2024.03.0=2024.03.0-5nk4sjpusp6cilzp3jkmubonty5q2w62
==> Warning: Module file /g/data/.../dev_instructions/release/modules/linux-rocky8-x86_64/access-om2/2024.03.0 exists and will not be overwritten
==> Warning: Module file /g/data/.../dev_instructions/release/modules/linux-rocky8-x86_64/mom5/2023.11.09 exists and will not be overwritten
```

This was a simple example to show the development workflow: `change code > test compilation > change code > test` compilation cycle.

#### Notes

By default `spack` clones the source code repository into the environment directory:
```
$ ls -gd mom5/
drwxr-s--- 11 tm70 4096 Apr 22 16:09 mom5/
```
it is possible to specify any [git reference](https://docs.github.com/en/rest/git/refs?apiVersion=2022-11-28) tag or branch 

If you have an existing source code repository clone you wish to use specify the path with the `-p` option:
```bash
$ spack develop -p ../../sources/MOM5/ mom5@git.master 
==> Configuring spec mom5@git.master=2023.11.09 for development at path ../../sources/MOM5/
```

### Developing more than one model component

To develop more than one component simultaneously use `spack develop` to mark it as a development component, and if it isn't already in the list of `specs` for the environment use `spack add` to add it.

```bash
$ spack env create -d . ../ACCESS-OM2/spack.yaml
==> Created environment in /g/data/tm70/aph502/dev_instructions/envs/mom5-cice5-dev
==> You can activate this environment with:
==>   spack env activate /g/data/tm70/aph502/dev_instructions/envs/mom5-cice5-dev
==> Warning: upstream not found: /g/data/vk83/apps/spack/0.21/release/.spack-db/index.json
$ spack env activate .
$ spack develop mom5@git.2023.11.09
==> Configuring spec mom5@git.2023.11.09=2023.11.09 for development at path mom5
$ spack develop cice5@git.2023.10.19
==> Configuring spec cice5@git.2023.10.19=2023.10.19 for development at path cice5
$ spack add mom5@git.2023.11.09
==> Adding mom5@git.2023.11.09=2023.11.09 to environment /g/data/tm70/aph502/dev_instructions/envs/mom5-cice5-dev
$ spack add cice5@git.2023.10.19
==> Adding cice5@git.2023.10.19=2023.10.19 to environment /g/data/tm70/aph502/dev_instructions/envs/mom5-cice5-dev
```

```bash
$ spack concretize -f
==> Warning: upstream not found: /g/data/vk83/apps/spack/0.21/release/.spack-db/index.json
==> Concretized access-om2@git.2024.03.0=2024.03.0
 -   2rd6qte  access-om2@git.2024.03.0=2024.03.0%intel@19.0.5.281~deterministic build_system=bundle arch=linux-rocky8-x86_64
 -   tnz4lu7      ^cice5@git.2023.10.19=2023.10.19%intel@19.0.5.281~deterministic~optimisation_report build_system=makefile dev_path=/g/data/tm70/aph502/dev_instructions/envs/mom5-cice5-dev/cice5 arch=linux-rocky8-x86_64
[^]  aretozi          ^datetime-fortran@1.7.0%intel@19.0.5.281 build_system=autotools patches=80b9577 arch=linux-rocky8-x86_64
[^]  ugorlm6          ^gmake@4.4.1%intel@19.0.5.281~guile build_system=autotools arch=linux-rocky8-x86_64
[^]  i3inxza          ^netcdf-c@4.7.4%intel@19.0.5.281~blosc~byterange~dap~fsync~hdf4~jna+mpi~nczarr_zip+optimize~parallel-netcdf+pic+shared~szip~zstd build_system=autotools arch=linux-rocky8-x86_64
[^]  a6mpuk7              ^hdf5@1.14.1-2%intel@19.0.5.281~cxx~fortran+hl~ipo~java~map+mpi+shared~szip~threadsafe+tools api=default build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[^]  arj4bfz                  ^zlib@1.2.13%intel@19.0.5.281+optimize+pic+shared build_system=makefile arch=linux-rocky8-x86_64
[^]  mnx4ggh          ^netcdf-fortran@4.5.2%intel@19.0.5.281~doc+pic+shared build_system=autotools patches=b050dbd arch=linux-rocky8-x86_64
[^]  sh2anmt          ^oasis3-mct@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report build_system=makefile arch=linux-rocky8-x86_64
[e]  4jwtg3n          ^openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~java~legacylaunchers~lustre~memchecker~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64
[^]  nwolfzy          ^parallelio@2.5.2%intel@19.0.5.281+fortran~ipo~logging+mpi~pnetcdf~shared~timing build_system=cmake build_type=Release generator=make patches=55a6d7a arch=linux-rocky8-x86_64
[^]  ltfg7jc      ^libaccessom2@git.2023.10.26=2023.10.26%intel@19.0.5.281~deterministic~ipo~optimisation_report build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[e]  y2zohex          ^cmake@3.24.2%intel@19.0.5.281~doc+ncurses+ownlibs~qt build_system=generic build_type=Release arch=linux-rocky8-x86_64
[^]  nyxvikk          ^json-fortran@8.3.0%intel@19.0.5.281~ipo build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[^]  uyq7tvp          ^pkgconf@1.9.5%intel@19.0.5.281 build_system=autotools arch=linux-rocky8-x86_64
 -   ig2eod2      ^mom5@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report+restart_repro build_system=makefile dev_path=/g/data/tm70/aph502/dev_instructions/envs/mom5-cice5-dev/mom5 type=ACCESS-OM arch=linux-rocky8-x86_64

==> Concretized mom5@git.2023.11.09=2023.11.09
# trimming concretization diagram for brevity...

==> Concretized cice5@git.2023.10.19=2023.10.19
# trimming concretization diagram for brevity...

==> Updating view at /g/data/tm70/aph502/dev_instructions/envs/mom5-cice5-dev/.spack-env/view
==> Warning: Skipping external package: openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~java~legacylaunchers~lustre~memchecker~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64/4jwtg3n
```

Now there are source subdirectories for `mom5` and `cice5` 

```bash
$ ls -lg
total 28
drwxr-s--- 20 tm70  4096 Apr 30 20:49 cice5
drwxr-s--- 11 tm70  4096 Apr 30 20:49 mom5
-rw-r-----  1 tm70 15477 Apr 30 20:50 spack.lock
-rw-r-----  1 tm70  1586 Apr 30 20:50 spack.yaml
```

Both can be compiled with a single call to `spack install`.
