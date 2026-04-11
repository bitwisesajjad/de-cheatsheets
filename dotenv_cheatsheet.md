# `python-dotenv` Cheat Sheet for Data Engineering

A practical reference for managing environment variables and secrets using `.env` files in Python.

---

## Setup

```bash
pip install python-dotenv
```

```python
from dotenv import load_dotenv
import os
```

---

## The `.env` File

A plain text file that stores key-value pairs. Never commit this file to Git.

```dotenv
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mydb
DB_USER=admin
DB_PASSWORD=secret

S3_BUCKET=my-data-bucket
API_KEY=abc123xyz
ENV=development
```

---

## Loading Variables

### Load `.env` into the environment
```python
load_dotenv()
```

### Load a specific `.env` file
```python
load_dotenv("/path/to/.env.production")
```

### Load without overriding existing environment variables
```python
load_dotenv(override=False)
```

### Load and override existing environment variables
```python
load_dotenv(override=True)
```

---

## Reading Variables

### Read a variable after loading
```python
load_dotenv()
db_host = os.environ.get("DB_HOST")
```

### Read with a fallback default
```python
db_port = os.environ.get("DB_PORT", "5432")
```

### Read a required variable — raise an error if missing
```python
api_key = os.environ.get("API_KEY")
if not api_key:
    raise ValueError("API_KEY is not set in the environment")
```

---

## dotenv_values — Read Without Polluting the Environment

Reads the `.env` file into a plain dictionary without setting system environment variables.

```python
from dotenv import dotenv_values

config = dotenv_values(".env")
db_host = config["DB_HOST"]
```

### Merge multiple env files
```python
config = {
    **dotenv_values(".env"),
    **dotenv_values(".env.local"),
    **os.environ
}
```

---

## `.gitignore` — Always Exclude `.env`

Add this to your `.gitignore` to prevent secrets from being committed:

```
.env
.env.*
!.env.example
```

---

## `.env.example` — Safe Template to Share

Commit this file instead of `.env` — same keys, no real values:

```dotenv
DB_HOST=
DB_PORT=
DB_NAME=
DB_USER=
DB_PASSWORD=

S3_BUCKET=
API_KEY=
ENV=
```

---

## Common Patterns

### Standard pipeline setup at the top of a script
```python
from dotenv import load_dotenv
import os

load_dotenv()

DB_HOST = os.environ.get("DB_HOST", "localhost")
DB_PORT = os.environ.get("DB_PORT", "5432")
DB_NAME = os.environ.get("DB_NAME")
DB_USER = os.environ.get("DB_USER")
DB_PASSWORD = os.environ.get("DB_PASSWORD")
```

### Build a database connection string from env variables
```python
load_dotenv()

conn_str = (
    f"postgresql://{os.environ['DB_USER']}:{os.environ['DB_PASSWORD']}"
    f"@{os.environ['DB_HOST']}:{os.environ['DB_PORT']}/{os.environ['DB_NAME']}"
)
```

### Switch behavior based on environment
```python
load_dotenv()

env = os.environ.get("ENV", "development")
if env == "production":
    output_path = "/data/prod/output"
else:
    output_path = "/data/dev/output"
```

### Load env file based on the current environment
```python
env = os.environ.get("ENV", "development")
load_dotenv(f".env.{env}")
```

### Use with Docker — pass `.env` file at runtime
```bash
docker run --env-file .env my_pipeline python pipeline.py
```

---

*Last updated: 2026*
