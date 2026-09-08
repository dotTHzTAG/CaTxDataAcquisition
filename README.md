# dotTHz Data Acquisition

Data Acquisition is part of the dotTHz project: a Python/PyQt6 desktop tool for terahertz time-domain acquisition and `.thz` HDF5 project management. Originally developed for Menlo Systems users, it is shared as a starting point for other commercial and home-built THz systems, including customisation with AI coding assistants.

**The supplied live acquisition backend supports Menlo ScanControl only.** Other systems require code changes; selecting a spectrometer profile does not select or install a driver. Project browsing and editing do not require connected hardware.

## Documentation

- [Installation](Installation_guide.txt): source setup and portable Windows releases.
- [Illustrated user manual](docs/USER_MANUAL.md): window tour, worked acquisition example, profiles, calibration, and data management.
- [Customisation guide](docs/CUSTOMISATION.md): modification direction, code map, backend requirements, and an AI handoff template.
- [Stored data layout](docs/DATA_FORMAT.md): current HDF5 layout and plotting assumptions.
- [AI contributor instructions](AGENTS.md): repository guidance for coding assistants.

## Features

[![Data Manager displaying sample, reference, and baseline waveforms](Images/data_manager_window02.png)](docs/USER_MANUAL.md#data-manager-datasets)

*Inspect saved measurements in Data Manager. Start with the [visual window guide](docs/USER_MANUAL.md#visual-guide-to-the-windows), or follow the [worked acquisition example](docs/USER_MANUAL.md#worked-acquisition-example).*

- Menlo ScanControl connection and acquisition
- Baseline, reference, and sample waveform capture
- dotTHz project storage with per-measurement HDF5 groups
- Editable measurement metadata and profile management
- HDF5 tree browsing, attribute editing, and measurement removal
- Waveform and frequency-domain dataset plotting

## Installation

Use Python 3.9 or newer in a dedicated environment, as declared in `pyproject.toml`. Windows is the current portable release target; other platforms require validation.

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## Run

```powershell
.\.venv\Scripts\python.exe main.py
```

## Build a Release

Install the build dependency once, then create a portable Windows release:

```powershell
.\.venv\Scripts\python.exe -m pip install "pyinstaller>=6"
.\.venv\Scripts\python.exe build.py
```

The output is `release/DataAcquisition.zip`. Extract the entire archive and run `DataAcquisition/DataAcquisition.exe`. The script clears `build/`, removes any old `release/DataAcquisition/` directory, and replaces the previous archive. It removes temporary `build/dist` after archiving, so no unpacked release remains.

For the supplied backend, Menlo ScanControl must be running with its remote interface available (default `localhost:8002`) and Idle before acquisition. See the manual for calibration and saving behaviour.

## Project Structure

- `catx/`: models, repositories, services, and ScanControl integration
- `ui/`: PyQt6 controllers and Designer `.ui` files
- `resources/`: application icons
- `docs/` and `Images/`: guides and screenshots; keep them together for the illustrated manual.
- Projects are saved wherever the user chooses; profiles and the application log reside in the application directory, which must be writable.

Runtime release metadata is stored in `version.json`; Python package metadata is in `pyproject.toml`. Hardware support claims should reflect the adapters actually supplied and tested.

## Publishing on GitHub

Commit the source and documentation. Generated `build/` and `release/` folders, Python caches, logs, and local `.thz` measurements are excluded by `.gitignore`. Keep the bundled profile database limited to shareable records.

After pushing the source, create a GitHub Release and attach `release/DataAcquisition.zip` as a downloadable asset. Do not commit the archive: GitHub blocks ordinary Git files larger than 100 MiB. See [GitHub's large-file guidance](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github).

If a generated archive was already committed, adding an ignore rule or deleting it in a later commit is insufficient: it must also be removed from the commits being pushed. For a file introduced in the latest unpushed commit, remove it from the index with `git rm --cached` and amend that commit, preserving the local file. Earlier commits need a separate history cleanup.

The repository includes the [GNU GPL version 3 license text](LICENSE). Preserve applicable attribution when adapting or redistributing the source.
