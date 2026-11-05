# Compact Darwin reference inputs

These files contain a losslessly compressed subset of the original Darwin
reference fields from the historical `birdy1123/dar1` Docker image, digest
`sha256:9eb6f57a9647e3667deb2c1670ef6426876b72da6b11808a604d8771662389b3`.

| File | Contents |
|---|---|
| `surface_tracers.nc` | `TRAC01`-`TRAC70`, surface, first time record |
| `surface_temperature.nc` | `Ttave`, surface, first time record |
| `par_upper4.nc` | `Ed001`-`Ed013`, `Es001`-`Es013`, first four levels, first time record |

The original 360 x 160 grid, zero-valued masked locations, numerical types,
and values are preserved. The one-time extraction verified equality with the
source files. The compact reference inputs support the example tests and the
full 16-tile Bray-Curtis reproduction. Their content is versioned by Git.
