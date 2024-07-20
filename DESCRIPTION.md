# OSU MPI Micro-Benchmarks

## Purpose

The OSU Micro-Benchmarks (OSU) are a set of MPI performance benchmarks for point-to-point
(and global) communication. Here, OSU is used for CPUs and GPUs to measure latency and bandwidth of
point-to-point communication.

_While this description tries to be agnostic with respect to the benchmarking infrastructure, we consider JUBE as our reference and give examples with it._

## Source

Archive name: `osu-bench.tar.gz`

The file holds instructions to run the benchmark, according JUBE scripts, and
source files to run OSU. We are using OSU version 6.2 which is included in the archive. It can also be downloaded from
https://mvapich.cse.ohio-state.edu/benchmarks/

Using JUBE, the source is built automatically.

## Building

Information about the build process can be retrieved from
https://mvapich.cse.ohio-state.edu/static/media/mvapich/README-OMB.txt

Alternatively, JUBE can be used to build the benchmark.

### Modification

No source code modifications are allowed.

### Multi-Threading

Not applicable.

### JUBE

The JUBE step `compile` takes care of building the benchmark. It untars the
sources, configures, and builds the benchmark.

## Execution

The executables picked for the benchmark are called `osu_bibw` and `osu_latency`; also `osu_bw`. For each, there are two test cases:

- GPU
  1. Intra-node: Point-to-point communication between all GPUs of a single node (one-to-all); report the **maximum bandwidth**/**minimum latency** achieved for the GPU-GPU combination with the lowest theoretical peak bandwidth
  2. Inter-node: Point-to-point communication between GPUs of two nodes; report the **maximum bandwidth**/**minimum latency** achieved for the GPU-GPU combination with the lowest theoretical peak bandwidth
- CPU
  1. Intra-node: Point-to-point communication between two cores of a node; report the **maximum bandwidth**/**minimum latency** achieved for the core-core combination with the lowest theoretical peak bandwidth
  2. Inter-node: Point-to-point communication between two nodes (one core per node); report the **maximum bandwidth**/**minimum latency** achieved for the node-node combination with the lowest theoretical peak bandwidth

The executables should be called with different message sizes:

* `osu_bw` and `osu_bibw`: 67108864
* `osu_latency`: 0


### Command Line

Please call the benchmark with

- GPU: 
  ```
  [mpiexec] ./osu_[bw|bibw|latency] -m ${msg_size}:${msg_size} -d ${buffer} ${target}
  ```

  For example, using CUDA
  ```
  [mpiexec] ./osu_[bw|bibw|latency] -m ${msg_size}:${msg_size} -d cuda D D
  ```
- CPU:
  ```
  [mpiexec] ./osu_[bw|bibw|latency] -m ${msg_size}:${msg_size}
  ```

### JUBE

The JUBE step `execute` calls the aforementioned command line with the correct
modules. It also cares about the MPI distribution by submitting a script to the
batch system. The latter is achieved by populating a batch submission script
template (via `platform.xml`) with information specified in the top of the
script relating to the number of nodes and tasks per node. Via dependencies, the
JUBE step `execute` calls the JUBE step `compile` automatically.

To submit a fully-contained benchmark run to the batch system, call

- GPU: `jube run benchmark/jube/osu_gpu.xml`
- CPU: `jube run benchmark/jube/osu_cpu.xml`

JUBE will generate the necessary configuration and files, and
submit the benchmark to the batch engine.

The parameters `queue` (Slurm batch queue) and `load_odules` (environment modules) of the JUBE script might need to be adapted.

After a run, JUBE can also be used to
extract the runtime of the program (with `jube analyse` and `jube result`).

## Verification

Not applicable.

## Results

The benchmark prints a bandwidth or latency value for the given message size to the standard output.

Both, message size and bandwidth/latency have to be reported.

### JUBE

Using `jube analyse` and a subsequent `jube result` prints an overview table
with the number of nodes, tasks per node, runtime, and further values.

- GPU: `jube result benchmark/jube/GPU_procure_run -a`
- CPU: `jube result benchmark/jube/CPU_procure_run -a`

The metrics are shown in the columns `size` and `result`.

Example JUBE output for the GPU part, abbreviated:

| nodes |Tasks / Node| cuda_vis |       osu_exe | msg_size | runtime[sec] |     size |    result |
|-------|------------|----------|---------------|----------|--------------|----------|-----------|
|     1 |           2|      0,1 |    `osu_bibw` | 67108864 |         3.92 | 67108864 | 176667.35 |
|     2 |           1|        1 |    `osu_bibw` | 67108864 |        11.42 | 67108864 |  34229.53 |
|     1 |           2|      0,1 |      `osu_bw` | 67108864 |         4.37 | 67108864 |  93500.13 |
|     2 |           1|        1 |      `osu_bw` | 67108864 |         8.14 | 67108864 |  24162.07 |
|     1 |           2|      0,1 | `osu_latency` |        0 |         3.02 |        0 |      0.74 |
|     2 |           1|        1 | `osu_latency` |        0 |         1.94 |        0 |      2.43 |

## Commitment

The following combinations are used for the quantitative evaluation:

- GPU: `osu_bibw` (inter-node) and `osu_latency` (inter-node)
- CPU: `osu_bibw` (inter-node) and `osu_latency` (inter-node)

The remaining combinations are used for the qualitative evaluation (intra-node, `osu_bw`).
