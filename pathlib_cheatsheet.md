# Python `pathlib` Cheat Sheet for Data Engineering

A practical reference for `pathlib` — the modern, cleaner way to work with file paths in Python. Prefer this over `os.path` in new code.

---

## Setup

```python
from pathlib import Path
```

---

## Creating a Path

| Command | What it does |
|---------|--------------|
| `Path("data/raw/file.csv")` | Create a path object |
| `Path.cwd()` | Current working directory |
| `Path.home()` | Home directory of the current user |
| `Path("/data") / "raw" / "file.csv"` | Build a path with `/` operator |

---

## Inspecting a Path

| Command | What it does |
|---------|--------------|
| `p.name` | Filename with extension: `file.csv` |
| `p.stem` | Filename without extension: `file` |
| `p.suffix` | Extension only: `.csv` |
| `p.suffixes` | All extensions: `[".tar", ".gz"]` |
| `p.parent` | Parent directory |
| `p.parts` | Path split into a tuple of parts |
| `p.resolve()` | Absolute path, resolves symlinks |

---

## Checking a Path

| Command | What it does |
|---------|--------------|
| `p.exists()` | True if the path exists |
| `p.is_file()` | True if it is a file |
| `p.is_dir()` | True if it is a directory |
| `p.stat().st_size` | File size in bytes |

---

## Creating & Removing

| Command | What it does |
|---------|--------------|
| `p.mkdir()` | Create a directory |
| `p.mkdir(parents=True, exist_ok=True)` | Create nested dirs, no error if exists |
| `p.touch()` | Create an empty file |
| `p.unlink()` | Delete a file |
| `p.rmdir()` | Delete an empty directory |
| `p.rename(new_path)` | Rename or move a file |

---

## Reading & Writing Files

| Command | What it does |
|---------|--------------|
| `p.read_text()` | Read file as a string |
| `p.read_text(encoding="utf-8")` | Read with explicit encoding |
| `p.write_text("content")` | Write a string to a file |
| `p.read_bytes()` | Read file as bytes |
| `p.write_bytes(b"data")` | Write bytes to a file |

---

## Listing & Searching

| Command | What it does |
|---------|--------------|
| `p.iterdir()` | Iterate over contents of a directory |
| `p.glob("*.csv")` | Find files matching a pattern |
| `p.glob("**/*.csv")` | Find CSV files recursively |
| `p.rglob("*.csv")` | Shorthand for recursive glob |

---

## ETL Patterns

### Build a path cleanly with the `/` operator
```python
path = Path("/data") / "raw" / "2024" / "orders.csv"
```

### Create output directory if it does not exist
```python
output_dir = Path("/data/processed")
output_dir.mkdir(parents=True, exist_ok=True)
```

### List all CSV files in a folder
```python
csv_files = list(Path("/data/raw").glob("*.csv"))
```

### Find all CSV files recursively
```python
csv_files = list(Path("/data").rglob("*.csv"))
```

### Get the filename without extension
```python
path = Path("/data/raw/orders_2024.csv")
name = path.stem   # "orders_2024"
```

### Build an output path based on the input filename
```python
input_path = Path("/data/raw/orders_2024.csv")
output_path = Path("/data/processed") / (input_path.stem + "_clean.csv")
# /data/processed/orders_2024_clean.csv
```

### Check if input file exists before processing
```python
input_path = Path("/data/raw/orders.csv")
if not input_path.exists():
    raise FileNotFoundError(f"Input file not found: {input_path}")
```

### Read a config file
```python
config_text = Path("config.json").read_text(encoding="utf-8")
```

### Iterate over all files in a folder and process each one
```python
for file in Path("/data/raw").glob("*.csv"):
    df = pd.read_csv(file)
    df.to_parquet(Path("/data/processed") / file.with_suffix(".parquet").name)
```

---

*Last updated: 2026*
