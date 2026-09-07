# Direction for contributors and AI coding assistants

This is dotTHz Data Acquisition, originally developed for Menlo Systems ScanControl. The release direction is a reusable acquisition and dotTHz storage application that users can adapt to other commercial or home-built THz systems with AI assistance.

Read [docs/CUSTOMISATION.md](docs/CUSTOMISATION.md), [docs/DATA_FORMAT.md](docs/DATA_FORMAT.md), and [docs/USER_MANUAL.md](docs/USER_MANUAL.md) before changes affecting acquisition, data, or user workflows.

- Distinguish implemented behaviour from proposed extensions. Only Menlo ScanControl is currently supplied; there is no driver selector, plugin discovery, or simulation mode.
- Keep device SDK calls, device state, units conversion, and acquisition cleanup at a hardware boundary. Keep HDF5 writing in repositories and Qt updates in the UI thread.
- Preserve Menlo operation when adding a backend. Introduce a neutral backend contract and inject the selected implementation at application construction when that work is requested. Avoid rewriting the application to add one instrument.
- Obtain the target device's actual API documentation and representative waveforms. Do not invent SDK calls, time units, motion-to-delay conversions, averaging semantics, or hardware capabilities. Record unresolved details before implementing dependent code.
- Preserve stored dataset names, shapes, and metadata unless an explicit format change includes compatibility handling and documentation. The local data-layout guide is not the full dotTHz specification.
- Validate new adapters with deterministic synthetic input, failure/stop scenarios, file round trips, and then separately with actual hardware. Label simulated results clearly and state what remains untested.
- Update README, installation instructions, manual, customisation/data guides, and About text when affected. Include documentation in packaged releases. Do not claim support for untested systems.
- Keep the internal `catx` package and existing settings identifiers unless a requested migration handles compatibility. A displayed profile is metadata, not driver configuration.
- Do not add experimental measurements, personal profile records, logs, SDK credentials, or generated releases to source changes. Preserve existing user data and applicable license/attribution.

For documentation-only work, verify references and affected UI text; a hardware refactor is not required. The user's explicit task determines the scope.
