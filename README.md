# ACCESS-OM2: ACCESS Ocean-Ice Model Release Configurations

## About the model

ACCESS-OM2 is a global coupled ocean - sea ice model developed by [COSIMA](http://www.cosima.org.au).

ACCESS-OM2 consists of the [MOM 5](https://github.com/ACCESS-NRI/MOM5) ocean model, [CICE 5](https://github.com/ACCESS-NRI/cice5) sea ice model, and a file-based atmosphere called [YATM](https://github.com/ACCESS-NRI/libaccessom2) coupled together using [OASIS3-MCT v2.0](https://github.com/ACCESS-NRI/oasis3-mct). ACCESS-OM2 builds on the ACCESS-OM ([Bi et al., 2013](http://www.bom.gov.au/jshess/docs/2013/bi2_hres.pdf)) and AusCOM ([Roberts et al., 2007](https://50years.acs.org.au/content/dam/acs/50-years/journals/jrpit/JRPIT39.2.137.pdf); [Bi and Marsland, 2010](https://www.cawcr.gov.au/technical-reports/CTR_027.pdf)) models originally developed at [CSIRO](http://www.csiro.au).

The model code, configurations and performance were described in [Kiss et al. (2020)](https://doi.org/10.5194/gmd-13-401-2020), with further details in the draft [ACCESS-OM2 technical report](https://github.com/COSIMA/ACCESS-OM2-1-025-010deg-report). The current code and configurations differ from this version in a number of ways (biogeochemistry, updated forcing, improvements and bug fixes), as described by [Solodoch et al. (2022)](https://doi.org/10.1029/2021GL097211), [Hayashida et al. (2023)](https://dx.doi.org/10.1029/2023JC019697), [Menviel et al. (2023)](https://doi.org/10.5194/egusphere-2023-390) and [Wang et al. (2023)](https://doi.org/10.5194/gmd-2023-123).

## Support

[ACCESS-NRI](https://www.access-nri.org.au) has assumed responsibility for supporting ACCESS-OM2 for the Australian Research Community. As part of this support ACCESS-NRI has developed a new build and deployment system for ACCESS-OM2 to align with plans for supporting a range of Earth System Models.

Any questions about ACCESS-NRI releases of ACCESS-OM2 should be done through the [ACCESS-Hive Forum](https://forum.access-hive.org.au/). See the [ACCESS Help and Support topic](https://forum.access-hive.org.au/t/access-help-and-support/908) for details on how to do this.

### Build

ACCESS-NRI is using [spack](https://spack.io), a build from source package manager designed for use with high performance computing. This repository contains a [spack environment](https://spack.readthedocs.io/en/latest/environments.html) definition file ([`spack.yaml`](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/main/spack.yaml)) that defines all the essential components of the ACCESS-OM2 model, including exact versions.

Spack automatically builds all the components and their dependencies, producing model component executables. Spack already contains support for compiling thousands of common software packages. Spack packages for the components in ACCESS-OM2 are defined in the [spack packages repository](https://github.com/ACCESS-NRI/spack_packages/).

ACCESS-OM2 is built and deployed automatically to `gadi` on NCI (see below). However it is possible to use spack to compile the model using the `spack.yaml` environment file in this repository. To do so follow the [instructions on the ACCESS Forum for configuring spack on `gadi`](https://forum.access-hive.org.au/t/how-to-build-access-om2-on-gadi/1545).

Then clone this repository and run the following commands on `gadi`:

```bash
spack env create access-om2 spack.yaml
spack env activate access-om2
spack install
```

to create a spack environment called `access-om2` and build all the ACCESS-OM2 components, the locations of which can be found using `spack find --paths`.

In contrast, the [COSIMA ACCESS-OM2 repository](https://github.com/COSIMA/access-om2) uses [submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) to bring all the code dependencies into a single repository and build all the models together.

### Deployment

ACCESS-OM2 is deployed automatically when a new version of the [`spack.yaml`](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/main/spack.yaml) file is committed to `main` or a dedicated `backport/VERSION` branch. All the ACCESS-OM2 components are built using `spack` on `gadi` and installed under the [`vk83`](https://my.nci.org.au/mancini/project/vk83) project in `/g/data/vk83`. It is necessary to be a member of [`vk83`](https://my.nci.org.au/mancini/project/vk83) project to use ACCESS-NRI deployments of ACCESS-OM2.

The deployment process also creates a GitHub release with the same tag. All releases are available under the [Releases page](https://github.com/ACCESS-NRI/ACCESS-OM2/releases). Each release has a changelog and meta-data with detailed information about the build and deployment, including:

- paths on `gadi` to all executables built in the deployment process (`spack.location`)
- a `spack.lock` file, which is a complete build provenance document, listing all the components that were built and their dependencies, versions, compiler version, build flags and build architecture
- the environment `spack.yaml` file used for deployment

Additionally the deployment creates environment modulefiles, the [standard method for deploying software on `gadi`](https://opus.nci.org.au/display/Help/Environment+Modules). To view available ACCESS-OM2 versions:

```bash
module use /g/data/vk83/modules  # module use /g/data/vk83/prerelease/modules for modules deployed in an open Pull Request
module avail access-om2
```

For users of ACCESS-OM2 model configurations released by ACCESS-NRI the exact location of the ACCESS-OM2 model executables is not required. Model configurations will be updated with new model components when necessary.

For information on contributing your own fixes to the ACCESS-OM2 `spack.yaml`, see the [CONTRIBUTING.md](./CONTRIBUTING.md) file.
