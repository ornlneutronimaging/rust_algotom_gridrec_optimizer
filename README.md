# AlgoTom GridRec Optimizer

Standalone GUI to tune AlgoTom GridRec reconstruction parameters on a
pre-processed CT checkpoint (the HDF5 written by `rust_ct_reconstruction`:
attenuation data with `/angles_rad` and `/center_of_rotation`). Uses
algotom's `gridrec_reconstruction` (a wrapper of tomopy's gridrec), like
the pipeline's white-beam CLI.

## Workflow

1. Open a checkpoint (command-line argument or the 📂 button).
2. Pick two slices on the projection view (red and cyan lines).
3. Adjust the parameters — the **filter** (none, shepp, cosine, hann,
   hamming, ramlak, parzen, butterworth) in the open section; the circle
   mask ratio, FFT padding, butterworth cutoff and the center of rotation
   behind the password-locked **Advanced** section.
4. **▶ Evaluate** reconstructs the two selected slices through the real
   `algotom` (from the `all_ct_reconstruction_development` pixi
   environment) and shows them side by side — gridrec reconstructs each
   sinogram row independently and is very fast. Every run lands in the
   **Run history** with slice thumbnails (hover to enlarge); its `use`
   buttons restore the parameters of a previous run.
5. **💾 Save** writes `algotom_gridrec_config` (JSON) into the
   checkpoint's `/metadata`; `rust_ct_reconstruction` restores it
   automatically and later GridRec reconstructions use these parameters.

Defaults follow the pipeline: `shepp` filter, padding 100, mask ratio
1.0, center seeded from the checkpoint.

## Running

```bash
./launch_algotom_gridrec_optimizer.sh [checkpoint.h5]
```

Requires a graphical session; the launch script rebuilds when sources
changed. `--called-from-app` additionally prints the saved JSON on stdout
for a driving application.
