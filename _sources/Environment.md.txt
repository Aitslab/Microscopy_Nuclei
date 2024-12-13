# Conda Environment

For training UNet models We have created a Conda environment and export it
through:
```console
conda env export --from-history
```

It is possible to recreate conda env through
```console
conda env create -f environment.yml
```
and run the code in that environment or use the containerized version.

# Apptainer container
For creating an apptainer image of conda environment a docker definition file is defined as follows:
```bash
Bootstrap: docker

From: continuumio/miniconda3

%files
    environment_ubuntu.yml

%post
    conda env create -f environment_ubuntu.yml

%runscript
    exec conda/envs/$(head -n 1 environment.yml | cut -f 2 -d ' ')/bin/"$@"
```



The Windows Subsystem for Linux (WSL) tool were installed as well as Apptainer.

In ubuntu terminal the singulairity/apptainer image(.sif) file were built.

```bash
sudo apptainer build docker_def.def container_name.sif
```

and then activate the apptainer shell:

```bash
apptainer shell your-container-name.sif
```

