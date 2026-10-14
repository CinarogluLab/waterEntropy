# waterEntropy (CinarogluLab fork)

<p align="center">
<img src="docs/source/images/waterentropy_logo_grey.png" width="500px" />
</p>

This is a fork of [jkalayan/waterEntropy](https://github.com/jkalayan/waterEntropy), maintained by the Çınaroğlu
lab. It is based on waterEntropy **2.2.0** and is the version used by the
[CinarogluLab fork of CodeEntropy](https://github.com/CinarogluLab/CodeEntropy).

waterEntropy calculates the entropy of water molecules from molecular dynamics trajectories, in particular of
the waters in the first hydration shell around a solute such as a protein or a ligand. For each water it
identifies its neighbours with the relative angular distance (RAD) method, assigns hydrogen bonds, and computes
vibrational (translational and rotational) and orientational entropy.

## How it differs from the original

### Much faster, with identical results

Almost all of the original run time was spent on bookkeeping rather than on the entropy calculation itself:
copying the full coordinate array of the system for every neighbour checked, looking up bonds and atom
properties through slow selection queries inside the innermost loops, and testing neighbour pairs one at a
time. These are now done once per frame or vectorised, and the results stay exactly the same. For a solvated
protein system, a frame that previously took many minutes now takes about a second.

An optional faster neighbour search is available with `CODEENTROPY_FAST_NEIGHBOURS=1`. It gives identical
results; it was tested on many random systems, including non-rectangular boxes.

### One correction

- **Orientational entropy averaging.** When orientational entropies were averaged over different neighbour
  environments, part of the average used the wrong count, so some environments carried the wrong weight. In
  pure water this made the orientational entropy too low. It is now a correct population-weighted average. A
  check of the fix: in pure water, the hydrogen-bond-based and the simple orientational models now agree
  exactly, as they must. Upstream's development version contains an equivalent fix, not yet released.

### Robustness

- The Dask worker pool no longer restarts after every small batch of frames, which could stall long runs.

## Installation

```bash
pip install "waterEntropy @ git+https://github.com/CinarogluLab/waterEntropy"
```

To use it with CodeEntropy, install this package first and then the
[CodeEntropy fork](https://github.com/CinarogluLab/CodeEntropy). That stops pip from fetching the original
waterEntropy from PyPI. 

## Usage

Usage is the same as the original; see the
[waterEntropy documentation](https://waterentropy.readthedocs.io/en/latest/). Most users will run it through
CodeEntropy with `--water_entropy True`.

## Known limitations

These were noted in a code audit and are left as in the original. Keep them in mind when interpreting results:

- **Orientational entropy is floored at zero.** The published equation can give negative values for waters with
  few neighbours, as happens at protein surfaces, but the code sets them to zero. This matters most for the
  hydrogen-bond-biased orientational model.
- **Angle calculations assume a rectangular simulation box.**
- **Each polar hydrogen is always assigned a hydrogen-bond partner** (the nearest neighbour if none meets the
  criteria). In our tests this affected under 1% of assignments.

## Credit and licence

waterEntropy is developed by Jas Kalayan and contributors. Please cite the original package and the methods it
implements when using this fork, and mention the fork and its version.

Distributed under the same licence as the original (see `LICENSE`). Report problems with this fork in its
[issue tracker](https://github.com/CinarogluLab/waterEntropy/issues).
