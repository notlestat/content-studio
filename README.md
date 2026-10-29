# Content studio

A local Python CLI operated through Codex for transcript review, edit planning, shorts, captions, graphics, and supporting copy. Saved client preferences guide each project. Human review controls editing choices and final delivery.

## Install the application

Use Python 3.12+, Node.js 22+, and system `ffprobe`. Python dependencies include a subtitle-capable FFmpeg fallback.

```sh
git clone https://github.com/notlestat/content-studio.git
cd content-studio
python3 -m venv content-studio/.venv
content-studio/.venv/bin/python -m pip install -e './content-studio[local,dev]'
npm ci --prefix content-studio/tools/graphics
content-studio/.venv/bin/studio doctor
```

The final command reports local dependencies and renderer capabilities. Resolve missing prerequisites before processing media.

To install the bundled Codex instructions, run `python3 tools/install-skill.py`. The installer prints the destination and binds it to this checkout. It refuses to overwrite an existing installation. Keep the checkout in place, or update `app_path` in the installed skill's `config.local.json` after moving it.

## Start a project

Open this repository in Codex and provide the source path, client, deliverables, and editing constraints. For example:

> Process the episode at /path/to/episode.mp4 for this client. Make five 30–60 second shorts with clean captions and restrained graphics. No longform edit.

Read [the operating guide](content-studio/README.md) for client setup, manual commands, review checkpoints, and output paths. [AGENTS.md](AGENTS.md) defines the operator workflow. Client media and working projects are excluded from Git.

Local Whisper transcription needs no API key. Its first run downloads the model. The optional ElevenLabs provider reads `ELEVENLABS_API_KEY` and sends transcription input to that service. Codex performs editorial analysis. Unknown token counts and costs remain unknown in project reports.

## Check a change

After installation, run from the repository root:

```sh
cd content-studio
.venv/bin/pytest -q
.venv/bin/ruff check studio tests ../skills ../tools
.venv/bin/mypy --config-file pyproject.toml
```

The tests include local media operations. Technical checks do not replace watching and listening to final outputs. `--auto` is for explicit experiments and does not record human QC or delivery approval.

## Find the implementation

| Location | Purpose |
| --- | --- |
| `content-studio/studio/` | CLI and media workflow |
| `content-studio/templates/` | Client, caption, and graphics templates |
| `skills/` | Bundled Codex instructions and launcher |
| `tools/install-skill.py` | Local installer |
| [Graphics guidance](content-studio/docs/graphics.md) | Custom animation slots |
| [Upstream review](content-studio/docs/upstream-review.md) | Reuse, pinned sources, and licenses |

The application has no publishing integration. Rendered files require current human QC before delivery.
