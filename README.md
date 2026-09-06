# Steering demo webpage generator

Generates a static, GitHub-Pages-ready page of single-concept steering examples.
For each **prompt**, each steered concept is shown as a grid: **rows = DiT layers**,
**columns = steering alpha** (`α=0` is the unsteered reference), at a single fixed
**denoising timestep** shown beside the prompt. Each cell = mel-spectrogram + audio
player. Bootstrap column grid, same spirit as the MorphFader supplementary page.

## Install

```bash
pip install pyyaml librosa matplotlib numpy soundfile
```

## Input folder layout

```
samples/
  prompt_1/
    birds_alpha0.wav              # unsteered (layer-independent) — one per concept
    birds_layer8_alpha10.wav      # layer 8,  α=10
    birds_layer8_alpha20.wav      # layer 8,  α=20
    birds_layer12_alpha10.wav     # layer 12, α=10
    birds_layer12_alpha20.wav
    birds_layer16_alpha10.wav
    birds_layer16_alpha20.wav
  prompt_2/
    thunder_alpha0.wav
    thunder_layer8_alpha10.wav
    ...
```

- **Steered** files: `{sound}_layer{layer}_alpha{alpha}{ext}` (via `filename_template`).
- **Unsteered** (`α=0`): layer-independent by default, so you supply **one** file per
  concept — `{sound}_alpha0{ext}` (via `unsteered_filename_template`) — reused across all
  layer rows. Set `unsteered_layer_independent: false` if your pipeline writes a separate
  unsteered file per layer.
- Both templates accept `{sound} {layer} {alpha} {timestep} {ext}`; unused placeholders are
  ignored, so you can bake the timestep into filenames if you prefer.

## Run

```bash
python build_steering_webpage.py \
    --config config.yaml \
    --audio-dir ./samples \
    --output-dir ./site

python build_steering_webpage.py ... --force   # regenerate spectrograms / recopy audio
```

## Output (commit this whole folder to gh-pages)

```
site/
  index.html
  assets/style.css
  assets/audio/prompt_1/...          # wavs copied in (relative paths)
  assets/spectrograms/prompt_1/...   # PNGs generated from the wavs
```

## Config

See `config.example.yaml` for a fully-commented template. Key fields:

- `title` / `subtitle` / `description` / `footer` — page text.
- `alphas: [0, 10, 20]` — the columns. `unsteered_alpha` marks the reference column.
- `layers: [8, 12, 16]` — the rows (per-prompt override via `layers:` inside a prompt).
- `timestep: 0.5` — fixed denoising timestep, shown as a badge beside every prompt
  (per-prompt override via `timestep:` inside a prompt).
- `filename_template` / `unsteered_filename_template` — how files are located.
- `unsteered_layer_independent` — reuse one α=0 file across layers (default `true`).
- `spectrograms:` — `generate: true` renders mel-spectrograms; `false` reuses a PNG next
  to each wav (same stem). Tune `n_mels`, `fmax`, `cmap`, etc.
- `prompts:` — one entry per `prompt_N` folder: `prompt` text, optional `context`, optional
  `timestep`/`layers` overrides, and a list of `sounds` (`id` + display `label`, optional
  `note`). Each sound renders as its own layer × alpha grid.

Optional per-sound explicit filenames (override the templates):

```yaml
sounds:
  - id: birds
    label: "add: birds chirping"
    files:
      unsteered: dog_only.wav                 # layer-independent α=0
      12: {10: L12_a10.wav, 20: L12_a20.wav}  # nested layer -> alpha
```

## Notes

- Spectrogram generation is idempotent + atomic (temp file + `os.replace`); re-runs skip
  existing PNGs unless `--force`. `fmax` is clamped to Nyquist automatically.
- Missing samples render as a labeled placeholder and are reported in the console.
- Bootstrap + fonts load from CDN (fine on GitHub Pages). To vendor them, drop
  `bootstrap.min.css` into `assets/` and repoint the `<link>` in the template.
# concept_discovery_web
