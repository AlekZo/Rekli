# Local model validation — October 13, 2026

The added Moonshine Tiny/Base and Russian Zipformer Small models were downloaded
through the application's installer and decoded real speech using the pinned
`sherpa-onnx==1.13.4` Windows x64 runtime. Downloads were kept in a workspace test
directory. Existing application model files supplied the Parakeet/Whisper baseline;
the user's saved selection was not changed.

After downloading the models and samples, Python `requests.get` was replaced with
a function that raises on any request during all recognition runs. All five models
returned non-empty local transcripts.

## Warm CPU timings

Hardware: **Intel Core Ultra 5 225H**, Windows 11 x64 (build 26300).

Each value is the median of three calls after a first call to load the recognizer.
The calls include WAV conversion and recognition but exclude microphone capture,
text insertion, download, and initial loading. The application uses up to four
inference threads for Parakeet and up to two for the other models. An explicit
`en`/`ru` hint was passed, so the Whisper results exclude automatic language
identification. Recognition used `provider="cpu"` throughout.

| Model | English clip: 3.845 s | Russian clip: 7.080 s |
| --- | ---: | ---: |
| Parakeet TDT 0.6B v3 INT8 | 0.457 s | 0.614 s |
| Whisper Tiny INT8 | 0.344 s | 0.546 s |
| Moonshine Tiny, February 2026 v2 export | 0.056 s | Unsupported |
| Moonshine Base, February 2026 v2 export | 0.093 s | Unsupported |
| Zipformer Small Russian INT8 | Unsupported | 0.102 s |

The English sample was [the Moonshine export's test WAV](https://huggingface.co/csukuangfj2/sherpa-onnx-moonshine-tiny-en-quantized-2026-02-27/blob/main/test_wavs/0.wav)
(24 kHz, resampled internally by sherpa-onnx). The Russian sample was
[the Vosk model's test WAV](https://huggingface.co/alphacep/vosk-model-small-ru/blob/main/test.wav)
(16 kHz). Both Moonshine variants transcribed the English sample correctly.
Zipformer produced the expected Russian words without capitalization or punctuation.

These two short samples establish that installation, decoding, and the existing
recognition path work. They are not a representative accuracy benchmark, a
cross-CPU comparison, a measured RAM requirement, or a battery-life measurement.
There was no controlled system-load or power measurement. Test your own microphone
audio before treating the timing ratios as an expectation.

## Automated checks

The full unit suite passed: **90 tests**. Added coverage checks catalog/installer
consistency, required file validation, nested upstream download paths, reuse of
installed files, CPU recognizer construction, cache reuse, incompatible language
hints, and recognition without network calls or a separate language detector for
the single-language models.

## Fast automatic English/Russian

`local-fast-auto` combines Whisper Tiny spoken-language identification with
Moonshine Tiny for English and Zipformer Small for Russian. Detection happens on
every completed phrase and ignores saved language hints. Other detected languages
use Whisper Tiny transcription with that detected language, avoiding another
detection pass. The app preloads the detector and fast pair in a background thread
at startup, when the mode is saved, and after the bundle is installed.

The same two clips above were tested with all three components in a workspace
model directory. Network requests raised an error for the entire check. English
and Russian recordings alternated three times under the same model ID, without
passing language hints. Both produced non-empty transcripts in the correct language.
An initial pair of calls with deliberately incompatible saved hints also worked;
the hints did not affect routing. The first one-second excerpt of each clip was
identified correctly, which does not establish reliability for all short speech.

Warm median timings **including detection, routing, WAV conversion, and decoding**:

| Automatic mode | Audio duration | Recognition time |
| --- | ---: | ---: |
| English → Moonshine Tiny | 3.845 s | 0.249 s |
| Russian → Zipformer Small | 7.080 s | 0.330 s |

Background loading took 4.13 s in this run. Recognition started after preload
completed. These timings come from a separate run from the single-model table;
system load was not controlled. They are examples, not a universal speed guarantee.
Language detection can misidentify short/noisy or mixed-language phrases. The
single-language recognizers assume one dominant language per phrase; this mode is
not an accuracy-equivalent replacement for Parakeet.

Additional regression tests cover changing languages between successive calls,
ignoring inherited hints, fallback to Whisper for other languages, readiness of
all bundle files, monotonic installer progress, cached preparation without audio
decoding, background preload deduplication, and automatic-language selection in
the setup wizard.
