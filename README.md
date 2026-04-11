# DE Cheat Sheets

A personal collection of cheat sheets for tools, languages, and frameworks I use in data engineering work. I come back to these when I forget a command, want to review a concept, or need a quick reference without digging through documentation.

These are living documents! I add to them as I encounter new patterns, discover better ways of doing things, or start working with new tools.

---

## What's in Here

### Core Languages & Libraries

| File                   | What it covers                                                                                                                                                                                  |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `python_cheatsheet.md` | Beyond-the-basics Python — error handling, file I/O, logging, argparse, type hints, dataclasses, generators, decorators, context managers, subprocess, regex, collections, itertools, functools |
| `pandas_cheatsheet.md` | Reading, selecting, transforming, cleaning, aggregating, joining, date handling, writing data, ETL patterns                                                                                     |
| `numpy_cheatsheet.md`  | Array creation, indexing, reshaping, math, aggregations, normalization, ETL patterns                                                                                                            |

### Data Engineering & Pipelines

| File                    | What it covers                                                                                                                                                                            |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pyspark_cheatsheet.md` | Spark sessions, reading/writing data, transformations, cleaning, aggregations, joins, window functions, performance, Spark ML (regression, classification, clustering, pipelines, tuning) |
| `sql_cheatsheet.md`     | Table management, querying, aggregations, joins, CTEs, window functions, date/string functions, views, indexes, ETL patterns                                                              |
| `airflow_cheatsheet.md` | DAG definition, operators, task dependencies, XCom, variables, connections, CLI, ETL patterns                                                                                             |
| `kafka_cheatsheet.md`   | Producers and consumers with kafka-python and confluent-kafka, topic management, key concepts, ETL patterns                                                                               |

### Machine Learning

| File               | What it covers                                                                                                             |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `ml_cheatsheet.md` | scikit-learn — shared workflow, regression, classification, clustering, model tuning, pipelines, saving and loading models |

### Infrastructure & Tools

| File                   | What it covers                                                                                                                          |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `docker_cheatsheet.md` | Images, containers, volumes, networks, Docker Compose, Dockerfile reference, ETL patterns                                               |
| `git_cheatsheet.md`    | First-time setup, daily workflow, branching, merging, undoing mistakes, remotes, stashing, .gitignore, ETL patterns                     |
| `linux_cheatsheet.md`  | Navigation, file operations, viewing files, searching, permissions, disk usage, processes, compression, networking, pipes, ETL patterns |

### Python Utilities

| File                    | What it covers                                                                                 |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| `os_cheatsheet.md`      | Working with paths, directories, environment variables, shell commands using the `os` module   |
| `pathlib_cheatsheet.md` | Modern path handling — building paths, listing files, glob patterns, reading and writing files |
| `dotenv_cheatsheet.md`  | Managing secrets and config with `.env` files using python-dotenv                              |

---

## How I Use These

- **When I forget a command** — open the relevant file and scan the tables
- **When starting something new** — read the ETL Patterns section at the bottom of each file, they show the most common real-world combinations
- **When reviewing** — go through a file before starting a new project or task that involves that tool
- **Cross-references** — the `ml_cheatsheet.md` covers scikit-learn; for the PySpark equivalent see the Spark ML section at the bottom of `pyspark_cheatsheet.md`

---

_Last updated: 2026_
