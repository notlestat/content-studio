# Review and render a content project

Install the application using [the repository overview](../README.md#install-the-application), then follow the steps below. Example client names and clip IDs describe sample commands, not completed client work.

## First client

From the repository root, activate the installed environment and create the example client and project:

```sh
cd content-studio
source .venv/bin/activate
studio doctor
studio client create 'Rui Fu'
studio project create rui-fu 'AI Business Podcast'
```

Edit `clients/rui-fu/client.yaml` to save Rui's preferences. Longform defaults to
disabled; set `longform.enabled: true` when the client wants a YouTube edit.
`target_duration`, short durations and all timestamps are seconds. Null longform
target duration means preserve the editorially useful length.

Add media with either command:

```sh
studio ingest rui-fu ai-business-podcast --file '/absolute/path/episode.mp4'
studio ingest rui-fu ai-business-podcast --youtube 'https://www.youtube.com/watch?v=VIDEO_ID'
```

Or copy the source into `clients/rui-fu/projects/ai-business-podcast/01_raw/`.
Then tell Codex, **"Process Rui's latest project."** [The repository instructions](../AGENTS.md) define
how Codex resumes the workflow. It reads Rui's profile every time.

Run it yourself:

```sh
studio process rui-fu
```

This resumes the newest project and stops at the next required review. Codex authors and imports the editorial plan, and you approve the choices.

## Review the transcript

Read the word-timed transcript and check names, numbers, and timing against the source:

```sh
studio review rui-fu ai-business-podcast transcript
```

If you edit `transcript.json`, rebuild the readable transcript through the importer:

```sh
studio transcribe rui-fu ai-business-podcast --import-json clients/rui-fu/projects/ai-business-podcast/02_transcript/transcript.json
```

After completing the review, record it:

```sh
studio approve rui-fu ai-business-podcast transcript --note 'Checked names, numbers and timing against source'
```

## Prepare and review the edit plan

Export the analysis brief:

```sh
studio analyze prepare rui-fu ai-business-podcast
```

Ask Codex to read the brief and every coverage chunk, then write `analysis-proposal.json`. Import the completed proposal and inspect the plan:

```sh
studio analyze import rui-fu ai-business-podcast clients/rui-fu/projects/ai-business-podcast/03_analysis/analysis-proposal.json
studio review rui-fu ai-business-podcast plan
```

Use IDs from your generated plan to change selections. These commands illustrate the available actions:

```sh
studio clips reject rui-fu ai-business-podcast c02
studio clips approve rui-fu ai-business-podcast c07
studio clips replace rui-fu ai-business-podcast c03 c09
studio graphics reject rui-fu ai-business-podcast g02
studio review rui-fu ai-business-podcast graphics
```

## Render approved selections

After reviewing the current selections and graphics, record the decisions and render:

```sh
studio approve rui-fu ai-business-podcast plan --note 'Approved the keep ranges and selected shorts'
studio approve rui-fu ai-business-podcast graphics --note 'Approved the chosen concepts and timing'
studio render rui-fu ai-business-podcast
```

## Inspect the final outputs

Run technical QC and generate inspection evidence for the outputs you rendered:

```sh
studio qc rui-fu ai-business-podcast
studio qc rui-fu ai-business-podcast --inspect c01
```

For a longform output, use `--inspect longform`. Watch and listen to every output. After checking cuts, sync, captions, and graphics, record human QC and generate the report:

```sh
studio approve rui-fu ai-business-podcast qc --note 'Watched every output, checked cuts, sync, captions and graphics'
studio report rui-fu ai-business-podcast
```

Approve only after doing the stated check. IDs are stable within an analysis.
Replacing a clip may need copy or new visual opportunities from Codex. Edit the
proposal and reimport it. Imports recalculate the default selection, so apply
your final clip choices afterwards.

Approvals expire when relevant inputs or choices change. A render cannot use a
stale analysis. Raw files are hashed and checked before processing. Symlinks and
hardlinked working files are refused. Originals are never output destinations.

## Run an explicit experiment

For an experiment you have chosen to run, `--auto` bypasses review checkpoints:

```sh
studio process rui-fu ai-business-podcast --auto
studio render rui-fu ai-business-podcast --auto
```

The first command still needs a real Codex-authored analysis if none exists. It
exports a brief and exits 2. Neither command marks experimental output ready to
publish. All normal review commands remain available afterwards.

Read the [reference](docs/reference.md) for output paths, dependencies, transcription providers, cost records, and exit codes.
