---
License: Copyright © 2026 Durham University, SHAREing Project, MIT Licensed
Creator: Andrew Naden
Contributors: Andrew Naden; Ananya Gangopadhyay
Summary: Pre-assessment report template document
---

# Pre-assessment template

> [!NOTE]
>
> 1. Guidance throughout this template is provided using information boxes, along with example text and placeholders. The example text is not meant to be a complete comment, but a suggestion to get you started. Please remove the information boxes and replace the example text in your report.
>
> 2. The template is based on Chapter 4 of the [performance assessment guidebook](https://shareing-dri.github.io/performance-assessment/guidebook). However, the guidebook should not be necessary to perform the pre-assessment using this template.
>
> 3. This template is not exhaustive, but should provide the framework for completing a pre-assessment using the information submitted via the [assessment submission form](https://forms.office.com/Pages/ResponsePage.aspx?id=i9hQcmhLKUW-RNWaLYpvlIUXnqx3D81Bt-KemxGOyY5UOU9RQkFSOE5UU1Y1QVZNS0QyNlFYNjRNOS4u), by the submitter.

## Assessment objective

>[!IMPORTANT]
> Is this a test submission, internal review or external review? If it is part of a larger project or funding, add that information here. You may also include your (the assessor's) name, optionally with contact details.

This is a pre-assessment study of the performance of the planewave Density Functional Theory (DFT) code CASTEP, performed by Oscar van Vuren at Cardiff University, in collaboration with Phil Hasnip of the University of York and Martyn Guest of ARCCA. CASTEP allows for the accurate simulation of the electronic structure of condensed matter, to scales of hundreds to thousands of atoms. The CASTEP code has seen historical use for benchmarking flagship HPC facilities in the UK, including HECToR, ARCHER and ARCHER2, and sees contemporary use in condensed matter simulations across the globe.

The associated SHAREing work package for this pre assessment of CASTEP is WP 019. The objective of this assessment is to provide SHAREing with a preliminary set of performance data, benchmarks and compilation instructions to allow for a more detailed, rigorous benchmark of CASTEP in future by SHAREing.

CASTEP version 26 was benchmarked using CPU only and GPU accelerated workloads on three different HPC architectures

A test submission of `<repository/website>`, version `<version_number>` using the `<benchmark_name>` benchmark. This assessment is performed by `<assessor_name>` of `<assessor_affiliation>` on `<assessment_date>`.

## Disclaimers

1. This report is not a commentary on code quality, but an indicator of the quality of the current SHAREing testing methodology as of `<guidance_date>`.
2. The pre-assessment is only a preliminary assessment of submission suitability and does not guarantee a full assessment. It will be provided to the submitter indicating if the full assessment will be undertaken or detail reasons for rejection.

## Table of contents

>[!TIP]
> Place `x` inside the box when complete to mark the checkbox.

- [ ] [1: Benchmark setup](#1-benchmark-setup)
- [ ] [2: Description of working environment](#2-description-of-working-environment)
- [ ] [3: Compiler setup and optimisations](#3-compiler-setup-and-optimisations)
- [ ] [4: Computational complexity and scaling](#4-computational-complexity-and-scaling)
- [ ] [5: Memory, storage and I/O](#5-memory-storage-and-io)
- [ ] [6: Additional comments from submitter](#6-additional-comments-from-submitter)
- [ ] [7: Pre-assessment outcome](#7-pre-assessment-outcome)

## 1: Benchmark setup

>[!IMPORTANT]
> This section should be composed of information provided by the submitter.

### Fetch and build program

>[!IMPORTANT]
> Provide commands to fetch and build the program. Include version numbers where possible.

The submitter has requested an assessment of `<repository/program_name>`, version `<version_number`>/branch `<branch_name>`. It can be fetched/installed as follows:

```bash
# Example 1: Install specific version with a package manager
spack install <program_name>@<version_number>

# Example 2: Clone a specific branch of a repository
git clone -p <branch_name> <repository> && make release
```

### Fetch and run benchmark

>[!IMPORTANT]
> Provide commands to fetch the benchmark. If it is directly provided by the submitter, note that here and provide as an attachment if permitted.

The benchmark can be obtained from:

```bash
git clone <benchmark_repository>
```

>[!IMPORTANT]
> Provide instructions on how to run the benchmark and indicate the expected I/O.

To run the benchmark:

```bash
cd <benchmark_folder>
mpirun -np 24 <program_executable> <benchmark_name> <output_file>
```

The following output is expected every `<n>` iterations:

```txt
<example-output>
```

### Reference architecture

>[!IMPORTANT]
> Add details of the reference architecture as provided by the submitter. Add any relevant details you may find regarding the architecture/system online. If the same system is accessible to you for the assessment, then indicate that here and detail the information in the [next section](#hardware-information).

## 2: Description of working environment

### Hardware information

>[!IMPORTANT]
> Add the hardware information used, including where applicable the queue information if necessary. Comment on expected normal limit for the hardware, e.g. size of the largest interconnected set of nodes, or memory limitations. Compare with the reference architecture (if different), indicating any issues you may expect to see due to the differences.

The `<cluster_name>` system based at `<university/organisation>` was used for this assessment.

>[!IMPORTANT]
> Provide processor, memory and cache information as well as interconnect information (e.g. Infiniband, NVlink - if across multiple nodes) of the system the assessment is to be performed on.

The hardware details for `<cluster_name>` are `[available at](<URL>)`. This information is corroborated by running `cat /proc/cpuinfo` on one of the compute nodes via an interactive session. There are `<num_nodes>` compute nodes on the system:

>[!IMPORTANT]
> The table below is an example and should be adapted for "non-standard" architectures, e.g. systems with Grace Hopper nodes. A placeholder for hardware accelerators is provided. For most assessment projects, these are likely to be GPUs (generally Nvidia or AMD). But some cases may also work with FPGAs, ASICs etc. Include any relevant specifications.

| Specification             | Per node                                                                     |
| ------------------------- | ---------------------------------------------------------------------------- |
| Processors                | `<num_procs>` $\times$ `[<processor_name>](<link-to-online-details>)`        |
| `<Hardware accelerators>` | `<num_gpus/fpgas>` $\times$ `[<accelerator_name>](<link-to-online-details>)` |
| Clock speed per CPU       | `<freq>`MHz                                                                  |
| Sockets                   | `<num_sockets>`                                                              |
| Cores                     | `<num_cores>`                                                                |
| RAM                       | `<memory_size>`GB `<memory_type>`                                            |
| Local storage             | `<storage_size>`GB `<storage_type>`                                          |

>[!TIP]
> Add details that you can obtain from the system or online information about sockets, NUMA regions, interconnects and cache sizes.

### Libraries and modules

>[!IMPORTANT]
> Provide information on the libraries that need to be installed or modules that must be loaded based on the compilation information provided by the submitter.

### Assessment tools

> [!IMPORTANT]
>
> 1. Limit pre-assessment tools to those with very low runtime. Mostly just focus on whether the program is running as expected. Do not assess the results of the benchmark for correctness as that requires domain-specific knowledge.
> 2. High level assessment tools and techniques which are expected to be useful, like global measures such as wall time.
> 3. If additional information is provided, you can address the low-level assessment that may be required, and if you may require privileges on the system. Only address this section if you confident that enough domain information has been provided, with respect to scaling of the compute and memory with the problem size.

1. Pre-assessment:
   - `<tool_1>`
   - `<tool_2>`

2. High-level assessment:
   - `<tool_1>`
   - `<tool_2>`

3. Low-level assessment:
   - `<tool_1>`
   - `<tool_2>`

## 3: Compiler setup and optimisations

>[!IMPORTANT]
> Based on the compilation information provided by submitter, comment on the following (where applicable):
>
> - package manager (e.g. `spack`)
> - build toolchain (e.g. `cmake`)
> - main compiler version (e.g. GCC 11)
> - compiler optimisations (e.g. -O3, `--fast-math`)
> - additional accelerator libraries and versions (e.g. SYCL revision 11, Kokkos 5.1)
> - any feature sets which are toggled on (e.g. vectorisation)
>
> Add additional information about the impact of optimisations on convergence or correctness of results if provided by the submitter.  If there are any issues with compatibility on the machine you are testing on, or any build issues experienced, provide details

Build instructions from the user `<build_toolchain>` to build the `<program_name>` with `<optimisation_level>`.

>[!tip]
> MAQAO should present missed compiler optimisation opportunities. Increasing the optimisation level may require re-converging the system to confirm accuracy. This may be outside the scope of the assessment.

## 4: Computational complexity and scaling

>[!IMPORTANT]
> Comment on the possibility of scaling the problem up and down, both in strong (changing number of work units e.g. CPUs, but keeping the problem size constant) and weak (changing the problem size but keeping number of work units the same) contexts. Add any information provided by the submitter regarding the scaling of _computation (i.e. work)_, _memory_ and _execution time_ as the problem size or work units are increased.
>
> If there is existing scaling information (graphs or raw data) available, attach it to this report or add links to access it.

The `<parameter>` value can be varied to increase the problem size for scaling tests.

## 5: Memory, storage and I/O

>[!IMPORTANT]
> Comment on the expected in memory size of the program at runtime, including data. An estimate of this information should be provided as part of the submission. For jobs submitted to Hamilton as part of early assessment, the Hamilton dashboard can be used to gauge memory usage (see [Hamilton Portal Performance](https://www.durham.ac.uk/research/institutes-and-centres/advanced-research-computing/hamilton-supercomputer/usage/portal/performance/)).

According to the submitter, the expected memory required for the benchmark is `<memory_size>`GB.

>[!IMPORTANT]
> Comment on the expected storage requirements of the program, are there large amounts of temporary files (either in quantity or in total size)? An estimate of this information should be provided as part of the submission. A program that produces a large amount of temporary checkpoint files should have checkpoints turned off where possible.

The benchmark also outputs files totalling `<storage_size>`MB.

>[!IMPORTANT]
> Comment on the expected output, including when the I/O is performed, and your observations when running the benchmark. This output should be minimal when testing the working performance of the program rather than the I/O saturation. Excessive I/O will result in an inaccurate performance assessment and may result in rejection.

The benchmark writes to the console output every `<n>` iterations [as indicated by the submitter](#fetch-and-run-benchmark).

## 6: Additional comments from submitter

>[!IMPORTANT]
> Include any additional information from the submitter that does not fit the previous sections.

## 7: Pre-assessment outcome

>[!IMPORTANT]
> Indicate whether the assessment will proceed to the high-level stage. If the assessment is rejected here, comment on why and how to proceed.
