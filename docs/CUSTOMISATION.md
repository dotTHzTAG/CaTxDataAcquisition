# Adapting Data Acquisition to another THz system

## Direction and current boundary

Reuse the dotTHz project workflow, metadata editor, and data viewer while replacing instrument-specific acquisition. AI coding assistants can help when supplied with the device API and experimental requirements. They cannot infer correct timing, units, or the physical acquisition procedure from a device name alone.

Today, only Menlo ScanControl is implemented. There is no backend registry, generic protocol, runtime device selector, or simulator. This is a modification guide, not a claim those features already exist. A spectrometer profile only changes recorded metadata.

## Code map

The manual's [visual window guide](USER_MANUAL.md#visual-guide-to-the-windows) and [worked example](USER_MANUAL.md#worked-acquisition-example) connect the existing controls to saved measurement views. Use them to identify UI labels and instructions affected by an adaptation. Keep the referenced `Images/` screenshots alongside the guides in releases.

| File | Responsibility and adaptation point |
| --- | --- |
| `catx/core/application.py` | Constructs one service shared by the windows; select/inject a backend here. |
| `catx/services/acquisition.py` | Validates plans, consumes waveforms, stores samples, and sends calibration/progress callbacks. Currently imports Menlo types and defaults its `menlo` field to a Menlo client. |
| `catx/menlo/client.py` | Menlo status, connection loop, averaging, timing, pulse decoding, and cleanup. Keep vendor details here. |
| `scancontrolclient.py`, `pywebchannel/` | Existing ScanControl transport; not a generic device interface. |
| `catx/models/acquisition.py` | AcquisitionPlan, Waveform, AcquisitionProgress, and enums. |
| `catx/repositories/project.py` | HDF5 writes. Keep hardware calls out of this layer. |
| `ui/acquisition_main.py`, `.ui` | Connection/acquisition workers, session calibration, metadata, controls, and Menlo messages. |
| `ui/data_manager.py`, `.ui` | HDF5 browsing and plotting; assumes time in ps for THz FFT axes. |
| `ui/profile_manager.py` | User/instrument descriptions; does not configure hardware. |

## Information to supply before coding

Record the instrument model or home-built components, OS, SDK/driver versions, transport and endpoint, and a minimal working acquisition example. Include API documentation for connect/status/start/read/stop/disconnect, a real waveform with dimensions and units, and timeout/error behaviour.

Specify the waveform channel, time-axis direction and units, amplitude scaling, hardware versus software averaging, scan count/rate meaning, and how to identify fresh data. For a mechanical delay line, supply its calibrated displacement-to-time relationship and acquisition sequence. Describe reference/baseline procedures and whether pause stops hardware or delays collection. Keep credentials and lab-specific endpoints outside committed source.

## Recommended implementation sequence

1. Capture existing Menlo behaviour and representative `.thz` output. Add a target backend in a separate module such as `catx/backends/my_system.py`.
2. Introduce a neutral status type and backend interface. The current effective contract is `async status()` returning an object with `name` (Menlo also has integer `code`), and `acquire(plan, stop_event, pause_event)` returning an async iterator of `Waveform`. Rename the service's `menlo` dependency to a neutral name and update annotations/callers together. Keep Menlo as default and inject the chosen backend in `create_main_window()`.
3. Preserve that contract in a Menlo wrapper or the existing implementation. Do not reuse Menlo status integers for unrelated states. Define readiness and unsupported operations. Add connection settings/selection only as needed; no selector exists today.
4. Implement connection, acquisition, validation, and cleanup. The adapter currently owns count/time termination, averaging, and intervals; the service validates and consumes its output. Yield one complete averaged waveform per measurement.
5. Update Menlo-specific connection worker names, messages, and control labels for the new backend. Disable or explain unavailable features. Transmission/Reflection is metadata, not a hardware command.
6. Validate storage/plotting with synthetic input and then actual hardware. Update documentation and About support claims with the integration and test results.

A small interface plus explicit construction is enough for one adaptation; a plugin framework is optional future work. If retaining `menlo=` injection temporarily, describe it as a transitional vendor-named field, not a completed neutral interface.

## Waveform and execution contract

- Produce independent one-dimensional numeric arrays of equal nonzero length for `time_axis` and `amplitude`; copy SDK buffers that can be reused. Validate finite data. Convert time to **picoseconds** for the current viewer. FFT display needs at least two samples on an increasing, uniformly spaced axis. Reject or explicitly resample nonuniform data with documented provenance.
- Use a timezone-aware acquisition timestamp, preferably UTC. If provided, `rate` means raw scans per second for the existing estimate (`duration * rate / average`). Use `None` when that meaning cannot be supplied.
- `scancontrol_timestamp` and `pulse_flags` are legacy Menlo fields. Leave the timestamp `None` and flags `0` for other devices unless a documented compatible meaning exists. Use explicit names for distinct vendor metadata.
- Match calibration/sample axes and amplitude scaling. Storage does not align or convert calibration. Reacquire calibration after changing the time window or instrument settings.
- Qt workers run service coroutines outside the UI thread. Deliver progress/calibration through Qt signals. Keep blocking SDK reads off the GUI/event loop, respect SDK thread affinity, and serialize device access.
- Poll stop requests during reads/waits with bounded timeouts. Use `finally` cleanup to stop acquisition started by the adapter, detach callbacks, and release resources. Propagate actionable errors. Define close/reconnect handling when introducing the interface; the current service has no generic disconnect method.
- Match or document count/time and interval semantics. Exclusive interval waits after a measurement; inclusive interval targets spacing between starts without extra waiting when acquisition exceeds the interval. Current Menlo elapsed time includes pauses/waits; deadlines are not hard real-time cutoffs.
- Current pause delays collection/reset progression and does not send a hardware pause. Document any difference in another backend, avoid stale queued frames on resume/stop, and never silently ignore unsupported options.

## Verification before claiming support

Use a deterministic fake backend for development and label synthetic data. Check known axes/amplitudes through the service into a temporary project, then reopen with h5py and Data Manager. Exercise single scan, count/time modes, averaging, both intervals, pause/resume/stop, disconnect, timeout, and cleanup after failure. Check fresh-frame detection, UI responsiveness, metadata, and Menlo defaults after shared changes.

On actual hardware, compare waveform/time/amplitude scaling against the vendor tool or a known procedure. Check averaging and measured timing, stop during a read, reconnection, and calibration axes. Record device/SDK/OS versions, settings, results, and limitations. Validate files with the intended downstream dotTHz consumer; [DATA_FORMAT.md](DATA_FORMAT.md) describes this application's layout.

## Prompt to hand to an AI coding assistant

```text
Adapt dotTHz Data Acquisition for [system/components].
Read AGENTS.md and docs/CUSTOMISATION.md, DATA_FORMAT.md, USER_MANUAL.md.
The supplied live backend is Menlo only. Preserve its default operation.
Target OS / SDK / transport: [...]
API documentation and working acquisition example: [...]
Example waveform, channel, time units, amplitude units, averaging: [...]
Required count/time, interval, pause, stop and calibration behaviour: [...]
Implement a small neutral backend boundary and inject the target backend.
Preserve the .thz layout; explain necessary compatibility changes.
Ask about missing device facts rather than inventing APIs or conversions.
Validate with deterministic synthetic input; report hardware tests separately.
Update installation/user docs and About for the actual supported behaviour.
```

## Release handoff

Keep documentation beside this application when placing it in the dotTHz GitHub repository; relative links do not depend on a repository URL. Include guides in the release archive. Review the bundled profile database for personal/lab data and distribute shareable example records. Preserve applicable license notices. Update `version.json` and `pyproject.toml` together when assigning versions; a documentation edit does not require an invented release date.
