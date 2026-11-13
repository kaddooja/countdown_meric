# Installing COUNTDOWN-MERIC with EasyBuild

`COUNTDOWN-1.0-gompi-2025b-MERIC.eb` builds `libcntd.so` (COUNTDOWN with the
MERIC energy library linked in) and creates the module
`COUNTDOWN/1.0-gompi-2025b-MERIC`.

EasyBuild downloads the code from this repository by itself. MERIC is included
in the repository (`meric/meric-external.tar.gz`, BSD-3 licence), so there is
nothing else to download.

## Requirements

- EasyBuild 5.x and Lmod (or Environment Modules)
- The `gompi/2025b` toolchain (GCC 14.3.0 + OpenMPI 5.0.8), CMake 4.0.3 and
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

**Cluster with internet:**

```bash
git clone https://github.com/kaddooja/countdown_meric.git
cd countdown_meric/easybuild
ml EasyBuild
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot --dry-run   # shows what will be built
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot
```

`--dry-run` builds nothing; it only lists the modules. `[x]` means already
installed and `[ ]` means it will be built. Normally only COUNTDOWN is `[ ]`.
If GCC or OpenMPI are `[ ]`, use a toolchain your site already has (see below).

The build takes a few minutes and ends with
`COMPLETED: Installation ended successfully`.

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

To remove the test installation afterwards: `rm -rf $HOME/eb-test`

**Using a different toolchain** (when your site has another one, see `ml av gompi`):

```bash
eb COUNTDOWN-1.0-gompi-2025b-MERIC.eb --robot --try-toolchain=gompi,2024a
```

**Common problems:**

- `Missing modules for dependencies: CMake/...` or `hwloc/...`: add `--robot`,
  or change the version in the recipe to one your site has (`ml av CMake hwloc`).
- Warning about `non-allowed loaded modules`: run `ml purge` and then
  `ml EasyBuild` before `eb`.

## 3. Use

```bash
module load COUNTDOWN/1.0-gompi-2025b-MERIC

mpirun --bind-to core -x LD_PRELOAD=$CNTD_LIBRARY ./your_mpi_app
# or with Slurm:
srun --cpu-bind=cores --export=ALL,LD_PRELOAD=$CNTD_LIBRARY ./your_mpi_app
```

- Compile your application with the same MPI (loading the module loads it).
- Each MPI rank must be pinned to its own CPU core.
- Pass `LD_PRELOAD` with `-x` / `--export` as shown, instead of
  `export LD_PRELOAD=...` in your shell, so only the application loads COUNTDOWN.
- Run on compute nodes. COUNTDOWN needs the CPU frequency files in
  `/sys/devices/system/cpu/cpu0/cpufreq/`, which many login nodes don't have
  (error: `Failed to open file: .../cpuinfo_max_freq`).

See the main README for the `CNTD_*` runtime settings.
