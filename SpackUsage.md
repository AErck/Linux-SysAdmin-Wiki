# Installing things with Spack

## Starting from Log In

* Make sure to start with `module purge` to clear any pre-loaded modules
* Next load your compiler `module load gnu7`
* Load Spack Apps `module load spack-apps`
* Load Spack `module load spack`
* Install a new module, for example python `spack install python@3.7.2%gcc@7.3.0` 
    * In this case we attempt the install of Python version 3.7.2 with the GCC compiler version 7.3.0
