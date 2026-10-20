# Rekli for Windows

[English](README.md) | [Русский](README.ru.md)

**Speak into a microphone and put your words into the Windows app you are using.**

The main advantage is **fully offline dictation on your CPU**. After a one-time model download, speech recognition runs on your PC. You do not need a dedicated GPU, an NVIDIA graphics card, an API key, a paid speech service, or an internet connection to dictate locally.

Cloud recognition is optional. If you prefer an external service, you can select Groq, OpenAI, or a custom OpenAI-compatible transcription API. The local speech engines are complete recognition backends, not a demo or a fallback that requires a cloud account.

[Download Rekli.exe](https://github.com/AlekZo/Rekli/releases/latest/download/Rekli.exe) · [Release notes](https://github.com/AlekZo/Rekli/releases/latest) · [Report an issue](https://github.com/AlekZo/Rekli/issues)

## Quick start

1. Run **Rekli.exe**. A prebuilt executable includes Python and the application dependencies. [Published releases](https://github.com/AlekZo/Rekli/releases/latest) are hosted on GitHub.
2. Follow the **Setup wizard**: choose English or Russian. This sets both the interface and the initial recognition language.
3. Choose **Parakeet TDT v3** (recommended by default) and click **Download model**. The model downloads once and then works offline.
4. Choose a microphone, click **Test microphone**, and speak until the live meter confirms a signal.
5. Review the default **Ctrl+Alt+Space** shortcut or record a different one. The final page shows the same steps for daily dictation.
6. Click into the text field where you want the result, press the shortcut, speak, then press it again or click **Stop & Insert**. Keep the original field selected while recognition finishes and inserts the text.

The wizard can be reopened from **Tray → Setup wizard** or **Settings → Setup wizard**. Change the display language later in **Settings → Behavior → Interface language**; the speech-recognition hint remains separate in **Settings → Speech**. Microphone testing reads levels without saving audio. Model installation needs internet access once; local dictation does not.

To check an input later, open **Settings → Microphone**, select the device, click **Test microphone**, and speak. Confirm that the meter moves, then click **Save**. If there is no signal, check mute and Windows microphone access or test another input.

## Offline recognition and CPU support

All local engines run through **sherpa-onnx on the CPU**. Parakeet, Whisper, and Zipformer use quantized ONNX weights; Moonshine uses the quantized ORT exports documented by sherpa-onnx as Moonshine v2. The application explicitly selects `provider="cpu"` for recognition and local Whisper language identification. It does not require CUDA and currently does not expose GPU acceleration.

| Mode | How it works | Approximate download | CPU inference threads |
| --- | --- | --- | --- |
| **Automatic (compatibility mode)** | Keeps Parakeet TDT v3 on AC power and battery. | 640 MB | Up to 4 |
| **Parakeet TDT v3** | Uses Parakeet regardless of power source. The app's preferred option for English/Russian dictation. | 640 MB | Up to 4 |
| **Fast automatic · English/Russian** | Detects each phrase locally; Moonshine Tiny for English, Zipformer Small for Russian. No manual language switching. | 173 MB total | Up to 2 per model |
| **Whisper Tiny INT8** | A smaller alternative with a lower model footprint. Uses Whisper regardless of power source. | 100 MB | Up to 2 |
| **Moonshine Tiny (English only)** | Lightweight option for short English phrases. Quantized export dated February 27, 2026. | 45 MB | Up to 2 |
| **Moonshine Base (English only)** | Larger alternative to Moonshine Tiny for a different quality/size tradeoff. Same February 2026 export format. | 142 MB | Up to 2 |
| **Zipformer Small INT8 (Russian only)** | Small Vosk/Zipformer model for Russian. Older September 2024 export, added for its low footprint. | 28 MB | Up to 2 |

The default is **Parakeet TDT v3**. Existing Automatic preferences also resolve to Parakeet on both AC power and battery, with no automatic switch to a smaller model. The additional modes are explicit choices in both **Settings → Speech** and the **Setup wizard**. Select one and click **Install model** (or **Download model** in the wizard). Only the selected model or bundle is downloaded. Settings saves the selection when you click **Install model**; the wizard applies it when you click **Save & start**.

**One hotkey for English and Russian:** choose **Local · Fast automatic · English/Russian** once and install its bundle. Then use your normal hotkey and speak either language, without choosing a language or changing models between recordings. The language field is automatically cleared and disabled for this mode; previously saved language hints are ignored. Whisper Tiny identifies each completed phrase, Moonshine Tiny recognizes English, and Zipformer Small recognizes Russian. A different detected language is transcribed by Whisper Tiny. Detection and recognition remain fully local.

The bundle installs **Whisper Tiny + Moonshine Tiny + Zipformer Small (~173 MB)** and reuses existing files. The detector and fast pair load in the background after installation, when the mode is saved, and at startup. The first phrase can still take longer if background loading has not completed. Language is detected again for every completed recording or progressive phrase. Including detection, the warm pipeline took about **0.25 s for a 3.85 s English clip** and **0.33 s for a 7.08 s Russian clip** in the local test. See [validation details](docs/local-model-validation.md) and [the local language detector documentation](https://k2-fsa.github.io/sherpa/onnx/spoken-language-identification/pretrained_models.html).

Fast automatic assumes one dominant language per phrase. Very short or noisy speech can be misidentified, and switching languages inside one phrase is not reliably supported by the single-language models. This Zipformer export provides neither automatic punctuation nor capitalization. **Parakeet TDT v3** with an empty language hint also requires no manual language switching and remains the default for recognition quality.

**Quality first:** keep Parakeet selected for English/Russian dictation. Fast automatic, Whisper Tiny, Moonshine Tiny/Base, and Zipformer Small are optional size/speed tradeoffs, not verified replacements of equal or better accuracy. Choose a smaller model only after checking it on your own speech. These choices have no API charges; Fast automatic adds a separate language detection step. Their smaller footprint and two-thread cap do not establish a battery-life guarantee or a universal speed advantage over Parakeet.

Moonshine supports **English only**, and Zipformer Small supports **Russian only**. Leave the language hint empty or use `en` for Moonshine / `ru` for Zipformer. An incompatible explicit hint produces a clear error; automatic language mode does not make these models multilingual. This Zipformer Small export outputs lowercase words without automatic punctuation. Test your own phrases before choosing a smaller model for accuracy-sensitive dictation.

### Punctuation in English and Russian

**If you need punctuation in both languages, choose Parakeet or Whisper Tiny.** Fast automatic changes formatting with the detected language: its English model can add punctuation, while its Russian model cannot. The **Punctuation & capitalization** summary in **Settings → Speech** and the setup wizard explains this for each choice.

| Local model/mode | English punctuation | Russian punctuation | Capitalization / evidence |
| --- | --- | --- | --- |
| Parakeet TDT v3 | Supported automatically | Supported automatically | Supports capitalization in both languages. NVIDIA documents punctuated output; marks may still be omitted. |
| Automatic (compatibility mode) | Same as Parakeet | Same as Parakeet | Uses the same Parakeet recognizer. |
| Whisper Tiny INT8 | Supported automatically | Supported automatically | Capitalization and periods observed in both sample transcripts. |
| Moonshine Tiny / Base, installed v2 exports | Supported; periods observed | Language unsupported | Capitalized English output observed in both installed exports. |
| Zipformer Small INT8, installed Russian export | Language unsupported | **No automatic punctuation** | Lowercase words; its token vocabulary contains no sentence punctuation or Russian capital letters. |
| Fast automatic · English/Russian | Supported via Moonshine Tiny | **No automatic punctuation via Zipformer Small** | English capitals; lowercase Russian. Language detection does not restore punctuation. |

Supported punctuation is a model capability, not a promise that every comma,
question mark, or sentence ending will be correct. The Russian Parakeet test clip,
for example, had capitalization but no final period. Short progressive chunks can
lose sentence context. Rekli preserves the returned text and has **no separate
punctuation-restoration or capitalization pass**. See [actual sample transcripts](docs/recognition-quality-check.json),
[NVIDIA's Parakeet model card](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3),
and [sherpa-onnx's Moonshine v2 examples](https://k2-fsa.github.io/sherpa/onnx/moonshine/models-v2.html).

The Moonshine entries use the February 2026 **non-streaming v2 exports** supported by the pinned `sherpa-onnx==1.13.4` runtime. They are distinct from the separate Moonshine Streaming Tiny/Small/Medium family. The app continues to transcribe completed audio chunks for progressive insertion.

The thread count is capped by the number of logical processors reported by the system. Whisper automatic language detection also uses up to two CPU inference threads. These caps describe inference configuration, not total application thread count.

**Hardware:** the Windows release targets x64 PCs, including Intel and AMD processors. A microphone and sufficient free memory and disk space are required. No specific minimum CPU generation, instruction-set matrix, or minimum RAM configuration has been qualified. ARM64 and 32-bit Windows builds are not currently qualified.

Parakeet's built-in model description estimates around 2 GB of memory for recognition; this is a rough guide, not a measured minimum for the whole PC. Leave memory available for Windows and your other applications. Try a smaller model for your language if the larger model strains your machine or if a smaller download matters more. Switching models can keep previously loaded recognizers in memory; restart the app to measure one model's footprint in isolation.

Recognition speed and quality depend on CPU performance, microphone quality, speech, noise, and model selection. The project does not publish a verified cross-CPU speed benchmark or a guaranteed real-time ratio. In a local Windows CPU check on October 13, 2026, warm recognition of a 3.85-second English clip took 0.056 s with Moonshine Tiny, 0.093 s with Moonshine Base, and 0.457 s with Parakeet. A 7.08-second Russian clip took 0.102 s with Zipformer Small and 0.614 s with Parakeet. These are medians of three runs on two short samples, not general speed or accuracy guarantees; see [validation details](docs/local-model-validation.md). The first recognition can take longer because the model must be loaded; recognizers are cached in memory for subsequent use.

English and Russian are the app's main use cases. Parakeet handles language detection in its recognizer; Whisper can detect the utterance language locally or use an English/Russian hint. Underlying model language support is broader, but the app has not qualified every supported language or mixed-language dictation scenario.

Model/runtime references: [NVIDIA Parakeet TDT v3](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3), [sherpa-onnx Whisper documentation](https://k2-fsa.github.io/sherpa/onnx/pretrained_models/whisper/index.html), [Moonshine v2 exports](https://k2-fsa.github.io/sherpa/onnx/moonshine/models-v2.html), [Russian Zipformer Small](https://k2-fsa.github.io/sherpa/onnx/pretrained_models/offline-transducer/zipformer-transducer-models.html#sherpa-onnx-small-zipformer-ru-2024-09-18-russian).

## Recognition quality policy

Parakeet TDT 0.6B v3 INT8 is the local baseline used by this application. The rename keeps its model files and CPU recognizer configuration. Automatic mode uses that same backend on both power sources. New installations leave progressive insertion off so the recognizer receives the complete recording; existing explicit preferences are preserved. Progressive insertion can lose context because it recognizes shorter phrases.

A local check on two existing English/Russian recordings verified that the default and Automatic paths produce exactly the same transcripts as explicit Parakeet selection on AC and battery. Smaller models returned different Russian wording or punctuation. Two clips do not establish general word accuracy, and matching Parakeet's output does not prove equivalence to NVIDIA's original full-precision checkpoint. The CPU app uses an INT8 ONNX conversion. See [quality check](docs/recognition-quality.md) for scope, outputs, and reproduction.

## Optional external APIs

Open **Settings → Speech**, or select a cloud provider in the setup wizard. Choose the provider/model, supply its API key, and save. For a custom provider, specify its transcription endpoint, model ID, and API key.

| Backend | Choices included in the app |
| --- | --- |
| Local | Automatic, Parakeet TDT v3, Fast automatic (English/Russian), Whisper Tiny INT8, Moonshine Tiny/Base (English), Zipformer Small INT8 (Russian) |
| Groq | Whisper Large v3 Turbo, Whisper Large v3; legacy Distil-Whisper English entry is retired |
| OpenAI | Whisper (`whisper-1`), GPT-4o Transcribe, GPT-4o-mini Transcribe |
| Custom | An OpenAI-compatible `/audio/transcriptions` API |

Cloud mode sends recorded audio to the chosen endpoint and requires access to that service. Its availability, supported models, prices, and data handling are determined by the provider. Figures displayed by the app are estimates, not live pricing. Local recognition remains free of API charges.

**Cloud punctuation:** Groq Whisper Large v3/Turbo and OpenAI Whisper, GPT-4o
Transcribe, and GPT-4o-mini Transcribe are expected to return punctuation and
capitalization for English/Russian transcription. This is an expectation from
their transcription capabilities, not a local test of the paid services; exact
formatting can vary. A custom endpoint determines its own language and punctuation
support. See [Groq speech-to-text documentation](https://console.groq.com/docs/speech-to-text)
and [official OpenAI transcription guidance](https://developers.openai.com/api/docs/guides/speech-to-text).

The saved `distil-whisper-large-v3-en` entry is **retired by Groq** (August 23,
2025), and was English-only. It is labeled as retired in Settings for existing
configurations; use a currently supported Groq model for new dictation.
[Groq deprecation notice](https://console.groq.com/docs/deprecations).

**Rekli is an independent project**, with no affiliation to Groq, OpenAI, NVIDIA, or xAI. Local recognition requires no provider subscription.

## Recording and insertion

- The overlay starts in **compact mode**. **Details** opens the full panel with microphone diagnostics and five seconds of input-level history. **Compact** shrinks it again, and the view choice is remembered.
- In the detailed view, **Glass** changes panel opacity between 84%, 68%, and 96%. Text and controls stay readable; this is transparency, not desktop blur.
- The panel shows elapsed time, input levels, silence/quiet/clipping warnings, and a **Transcribing…** state after Stop.
- After Stop, **both compact and detailed panels show an estimated countdown**, such as `~7s`. It uses recording length and the latest successful timings for the selected model on your PC, with an allowance for initial model loading. It learns as you dictate. In progressive mode it estimates only pending/in-flight audio. If the estimate expires, the panel says **Still working…** and stays visible until transcription actually finishes. The countdown is an estimate, not a completion guarantee.
- **Progressive typing** can insert completed phrases while you speak when using a local model. Disable it in **Settings → Behavior** to receive the text after Stop instead.
- Cancel discards remaining audio. It cannot undo phrases that have already been inserted.
- Text is inserted through clipboard paste or simulated keystrokes. If delivery fails, use **Copy undelivered text** or, when safe, **Retry undelivered text** from the tray menu.

The overlay avoids activating the target window. Protected fields, secure desktops, elevated applications, and some custom controls may reject simulated input.

## Windows compatibility and startup

Release targets: **x64 Windows 10 22H2** and **x64 Windows 11**. The local development build and unit tests have run on Windows 11. Complete clean-machine acceptance testing on Windows 10 remains pending. macOS and Linux are not supported by this Windows application.

Enable **Start automatically when Windows starts** in **Settings → Behavior** or in the setup wizard. It registers startup for the current Windows user and launches the app in the tray. Keep the executable in the same folder. If you move it, turn the option off and on to update the path; also check **Task Manager → Startup apps** if Windows has disabled it.

## Updates

Rekli checks the [releases page](https://github.com/AlekZo/Rekli/releases) at startup (at most every 6 hours; turn it off in **Settings → Behavior → Updates**). **Check for updates…** in the tray menu checks on demand. When a newer version exists, choose **Install update** in the tray menu: Rekli asks first, downloads `Rekli.exe`, verifies its SHA-256 against the release, replaces itself and restarts. Settings, credentials and models are kept, and Rekli reports the new version after restarting. Nothing is installed without your confirmation. You can also download any release manually and compare it with `SHA256SUMS.txt`.

## Optional NanoKVM output

Configure devices in **Settings → NanoKVM**. Output can go to the current Windows app, always to the first configured NanoKVM device, or automatically to a device matched by the captured browser-tab title.

NanoKVM sends recognized text to the remote computer through its HID paste interface. With local recognition, microphone audio stays on your PC. Use HTTPS or a trusted VPN for remote access. Russian output requires the Russian keyboard layout on the remote computer; mixed English/Russian transcripts and unsupported characters are rejected to avoid incorrect keystrokes.

## Privacy and local storage

| Item | What happens |
| --- | --- |
| Local dictation | Microphone audio is processed on your PC; the recognition path makes no network requests. |
| Model installation | Files are downloaded once from Hugging Face. |
| Cloud dictation | Audio is sent only to the selected transcription endpoint. |
| Audio recordings | Held in memory; the app does not maintain a recording archive. |
| Usage history | Local statistics such as app, model, duration, latency, and character count. The transcript is not saved in the usage log. |
| Secrets | API keys and NanoKVM passwords use Windows Credential Manager. |
| Clipboard | Paste mode places the recognized text on the Windows clipboard. |

Configuration, usage history, installed models, and runtime status remain under `%APPDATA%\GrokVoice\`. The legacy directory and credential targets are retained so the Rekli rename reuses your settings, downloads, history, and saved keys. Use **Open config folder** from the tray or Settings. The app has no built-in telemetry upload. Explicit cloud, model-download, and NanoKVM operations use the network as described above.

## Troubleshooting

| Problem | What to try |
| --- | --- |
| No signal or no text after Stop | Test the microphone in Settings. Check mute, device connection, Windows input selection, and desktop microphone permissions. |
| Microphone error `-9996` on Windows 10 | PortAudio reports an invalid input device. Exit the app, reconnect the mic, select it explicitly in Settings, and test it. Confirm that Windows detects the same input. If it fails only in this app, report the complete error and device model; backend/format compatibility may need investigation. |
| Microphone cannot be opened | In Windows microphone privacy settings, allow device, app, and desktop-app microphone access. Reopen the app after changes. |
| Model is not installed | Use the wizard or Settings → Speech to install it. Automatic needs only Parakeet. Fast automatic installs Whisper Tiny, Moonshine Tiny, and Zipformer Small together. |
| Dictation is slow on this CPU | Try Moonshine Tiny for English or Zipformer Small for Russian; Whisper Tiny is a compact multilingual alternative. Reduce competing CPU/memory load or use an optional external API. Measure on your own machine; a smaller model is not a guarantee of faster recognition. |
| Shortcut does not respond | Check Settings → Hotkeys, try a different combination, and ensure that another running instance is not using the shortcut. |
| Text was recognized but not delivered | Keep the original field selected. Copy undelivered text from the tray; retry only if the app reports it is safe. |
| Startup is enabled but nothing launches | Check Task Manager → Startup apps and re-enable the option if the executable has moved. |
| Cloud request fails | Verify the provider's endpoint, model ID, API key, and current account status. |

Microphone references: [PortAudio error definitions](https://portaudio.com/docs/v19-doxydocs/portaudio_8h_source.html), [Windows backend format-error discussion](https://github.com/PortAudio/portaudio/issues/314), [Microsoft microphone permissions](https://support.microsoft.com/en-us/windows/privacy/turn-on-app-permissions-for-your-microphone-in-windows).

## License and support

Rekli is provided as is, without warranty. Dependencies and downloaded models keep their own licenses. Built with PySide6, sounddevice, NumPy, sherpa-onnx, Parakeet, Whisper, Moonshine, and Vosk/Zipformer.

[Report an issue](https://github.com/AlekZo/Rekli/issues) or [support development](https://boosty.to/donate_zotov/donate).
