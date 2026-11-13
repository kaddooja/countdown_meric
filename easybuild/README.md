# Installing COUNTDOWN-MERIC with EasyBuild

`COUNTDOWN-1.0-gompi-2025b-MERIC.eb` builds `libcntd.so` (COUNTDOWN with the
MERIC energy library linked in) and creates the module
`COUNTDOWN/1.0-gompi-2025b-MERIC`.

## Requirements

- EasyBuild 5.x and Lmod (or Environment Modules)
- The `gompi/2025b` toolchain (GCC 14.3.0 + OpenMPI 5.0.8), CMake 3.31.8 and
  hwloc 2.12.1. If your site doesn't have them, `--robot` builds them, but that
  takes a few hours. You can also use another toolchain with `--try-toolchain`
  (see below).
- Python 3 on the build node (MERIC builds with waf)

## 1. Choose the energy domain

Open the `.eb` file and set the domain for your hardware:

```python
local_energy_domain = 'NVML'   # NVML, RAPL, A64FX, HDEEM or ROCM
```

| Hardware                         | Domain  |
|----------------------------------|---------|
| NVIDIA GPUs (e.g. GH200)         | `NVML`  |
| Intel or AMD x86 CPUs            | `RAPL`  |
| Fujitsu A64FX                    | `A64FX` |

For `NVML`, also check `local_nvml_inc` (the folder with `nvml.h`, from CUDA)
and `local_nvml_lib` (the folder with `libnvidia-ml.so`, from the driver).

## 2. Build

The recipe downloads COUNTDOWN from this repo and MERIC from the `v1.0`
release of this repo.

**Cluster with internet:**

```bash
git clone https://github.com/kaddooja/countdown_meric.git
cd countdown_meric/easybuild
ml EasyBuild
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot --dry-run   # shows what will be built
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot
```

**Build nodes without internet:** download the sources on a login node first,
then build on a compute node:

```bash
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --fetch       # login node
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot       # compute node / Slurm job
```

**Testing in your home directory**, without touching the site's software stack:

```bash
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot --prefix=$HOME/eb-test
module use $HOME/eb-test/modules/all
```

**Using a different toolchain** (when your site has another one):

```bash
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot --try-toolchain=gompi,2024a
```

## 3. Use

```bash
module load COUNTDOWN/1.0-gompi-2025b-MERIC
export LD_PRELOAD=$CNTD_LIBRARY
srun ./your_mpi_app          # or mpirun
```

Each MPI rank must be pinned to its own CPU. See the main README for the
`CNTD_*` runtime settings.
