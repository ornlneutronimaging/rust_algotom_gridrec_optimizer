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
4. Choose the **Test data** resolution: the checkpoint's own (the
   default — gridrec is fast enough), or the projections n×n
   block-averaged first (2x2, 3x3, 4x4, 6x6 — the same block mean as the
   pre-processing rebin step), which reconstructs faster at a coarser
   resolution. This is a shortcut for the test only: the parameters are
   saved in the checkpoint's pixel units and the full reconstruction runs
   on the checkpoint as is (only the center of rotation is converted for
   the rebinned test run, following the pixel grid like the pre-processing
   rebin does: `(center + 0.5) / n - 0.5`).
5. **▶ Evaluate** reconstructs the two selected slices through the real
   `algotom` (from the `all_ct_reconstruction_development` pixi
   environment) and shows them side by side — gridrec reconstructs each
   sinogram row independently and is very fast. Every run lands in the
   **Run history** with slice thumbnails (hover to enlarge); its `use`
   buttons restore the parameters (and test resolution) of a previous run.
6. **💾 Save** writes `algotom_gridrec_config` (JSON) into the
   checkpoint's `/metadata`; `rust_ct_reconstruction` restores it
   automatically and later GridRec reconstructions use these parameters.

Defaults follow the pipeline: `shepp` filter, padding 100, mask ratio
1.0, center seeded from the checkpoint.

## Center of rotation and tilt

The projection view draws the **center of rotation**: the checkpoint's
value as a dashed orange line and, once the center is changed in the
Advanced section, the current one as a solid green line, with a readout
of both values and the move between them (`↺ checkpoint value` goes
back). Drag the center field or use the arrow keys; holding Shift moves
it 10× faster. The center is an absolute column in pixels, seeded from
the checkpoint's `/center_of_rotation` (the middle of the detector when
the checkpoint carries none).

Under the projection, **Tilt correction** lists what the checkpoint
records: the pre-processing step of `rust_ct_reconstruction`
(`tilt_correction`) and every run of the standalone tool
(`metadata/tilt_center_of_rotation`, JSON records), plus the
pre-processing rebin when there was one.

**🎯 Open the tilt & center-of-rotation tool** launches
`rust_tilt_center_of_rotation` on the checkpoint itself (the more robust
estimators: sub-pixel 0°/180° registration, all-pairs consensus, gridrec
test slices). Applying & saving there rewrites the checkpoint's
projections and center of rotation; when the tool closes, this window
detects the new correction record, reloads the file and re-seeds the
center from the new value — save the parameters to keep it. Closing the
tool without applying changes nothing. Evaluate, Save and Return are
disabled while the tool is open.

## Running

```bash
./launch_algotom_gridrec_optimizer.sh [checkpoint.h5]
```

Requires a graphical session; the launch script rebuilds when sources
changed. `--called-from-app` additionally prints the saved JSON on stdout
for a driving application.
