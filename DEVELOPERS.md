# Developers Guide

This guide steps through the process to develop the ACCESS-OM2 model using [spack](spack.io), the build from source package manager that is used to build and deploy the model.  Spack automatically downloads package sources to a cache, compiles and installs them.  However spack also supports model development using a modified environment where the components that are being actively developed are compiled in a local source directory, and the rest built in the normal spack manner.

This guide is to assist developers who wish to modify one or more of the model components of ACCESS-OM2 and compile the modified code. Typically there is a development cycle, where the code is modified, compiled, tested and further modified based on testing.

It is assumed the modifications and compilation will be done on the [NCI HPC system `gadi`](https://opus.nci.org.au/display/Help/Gadi+User+Guide).

Much of this is adapted from the [spack Developer Workflows Tutorial](https://spack-tutorial.readthedocs.io/en/latest/tutorial_developer_workflows.html).

## Initial set-up 

It is necessary to make local copies of the following repositories using `git clone`: 
- [`spack`](https://github.com/ACCESS-NRI/spack.git) - the build from source package manager
- [`spack-config`](https://github.com/ACCESS-NRI/spack-config.git) - a spack configuration for `gadi`
- [`spack-packages`](https://github.com/ACCESS-NRI/spack-packages.git) - ACCESS-NRI package repository which defines a number of the packages used by ACCESS-OM2

```bash
git clone -c feature.manyFiles=true https://github.com/ACCESS-NRI/spack.git --branch releases/v0.21 --single-branch --depth=1
git clone https://github.com/ACCESS-NRI/spack-packages.git --branch main
git clone https://github.com/ACCESS-NRI/spack-config.git --branch main
```

Then link all the `spack-config` settings to your local spack instance:
```bash
ln -s -r -v spack-config/v0.21/gadi/* spack/etc/spack/
```

> [!NOTE]
>
> The steps in this section need only be done once, unless the cloned repositories need to be updated to a newer version.
>
> This guide is adapted from [instructions on the ACCESS-Hive Forum](https://forum.access-hive.org.au/t/how-to-build-access-om2-on-gadi/1545)
>
> ACCESS-NRI maintains a fork of spack to enable back-porting fixes from more recent spack versions. This fork is the one used in these instructions.

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

## Example

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
$ spack develop mom5@git.2023.11.09
==> Configuring spec mom5@git.2023.11.09=2023.11.09 for development at path mom5
```

2. Concretize the updated environment. This will update what spack thinks is required to build the defined specs, in this case `access-om2` and `mom5`. Note that the `mom5` spec has a `dev_path` argument that points to the location of the sources it will use to build the package.

```bash
$ spack concretize -f
==> Concretized access-om2@git.2024.03.0=2024.03.0
 -   s6gpbn7  access-om2@git.2024.03.0=2024.03.0%intel@19.0.5.281~deterministic build_system=bundle arch=linux-rocky8-x86_64
 -   lx7r646      ^cice5@git.2023.10.19=2023.10.19%intel@19.0.5.281~deterministic~optimisation_report build_system=makefile arch=linux-rocky8-x86_64
 -   66vc6b7          ^datetime-fortran@1.7.0%intel@19.0.5.281 build_system=autotools patches=80b9577 arch=linux-rocky8-x86_64
 -   v2qgeyg          ^gmake@4.4.1%intel@19.0.5.281~guile build_system=generic arch=linux-rocky8-x86_64
 -   lxcoiut          ^netcdf-c@4.7.4%intel@19.0.5.281~blosc~byterange~dap~fsync~hdf4~jna+mpi~nczarr_zip+optimize~parallel-netcdf+pic+shared~szip~zstd build_system=autotools arch=linux-rocky8-x86_64
 -   nq76l75              ^hdf5@1.14.3%intel@19.0.5.281~cxx~fortran+hl~ipo~java~map+mpi+shared~szip~threadsafe+tools api=default build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
 -   lwraula                  ^zlib-ng@2.1.4%intel@19.0.5.281+compat+opt build_system=autotools arch=linux-rocky8-x86_64
 -   2wcjlli          ^netcdf-fortran@4.5.2%intel@19.0.5.281~doc+pic+shared build_system=autotools patches=b050dbd arch=linux-rocky8-x86_64
 -   ywm4cjc          ^oasis3-mct@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report build_system=makefile arch=linux-rocky8-x86_64
[e]  mxcytar          ^openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~internal-pmix~java~legacylaunchers~lustre~memchecker~openshmem~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64
 -   mhmz5m4          ^parallelio@2.5.2%intel@19.0.5.281+fortran~ipo~logging+mpi~pnetcdf~shared~timing build_system=cmake build_type=Release generator=make patches=55a6d7a arch=linux-rocky8-x86_64
 -   zo4a6ll      ^libaccessom2@git.2023.10.26=2023.10.26%intel@19.0.5.281~deterministic~ipo~optimisation_report build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
[e]  nemw52e          ^cmake@3.24.2%intel@19.0.5.281~doc+ncurses+ownlibs build_system=generic build_type=Release arch=linux-rocky8-x86_64
 -   hevplm4          ^json-fortran@8.3.0%intel@19.0.5.281~ipo build_system=cmake build_type=Release generator=make arch=linux-rocky8-x86_64
 -   tm433j6          ^pkgconf@1.9.5%intel@19.0.5.281 build_system=autotools arch=linux-rocky8-x86_64
 -   uwi337q      ^mom5@git.2023.11.09=2023.11.09%intel@19.0.5.281~deterministic~optimisation_report+restart_repro build_system=makefile dev_path=/scratch/.../envs/mom5-dev/mom5 type=ACCESS-CM arch=linux-rocky8-x86_64
```

3. Test compilation works without changes. Before modifications it is a good idea to test the component will build as-is:

```bash
$ spack install
==> Installing gmake-4.4.1-v2qgeygrsw6xckypqqagh2pc4ib3chw6 [1/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/gmake-4.4.1-v2qgeygrsw6xckypqqagh2pc4ib3chw6
[+] /apps/cmake/3.24.2 (external cmake-3.24.2-nemw52evn6pvohthm7hxmndyxlfm2637)
==> openmpi@4.0.2 : has external module in ['openmpi/4.0.2']
[+] /apps/openmpi/4.0.2 (external openmpi-4.0.2-mxcytarrq3lnlmgidspiqgftznsoqnoq)
==> Installing zlib-ng-2.1.4-lwraula2ptlbxda76lu3yt3rdupgcukm [4/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/zlib-ng-2.1.4-lwraula2ptlbxda76lu3yt3rdupgcukm 
==> Installing datetime-fortran-1.7.0-66vc6b7gy6bldpgyqf5oasnkwb6e3scw [5/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/datetime-fortran-1.7.0-66vc6b7gy6bldpgyqf5oasnkwb6e3scw   
==> Installing pkgconf-1.9.5-tm433j6fyaqdfcgwpdykshqseqglmnrn [6/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/pkgconf-1.9.5-tm433j6fyaqdfcgwpdykshqseqglmnrn
==> Installing json-fortran-8.3.0-hevplm4n45oxuosrx3ldrgecd57b4vd4 [7/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/json-fortran-8.3.0-hevplm4n45oxuosrx3ldrgecd57b4vd4
==> Installing hdf5-1.14.3-nq76l75erusuueallosn3lzuuav5omrf [8/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/hdf5-1.14.3-nq76l75erusuueallosn3lzuuav5omrf
==> Installing netcdf-c-4.7.4-lxcoiutuqt6ghbo3pitqrazqbubmg4go [9/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-c-4.7.4-lxcoiutuqt6ghbo3pitqrazqbubmg4go
==> Installing netcdf-fortran-4.5.2-2wcjlli6necmfrsztdsl3s57bkafnbiz [10/16]                                                                  
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/netcdf-fortran-4.5.2-2wcjlli6necmfrsztdsl3s57bkafnbiz
==> Installing oasis3-mct-git.2023.11.09=2023.11.09-ywm4cjcl3b7zvnisyi24jsjzgtdcfz4a [11/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/oasis3-mct-git.2023.11.09_2023.11.09-ywm4cjcl3b7zvnisyi24jsjzgtdcfz4a
==> Installing parallelio-2.5.2-mhmz5m4k3fs6hs6bsi22nemdtz5yba4x [12/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/parallelio-2.5.2-mhmz5m4k3fs6hs6bsi22nemdtz5yba4x
==> Installing libaccessom2-git.2023.10.26=2023.10.26-zo4a6ll4cr3aoh3yxeicm4nocm6gzmlj [13/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/libaccessom2-git.2023.10.26_2023.10.26-zo4a6ll4cr3aoh3yxeicm4nocm6gzmlj
==> Installing mom5-git.2023.11.09=2023.11.09-uwi337qmi26zojqvcaag2yi3ukml6sxc [14/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/mom5-git.2023.11.09_2023.11.09-uwi337qmi26zojqvcaag2yi3ukml6sxc
==> Installing cice5-git.2023.10.19=2023.10.19-lx7r646nefy37hxpiv5ct6skrxm3l5sa [15/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/cice5-git.2023.10.19_2023.10.19-lx7r646nefy37hxpiv5ct6skrxm3l5sa
==> Installing access-om2-git.2024.03.0=2024.03.0-s6gpbn7eottnw7wk424rj362ccye3o37 [16/16]
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64/intel-19.0.5.281/access-om2-git.2024.03.0_2024.03.0-s6gpbn7eottnw7wk424rj362ccye3o37
==> Updating view at /scratch/.../tmp/dev-docs/envs/mom5-dev/.spack-env/view
==> Warning: Skipping external package: openmpi@4.0.2%intel@19.0.5.281~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~internal-pmix~java~legacylaunchers~lustre~memchecker~openshmem~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none patches=073477a,60ce20b schedulers=none arch=linux-rocky8-x86_64/mxcytar
```
The output has been truncated for brevity, but spack will attempt to install 16 packages on which the build depends. This can take a while the first time it is run (of the order of 30 minutes), but subsequent runs will re-use the dependencies and will not have to be re-built every time.

4. Modify sources as required. For illustrative purposes a `use` statement has been commented out to force an error in compilation:

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
spack install
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/datetime-fortran-1.7.0-oy47sjopcbyjxejfinwe3ept564idest
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/json-fortran-8.3.0-dlmgtabad5koosoqhjfcs7323itzzrxx
==> openmpi@4.0.2 : has external module in ['openmpi/4.0.2']
[+] /apps/openmpi/4.0.2 (external openmpi-4.0.2-skdxaycsjbzb33gqqp252d7vu6zuai5e)
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/pkgconf-1.9.5-3f2l3qwlgatzrt6kdcigbdiadyydy6ld
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/zlib-ng-2.1.4-jugwcxhg3jwqkp4rzchowtra6zlmoxn3
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/gmake-4.4.1-ggoluv3v37ruoyyjo4idshs6lcfcnpyb
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/hdf5-1.14.3-a7oy4ouzbbrsyajhbwyre57iokthvdg5
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/netcdf-c-4.7.4-7mvqb2xq7nd7vxmw3d27peuoshrviu4y
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/netcdf-fortran-4.5.2-gwkizyb62yqltwm2doeui4dqdtniauje
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/parallelio-2.5.2-mv2hsjyyxrxefozyl2zm353s4g4yn4bt      
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/oasis3-mct-git.2023.11.09_2023.11.09-54ylj75ghxllo7feyxy
o2cpa23zbx7lf                     
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/libaccessom2-git.2023.10.26_2023.10.26-44vqfykpu37f4payz
6xsljnov7s3az6l
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/cice5-git.2023.10.19_2023.10.19-3hxisc2ywncbzlz5a3yg5f5v
5tahn5ta                                                                                                                                     
==> Installing mom5-git.2023.11.09=2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm [14/15]
==> No binary for mom5-git.2023.11.09=2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm found: installing from source                              
==> No patches needed for mom5                                                                                                               
==> mom5: Executing phase: 'edit'                                                                                                            
==> mom5: Executing phase: 'build'                                                                                                           
==> Error: ProcessError: Command exited with status 1:                                                                                       
    './MOM_compile.csh' '--no_version' '--type' 'ACCESS-OM' '--platform' 'spack' '--no_environ'                                              
                                                                                                                                             
2 errors found in build log:                                                                                                                 
     177    make: Nothing to be done for 'all'.                                                                                                    178    ..................................................................................................................................
...................................                                                                                                                       . Makefile is ready.
     179    make: Nothing to be done for 'all'.                                                                                              
     180    ..... Makefile is ready.                                                                                                         
     181    mpifort -Duse_netCDF -Duse_libMPI -DACCESS_OM  -fpp -Wp,-w -I/scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/inte
l-19.0.5.281/oasis3-mct-git.2023.11
            .09_2023.11.09-54ylj75ghxllo7feyxyo2cpa23zbx7lf/include/psmile.MPI1 -I/scratch/.../tmp/dev-docs/release/linux-rocky8-x86_6
4_v4/intel-19.0.5.281/oasis3-mct-gi 
            t.2023.11.09_2023.11.09-54ylj75ghxllo7feyxyo2cpa23zbx7lf/include -I/scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v
4/intel-19.0.5.281/libaccessom2-git                                                                                                          
            .2023.10.26_2023.10.26-44vqfykpu37f4payz6xsljnov7s3az6l/include -I/scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/netcdf-fortran-4.
            5.2-gwkizyb62yqltwm2doeui4dqdtniauje/include -fno-alias -safe-cray-ptr -fpe0 -ftz -assume byterecl -i4 -r8 -nowarn -check noarg_temp_created -assume nobuffered_io -c
            onvert big_endian -grecord-gcc-switches -align all -fp-model precise -fp-model source -align all -g3 -O2 -xCORE-AVX2 -debug all -check none -traceback -I/scratch/.../tmp/dev-docs/envs/mom5-dev/mom5/exec/spack/lib_FMS -I/scratch/.../tmp/dev-docs/envs/mom5-dev/mom5/exec/spack/ACCE
SS-OM/lib_ocean -c        /scratch/tm70/ap
            h502/tmp/dev-docs/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90
     182    ifort: command line warning #10121: overriding '-march=skylake-avx512' with '-xCORE-AVX2'
  >> 183    /scratch/.../tmp/dev-docs/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90(365): error #6404: This name does not have a type, and must have an explicit
             type.   [SECONDS_PER_HOUR]
     184        sfix_seconds = sfix_hours * SECONDS_PER_HOUR
     185    --------------------------------^ 
     186    compilation aborted for /scratch/.../tmp/dev-docs/envs/mom5-dev/mom5/src/accessom_coupler/ocean_solo.F90 (code 1)
  >> 187    make: *** [Makefile:20: ocean_solo.o] Error 1
     188    Make failed to create the ACCESS-OM executable
     189    ==> mom5: Executing phase: 'install'
     190    ==> [2024-05-31-00:37:24.137440] Installing exec/spack/ACCESS-OM/fms_ACCESS-OM.x to /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281
            /mom5-git.2023.11.09_2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm/bin
     191    ==> [2024-05-31-00:37:24.210207] Installing bin/mppnccombine.spack to /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/mom5-git.2023
            .11.09_2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm/bin

See build log for details:
  /scratch/.../tmp/dev-docs/envs/mom5-dev/mom5/spack-build-out.txt

==> Warning: Skipping build of access-om2-git.2024.03.0=2024.03.0-xtrvyahghgfmonn732mwhrl6nffuh2cq since mom5-git.2023.11.09=2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm failed
==> Error: access-om2-git.2024.03.0=2024.03.0-xtrvyahghgfmonn732mwhrl6nffuh2cq: Package was not installed
==> Warning: Module file /scratch/.../tmp/dev-docs/release/modules/linux-rocky8-x86_64_v4/access-om2/2024.03.0 exists and will not be overwritten
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

Reverting that change and `spack install` works as expected:
```bash
$ spack install

[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/datetime-fortran-1.7.0-oy47sjopcbyjxejfinwe3ept564idest
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/json-fortran-8.3.0-dlmgtabad5koosoqhjfcs7323itzzrxx
==> openmpi@4.0.2 : has external module in ['openmpi/4.0.2']
[+] /apps/openmpi/4.0.2 (external openmpi-4.0.2-skdxaycsjbzb33gqqp252d7vu6zuai5e)
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/pkgconf-1.9.5-3f2l3qwlgatzrt6kdcigbdiadyydy6ld
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/zlib-ng-2.1.4-jugwcxhg3jwqkp4rzchowtra6zlmoxn3
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/gmake-4.4.1-ggoluv3v37ruoyyjo4idshs6lcfcnpyb
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/hdf5-1.14.3-a7oy4ouzbbrsyajhbwyre57iokthvdg5
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/netcdf-c-4.7.4-7mvqb2xq7nd7vxmw3d27peuoshrviu4y
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/netcdf-fortran-4.5.2-gwkizyb62yqltwm2doeui4dqdtniauje
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/parallelio-2.5.2-mv2hsjyyxrxefozyl2zm353s4g4yn4bt
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/oasis3-mct-git.2023.11.09_2023.11.09-54ylj75ghxllo7feyxy
o2cpa23zbx7lf
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/libaccessom2-git.2023.10.26_2023.10.26-44vqfykpu37f4payz
6xsljnov7s3az6l
==> Installing mom5-git.2023.11.09=2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm [13/15]
==> No binary for mom5-git.2023.11.09=2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm found: installing from source
==> No patches needed for mom5
==> mom5: Executing phase: 'edit'
==> mom5: Executing phase: 'build'
==> mom5: Executing phase: 'install'
==> mom5: Successfully installed mom5-git.2023.11.09=2023.11.09-sn3ycjygtex7ppdyh7t5ihasyvlr3ofm
  Stage: 0.00s.  Edit: 0.06s.  Build: 9.21s.  Install: 0.17s.  Post-install: 0.24s.  Total: 10.72s
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/mom5-git.2023.11.09_2023.11.09-sn3ycjygtex7ppdyh7t5ihasy
vlr3ofm
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/cice5-git.2023.10.19_2023.10.19-3hxisc2ywncbzlz5a3yg5f5v
5tahn5ta
==> Installing access-om2-git.2024.03.0=2024.03.0-xtrvyahghgfmonn732mwhrl6nffuh2cq [15/15]
==> No binary for access-om2-git.2024.03.0=2024.03.0-xtrvyahghgfmonn732mwhrl6nffuh2cq found: installing from source
==> No patches needed for access-om2
==> access-om2: Executing phase: 'install'
==> access-om2: Successfully installed access-om2-git.2024.03.0=2024.03.0-xtrvyahghgfmonn732mwhrl6nffuh2cq
  Stage: 0.00s.  Install: 0.00s.  Post-install: 0.12s.  Total: 0.59s
[+] /scratch/.../tmp/dev-docs/release/linux-rocky8-x86_64_v4/intel-19.0.5.281/access-om2-git.2024.03.0_2024.03.0-xtrvyahghgfmonn732mwh
rl6nffuh2cq
==> Warning: Module file /scratch/.../tmp/dev-docs/release/modules/linux-rocky8-x86_64_v4/access-om2/2024.03.0 exists and will not be
overwritten
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