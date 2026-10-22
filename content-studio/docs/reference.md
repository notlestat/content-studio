# Content studio reference

See [the operating guide](../README.md) for client setup and review steps.

## Files

```text
content-studio/
  clients/<client>/
    client.yaml
    brand/{logos,assets,references}/
    projects/<project>/
      project.json
      sources.json
      01_raw/           original media, never modified
      02_transcript/    transcript.json, transcript.md, per-source ASR cache
      03_analysis/      brief, coverage chunks, analysis, candidates, selection
                        visual opportunities, animation slots, review notes
      04_longform/      EDL, output transcript, framing and QC evidence
      05_shorts/<id>/   EDL, ASS captions, framing and QC evidence
      06_copy/<id>/     hook, title, caption, YouTube title, description,
                        LinkedIn, X, alternatives, CTA and metadata
      07_final/
        youtube/       final-youtube.mp4, thumbnail-notes.md, youtube-copy.md
        shorts/01/     clip.mp4, copy.md, metadata.json
        REVIEW-STATUS.md
      logs/            events, usage, approvals and render manifest
      report.md
  templates/           default client, caption presets, graphic source
  tools/               local render dependencies and model cache
  config/studio.yaml
  studio/              Python implementation
  vendor/video-use/    pinned MIT helpers and license
```

Project names become slugs. `latest` means most recently created project, not the
one with the most recently modified file. Use an explicit slug when processing an
older episode. Multiple raw files are separate takes; the plan can combine them.
Audio-only files produce a branded still with the conversation audio and captions.

## Transcription, dependencies and API keys

Default: `faster-whisper` small.en on CPU with int8 and word timestamps. The first
run downloads the model to `tools/cache/whisper`; later transcription is local.
There is no additional API charge. It may omit fillers and mishear proper names.
It has no speaker diarization. Review it before precision editing.

For video-use's preferred verbatim transcription with diarization:

```sh
export ELEVENLABS_API_KEY='your-key'
studio transcribe rui-fu ai-business-podcast --provider elevenlabs
```

Store secrets in your shell environment or your own local secret manager. Do not
commit them or put them in a client profile. The studio does not need an OpenAI,
Anthropic or Gemini key because Codex is the editorial operator. A transcript
cache is reused only when source hash, provider, model, language and audio track
match. Changing these settings selects a separate cache or transcribes again.

Set `audio_track` in `config/studio.yaml` for multi-track camera/OBS recordings.
The same track is used for transcription and output. Track numbering starts at 0.
Set language to null for automatic language detection, or choose a language code.
Use a multilingual model rather than small.en for non-English sources.

FFmpeg's `subtitles` filter is required. The original Mac's Homebrew build lacked it during
development, so studio uses the local imageio-ffmpeg binary with libass. ffprobe
still comes from the system. `STUDIO_FFMPEG` can select an explicit FFmpeg build.
Node 22+ is required for HyperFrames. No cloud rendering is used.

Installation commands are in [the repository overview](../../README.md#install-the-application).

`requirements-lock.txt` records the Python versions used in the verified Mac
environment. The npm lockfile pins graphics dependencies. Private media, model
caches, dependencies and secrets are ignored by Git. ffmpeg binaries and fonts
have their own licenses; the setup does not relicense them.

## What the MVP handles

- Full-source transcript reading with coverage checks, 15-30 candidate proposals
  where the material supports them, topic grouping and editorial ranking.
- Keep-range edits across takes, restrained punch-ins, 30ms cut fades, audio
  normalization, client caption presets and selective animation slots.
- Conservative face-aware crops and explicit manual crop positions. Multiple or
  changing faces fall back to full-picture fit. It does not claim active-speaker
  tracking from audio. Codex can split a shot and specify speaker crops after
  inspecting it. Source-time `crop_keyframes` change the crop at camera switches
  within a continuous audio range, without adding unnecessary audio edits.
- clean, minimal, bold, podcast and creator captions, with optional word emphasis.
- HyperFrames cards and lists, external HyperFrames/Remotion compositions, and
  simple PIL cards. See [Graphics guidance](graphics.md) for custom charts and UI graphics.
- Source hashes, resumable ASR/render caches, review records, per-project logs,
  technical QC and human sign-off.

B-roll acquisition, advanced tracking, automatic retake judgment, bespoke charts,
colour decisions and editorial polish remain Codex/operator work, backed by the
saved profile and evidence. The CLI never invents those decisions. This MVP does
not independently learn a client's taste or predict a clip's view count.

## Costs and margin

`report.md` and `logs/cost-summary.json` show processing seconds, API attempts,
known cost, unknown entries, and per-stage usage. Supply your current provider
rates in config/studio.yaml when needed. Unknown prices and unavailable token
counts remain null. Failed API attempts remain recorded. Codex subscription
usage and local electricity are not converted into a guessed per-project cost.

## Validation

```sh
pytest -q
ruff check studio tests
mypy --config-file pyproject.toml
```

The tests include a real FFmpeg render and verify source preservation, caption
timing after edits, review invalidation, path safety, full analysis coverage,
selection replacement and unknown-cost handling. The original Mac also retains
a local docs/development-test.md report with private output paths. That report
and all client data are excluded from the GitHub repository.

Exit codes: 0 success, 1 error, 2 expected review/Codex checkpoint, 130 interrupted.
Rerun after an interruption to reuse completed cached work. Errors never imply
that empty/missing outputs are successful. Read project logs for provider or
render failures.

Upstream licenses, inspected commits and exact reuse are documented in
[Upstream review](upstream-review.md). No AutoClip services were installed.
