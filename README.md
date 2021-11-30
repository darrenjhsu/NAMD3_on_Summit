
# Using NAMD3 on Summit

This note includes compilation of NAMD3 and some basic commands to run NAMD3 simulations on Summit.
This is not an official document.

## Compilation

I've followed and adapted Josh Vermaas's instruction on [GitHub](https://github.com/jvermaas/Software-Building-Instructions/blob/main/NAMD.md)

```bash
CUDA_PREFIX=/sw/summit/cuda/11.0.3
module load cuda/11.0.3 fftw
mkdir NAMDbuild
cd NAMDbuild
git clone https://github.com/UIUC-PPL/charm.git
cd charm
git checkout v6.10.2
./build charm++ pamilrts-linux-ppc64le   smp     -j16  --with-production
cd ..
git clone NAMD_SOURCE (get the GitLab access from NAMD dev team)
cd namd
ln -s ../charm .
# Get the NAMD3 development branch
git checkout devel
./config Linux-POWER-g++.summit30a9 --charm-arch pamilrts-linux-ppc64le-smp --with-fftw3 --with-cuda --cuda-prefix $CUDA_PREFIX --with-single-node-cuda
cd Linux-POWER-g++.summit30a9
make -j8
```

## Running NAMD3

Request an interactive session.

```bash
bsub -nnodes 1 -W 0:30 -P ABC123 -Is /bin/bash
```

When in, set up environment

```bash
module load cuda fftw
PATH=/gpfs/alpine/the/correct/path
NAMD_PATH=/home/path/to/namd
cd $PATH
```

Submitting NAMD3 runs is quite easy.
The easiest configuration, which is already pretty powerful, is with one CPU core and one GPU (NVIDIA V100). (Line 1)
One can request more than 1 core to accompany 1 GPU. (Line 2)
Alternatively, one can request multiple GPUs and an equal number of CPUs. (Line 3)

```bash
jsrun -r1 -c1 -g1 -a1 $NAMD_PATH/namd3 +devices 0 +p1 <input conf> | tee <output log>
jsrun -r1 -c<M> -g1 -a1 $NAMD_PATH/namd3 +devices 0 +p<M> <input conf> | tee <output log>
jsrun -r1 -c<N> -g <N> -a1 $NAMD_PATH/namd3 +devices 0,1,2...<N-1> +p<N> +setcpuaffinity <input conf> | tee <output log>
```


## Benchmarking (unofficial)

These are just by me looking at the runtime output and they are not extremely accurate.

### Apoa1 (92K atoms)

M cores, 1 GPU without `+setcpuaffinity`
```
jsrun -r1 -c<M> -g1 -a1 $NAMD_PATH/namd3 +devices 0 +p<M> <input conf> | tee <output log>
Using 1 GPU and 1 CPU core:    95 ns / day
Using 1 GPU and 2 CPU cores:   94 ns / day
Using 1 GPU and 4 CPU cores:   92 ns / day
```

M cores, 1 GPU with `+setcpuaffinity`
```
jsrun -r1 -c<M> -g1 -a1 $NAMD_PATH/namd3 +devices 0 +p<M> +setcpuaffinity <input conf> | tee <output log>
Using 1 GPU and 1 CPU core:    95 ns / day
Using 1 GPU and 2 CPU cores:   94 ns / day
Using 1 GPU and 4 CPU cores:   94 ns / day
```


N cores, N GPUs without `+setcpuaffinity`
```
jsrun -r1 -c<N> -g <N> -a1 $NAMD_PATH/namd3 +devices 0 +p<N> <input conf> | tee <output log>
Using 2 GPUs and 2 CPU cores: 126 ns / day
Using 3 GPUs and 3 CPU cores: 135 ns / day
Using 4 GPUs and 4 CPU cores: 125 ns / day
Using 6 GPUs and 6 CPU cores: Crashes
```

N cores, N GPUs with `+setcpuaffinity`
```
jsrun -r1 -c<N> -g <N> -a1 $NAMD_PATH/namd3 +devices 0,1,2...<N-1> +p<N> +setcpuaffinity <input conf> | tee <output log>
Using 1 GPU and 1 CPU core: 95 ns / day
Using 2 GPUs and 2 CPU cores: 132 ns / day
Using 3 GPUs and 3 CPU cores: 151 ns / day (!)
Using 4 GPUs and 4 CPU cores: 140 ns / day
Using 6 GPUs and 6 CPU cores: 125 ns / day
```

Bonus: Asking 2 cores per GPU
```
jsrun -r1 -c<2*N> -g <N> -a1 $NAMD_PATH/namd3 +devices 0,1,2...<N-1> +p<2*N> +setcpuaffinity <input conf> | tee <output log>
Using 2 GPUs and 4 CPU cores: 131 ns / day
Using 3 GPUs and 6 CPU cores: 143 ns / day

```

Suggestion: Always use `+setcpuaffinity`. If using more than one GPU, ask for same number of CPU cores. 


### STMV (1.07M atoms)

M cores, 1 GPU with `+setcpuaffinity`
```
jsrun -r1 -c<M> -g1 -a1 $NAMD_PATH/namd3 +devices 0 +p<M> +setcpuaffinity <input conf> | tee <output log>
Using 1 GPU and 1 CPU core:    8.5 ns / day
Using 1 GPU and 4 CPU cores:   8.4 ns / day
```

N cores, N GPUs with `+setcpuaffinity`
```
jsrun -r1 -c<N> -g <N> -a1 $NAMD_PATH/namd3 +devices 0,1,2...<N-1> +p<N> +setcpuaffinity <input conf> | tee <output log>
Using 1 GPU and 1 CPU core:    8.5 ns / day
Using 4 GPUs and 4 CPU cores: 24.3 ns / day
Using 6 GPUs and 6 CPU cores: 31.4 ns / day
```

The trend here roughly corresponds to that posted on [NAMD3 announcement](http://www.ks.uiuc.edu/Research/namd/alpha/3.0alpha/) with slightly lower performance probably due to other settings.
