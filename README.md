[English](README.md) | [简体中文](README.zh-CN.md)

# STFT-Based Melody Extraction and Chord Recognition

A MATLAB project for the **Signal Analysis and Processing** course at Zhejiang University. It implements an audio-to-musical-structure pipeline using short-time Fourier analysis, multi-pitch tracking and chord-template matching.

## Processing pipeline

### Audio preprocessing

Normalize amplitude, then apply a fourth-order Chebyshev Type I band-pass filter with 1 dB ripple and a nominal 20–2000 Hz passband. Butterworth filtering is also supported. Zero-phase `filtfilt` is used. Frames have 2048 samples, approximately 42.7 ms at 48 kHz, 70% overlap and a periodic Hann window.

```matlab
processed_audio = input_audio / max(abs(input_audio));
Wn = cutoff_freqs / (fs/2);
Wn = max(min(Wn, 0.99), 0.01);
[b, a] = cheby1(4, 1, Wn, 'bandpass');
filtered_audio = filtfilt(b, a, audio);
window = hann(window_length, 'periodic');
```

### Multi-pitch extraction

Compute the STFT magnitude spectrum and select up to four strongest spectral peaks per frame with `findpeaks`. Each peak's confidence is normalized by that frame's maximum spectral magnitude. Frame-energy plots help locate active passages; the highest-energy frame's spectrum highlights the 200–800 Hz region for inspection.

### Chord recognition

The reference set contains 84 chords: twelve roots and seven types. Frequencies are mapped to MIDI notes with equal temperament, using `f = 440 * 2^((m - 69)/12)`. The documented piano mapping spans C4–G5 (MIDI 60–79).

| Type | Semitone intervals | Minimum notes |
| --- | --- | ---: |
| Major | 0, 4, 7 | 3 |
| Minor | 0, 3, 7 | 3 |
| Augmented | 0, 4, 8 | 3 |
| Diminished | 0, 3, 6 | 3 |
| Dominant seventh | 0, 4, 7, 10 | 4 |
| Major seventh | 0, 4, 7, 11 | 4 |
| Minor seventh | 0, 3, 7, 10 | 4 |

Detected notes are deduplicated and their confidences combined. Candidate roots and interval templates are scored to choose a chord. An isolated frame that disagrees with both neighbors is marked unknown; endpoint frames must agree with their adjacent frame.

Potential uses discussed in the report include music-education analysis, accompaniment and composition assistance, and harmony-based audio retrieval.

## Files and outputs

| Path | Purpose |
| --- | --- |
| `Code/main.m` | Main workflow |
| `Code/preprocessing.m` | Normalization, filtering and framing |
| `Code/melody_extraction.m` | Multi-pitch and frame-energy analysis |
| `Code/chord_recognition.m` | Chord matching and postprocessing |
| `Code/generate_visualizations.m` | Analysis figures |
| `Code/GUI.htm` | Results interface |
| `Report/Project_Report.pdf` | Technical report |
| `Documentation/` | Reference material |
| `Test/Sources/Chords/` | 84 synthetic chord samples and MuseScore/PDF scores |
| `Test/Sources/Songs/` | Song-based course evaluation materials; restrictions below |

Outputs include `multi_pitch.csv`, `chords.csv`, `multi_pitch_visualization.png`, `chord_recognition.png`, `frame_energy.png` and `max_energy_frame_spectrum.png`.

## Run and debug

The project was developed with **MATLAB R2024a** and **MuseScore 3**. Signal Processing Toolbox and Audio Toolbox are required:

```matlab
license('test', 'signal_toolbox')
license('test', 'audio_toolbox')
```

Run `Code/main.m` with the intended audio input configured. To inspect preprocessing, plot a spectrogram of the processed signal. Inspect `confidences` and `multi_pitch` for pitch estimation, `chord_conf` for matching scores, and the generated chord-duration visualization for temporal consistency.

## Contributors

| Contributor | Work |
| --- | --- |
| Cavalcdor | Coordination, core algorithms, piano-frequency reference and MuseScore inputs/debugging |
| Atopos | Technical report, experimental data analysis and algorithm testing |
| luvwy | Visualization and GUI design |
| ing | Technical plan, presentation slides and demonstration materials |

## Copyright, licensing and evaluation data

This course assignment was independently completed by four 2023-entry undergraduates in Zhejiang University's College of Control Science and Engineering. Code and documentation belong to the team. The original documentation identifies the files under `Code/` as MIT-licensed; see [LICENSE](LICENSE) for the license text. It also states academic/noncommercial-use conditions, preservation of notices, disclosure of modifications and no misrepresentation of authorship. This documentation translation does not change those statements or the repository's license.

Technical reports and other documents are outside the stated open-source scope; they are provided for reference and should not be copied or redistributed without permission.

The song recordings and chord annotations under `Test/Sources/Songs/` are **unofficial, manually interpreted coursework evaluation data**, not authoritative scores. Original music rights belong to the respective authors and record labels, and the project states that it has not obtained authorization for those works. The annotations may differ from the original harmony and must not be distributed as official scores or used for teaching or commercial arranging.

The original usage restriction limits those materials to internal academic submission and classroom demonstrations, prohibiting public distribution, open-source release and profit-making use. Users are responsible for compliance with applicable copyright rules. Use self-owned or public-domain audio when testing independently.

**Team:** Cavalcdor, Atopos, luvwy, ing. **Original statement date:** 2025-06-01.
