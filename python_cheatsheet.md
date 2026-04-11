# Python Cheat Sheet — Beyond the Basics

A practical reference for Python patterns and tools commonly used by data engineers, data analysts, and data scientists. Not the basics — the stuff that actually comes up in professional work.

---

## Error Handling

### try / except / else / finally
```python
try:
    result = int("abc")
except ValueError as e:
    print(f"Conversion failed: {e}")
except (TypeError, KeyError) as e:
    print(f"Type or key error: {e}")
else:
    print("No error occurred")       # runs only if no exception was raised
finally:
    print("This always runs")        # cleanup — always executes
```

### Raise a custom error
```python
def load_file(path):
    if not path.endswith(".csv"):
        raise ValueError(f"Expected a CSV file, got: {path}")
```

### Catch all exceptions and log them
```python
try:
    process()
except Exception as e:
    print(f"Unexpected error: {type(e).__name__}: {e}")
```

---

## File Handling

### Read a file
```python
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()           # entire file as a string
    lines = f.readlines()        # list of lines
```

### Write to a file
```python
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("hello\n")
```

### Append to a file
```python
with open("log.txt", "a", encoding="utf-8") as f:
    f.write("new line\n")
```

### Read a file line by line (memory efficient)
```python
with open("large_file.txt", "r") as f:
    for line in f:
        process(line.strip())
```

### Read and write JSON
```python
import json

with open("config.json", "r") as f:
    config = json.load(f)

with open("output.json", "w") as f:
    json.dump(data, f, indent=2)
```

### Read and write JSON lines (one JSON object per line)
```python
# Write
with open("data.jsonl", "w") as f:
    for record in records:
        f.write(json.dumps(record) + "\n")

# Read
with open("data.jsonl", "r") as f:
    records = [json.loads(line) for line in f]
```

---

## Logging

### Basic logging setup — use instead of print in pipelines
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

logger = logging.getLogger(__name__)

logger.debug("debug detail")
logger.info("pipeline started")
logger.warning("missing values detected")
logger.error("failed to connect to database")
logger.critical("pipeline aborted")
```

### Log to both console and a file
```python
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

console_handler = logging.StreamHandler()
file_handler = logging.FileHandler("pipeline.log")

formatter = logging.Formatter("%(asctime)s | %(levelname)s | %(message)s")
console_handler.setFormatter(formatter)
file_handler.setFormatter(formatter)

logger.addHandler(console_handler)
logger.addHandler(file_handler)
```

---

## Command Line Arguments

### Accept arguments when running a script
```python
import argparse

parser = argparse.ArgumentParser(description="Run the data pipeline")
parser.add_argument("--input",  required=True, help="Path to input file")
parser.add_argument("--output", required=True, help="Path to output file")
parser.add_argument("--env",    default="dev", help="Environment: dev or prod")
parser.add_argument("--dry-run", action="store_true", help="Run without writing output")

args = parser.parse_args()

print(args.input)
print(args.env)
print(args.dry_run)    # True if flag is passed, False otherwise
```

### Run the script
```bash
python pipeline.py --input data/raw.csv --output data/clean.csv --env prod
```

---

## Type Hints

### Basic type hints
```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}"
```

### Common types from the `typing` module
```python
from typing import List, Dict, Tuple, Optional, Union, Any

def process(records: List[Dict[str, Any]]) -> List[str]:
    return [r["name"] for r in records]

def find_user(id: int) -> Optional[str]:    # can return str or None
    ...

def parse(value: Union[int, str]) -> str:   # accepts int or str
    return str(value)
```

### Modern syntax (Python 3.10+)
```python
def find_user(id: int) -> str | None:
    ...

def parse(value: int | str) -> str:
    ...
```

---

## Dataclasses

### Define a clean data structure without boilerplate
```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class PipelineConfig:
    input_path: str
    output_path: str
    batch_size: int = 1000
    tags: List[str] = field(default_factory=list)

config = PipelineConfig(input_path="data/raw.csv", output_path="data/clean.csv")
print(config.batch_size)    # 1000
```

---

## Comprehensions

### List comprehension
```python
squares = [x ** 2 for x in range(10)]
evens   = [x for x in range(20) if x % 2 == 0]
```

### Dict comprehension
```python
word_lengths = {word: len(word) for word in ["hello", "world"]}
flipped      = {v: k for k, v in original.items()}
```

### Set comprehension
```python
unique_domains = {email.split("@")[1] for email in emails}
```

### Nested comprehension — flatten a list of lists
```python
flat = [item for sublist in nested for item in sublist]
```

---

## Generators

### Generator function — yields one item at a time, memory efficient
```python
def read_chunks(filepath, chunk_size=1000):
    with open(filepath) as f:
        chunk = []
        for line in f:
            chunk.append(line)
            if len(chunk) == chunk_size:
                yield chunk
                chunk = []
        if chunk:
            yield chunk

for chunk in read_chunks("large_file.csv"):
    process(chunk)
```

### Generator expression
```python
total = sum(x ** 2 for x in range(1_000_000))   # no list created in memory
```

---

## Useful Built-ins

### enumerate — loop with index
```python
for i, item in enumerate(items):
    print(i, item)

for i, item in enumerate(items, start=1):    # start index at 1
    print(i, item)
```

### zip — loop over multiple lists together
```python
for name, score in zip(names, scores):
    print(name, score)

pairs = list(zip(names, scores))
```

### zip_longest — zip with unequal lengths
```python
from itertools import zip_longest

for a, b in zip_longest(list1, list2, fillvalue=None):
    ...
```

### map — apply a function to every item
```python
cleaned = list(map(str.strip, raw_strings))
numbers = list(map(int, string_numbers))
```

### filter — keep items matching a condition
```python
valid = list(filter(lambda x: x is not None, items))
```

### sorted with a key
```python
records = sorted(records, key=lambda r: r["date"], reverse=True)
```

### any / all
```python
any(x > 0 for x in values)     # True if at least one is positive
all(x > 0 for x in values)     # True if all are positive
```

---

## collections

### Counter — count occurrences
```python
from collections import Counter

counts = Counter(["a", "b", "a", "c", "a", "b"])
counts.most_common(3)    # [('a', 3), ('b', 2), ('c', 1)]
```

### defaultdict — dict with a default value for missing keys
```python
from collections import defaultdict

groups = defaultdict(list)
for record in records:
    groups[record["department"]].append(record)
```

### namedtuple — lightweight immutable data structure
```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
print(p.x, p.y)
```

### deque — fast append and pop from both ends
```python
from collections import deque

queue = deque(maxlen=100)    # fixed-size sliding window
queue.append(item)
queue.appendleft(item)
queue.pop()
queue.popleft()
```

---

## itertools

### chain — flatten multiple iterables into one
```python
from itertools import chain

combined = list(chain([1, 2], [3, 4], [5, 6]))
# [1, 2, 3, 4, 5, 6]
```

### islice — take the first N items from an iterator
```python
from itertools import islice

first_10 = list(islice(my_generator, 10))
```

### groupby — group consecutive items by a key
```python
from itertools import groupby

data = sorted(records, key=lambda r: r["region"])
for region, group in groupby(data, key=lambda r: r["region"]):
    print(region, list(group))
```

### batched — split an iterable into fixed-size batches (Python 3.12+)
```python
from itertools import batched

for batch in batched(records, 100):
    process(batch)
```

---

## functools

### lru_cache — cache the result of expensive function calls
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def get_config(env: str) -> dict:
    return load_from_db(env)
```

### partial — fix some arguments of a function
```python
from functools import partial

def multiply(x, y):
    return x * y

double = partial(multiply, y=2)
double(5)    # 10
```

### reduce — cumulatively apply a function
```python
from functools import reduce

total = reduce(lambda acc, x: acc + x, [1, 2, 3, 4, 5])
# 15
```

---

## Decorators

### Write a simple decorator
```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def run_pipeline():
    ...
```

### Write a decorator that accepts arguments
```python
def retry(max_attempts=3):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
        return wrapper
    return decorator

@retry(max_attempts=5)
def fetch_data():
    ...
```

---

## Context Managers

### Write your own context manager with a class
```python
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_to_db()
        return self.conn

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        return False    # don't suppress exceptions

with DatabaseConnection() as conn:
    conn.execute("SELECT 1")
```

### Write a context manager with contextlib
```python
from contextlib import contextmanager

@contextmanager
def timer(label=""):
    start = time.time()
    yield
    print(f"{label} took {time.time() - start:.2f}s")

with timer("data loading"):
    df = pd.read_csv("large.csv")
```

---

## subprocess — Run Shell Commands from Python

### Run a command and wait for it to finish
```python
import subprocess

subprocess.run(["python", "other_script.py"])
subprocess.run(["ls", "-lh", "/data"])
```

### Capture the output
```python
result = subprocess.run(
    ["ls", "-lh", "/data"],
    capture_output=True,
    text=True
)
print(result.stdout)
print(result.stderr)
```

### Run a shell command as a string
```python
result = subprocess.run("ls -lh /data | grep csv", shell=True, capture_output=True, text=True)
print(result.stdout)
```

### Check for errors
```python
result = subprocess.run(["python", "pipeline.py"], capture_output=True, text=True)
if result.returncode != 0:
    raise RuntimeError(f"Pipeline failed:\n{result.stderr}")
```

---

## datetime

### Get current date and time
```python
from datetime import datetime, date, timedelta

now   = datetime.now()
today = date.today()
```

### Format a datetime as a string
```python
now.strftime("%Y-%m-%d %H:%M:%S")    # "2024-06-15 08:30:00"
now.strftime("%Y%m%d")               # "20240615"
```

### Parse a string into a datetime
```python
dt = datetime.strptime("2024-06-15", "%Y-%m-%d")
```

### Add or subtract time
```python
yesterday  = datetime.now() - timedelta(days=1)
next_week  = datetime.now() + timedelta(weeks=1)
two_hours  = datetime.now() + timedelta(hours=2)
```

### Difference between two dates
```python
delta = datetime(2024, 12, 31) - datetime(2024, 1, 1)
print(delta.days)    # 365
```

---

## Regular Expressions

### Basic pattern matching
```python
import re

re.match(r"^\d+$", "1234")           # matches from the start
re.search(r"\d+", "order_123_abc")   # find anywhere in the string
re.findall(r"\d+", "a1 b22 c333")    # ["1", "22", "333"]
re.sub(r"\s+", "_", "hello world")   # "hello_world"
```

### Common patterns
```python
r"\d+"          # one or more digits
r"[a-zA-Z]+"    # one or more letters
r"^\d{4}-\d{2}-\d{2}$"    # date format YYYY-MM-DD
r"[\w.-]+@[\w.-]+\.\w+"   # email address
r"https?://\S+"            # URL
```

### Compile a pattern for reuse
```python
date_pattern = re.compile(r"^\d{4}-\d{2}-\d{2}$")
date_pattern.match("2024-06-15")
```

---

## ETL Patterns

### Retry a function with exponential backoff
```python
import time

def retry(func, max_attempts=3, wait=2):
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            time.sleep(wait ** attempt)
```

### Time a block of code
```python
import time

start = time.time()
run_pipeline()
print(f"Finished in {time.time() - start:.2f}s")
```

### Read a config file and validate required keys
```python
import json

with open("config.json") as f:
    config = json.load(f)

required = ["input_path", "output_path", "batch_size"]
missing = [k for k in required if k not in config]
if missing:
    raise ValueError(f"Missing config keys: {missing}")
```

### Process files in a folder and log progress
```python
from pathlib import Path
import logging

logger = logging.getLogger(__name__)

for file in Path("/data/raw").glob("*.csv"):
    try:
        logger.info(f"Processing {file.name}")
        process(file)
        logger.info(f"Done: {file.name}")
    except Exception as e:
        logger.error(f"Failed: {file.name} — {e}")
```

### Write a pipeline script with argparse and logging
```python
import argparse
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s | %(levelname)s | %(message)s")
logger = logging.getLogger(__name__)

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--input", required=True)
    parser.add_argument("--output", required=True)
    args = parser.parse_args()

    logger.info(f"Reading from {args.input}")
    # pipeline logic here
    logger.info(f"Writing to {args.output}")

if __name__ == "__main__":
    main()
```

---

*Last updated: 2026*
