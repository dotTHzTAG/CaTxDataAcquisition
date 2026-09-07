# dotTHz Data Acquisition user manual

## Choose your workflow

Data Acquisition collects THz time-domain waveforms into `.thz` HDF5 projects. The supplied live implementation is Menlo Systems ScanControl. Other commercial or home-built systems require [customisation](CUSTOMISATION.md); adding a spectrometer profile does not add device support.

Data Manager works on compatible existing projects without ScanControl or connected hardware. Follow [the installation guide](../Installation_guide.txt) to install or extract the application.

## Projects and profiles

1. Choose **Project > New** and a writable `.thz` location, or **Project > Open**. Creating at an existing path replaces the file when overwrite is accepted.
2. Profile Manager opens on creation. Select/add a user and spectrometer and click **Apply**. Reopen it with **Profile > Profile Manager** or **User Manager**. Exiting without Apply leaves the acquisition selection unchanged.
3. In Profile Manager, use **New**, enter details, then **Add/Update** to save the record. **Remove** deletes a record; **Load** selects an existing HDF5 profile database. Database edits happen independently of Apply.
4. Enter **Sample**, **Description**, and **Mode** (Transmission/Reflection). Mode records metadata, not hardware configuration.
5. Use **Add Row** for up to seven numeric metadata values. Select Sample/Reference, a category and unit, and a value. `mdDescription` is generated from these selections. **Remove Row** reduces active rows; **Reset** clears the metadata table.

The default database is `profile_database.h5` beside the app. Selected profile records are copied into project/measurement metadata. Profiles do not contain driver settings. Review their content before sharing a project.

## Connect Menlo ScanControl

Start ScanControl and enable its remote interface at `localhost:8002`, the default. Click **ScanControl Connect**; after connection this becomes **Refresh Status**. Leave ScanControl **Idle** before acquisition. Configure the instrument itself in ScanControl.

There is no GUI host/port editor. Source customisation can configure `MenloScanControlClient(host=..., port=...)` in application construction. See the customisation guide for other systems.

## Capture calibration

Prepare the baseline or reference required by your experiment, set **Scan Average** under **Reference Acquisition**, and click the corresponding **Acquire**. Each captures one averaged waveform.

Baseline/reference are held in session memory and copied into subsequent sample measurements when available. The displayed **Store local reference and baseline** control currently has no effect on this behaviour. Calibration is not saved as a standalone measurement or restored into memory when reopening a project. Create/Open, Reset Data, and application restart clear session calibration. **Remove** beside calibration clears only its in-memory waveform.

Samples can be acquired without calibration. Reacquire calibration after changing scan window, time grid, or amplitude scaling. The application does not automatically align axes.

## Acquire samples

- Set **Scan Average** under **Sample Acquisition**. **Single Scan** captures one averaged sample.
- For **Multi Scan**, choose **Count** and **Total Count**, or **Time** and **Total Time** with its unit.
- **Interval Time** adds a wait after each measurement. **Scan Time Inclusive** instead targets spacing between measurement starts, with no extra wait when a scan takes longer.
- **Pause** becomes **Resume**. In the supplied backend it delays collection/reset progression and does not send a hardware pause. Time mode includes pauses and waits and may finish after the requested duration while a wait or measurement completes.
- **Stop** requests termination and cleanup. Already stored measurements remain. Current Index counts completed waveforms in the run; Scan Count shows project measurement groups. Time Left is an estimate.

Wait for acquisition to finish or stop before switching projects, copying files, or editing/removing measurements. Calibration used by a run is a snapshot taken at its start.

## Saving and inspecting data

Each completed sample is written immediately. **Project > Save** saves current sample name, description, and mode as project metadata; profiles are saved on Apply. Save does not persist every control setting or session calibration. **Save As** copies the on-disk file and switches to that copy; use Save first to include recently edited project fields.

Open **Data Manager** and select a measurement in the HDF5 tree. Under **Attributes**, edit enabled fields and click **Update** to commit. Under **Datasets**, choose **Waveform** or **Spectrum** and its frequency range. **Subtract Baseline** changes plotted sample/reference when axes match, leaving stored data intact. Spectrum is FFT magnitude, not transmission, absorbance, or a fitted material property.

**Reload** rereads the project; commit pending edits first. **Remove** deletes a measurement after confirmation. In the acquisition window, **Reset Data** deletes all measurements and clears saved project metadata and session calibration after confirmation. There is no undo; preserve a separate copy when needed. See [DATA_FORMAT.md](DATA_FORMAT.md) for storage behaviour and compatibility limits.

## Logs, About, and troubleshooting

**Show Logs** displays connection/acquisition diagnostics. **Dump Logs** clears `data_acquisition.log` beside the app; retain needed diagnostics first. **Help > About Data Acquisition** shows dotTHz identity, supplied hardware support, documentation paths, and release information.

| Symptom | What to check |
| --- | --- |
| Cannot connect | Start ScanControl; check its remote interface, host/port, and Show Logs. |
| Device must be Idle | Stop an existing scan in ScanControl, Refresh Status, and retry. |
| Acquisition controls disabled | Connect successfully; create/open a project before acquiring. |
| Calibration disappeared | Reacquire after creating/reopening a project; it is session memory. |
| Cannot subtract baseline | Use calibration with the same sample time axis and settings. |
| Empty or misleading spectrum | Check time units (ps), at least two samples, uniform increasing spacing, and frequency range. |
| Cannot save or edit profiles | Use writable directories; avoid another process writing the same HDF5 file. |
| Startup/import failure | Use the environment where requirements were installed and Python 3.9 or newer. |
| No plot area | Install requirements including Matplotlib and restart. |

Report issues in the hosting dotTHz repository with About version/build, OS, backend/hardware, reproduction steps, and a relevant log excerpt. For data issues, include a small shareable file and identify hardware versus synthetic data.
