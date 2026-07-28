[---]: #
[License: Copyright © 2026 Durham University, SHAREing Project, MIT Licensed]: #
[Creator: Andrew Naden]: #
[Contributors: Andrew Naden; Ananya Gangopadhyay]: #
[Summary: Pre-assessment report template document]: #
[---]: #

# Pre-assessment of CASTEP for SHAREing Task 019

## Assessment objective

[> Is this a test submission, internal review or external review? If it is part of a larger project or funding, add that information here. You may also include your (the assessor's) name, optionally with contact details.]: #

This is a pre-assessment study of the performance of the planewave Density Functional Theory (DFT) code CASTEP, performed by Oscar van Vuren at Cardiff University, in collaboration with Phil Hasnip of the University of York and Martyn Guest of ARCCA. CASTEP allows for the accurate simulation of the electronic structure of condensed matter, to scales of hundreds to thousands of atoms. The CASTEP code has seen historical use for benchmarking flagship HPC facilities in the UK, including HECToR, ARCHER and ARCHER2, and sees contemporary use in condensed matter simulations across the globe.

The associated SHAREing work package for this pre assessment of CASTEP is WP 019. The objective of this assessment is to provide SHAREing with a preliminary set of performance data, benchmarks and compilation instructions to allow for a more detailed, rigorous benchmark of CASTEP in future by SHAREing.

CASTEP version 26 was benchmarked using CPU only and GPU accelerated workloads on three different HPC architectures


## Disclaimers

1. This report is not a commentary on code quality, but an indicator of the quality of the current SHAREing testing methodology as of `31/07/2026`.

[2. The pre-assessment is only a preliminary assessment of submission suitability and does not guarantee a full assessment. It will be provided to the submitter indicating if the full assessment will be undertaken or detail reasons for rejection.]: #

## Table of contents

[> Place `x` inside the box when complete to mark the checkbox.]: #

- [1: Benchmark setup](#1-benchmark-setup)
- [2: Description of working environment](#2-description-of-working-environment)
- [3: Compiler setup and optimisations](#3-compiler-setup-and-optimisations)
- [4: Computational complexity and scaling](#4-computational-complexity-and-scaling)
- [5: Memory, storage and I/O](#5-memory-storage-and-io)
- [6: Additional comments from submitter](#6-additional-comments-from-submitter)

## 1: Benchmark setup

Two regimes were explored in the benchmarking procedure; a metal system with a dense **k**-point mesh (Fe cell, 16 atoms) and a molecular system (periodic water box, 20 x 20 x 20 &angst; ) sampled at the gamma point only. This allows for the comparison of parallelisation in CASTEP over **k** points and over **G** vectors, and the acceleration possible in these two extremes of condensed matter simulation.

<center>
<figure>
   <img src="./figs/Fe.png" width="400">
   <figcaption> 16 iron atoms in a periodic box (black lines).
</figure>

<figure>
   <img src="./figs/h2o_box.png" width="400">
   <figcaption> 200 water molecules inside a periodic box spanning 20 &angst; x 20 &angst; x 20 &angst; (black lines).
</figure>
</center>

[> This section should be composed of information provided by the submitter.]: #

### Fetch and build program

[> Provide commands to fetch and build the program. Include version numbers where possible.]: #

CASTEP is licensed by STFC, providing a free of cost license for academic use. This is different from the commercial license for CASTEP, available from BIOVIA. To gain a license for the academic use of CASTEP, a request may be made at <https://licences.stfc.ac.uk/product/castep>. The source code of CASTEP version 26 may then be downloaded and installed using the Make scripts included in the `scripts` directory.

The 'scripts' directory contains shell scripts to install CASTEP for CPU only and GPU accelerated applications. These scripts require user input to define a correct combination of compilers and libraries. Notably, to build GPU accelerated code, the NVIDIA Fortran and C compilers from the NVIDIA HPC SDK must be used to build CASTEP. Combination of maths libraries, FFT libraries and compilers are allowed, though the use of the Intel MKL FFT library requires the use of the Intel MKL maths libraries. When compiling using the included scripts, make will request user input for the location of the FFT and maths libraries. If the libraries are in the PATH, the user can leave this prompt empty. Optionally, the location of the libraries can be specified by including the make directives MATHLIBDIR and FFTLIBDIR in the shell script, and uncommenting the relevant lines in the CASTEP top level `Makefile`. 

### Fetch and run benchmark

[> Provide commands to fetch the benchmark. If it is directly provided by the submitter, note that here and provide as an attachment if permitted.]: #

The benchmarks for this pre-assessment of CASTEP can be obtained from:

```bash
git clone https://github.com/OscarvanVuren/castep_bench.git
```
This repository contains the benchmarks in `benchmarks`, as well as the compilation scripts for building both CPU and GPU accelerated binaries of CASTEP in `scripts`. Additionally, reference data for the benchmarks performed as part of this pre-assessment are available in the `data` directory.

[> Provide instructions on how to run the benchmark and indicate the expected I/O.]: #

To run the benchmark, the compiled binary for CASTEP is called using 'mpirun' or 'srun' depending on the test platform. A seedfile name must be provided; "Fe" or "H2O_box" in the case of the iron or water box testcases, respectively. The overall syntax for a CASTEP run is, therefore:

```bash
export SEEDNAME=<seed_name>
export EXECUTABLE=<path/to/castep.mpi>
mpirun -np <n_tasks> $EXECUTABLE $SEEDNAME 
```
OR
```bash
srun -n <n_slurm_tasks> $EXECUTABLE $SEEDNAME
```

This will perform a CASTEP run using the number of tasks specified.
> [!NOTE]
> CASTEP will not always run with the number of tasks specified, but will instead use the most tasks possible when parallelising over **k**-points. The Fe benchmark has a total of 32 **k**-points, thus using multiples of 32 tasks, even when more have been requested. When using very few MPI tasks, this behaviour is ignored to ensure optimal performance at low core counts.

The `SEEDNAME` used in running the benchmarks determines the output filenames. Most file IO has been disabled, however CASTEP will generate two files during execution: a`SEEDNAME.castep` output file and a `.usp` on the fly generated (OTFG) pseudopotential for each species in the `SEEDNAME.cell` file.


### Reference architecture

[> Add details of the reference architecture as provided by the submitter. Add any relevant details you may find regarding the architecture/system online. If the same system is accessible to you for the assessment, then indicate that here and detail the information in the next section(#hardware-information).]: #

## 2: Description of working environment

### Hardware information

[> Add the hardware information used, including where applicable the queue information if necessary. Comment on expected normal limit for the hardware, e.g. size of the largest interconnected set of nodes, or memory limitations. Compare with the reference architecture (if different), indicating any issues you may expect to see due to the differences.]: #

Three different HPC clusters were used in this pre-assessment, chosen due to a combination of availability and hardware variety. This information is taken from the websites linked for each header and corroborated by running `cat /proc/cpuinfo` or `lscpu`, particularly `lscpu -C' for detailed cache information, on the compute nodes. Only relevant compute hardware has been mentioned in these descriptions; untested hardware has been omitted.

1. The [Falcon](https://wiki.arcca.cf.ac.uk/index.php/The_Falcon_Supercomputer) cluster at Cardiff University.
   * AMD Epyc Genoa 9654 CPU. 192 cores per node. 30 nodes total across partitions.
   * Intel Xeon (Emerald Rapids) 6530 CPU, NVIDIA H200 GPU. 64 CPU cores per node, 4 H200 GPUs per node. 1 node available.
   * Intel Xeon (Emerald Rapids) 6530 CPU, NVIDIA H100 GPU. 64 CPU cores per node, 4 H100 GPUs per node. 1 node available.
   * Intel Xeon (Sapphire Rapids) 6430 CPU, NVIDIA L40S GPU. 64 CPU cores per node, 8 L40S GPUs per node. 2 nodes available.
   * Intel Xeon (Cascade Lake) 6248 CPU, NVIDIA V100 GPU. 40 CPU cores per node, 2 V100 GPUs per node. 13 nodes available

2. The [Isambard3 Multi-Architecture Comparison System (MACS)](https://docs.isambard.ac.uk), hosted by Bristol University.
   * AMD EPYC Milan 7543P CPU, NVIDIA A100 GPU. 32 CPU cores per node, 4 A100 GPUs per node. 2 nodes available.
   * AMD EPYC Milan 7543P CPU, NVIDIA H100 GPU. 32 CPU cores per node, 4 H100 GPUs per node. 1 node available.

3. The [Bede](https://bede-documentation.readthedocs.io/en/latest/index.html) cluster from the N8 Group, hosted at Durham University.
   [* IBM POWER9 CPU, NVIDIA V100 GPU. 32 CPU cores with 4 hardware threads for 128 processes per node. 4 V100 GPUs per node. 32 nodes available.]: #
   * NVIDIA Grace-Hopper GH200 Superchip. NVIDIA Grace CPU, 72 cores per node. 1 H100 GPU per node. 8 nodes available.

[> Provide processor, memory and cache information as well as interconnect information (e.g. Infiniband, NVlink - if across multiple nodes) of the system the assessment is to be performed on.]: #

[> The table below is an example and should be adapted for "non-standard" architectures, e.g. systems with Grace Hopper nodes. A placeholder for hardware accelerators is provided. For most assessment projects, these are likely to be GPUs (generally Nvidia or AMD). But some cases may also work with FPGAs, ASICs etc. Include any relevant specifications.]: #

**Falcon CPU**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 2 $\times$ [AMD EPYC Genoa 9654](https://www.amd.com/en/products/processors/server/epyc/4th-generation-9004-and-8004-series/amd-epyc-9654.html)        |
| Clock speed per CPU       | 2.4 GHz                                                                     |
| Sockets                   | 2                                                                          |
| Cores (per socket)        | 192 (96)                                                                        |
| CPU Cache                 | <ul><li>L1d 6 MiB </li><li> L1i 6 MiB </li><li> L2 192 MiB </li><li> L3 768 MiB </li></ul>      |
| RAM                       | 768 GB DDR5                                                               |

**Falcon GPU H200**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 2 $\times$ [Intel Xeon Gold (Emerald Rapids) 6530](https://www.intel.com/content/www/us/en/products/sku/237249/intel-xeon-gold-6530-processor-160m-cache-2-10-ghz/specifications.html)        |
| GPU                       | 4 $\times$ [NVIDIA H200](https://www.nvidia.com/en-gb/data-center/h200/) |
| GPU Memory                | 140 GB                                                                         |
| GPU Connect               | SXM5                                                                        |
| Clock speed per CPU       | 4.0 GHz                                                                     |
| Sockets                   | 2                                                                          |
| Cores (per socket)        | 64 (32)                                                                        |
| CPU Cache                 | <ul><li>L1d 6 MiB </li><li> L1i 6 MiB </li><li> L2 192 MiB </li><li> L3 768 MiB </li></ul>     |
| RAM                       | 1000 GB DDR5                                                               |

**Falcon GPU H100**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 2 $\times$ [Intel Xeon Gold (Emerald Rapids) 6530](https://www.intel.com/content/www/us/en/products/sku/237249/intel-xeon-gold-6530-processor-160m-cache-2-10-ghz/specifications.html)        |
| GPU                       | 4 $\times$ [NVIDIA H100](https://www.nvidia.com/en-gb/data-center/h100/) |
| GPU Memory                | 80 GB                                                                         |
| GPU Connect               | SXM5                                                                        |
| Clock speed per CPU       | 4.0 GHz                                                                     |
| Sockets                   | 2                                                                          |
| Cores (per socket)        | 64 (32)                                                                       |
| CPU Cache                 | <ul><li>L1d 6 MiB </li><li> L1i 6 MiB </li><li> L2 192 MiB </li><li> L3 768 MiB </li></ul>                              |
| RAM                       | 1000 GB DDR5                                                               |

**Falcon GPU L40S**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 2 $\times$ [Intel Xeon Gold (Sapphire Rapids) 6430](https://www.intel.com/content/www/us/en/products/sku/231737/intel-xeon-gold-6430-processor-60m-cache-2-10-ghz/specifications.html)        |
| GPU                       | 8 $\times$ [NVIDIA L40S](https://www.nvidia.com/en-gb/data-center/l40s/) |
| GPU Memory                | 48 GB                                                                         |
| GPU Connect               | PCIe                                                                        |
| Clock speed per CPU       | 4.0 GHz                                                                     |
| Sockets                   | 2                                                                          |
| Cores (per socket)        | 64 (32)                                                                        |
| CPU Cache                 | <ul><li>L1d 6 MiB </li><li> L1i 6 MiB </li><li> L2 192 MiB </li><li> L3 768 MiB </li></ul>                              |
| RAM                       | 1000 GB DDR5                                                               |

**Falcon GPU V100**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 2 $\times$ [Intel Xeon Gold (Cascade Lake) 6248](https://www.intel.com/content/www/us/en/products/sku/192446/intel-xeon-gold-6248-processor-27-5m-cache-2-50-ghz/specifications.html)        |
| GPU                       | 2 $\times$ [NVIDIA V100](https://www.nvidia.com/en-gb/data-center/tesla-v100/) |
| GPU Memory                | 16 GB                                                                         |
| GPU Connect               | PCIe                                                                        |
| Clock speed per CPU       | 2.5 GHz                                                                     |
| Sockets                   | 2                                                                          |
| Cores (per socket)        | 40 (20)                                                                       |
| CPU Cache                 | <ul><li>L1d 1.3 MiB </li><li> L1i 1.3 MiB </li><li> L2 40 MiB </li><li> L3 55 MiB </li></ul>     |
| RAM                       | 360 GB DDR5                                                               |

**Isambard3 MACS "ampere"**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 1 $\times$ [AMD EPYC Milan 7543P](https://www.amd.com/en/products/processors/server/epyc/7003-series/amd-epyc-7543p.html)        |
| GPU                       | 4 $\times$ [NVIDIA A100](https://www.nvidia.com/en-gb/data-center/a100/) |
| GPU Memory                | 40 GB                                                                         |
| GPU Connect               | SXM4                                                                        |
| Clock speed per CPU       | 4.0 GHz                                                                     |
| Sockets                   | 1                                                                          |
| Cores (per socket)        | 32 (32)                                                                       |
| CPU Cache                 | <ul><li>L1d 1 MiB </li><li> L1i 1 MiB </li><li> L2 16 MiB </li><li> L3 256 MiB </li></ul>     |
| RAM                       | 256 GB DDR5                                                               |

**Isambard3 MACS "hopper"**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 1 $\times$ [AMD EPYC Milan 7543P](https://www.amd.com/en/products/processors/server/epyc/7003-series/amd-epyc-7543p.html)        |
| GPU                       | 4 $\times$ [NVIDIA H100](https://www.nvidia.com/en-gb/data-center/h100/) |
| GPU Memory                | 80 GB                                                                         |
| GPU Connect               | PCIe                                                                        |
| Clock speed per CPU       | 4.0 GHz                                                                     |
| Sockets                   | 1                                                                          |
| Cores (per socket)        | 32 (32)                                                                        |
| CPU Cache                 | <ul><li>L1d 1 MiB </li><li> L1i 1 MiB </li><li> L2 16 MiB </li><li> L3 256 MiB </li></ul>     |
| RAM                       | 256 GB DDR5                                                               |

[**Bede POWER9** ]: #
[IBM POWER9 CPUs have 4 hardware threads available per core. System summaries that do not contain an explicit thread count have one thread per CPU core. ]: #
[| Specification             | Per node                                                                     | ]: #
[| ------------------------- | ---------------------------------------------------------------------------- | ]: #
[| Processors                | 2 $\times$ IBM POWER9 CPU                                                      | ]: #
[| GPU                       | 4 $\times$ NVIDIA V100(https://www.nvidia.com/en-gb/data-center/tesla-v100/) | ]: #
[| GPU Connect               | NVLink 2.0                                                                     | ]: #
[| Clock speed per CPU       | 2.4 GHz                                                                     | ]: #
[| Sockets                   | 2                                                                          | ]: #
[| Cores (threads)           | 40 (160)                                                                    | ]: #
[| RAM                       | 512 GB DDR4                                                               | ]: #
[| Local storage             | 480 GB SATA SSD                                                           | ]: #

**Bede Grace Hopper**
| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | 1 $\times$ [NVIDIA GRACE](https://www.nvidia.com/en-gb/data-center/grace-cpu-superchip/)        |
| GPU                       | 1 $\times$ [NVIDIA H100](https://www.nvidia.com/en-gb/data-center/h100/) |
| GPU Memory                | 96 GB                                                                         |
| GPU Connect               | NVLink-C2C                                                                        |
| Clock speed per CPU       | 3.483 GHz                                                                     |
| Sockets                   | 1                                                                          |
| Cores (per socket)        | 72 (72)                                                                   |
| CPU Cache                 | <ul><li>L1d 9 MiB </li><li> L1i 9 MiB </li><li> L2 144 MiB </li><li> L3 228 MiB </li></ul>                            |
| RAM                       | 480 GB DDR5                                                               |

[> Add details that you can obtain from the system or online information about sockets, NUMA regions, interconnects and cache sizes.]: #

### Libraries and modules

[> Provide information on the libraries that need to be installed or modules that must be loaded based on the compilation information provided by the submitter.]: #

CASTEP depends on three core maths libraries: LAPACK, BLAS and an FFT library.

**LAPACK and BLAS**

CASTEP supports multiple implementations of BLAS and LAPACK: [flexiblas](https://www.mpi-magdeburg.mpg.de/projects/flexiblas),[openblas](https://www.openmathlib.org/OpenBLAS/docs/),[mkl](https://www.intel.com/content/www/us/en/developer/tools/oneapi/onemkl.html),[blis](https://github.com/flame/blis),[libflame](https://github.com/flame/libflame),[aocl](https://www.amd.com/en/developer/aocl.html),[atlas](https://math-atlas.sourceforge.net/), [essl](https://www.ibm.com/docs/en/aix/7.1.0?topic=techniques-calling-blas-essl-libraries), [scilib](https://cpe.ext.hpe.com/docs/latest/csml/cray_libsci.html), default, generic. Links for more information have been provided where possible. The Flame library (libflame) contains LAPACK only, and therefore must be paired with a BLAS library (BLIS is recommended). Options "default" and "generic" are synonyms and specify no particular BLAS and LAPACK library, searching for `libblas` and `liblapack` in the `PATH`.

**FFT Library**

The simplest FFT library to use in this case is the [Fastest Fourier Transform in the West (FFTW)](https://www.fftw.org/) library, which implements a performant and portable FFT algorithm. Another supported FFT library is included in the Intel oneAPI Maths Kernel Library (MKL), however using this FFT requires using the MKL versions of BLAS and LAPACK.

[### Assessment tools]: #

[> 1. Limit pre-assessment tools to those with very low runtime. Mostly just focus on whether the program is running as expected. Do not assess the results of the benchmark for correctness as that requires domain-specific knowledge.]: #
[> 2. High level assessment tools and techniques which are expected to be useful, like global measures such as wall time.]: #
[> 3. If additional information is provided, you can address the low-level assessment that may be required, and if you may require privileges on the system. Only address this section if you confident that enough domain information has been provided, with respect to scaling of the compute and memory with the problem size.]: #

[1. Pre-assessment:]:#
[   - `<tool_1>`]:#
[   - `<tool_2>`]:#

[2. High-level assessment:]:#
[   - `<tool_1>`]:#
[   - `<tool_2>`]:#

[3. Low-level assessment:]:#
[   - `<tool_1>`]:#
[   - `<tool_2>`]:#

## 3: Compiler setup and optimisations

[> Based on the compilation information provided by submitter, comment on the following (where applicable):]: #
[>]: #
[> - package manager (e.g. `spack`)]: #
[> - build toolchain (e.g. `cmake`)]: #
[> - main compiler version (e.g. GCC 11)]: #
[> - compiler optimisations (e.g. -O3, `--fast-math`)]: #
[> - additional accelerator libraries and versions (e.g. SYCL revision 11, Kokkos 5.1)]: #
[> - any feature sets which are toggled on (e.g. vectorisation)]: #
[>]: #
[> Add additional information about the impact of optimisations on convergence or correctness of results if provided by the submitter.  If there are any issues with compatibility on the machine you are testing on, or any build issues experienced, provide details]: #

Requirements of compiling on multiple systems and architectures has driven the choice of compilers and optimisations for the building of CASTEP. The build process is supported by GNU make and example scripts to build CPU only and GPU accelerated binaries are available in the `scripts` directory of the benchmark repository. For all GPU accelerated binaries, the NVIDIA Fortran and C compilers `nvfortran` and `nvc`. Optimisation was standardised at the CASTEP level, which allows for the automated building of standard `debug`, `intermediate`, `fast` or `coverage` binaries, controlled by the `BUILD` directive given to `make`. This pre-assessment has exclusively used `BUILD=fast` to ensure that compiler optimisation of CASTEP is performed as expected for production level simulations. For `gfortran`, the `fast` compiler flags are `-O3 -funroll-loops -fno-signed-zeros -g -fbacktrace`. For `nvfortran`, compiler optimisation flags are `-fastsse -O3`.

Whilst more aggressive optimisation of the compilation of CASTEP is possible, it would not be representative of the real-world applications of CASTEP, where often a centralised software build is controlled by HPC managers, rather than individually compiled by researchers. This incentivised the use of the CASTEP driven optimisation of the code, via the `BUILD` directive, ensuring effective and stable performance of CASTEP with minimal time investment.

[> MAQAO should present missed compiler optimisation opportunities. Increasing the optimisation level may require re-converging the system to confirm accuracy. This may be outside the scope of the assessment.]: #

## 4: Computational complexity and scaling

For the benchmarks presented thus far, the limits of strong scaling are reached. This is mainly due to the benchmarks themselves being optimised for intranode testing and GPU acceleration profiling within a single node environment. Expanding the tests to explore strong scaling across multiple nodes is possible by expanding the size of the system under consideration; model more atoms in both benchmarks.

[> Comment on the possibility of scaling the problem up and down, both in strong (changing number of work units e.g. CPUs, but keeping the problem size constant) and weak (changing the problem size but keeping number of work units the same) contexts. Add any information provided by the submitter regarding the scaling of _computation (i.e. work)_, _memory_ and _execution time_ as the problem size or work units are increased.]: #
[> If there is existing scaling information (graphs or raw data) available, attach it to this report or add links to access it.]: #

In the cases where the regular benchmarks were not large enough, the supercell size for the Fe benchmark was increased from 2x2x2 to 4x4x4 (an increase by factor of 8). Alternatively, the Fe benchmark can be scaled by increasing the number of **k**-points sampled, via increasing the `KPOINTS_MP_GRID` keyword in `Fe.cell` beyond `4 4 4`. Making the grid non-uniform, for example `KPOINTS_MP_GRID 5 6 7`, is not recommended for physical reasons. This will increase the computational complexity of the benchmark by a linear factor of the product of the integers given to `KPOINTS_MP_GRID`.

THe water box benchmark can be scaled by increasing the planewave cutoff energy in `H2O_box.param`  using the `cut_off_energy` keyword. Suggested cutoff energies to increase the complexity of the water box task are 600 eV or 800 eV.

## 5: Memory, storage and I/O

[> Comment on the expected in memory size of the program at runtime, including data. An estimate of this information should be provided as part of the submission. For jobs submitted to Hamilton as part of early assessment, the Hamilton dashboard can be used to gauge memory usage (see Hamilton Portal Performance(https://www.durham.ac.uk/research/institutes-and-centres/advanced-research-computing/hamilton-supercomputer/usage/portal/performance/)).]: #

CASTEP provides an estimate of the memory requirement for each run, based on approximations of the memory requirements of the code and static data, model inputs, and estimates of the memory needs of the electronic localisation procedure, force computation and stress computation. These estimates, for both benchmarks, are collated below for different CPU architectures. The overall memory requirements are mostly driven by the size of the CASTEP binary, which varies greatly between compilation and architecture, ranging from 25739.0 MB in the case of the AMD EPYC Genoa 9654 build, to 8681.0 MB for the Intel Xeon 6530 build. The estimated memory needs of the model and computations are identical between compilations and are tabulated below, constructed from serial runs of CASTEP for each benchmark.

**Fe 2x2x2**
Memory Use Case                             | Estimated Memory Need Per Process / MB |
--------------------------------------------|----------------------------------:|
Model and support data                      | 1542.3                          |
Electronic energy minimisation requirements | 2348.2                          |
Force calculation requirements              | 34.8                            |
Stress calculation requirements             | 34.8                            |

[CPU                       | Peak Memory Estimate / MB | Peak Memory Measured / MB |        ]: #
[--------------------------|------------------------------:|------------------------------:|]: #
[AMD EPYC Genoa 9654       | 4831065.6                     | 25973.128                     |]: #
[Intel Xeon 6530           | 575635.2                      | 9405.452                      |]: #
[NVIDIA Grace              | 53203.2                       | 1089.216                      |]: # 

**Water Box**
Memory Use Case                             | Estimated Memory Need Per Process / MB |
--------------------------------------------|----------------------------------:|
Model and support data                      | 7605.0                          |
Electronic energy minimisation requirements | 4664.4                          |
Force calculation requirements              | 262.8                           |
Stress calculation requirements             | 319.1                           |

[CPU                       | Peak Memory Estimate / MB | Peak Memory Measured / MB         |]: #
[--------------------------|------------------------------:|------------------------------:|]: #
[AMD EPYC Genoa 9654       | 4962067.2                     | 27325.188                     |]: #
[Intel Xeon 6530           | 570944.0                      | 10235.132                     |]: #
[NVIDIA Grace              | 58370.4                       | 1945.024                      | ]: #


[> Comment on the expected storage requirements of the program, are there large amounts of temporary files (either in quantity or in total size)? An estimate of this information should be provided as part of the submission. A program that produces a large amount of temporary checkpoint files should have checkpoints turned off where possible.]: #

Additionally, each benchmark will write output files to the disk, totalling approximately 1.5 MB for the water box benchmark and 2 MB for the Fe benchmark. There is also the potential for CASTEP to output a detailed timing profile using the code block:
```
%block devel_code
TRACE PROF: * :END PROF
%endblock devel_code
```
which will write a file `SEEDNAME.MPI_PROC.profile` for each MPI process used in the benchmark run. These files are only written after the simulation is complete and each one is around 350 KB.


[> Comment on the expected output, including when the I/O is performed, and your observations when running the benchmark. This output should be minimal when testing the working performance of the program rather than the I/O saturation. Excessive I/O will result in an inaccurate performance assessment and may result in rejection.]: #

## 6: Additional comments from submitter

### Benchmarking Data for CASTEP

Included in this section are the outcomes from preliminary benchmarking performed using the methods described above. Testing was performed on the HPC facilities [enumerated previously](#2-description-of-working-environment), with each CPU and CPU/GPU combination profiled individually.

#### CPU Benchmarking

**Fe 2x2x2**

Raw timing data for the CPU only performance of various architectures, alongside strong scaling up to 32 MPI processes are presented here.

<figure>
   <img src="./figs/plots/fe_cpu.png" width="600">
   <figcaption> Raw calculation timing data for the Fe 2x2x2 benchmark.
</figure>

<figure>
   <img src="./figs/plots/fe_cpu_strongScaling.png" width="600">
   <figcaption> Strong scaling of the Fe 2x2x2 benchmark, up to 32 MPI processes.
</figure>


**Waterbox**

<figure>
   <img src="./figs/plots/waterbox_cpu.png" width="600">
   <figcaption> Raw calculation timing data for the water box benchmark.
</figure>

<figure>
   <img src="./figs/plots/waterbox_cpu_strongScaling.png" width="600">
   <figcaption> Strong scaling of the water box benchmark, up to 32 MPI processes.
</figure>

#### GPU Benchmarking

**Fe 2x2x2**

<figure>
   <img src="./figs/plots/fe_gpu.png" width="600">
   <figcaption> Raw calculation timing data for the Fe 2x2x2 benchmark, when using GPU acceleration.
</figure>

<figure>
   <img src="./figs/plots/fe_gpu_speedup.png" width="600">
   <figcaption> Speedup from CPU by using a GPU for the Fe 2x2x2 benchmark.
</figure>

<figure>
   <img src="./figs/plots/fe_gpu_percentage.png" width="600">
   <figcaption> Computational cost saving from using a GPU for the Fe 2x2x2 benchmark. Higher is better.
</figure>

**Fe 4x4x4**

<figure>
   <img src="./figs/plots/multiGPU_testing.png" width="600">
   <figcaption> Calculation timing data for an expanded Fe 4x4x4 benchmark, when using GPU acceleration with multiple GPUs.
</figure>

**Waterbox**

<figure>
   <img src="./figs/plots/waterbox_gpu.png" width="600">
   <figcaption> Raw calculation timing data for the water box benchmark, when using GPU acceleration.
</figure>

<figure>
   <img src="./figs/plots/waterbox_gpu_speedup.png" width="600">
   <figcaption> Speedup from CPU by using a GPU for the water box benchmark.
</figure>

<figure>
   <img src="./figs/plots/waterbox_gpu_percentage.png" width="600">
   <figcaption> Computational cost saving from using a GPU for the water box benchmark. Higher is better.
</figure>

#### Compiler Testing

As well as profiling the performance of hardware on HPC facilities using these benchmarks, the effect of compiler and library combination has been explored. This was done using the Fe 2x2x2 benchmark, pinned at 16 MPI processes. Performance as a function of compiler and library stack was measured; comparing gfortran+openMPI+openBLAs+FFTW3 to an Intel stack of ifort+Intel MPI+MKL+MKL_FFT. Additionally, an NVIDIA toolchain was explored where possible, using the NVIDIA Performance Libraries (NVPL) and openMPI as shipped with the NVIDIA HPC Toolkit (nvfortran+openMPI+NVPL+FFTW3).

<figure>
   <img src="./figs/plots/compiler_testing.png" width="1000">
   <figcaption> Calculation time for an Fe 2x2x2 benchmark on 16 MPI processes, when compiing CASTEP using different compiler and library stacks.
</figure>

#### Data Availability

The data used to generate these plots is available as a `.csv` from the `data` directory. All raw data is also included as a tarball.

[> Include any additional information from the submitter that does not fit the previous sections.]: #

[## 7: Pre-assessment outcome]: #

[> Indicate whether the assessment will proceed to the high-level stage. If the assessment is rejected here, comment on why and how to proceed.]: #
