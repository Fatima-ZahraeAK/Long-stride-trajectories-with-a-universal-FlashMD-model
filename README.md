# Long-stride FlashMD trajectories: a mean-force test of the sampled distribution

Fatima-Zahrae Akhyar

## The question

[FlashMD](https://atomistic-cookbook.org/examples/flashmd/flashmd-demo.html) predicts molecular dynamics 16 to 64 fs
ahead in a single step. The learned map is not exactly symplectic or time-reversible, so a thermostatted FlashMD run
is not guaranteed to sample the Boltzmann distribution of the MLIP it was trained on. This notebook checks whether it
does, on alanine dipeptide in vacuum at 400 K.

## The idea

For a collective variable ξ (here the dihedral ψ), one trajectory gives the free energy F(ξ) in two independent ways:

1. from the **histogram** of ξ;
2. by integrating the **mean force**, the average of the MLIP forces projected on ξ (Eq. (21) of
   [arXiv:2512.17374](https://arxiv.org/abs/2512.17374) with k = 1).

The two agree only if the frames follow the correct Boltzmann distribution. When they disagree, the difference is the
error of the sampler projected on ξ, level set by level set. No reference simulation is needed: a short 0.5 fs MLIP
run is used only to measure the statistical noise of the test.

## Results

| run | length | gap F_hist vs F_mf (kJ/mol) | ⟨V⟩ − ⟨V⟩_control (kJ/mol) |
|---|---|---|---|
| MLIP, 0.5 fs (control) | 40 ps | 0.30 ± 0.24 | 0 |
| FlashMD, 16 fs | 96 ps | 0.33 ± 0.09 (+0.1σ) | +4.7 ± 1.6 (3.0σ) |
| FlashMD, 64 fs | 71 ps analysed (bond broke at 81 ps) | 14.9 ± 1.4 (10.4σ) | +25.0 ± 2.3 (10.7σ) |

- **16 fs:** no error detected along ψ. The test would have detected an excess gap larger than about 0.5 kJ/mol. The
  mean MLIP energy is nevertheless shifted by 3σ (10 blocks),it's an error the ψ test cannot see.
- **64 fs:** The local error is roughly constant at about +20 kJ/mol/rad along ψ. The potential energy
  and the hydrogen temperature (412 K) confirm it independently, and the run broke a bond.

Uncertainties are block-bootstrap standard errors over 10 blocks of consecutive frames.

## How to run it

1. Open `flashmd_mean_force_test.ipynb` in Google Colab and select a GPU (*Runtime → Change runtime type → T4 GPU*).
2. Run all cells. Trajectories are saved to Google Drive

| setting | what it does | time on a T4 |
|---|---|---|
| `MODE = "full"` | reproduces the results above | about 1.5 to 5 h from scratch (mostly the 0.5 fs run); minutes if the trajectories are already on Drive |
| `MODE = "quick"` | checks that the code runs; numbers are not meaningful | a few minutes |

Outside Colab, install `flashmd>=0.3.0,<0.4` and `chemiscope>=1.1,<2`; files are then written to `tmp/`.
Tested with flashmd 0.3.0, torch 2.14.1 and ase 3.29.0.

## Limitations

- The test only sees errors that change the free energy along ψ; the energy and temperature checks are reported
  alongside for this reason. Dynamical properties (rates, correlation times) are not tested.
- One molecule, in vacuum, one temperature, one pretrained model (PET-OMATPES-v2).

## Reference

F.-Z. Akhyar, W. Zhang, G. Stoltz, C. Schütte, *Generative modeling of conditional probability distributions on the
level-sets of collective variables*, [arXiv:2512.17374](https://arxiv.org/abs/2512.17374).
