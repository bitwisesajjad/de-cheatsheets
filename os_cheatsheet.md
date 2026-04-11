# Python `os` Module Cheat Sheet for Data Engineering

A practical reference for the `os` module — working with files, folders, paths, and the environment in Python.

---

## Setup

```python
import os
```

---

## Current Directory

| Command | What it does |
|---------|--------------|
| `os.getcwd()` | Get the current working directory |
| `os.chdir("/path/to/folder")` | Change the current working directory |

---

## Listing & Checking Paths

| Command | What it does |
|---------|--------------|
| `os.listdir(".")` | List files and folders in a directory |
| `os.listdir("/data/raw")` | List contents of a specific path |
| `os.path.exists("file.txt")` | Check if a file or folder exists |
| `os.path.isfile("file.txt")` | Check if the path is a file |
| `os.path.isdir("folder/")` | Check if the path is a directory |
| `os.path.getsize("file.csv")` | Get file size in bytes |

---

## Building Paths

| Command | What it does |
|---------|--------------|
| `os.path.join("data", "raw", "file.csv")` | Build a path safely across OS types |
| `os.path.basename("/data/raw/file.csv")` | Get just the filename: `file.csv` |
| `os.path.dirname("/data/raw/file.csv")` | Get just the directory: `/data/raw` |
| `os.path.splitext("file.csv")` | Split into name and extension: `("file", ".csv")` |
| `os.path.abspath("file.csv")` | Get the full absolute path |

---

## Creating & Removing

| Command | What it does |
|---------|--------------|
| `os.mkdir("new_folder")` | Create a single directory |
| `os.makedirs("a/b/c")` | Create nested directories |
| `os.makedirs("a/b/c", exist_ok=True)` | Create nested dirs, no error if they exist |
| `os.remove("file.txt")` | Delete a file |
| `os.rmdir("empty_folder")` | Delete an empty directory |
| `os.rename("old.txt", "new.txt")` | Rename a file or folder |

---

## Walking a Directory Tree

### `os.walk` — traverse all files in a folder recursively
```python
for root, dirs, files in os.walk("/data"):
    for file in files:
        full_path = os.path.join(root, file)
        print(full_path)
```

---

## Environment Variables

| Command | What it does |
|---------|--------------|
| `os.environ` | Dictionary of all environment variables |
| `os.environ.get("DB_HOST")` | Get a variable, returns `None` if missing |
| `os.environ.get("DB_HOST", "localhost")` | Get with a default fallback |
| `os.environ["DB_HOST"] = "localhost"` | Set a variable for the current process |

---

## Running Shell Commands

| Command | What it does |
|---------|--------------|
| `os.system("ls -l")` | Run a shell command (no output capture) |

> For capturing output, prefer `subprocess` over `os.system`.

---

## ETL Patterns

### Build a path from parts safely
```python
path = os.path.join("/data", "raw", "2024", "orders.csv")
```

### Create output folder if it does not exist
```python
os.makedirs("/data/processed", exist_ok=True)
```

### List only CSV files in a folder
```python
files = [f for f in os.listdir("/data/raw") if f.endswith(".csv")]
```

### Find all CSV files recursively in a folder
```python
csv_files = []
for root, dirs, files in os.walk("/data"):
    for f in files:
        if f.endswith(".csv"):
            csv_files.append(os.path.join(root, f))
```

### Read a required environment variable safely
```python
db_host = os.environ.get("DB_HOST")
if not db_host:
    raise ValueError("DB_HOST environment variable is not set")
```

### Get the file name without its extension
```python
name, ext = os.path.splitext("orders_2024.csv")
# name = "orders_2024", ext = ".csv"
```

### Check if input file exists before processing
```python
input_path = "/data/raw/orders.csv"
if not os.path.exists(input_path):
    raise FileNotFoundError(f"Input file not found: {input_path}")
```

---

*Last updated: 2026*
