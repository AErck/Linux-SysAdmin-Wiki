# Installing things with Spack

## Starting from Log In

* Make sure to start with `module purge` to clear any pre-loaded modules
* Next load your compiler `module load gnu7`
* Load Spack Apps `module load spack-apps`
* Load Spack `module load spack`
* Install a new module, for example python `spack install python@3.7.2%gcc@7.3.0` 
    * In this case we attempt the install of Python version 3.7.2 with the GCC compiler version 7.3.0

## Fix a No Checksum Warning

First we need to colect the new checksums with `spack checksum <module_name>`
When prompted enter a <number> to get the checksums for the last <number> of versions of <module_name>
Copy these new checksums for the module.
Run the command `spack edit <module_name>` to edit the spack build file for the designated module
