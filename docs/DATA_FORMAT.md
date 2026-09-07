# Data layout used by this application

This describes `catx/repositories/project.py`, not the complete dotTHz specification or a guarantee that arbitrary dotTHz files are supported. Validate changes with the intended downstream dotTHz software.

Projects have a `.thz` extension and are HDF5 files. New files receive root attributes `format="dotTHz"` and `created_utc`. `project_metadata` is JSON containing saved fields such as user, spectrometer, sample, description, and mode. `numbering_widths` is internal JSON bookkeeping.

Each sample measurement creates a top-level group from the sanitised sample name and numeric suffix, for example `sample_01`. Blank names use `measurement`. Padding depends on estimated count and existing suffixes; enumerate groups instead of assuming a width.

| Dataset | Contents | Presence |
| --- | --- | --- |
| `ds1` | Sample waveform | Always for a sample measurement |
| `ds2` | Session reference | When available at run start |
| `ds3` | Session baseline | When available at run start |

Each is a `2 x N` array: row 0 time, row 1 amplitude. The viewer labels time in **ps**, amplitude in arbitrary units, and FFT frequency in **THz**. The writer does not convert units or validate spacing; adapters must supply suitable data. Spectrum uses real FFT magnitude and mean time spacing without windowing, normalization, or sample/reference division. It assumes uniform increasing time samples. Baseline subtraction changes the display only, on matching axes.

Measurement attributes include `dsDescription` (comma-separated available labels), `time` (capture ISO timestamp), optional `rate`, optional `scancontrol_timestamp`, and `pulse_flags`. The UI adds `sample`, `description`, `mode`, JSON-encoded `user`, `spectrometer` and `instrument`, `scan_average`, `interval_seconds`, and `interval_inclusive`. Numeric metadata uses `md1` through `md7` and `mdDescription`. Viewer rows such as `thzVer` and `coordinate` are not automatically populated by this writer.

`ds3` can exist without `ds2`; do not assume sequential numbering. The viewer currently assigns description labels by sorted dataset position. Check sparse and externally authored layouts before claiming broad compatibility.

Calibration acquisition updates memory only. Subsequent sample runs store copies of calibration available at run start. Opening/creating a project clears session calibration; existing datasets are not restored into that session. Removing session calibration affects future samples only.

Samples are appended and flushed immediately. Project Save updates selected project metadata; Save As copies the on-disk file. Data Manager Update writes edited attributes; Remove deletes a measurement group. Reset Data recreates the file without measurement groups, project metadata, or numbering bookkeeping, preserving other root attributes. These operations have no undo history.
