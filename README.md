
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
The easiest configuration, which is already pretty powerful, is with one CPU and one GPU. (Line 1)
One can request more than 1 CPU to accompany 1 GPU. (Line 2)
Alternatively, one can request multiple GPUs and an equal number of CPUs. (Line 3)

```bash
jsrun -r1 -c1 -g1 -a1 $NAMD_PATH/namd3 +devices 0 +p1 <input conf> | tee <output log>
jsrun -r1 -c<M> -g1 -a1 $NAMD_PATH/namd3 +devices 0 +p<M> <input conf> | tee <output log>
jsrun -r1 -c<N> -g <N> -a1 $NAMD_PATH/namd3 +devices 0,1,2...<N-1> +p<N> +setcpuaffinity <input conf> | tee <output log>
```


# Benchmarking (unofficial)
