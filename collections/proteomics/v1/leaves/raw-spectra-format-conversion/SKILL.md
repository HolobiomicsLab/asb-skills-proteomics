---
name: raw-spectra-format-conversion
description: Use when the input spectra are vendor Thermo RAW files (spectra_type='raw')
  and downstream Oktoberfest rescoring or spectral library generation requires open
  formats such as mzML or HDF5.
when_to_use_negative:
- Input spectra are already in mzML or HDF5 format, so re-conversion would only add
  runtime (~40% of total pipeline time)
- The goal is downstream analysis outside the Oktoberfest/Prosit ecosystem (e.g. DIA
  search) that does not consume Oktoberfest-converted spectra
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Provides pp.convert_raw_to_mzml and pp.convert_d_to_hdf for vendor RAW to
    mzML/HDF conversion within the rescoring pipeline
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: spectrum_io
  role: File/data conversion package for mass spec data within the Oktoberfest ecosystem,
    backing the RAW-to-mzML step
  repo: https://github.com/wilhelm-lab/spectrum_io
- name: Python
  role: Runtime environment in which the Oktoberfest conversion functions are invoked
    and timed (Python 3.10+)
  repo: https://pypi.org/project/oktoberfest/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:d798d7aca4d2f06c915d1d01bbed2eb4e61407995677faeebecda8cda4d92238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Oktoberfest
  - Python
  - mokapot
  - spectrum_io
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# raw-spectra-format-conversion

## Summary

Converts vendor Thermo RAW mass-spectrometry files into open formats (mzML, and HDF5 where applicable) as the first step of the Oktoberfest rescoring pipeline, using pp.convert_raw_to_mzml and pp.convert_d_to_hdf, with elapsed time recorded for benchmarking.

## When to use

Use when the input spectra are vendor Thermo RAW files (spectra_type='raw') and downstream Oktoberfest rescoring or spectral library generation requires open formats such as mzML or HDF5.

## When NOT to use

- Input spectra are already in mzML or HDF5 format, so re-conversion would only add runtime (~40% of total pipeline time)
- The goal is downstream analysis outside the Oktoberfest/Prosit ecosystem (e.g. DIA search) that does not consume Oktoberfest-converted spectra

## Inputs

- Thermo RAW mass-spectrometry files (spectra_type='raw') from HLA immunopeptidomics runs
- Oktoberfest configuration file with 'Rescoring' job type and paths to search results and spectra in the inputs block

## Outputs

- mzML files produced by pp.convert_raw_to_mzml
- HDF5 (HDF) spectra files produced by pp.convert_d_to_hdf where applicable
- Recorded elapsed time for the conversion step (e.g. entries in rescoring_runtimes.csv)

## How to apply

In the Oktoberfest configuration file set the job type to 'Rescoring' (or 'SpectralLibraryGeneration') and specify the HLA search results and spectra paths in the inputs block with spectra_type set to 'raw'. Call Oktoberfest's pp.convert_raw_to_mzml to convert the Thermo RAW spectra to mzML, and apply pp.convert_d_to_hdf where applicable, wrapping each call with a timer to record elapsed time for the conversion step, since file conversion accounts for roughly 40% of total rescoring runtime. The conversion feeds the converted spectra into peak annotation and feature generation (re.generate_features) against Prosit predictions from a Koina server, so format fidelity and complete conversion of all raw files are essential. When benchmarking scaling with PSM counts (Figure 4-style evaluation), repeat the conversion for each of the six HLA raw-file combinations so per-step runtimes can be compared.

## Related tools

- **Oktoberfest** (Provides pp.convert_raw_to_mzml and pp.convert_d_to_hdf for vendor RAW to mzML/HDF conversion within the rescoring pipeline) — https://github.com/wilhelm-lab/oktoberfest
- **spectrum_io** (File/data conversion package for mass spec data within the Oktoberfest ecosystem, backing the RAW-to-mzML step) — https://github.com/wilhelm-lab/spectrum_io
- **Python** (Runtime environment in which the Oktoberfest conversion functions are invoked and timed (Python 3.10+)) — https://pypi.org/project/oktoberfest/

## Examples

```
import oktoberfest as pp; t0=time.time(); pp.convert_raw_to_mzml(raw_path='file.raw', mzml_path='file.mzml'); print('conversion', time.time()-t0)
```

## Evaluation signals

- Every input RAW file yields a corresponding mzML (and, where applicable, HDF5) output with no conversion errors before downstream pp.convert_search / re.generate_features steps run
- Recorded conversion elapsed time is roughly 40% of the total rescoring run time, matching the reported benchmark
- In a multi-run benchmark (six HLA raw-file combinations), conversion time grows consistently with the number/size of RAW files, supporting linear scaling of total runtime with PSM counts
- Subsequent peak annotation uses the default fragment mass tolerances (20 FTMS / 40 TOF / 0.35 ITMS) successfully on the converted spectra, indicating correct conversion of peak data

## Limitations

- Conversion is time-consuming: file conversion accounts for roughly 40% of total rescoring pipeline runtime, so it dominates wall-clock benchmarks
- pp.convert_d_to_hdf applies only 'where applicable' — not all instrument/vendor formats require or support the HDF conversion path
- Vendor RAW input requires spectra_type='raw' to be set in the configuration; mis-specifying spectra_type will prevent correct conversion

## Evidence

- [methods] pp.convert_raw_to_mzml / pp.convert_d_to_hdf / pp.load_spectra: "pp.convert_raw_to_mzml
   pp.convert_d_to_hdf
   pp.load_spectra"
- [other] spectra_type raw in inputs block: [evidence span withheld — anchor status: not_found]
- [other] conversion ~40% of total time: [evidence span withheld — anchor status: not_found]
- [readme] spectrum_io role: [evidence span withheld — anchor status: not_found]
- [methods] peak annotation tolerance: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
