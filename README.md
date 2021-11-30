
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


