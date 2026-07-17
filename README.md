# Perch_notebooks

Standalone Jupyter notebooks that run Google **Perch V2** directly from Kaggle — no
`perchlab` or `perch-hoplite` code involved. The notebooks import only TensorFlow,
kagglehub, librosa, soundfile, numpy and pandas, and reproduce Perch's official
preprocessing by hand.

## Setup

The environment is built with [uv](https://docs.astral.sh/uv/) from `pyproject.toml`
(Python 3.11 + TensorFlow 2.21). It is already installed in `.venv/`. To rebuild:

```bash
cd ~/projects/Perch_notebooks
uv sync
```

In VS Code, open a notebook and select the kernel **Perch Notebooks (3.11)** (or
`.venv/bin/python`). `ModuleNotFoundError: No module named 'tensorflow'` almost always
means the wrong kernel is selected.

### Kaggle credentials

Perch V2 downloads from Kaggle on first use (~780 MB, then cached in
`~/.cache/kagglehub`). Accept the model terms once on the
[model page](https://www.kaggle.com/models/google/bird-vocalization-classifier), create
a token (Kaggle → *Settings* → *API*), and export it **before** launching VS Code so the
kernel inherits it:

```bash
export KAGGLE_API_TOKEN=KGAT_xxxxxxxxxxxxxxxxxxxxxxxx
```

`~/.kaggle/kaggle.json` or `KAGGLE_USERNAME` + `KAGGLE_KEY` also work.

### Audio codecs

`.wav`, `.flac` and `.ogg` need nothing extra — `libsndfile` ships inside the
`soundfile` wheel. Only `.mp3` / `.m4a` require `sudo apt-get install -y ffmpeg`.

## Layout

```
Perch_notebooks/
  00_species_identification_guided.ipynb   Run Perch over recordings -> detections CSV.
  pyproject.toml                           Dependencies (uv).
  data/
    recordings/                            Put your audio here (searched recursively).
    outputs/                               Written by the notebooks.
```

Paths inside the notebook anchor to the folder containing `data/`, so they resolve the
same whether VS Code starts the kernel in the notebook's folder or you launch
`jupyter lab` from elsewhere.

## Notes

- Perch V2 is **not** amplitude-invariant. Each 5-second window must have its DC offset
  removed and its peak scaled to 0.25 before inference — the notebook's `peak_normalize`
  does this. Skipping it yields systematically low confidence scores.
- The model's window is fixed at **5 s** (the signature is `(None, 160000)`); only the
  *hop* between windows is tunable.
- `data/recordings/` currently holds **symlinks** to the 30 PIC02/PIC03 recordings in the
  neighbouring `PerchLab` project, so the notebook runs as-is. They are gitignored — a
  fresh clone starts with an empty `data/recordings/`. Delete them and drop your own
  audio in instead.
- **Symlinked folders are not searched.** The notebook finds files with
  `Path(root).rglob("*")`, and pathlib deliberately skips symlinked *sub-directories*
  when recursing (loop protection). Symlinked *files* are found fine, which is why the
  links above are per-file. If your audio lives on another drive, either symlink the
  individual files, make `data/recordings` itself the symlink, or just point `INPUT_DIR`
  straight at the real folder.
