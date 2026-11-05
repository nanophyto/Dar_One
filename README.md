# DAR1

Built off of Gael Forget's MITgcmTools! (https://github.com/gaelforget/MITgcmTools.jl)

[DAR1 Documentation](https://barbara42.github.io/Dar_One/build/getting_started/)
- [tutorials](https://barbara42.github.io/Dar_One/build/beginner_tutorials/)
- [case studies](https://barbara42.github.io/Dar_One/build/case_studies/)


Documentation for the darwin3 MITgcm fortran model can be found [here](https://darwin3.readthedocs.io/en/latest/phys_pkgs/darwin_bacteria.html) 
##  Overview of program structure 


DAR1 is a tool for designing and running experiments simulating the marine microbiome within a single cube of water. The biogeochemical forcing is powered by the [MIT Darwin Model](https://darwinproject.mit.edu/), and DAR1 provides a Julia interface to build, configure, and run experiments with a simple, streamlined workflow. DAR1 is composed of two main parts:

- [DAR1](https://github.com/barbara42/Dar_One) - Julia interface for organizing experiments and setting up parameters for model runs 
    - `MITgcm_path` variable should point to darwin3
- [darwin3](https://github.com/darwinproject/darwin3) - MITgcm source code set up to include all things Darwin 
    - `DAR1_config` - folder for the base configuration files for DAR1, which lives in the darwin3/verification/ folder. ([DAR1_config github](https://github.com/barbara42/dar_one_config))


DAR1 can be run in one of two ways. 
1) In a **docker container** using the DAR1_docker image, where all dependencies are handled for you 
2) On your local machine, where you have to set up the environment, download dependencies, etc 
    - this is more time consuming, and I strongly recommend using the docker image
    - instructions for local setup are below

# Setting up with docker

(1) Download and install [docker desktop](https://www.docker.com/).
- Run the docker application, which will start up the docker "daemon" 
- The docker daemon must be running in order to download the DAR1 image

(2) Get the [DAR1_docker image](https://hub.docker.com/repository/docker/birdy1123/dar1).
- using the command line, run 

    `docker pull birdy1123/DAR1_docker`

- If you go to the "Images" tab in the docker desktop UI, you should see "birdy1123/DAR1_docker"

(3) Run a container based on the DAR1_docker image
- using the command line, run 

    `docker run -it birdy1123/DAR1_docker` 
- this runs the container in interactive mode (`-i`) with terminal access (`-t`) 
- you should see the prompt change to `root@some-number:/DAR1_docker#`
    - if the container is already running, you can connect to it with `docker exec -it [CONTAINER_ID] bash`
    - Note: connecting with `bash` rather than `sh` gives you more a powerful terminal (i.e. with tab completion) 
  
You're now ready for the [beginner tutorial](beginner_tutorial)!
# Setting up on local machine

See instructions in the documentation: https://barbara42.github.io/Dar_One/build/getting_started/#Setting-up-on-local-machine

## Pinned Darwin sources

The tested Darwin3 and `dar_one_config` revisions are recorded in [`darwin-sources.env`](darwin-sources.env). CI reads this file; local scripts can load the same versions with `set -a; . ./darwin-sources.env; set +a`.

## Example integration tests

The example test runner executes all nine beginner tutorial and case-study scripts, including Fortran builds, simulations, and the tutorial plot. It uses compact Darwin reference inputs and short simulation durations. Run it with a Darwin checkout containing `verification/dar_one_config` and a working GNU Fortran toolchain:

```bash
DARONE_MITGCM_PATH=/path/to/darwin3 julia --project=. test/test_examples.jl
```

Inside the existing Docker image, the default `/dar_one_docker/darwin3` path is used automatically. Test runs receive unique experiment names; their results are written under Darwin's `verification/dar_one_config/run` directory. The runner also updates the Darwin configuration source files when rebuilding each example.

The tests use the bundled Darwin reference fields, run all 16 global tiles for one model year, and verify normal termination and nonempty NetCDF diagnostics. The beginner tutorial also verifies that its plotting example runs successfully.

# Contributing 

We have three main branches: `main`, `test`, and `dev`.

All other branches are feature branches. Once a feature has been implemented, tested, and documented you can make a pull request to merge it into the `dev` branch. 

`main` is the most stable version of the code. This branch is used for the docker image. 

## Manuscript Figure 4C reproduction

The independent ten-year, 16-tile reproduction remains in
[`paper/fig4c/`](paper/fig4c/). With a configured Darwin3 checkout and
working Julia/Fortran dependencies, run from the repository root:

```bash
export DARONE_MITGCM_PATH=/path/to/darwin3
export DARONE_DATA_DIR="$(mktemp -d)"
ln -s "$PWD/data/darwin/surface_tracers.nc" "$DARONE_DATA_DIR/3d.nc"
ln -s "$PWD/data/darwin/surface_temperature.nc" "$DARONE_DATA_DIR/tave.nc"
ln -s "$PWD/data/darwin/par_upper4.nc" "$DARONE_DATA_DIR/par.nc"
julia --project=. case_studies/global_steadystate_build.jl
export DARONE_BRAY_TILE_DIR="$(mktemp -d)"
JULIA_NUM_THREADS=4 bash paper/fig4c/run_bray_curtis_tiles.sh
julia --project=. paper/fig4c/plot_full_bray_curtis.jl \
  "$DARONE_BRAY_TILE_DIR" "$HOME/darone-fig4c"
```

The result is `~/darone-fig4c/Fig_Bray_Curtis.png` and its NetCDF field.
The experiment averages the final two annual outputs and compares 45
non-diazotroph biomass tracers (21-29 and 35-70) against the Darwin reference.
The paper workflow runs independently of the routine GitHub Actions tests.

## Bundled reference inputs

CI and the example integration tests use the compact reference fields in
[`data/darwin/`](data/darwin/). The test setup maps their descriptive filenames
to the historical `3d.nc`, `tave.nc`, and `par.nc` names expected by the examples.
The case-study scripts therefore retain their original input conventions.
