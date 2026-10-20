# Rekli recognition quality check

## Baseline and policy

Rekli retains the application's **Parakeet TDT 0.6B v3 INT8 CPU recognizer**. The model files, greedy decoding configuration, sample conversion, and four-thread cap are unchanged by the rename.

- New setups default to `local-parakeet-v3`.
- Existing `local-auto` selections resolve to the same Parakeet model on AC power and battery. They no longer select Whisper Tiny on battery.
- Automatic installation/readiness now requires only Parakeet.
- Smaller models remain explicit options. None is advertised as meeting or beating Parakeet accuracy.
- Progressive insertion is off for new setups so the entire recording reaches the recognizer. Existing explicit preferences remain intact. Phrase-based recognition can lose context.

## Real speech check

Recognition was run locally with `sherpa-onnx==1.13.4`, using the existing English and Russian WAV files from the [earlier local model validation](local-model-validation.md). No microphone was accessed, no settings were changed, and HTTP requests were blocked during recognition.

For each clip, the explicit Parakeet transcript was compared with the default model and Automatic under simulated AC and battery conditions. **All eight comparisons were exactly equal, including case and punctuation.** Power source simulation checks the selection policy; it is not a battery-performance measurement.

| Model/path | English sample, 3.845 seconds | Russian sample, 7.080 seconds |
| --- | --- | --- |
| Default Parakeet | Exact baseline output | Exact baseline output |
| Automatic on AC | Exact baseline output | Exact baseline output |
| Automatic on battery | Exact baseline output | Exact baseline output |
| Whisper Tiny | Same normalized words; punctuation differs | 4 word edits against Parakeet |
| Moonshine Tiny | Same normalized words; punctuation differs | Unsupported |
| Moonshine Base | Same normalized words; punctuation differs | Unsupported |
| Zipformer Small Russian | Unsupported | 2 word edits against Parakeet; capitalization differs |

The comparison lowercases words, maps `ё` to `е`, and ignores punctuation for the alternative-model word counts. **These counts measure disagreement with Parakeet, not word-error rate.** The baseline itself can make errors; an alternative's different wording may be better or worse. The exact transcripts and results are in [recognition-quality-check.json](recognition-quality-check.json).

## Scope of the conclusion

The default and Automatic paths use the actual Parakeet backend and produced matching transcripts in this check. This supports retaining the existing application's Parakeet recognition behavior instead of switching to a smaller engine.

Two short clips do **not** establish accuracy across accents, names, technical vocabulary, noisy microphones, mixed-language speech, or long dictation. No human-annotated representative accuracy corpus was evaluated. This check does not show that any smaller model is as accurate as Parakeet, or that the INT8 CPU conversion equals NVIDIA's original full-precision checkpoint.

Before promoting a smaller model as equal or better, evaluate it and Parakeet on the same representative Russian/English recordings with reviewed reference transcripts. Compare word and character error rates, names and numbers, punctuation, and complete-recording versus progressive insertion behavior. Until then, use Parakeet for the requested quality baseline.
