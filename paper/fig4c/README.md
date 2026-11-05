# Figure 4C - Bray-Curtis dissimilarity

Reproduce Figure 4C of the DAR1 manuscript from 16 independent 90 x 40 DAR1
simulations initialized with the bundled Darwin annual-mean reference fields.
The model runs for ten years (28,800 iterations per tile), and the plot compares
the mean of the last two annual outputs with the reference community. As in the
original analysis, the comparison uses 45 non-diazotroph biomass tracers
(21-29 and 35-70).

## Run from the repository root

Use a Darwin3 checkout with `verification/dar_one_config` and the standard
DarOne local build dependencies. The pinned source revisions are recorded in
[`darwin-sources.env`](../../darwin-sources.env).

Create the plotting environment once:

```bash
conda env create -f paper/fig4c/environment.yml
```

Configure the Darwin checkout and link the bundled reference fields using the
original filenames expected by the case-study scripts:

```bash
cd ~/Dar_One
export DARONE_MITGCM_PATH=/path/to/darwin3
export DARONE_DATA_DIR="$(mktemp -d)"
ln -s "$PWD/data/darwin/surface_tracers.nc" "$DARONE_DATA_DIR/3d.nc"
ln -s "$PWD/data/darwin/surface_temperature.nc" "$DARONE_DATA_DIR/tave.nc"
ln -s "$PWD/data/darwin/par_upper4.nc" "$DARONE_DATA_DIR/par.nc"
```

Build, run the 16 tiles, and generate Figure 4C:

```bash
julia --project=. case_studies/global_steadystate_build.jl
JULIA_NUM_THREADS=4 bash paper/fig4c/run_bray_curtis_tiles.sh "$HOME/darone-fig4c"
conda run -n darone-fig4c python paper/fig4c/plot_bray_curtis.py "$HOME/darone-fig4c"
```

The launcher uses up to `JULIA_NUM_THREADS` concurrent single-threaded model
processes. To choose a different concurrency, set `DARONE_GLOBAL_WORKERS`.

## Outputs

The analysis writes `Fig_Bray_Curtis.png` and `Fig_Bray_Curtis.nc` to
`~/darone-fig4c/`; per-tile simulation logs are under `~/darone-fig4c/logs/`.
The paper reproduction is independent of routine GitHub Actions tests.
