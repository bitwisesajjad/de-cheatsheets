# Linux & Bash Cheat Sheet for Data Engineering

A practical reference for Linux commands commonly used in data engineering, pipeline work, and server environments.

---

## Navigation

### `pwd`
Print the current working directory.

| Command | What it does |
|---------|--------------|
| `pwd` | Show full path of where you are |

---

### `cd`
Change directory.

| Command | What it does |
|---------|--------------|
| `cd folder_name` | Go into a folder |
| `cd ..` | Go up one level |
| `cd ../..` | Go up two levels |
| `cd -` | Go back to the previous directory |
| `cd ~` | Go to your home directory |
| `cd /` | Go to the root of the filesystem |

---

### `ls`
List contents of a directory.

| Command | What it does |
|---------|--------------|
| `ls` | List files and folders |
| `ls -l` | Long format with permissions, size, date |
| `ls -a` | Include hidden files (starting with `.`) |
| `ls -lh` | Long format with human-readable file sizes |
| `ls -lt` | Sort by modification time, newest first |
| `ls -R` | List recursively including subdirectories |

---

### `tree`
Show directory structure as a visual tree.

| Command | What it does |
|---------|--------------|
| `tree` | Show full tree of current directory |
| `tree -d` | Show only directories, no files |
| `tree -L 2` | Limit depth to 2 levels |
| `tree -a` | Include hidden files |
| `tree -h` | Show file sizes in human-readable format |

---

## Files & Folders

### `mkdir`
Create a new directory.

| Command | What it does |
|---------|--------------|
| `mkdir my_folder` | Create a single directory |
| `mkdir -p a/b/c` | Create nested directories in one shot |

---

### `touch`
Create a new empty file or update a file's timestamp.

| Command | What it does |
|---------|--------------|
| `touch file.txt` | Create an empty file |
| `touch file1.txt file2.txt` | Create multiple files at once |

---

### `cp`
Copy files or directories.

| Command | What it does |
|---------|--------------|
| `cp file.txt copy.txt` | Copy a file |
| `cp file.txt folder/` | Copy a file into a folder |
| `cp -r folder/ backup/` | Copy a folder and all its contents |
| `cp -u file.txt dest/` | Copy only if source is newer than destination |

---

### `mv`
Move or rename files and directories.

| Command | What it does |
|---------|--------------|
| `mv file.txt folder/` | Move a file into a folder |
| `mv old_name.txt new_name.txt` | Rename a file |
| `mv folder/ new_folder/` | Rename a folder |
| `mv *.csv data/` | Move all CSV files into a folder |

---

### `rm`
Delete files or directories.

| Command | What it does |
|---------|--------------|
| `rm file.txt` | Delete a file |
| `rm -i file.txt` | Ask for confirmation before deleting |
| `rm -r folder/` | Delete a folder and everything inside it |
| `rm -f file.txt` | Force delete without confirmation |
| `rm -rf folder/` | Force delete folder recursively — use with caution |

---

### `rename`
Rename multiple files at once using a pattern.

| Command | What it does |
|---------|--------------|
| `rename 's/.txt/.csv/' *.txt` | Rename all `.txt` files to `.csv` |

---

## Viewing & Reading Files

### `cat`
Print file contents to the terminal.

| Command | What it does |
|---------|--------------|
| `cat file.txt` | Print entire file |
| `cat file1.txt file2.txt` | Print multiple files in sequence |
| `cat -n file.txt` | Print with line numbers |

---

### `less`
View a file one screen at a time (use arrow keys to scroll, `q` to quit).

| Command | What it does |
|---------|--------------|
| `less file.txt` | Open file in scrollable view |
| `less +G file.txt` | Open and jump to the end |

---

### `head`
Show the first N lines of a file.

| Command | What it does |
|---------|--------------|
| `head file.txt` | Show first 10 lines |
| `head -n 50 file.txt` | Show first 50 lines |
| `head -n 1 file.csv` | Show only the header row of a CSV |

---

### `tail`
Show the last N lines of a file.

| Command | What it does |
|---------|--------------|
| `tail file.txt` | Show last 10 lines |
| `tail -n 50 file.txt` | Show last 50 lines |
| `tail -f log.txt` | Follow a file in real time as it grows |
| `tail -f -n 100 log.txt` | Follow and show last 100 lines |

---

### `wc`
Count lines, words, or characters in a file.

| Command | What it does |
|---------|--------------|
| `wc -l file.txt` | Count number of lines |
| `wc -w file.txt` | Count number of words |
| `wc -c file.txt` | Count number of bytes |

---

## Searching & Filtering

### `grep`
Search for a pattern inside files.

| Command | What it does |
|---------|--------------|
| `grep "error" log.txt` | Find lines containing "error" |
| `grep -i "error" log.txt` | Case-insensitive search |
| `grep -r "error" logs/` | Search recursively in a folder |
| `grep -n "error" log.txt` | Show line numbers with matches |
| `grep -v "error" log.txt` | Show lines that do NOT match |
| `grep -c "error" log.txt` | Count how many lines match |

---

### `find`
Search for files and directories by name, type, or date.

| Command | What it does |
|---------|--------------|
| `find . -name "*.csv"` | Find all CSV files from current directory |
| `find . -type d` | Find only directories |
| `find . -type f` | Find only files |
| `find . -name "*.log" -mtime -1` | Find log files modified in the last 1 day |
| `find . -size +100M` | Find files larger than 100 MB |
| `find . -empty` | Find empty files or folders |

---

### `sort`
Sort lines in a file.

| Command | What it does |
|---------|--------------|
| `sort file.txt` | Sort alphabetically |
| `sort -r file.txt` | Sort in reverse order |
| `sort -n file.txt` | Sort numerically |
| `sort -u file.txt` | Sort and remove duplicates |
| `sort -t',' -k2 file.csv` | Sort CSV by the second column |

---

### `uniq`
Remove or count duplicate lines (works best after `sort`).

| Command | What it does |
|---------|--------------|
| `uniq file.txt` | Remove consecutive duplicate lines |
| `sort file.txt \| uniq` | Remove all duplicates |
| `sort file.txt \| uniq -c` | Count how many times each line appears |
| `sort file.txt \| uniq -d` | Show only duplicate lines |

---

### `cut`
Extract specific columns from a file.

| Command | What it does |
|---------|--------------|
| `cut -d',' -f1 file.csv` | Extract the first column of a CSV |
| `cut -d',' -f1,3 file.csv` | Extract the first and third columns |
| `cut -d',' -f2- file.csv` | Extract from column 2 to the end |

---

### `awk`
Process and transform text by column — very powerful for data files.

| Command | What it does |
|---------|--------------|
| `awk -F',' '{print $1}' file.csv` | Print first column of a CSV |
| `awk -F',' '{print $1, $3}' file.csv` | Print first and third columns |
| `awk -F',' 'NR>1' file.csv` | Skip the header row |
| `awk -F',' '$2 > 100' file.csv` | Print rows where column 2 is greater than 100 |
| `awk -F',' '{sum += $2} END {print sum}' file.csv` | Sum all values in column 2 |

---

### `sed`
Find and replace text in files or streams.

| Command | What it does |
|---------|--------------|
| `sed 's/old/new/g' file.txt` | Replace all occurrences of "old" with "new" |
| `sed -i 's/old/new/g' file.txt` | Replace in-place (modifies the file) |
| `sed '1d' file.csv` | Delete the first line (e.g. remove header) |
| `sed -n '10,20p' file.txt` | Print only lines 10 to 20 |

---

## File Permissions

### `chmod`
Change file permissions.

| Command | What it does |
|---------|--------------|
| `chmod +x script.sh` | Make a script executable |
| `chmod 644 file.txt` | Owner can read/write, others can read |
| `chmod 755 folder/` | Owner full access, others can read and execute |
| `chmod -R 755 folder/` | Apply permissions recursively |

---

### `chown`
Change file ownership.

| Command | What it does |
|---------|--------------|
| `chown user file.txt` | Change owner of a file |
| `chown user:group file.txt` | Change owner and group |
| `chown -R user:group folder/` | Change ownership recursively |

---

## Disk & Storage

### `df`
Show disk space usage of the filesystem.

| Command | What it does |
|---------|--------------|
| `df -h` | Show disk usage in human-readable format |
| `df -h /home` | Show usage for a specific path |

---

### `du`
Show how much space a file or folder is using.

| Command | What it does |
|---------|--------------|
| `du -h file.txt` | Show size of a file |
| `du -sh folder/` | Show total size of a folder |
| `du -sh *` | Show size of every item in current directory |
| `du -sh * \| sort -h` | Show sizes sorted from smallest to largest |

---

## Processes & System

### `ps`
Show running processes.

| Command | What it does |
|---------|--------------|
| `ps aux` | Show all running processes with details |
| `ps aux \| grep python` | Find all running Python processes |

---

### `kill`
Stop a running process.

| Command | What it does |
|---------|--------------|
| `kill PID` | Gracefully stop a process by its ID |
| `kill -9 PID` | Force kill a process immediately |

---

### `top` / `htop`
Monitor system resource usage interactively.

| Command | What it does |
|---------|--------------|
| `top` | Show live CPU and memory usage |
| `htop` | Nicer interactive version of top (may need install) |

---

### `nohup`
Run a command that keeps running after you log out.

| Command | What it does |
|---------|--------------|
| `nohup python pipeline.py &` | Run a script in the background, persist after logout |
| `nohup python pipeline.py > out.log 2>&1 &` | Same, and save all output to a log file |

---

### `cron`
Schedule commands to run automatically at set times.

| Command | What it does |
|---------|--------------|
| `crontab -e` | Open the cron schedule for editing |
| `crontab -l` | List all scheduled cron jobs |
| `0 2 * * * python /path/pipeline.py` | Run a script every day at 2 AM |

---

## Compression & Archiving

### `tar`
Archive and compress folders.

| Command | What it does |
|---------|--------------|
| `tar -czf archive.tar.gz folder/` | Compress a folder into a `.tar.gz` file |
| `tar -xzf archive.tar.gz` | Extract a `.tar.gz` file |
| `tar -tzf archive.tar.gz` | List contents without extracting |

---

### `gzip` / `gunzip`
Compress or decompress individual files.

| Command | What it does |
|---------|--------------|
| `gzip file.csv` | Compress a file to `file.csv.gz` |
| `gunzip file.csv.gz` | Decompress back to `file.csv` |
| `gzip -k file.csv` | Compress but keep the original file |

---

### `zip` / `unzip`
Create and extract `.zip` archives.

| Command | What it does |
|---------|--------------|
| `zip archive.zip file1 file2` | Zip specific files |
| `zip -r archive.zip folder/` | Zip an entire folder |
| `unzip archive.zip` | Extract a zip archive |
| `unzip -l archive.zip` | List contents without extracting |

---

## Networking & File Transfer

### `wget`
Download a file from a URL.

| Command | What it does |
|---------|--------------|
| `wget https://example.com/file.csv` | Download a file |
| `wget -O output.csv https://example.com/file.csv` | Download and rename the file |
| `wget -r https://example.com/data/` | Download recursively from a URL |

---

### `curl`
Transfer data to or from a server — also used to call APIs.

| Command | What it does |
|---------|--------------|
| `curl https://example.com/file.csv` | Fetch and print response |
| `curl -o file.csv https://example.com/file.csv` | Save response to a file |
| `curl -X POST -H "Content-Type: application/json" -d '{"key":"val"}' https://api.example.com` | Send a POST request to an API |

---

### `scp`
Securely copy files between local and remote machines.

| Command | What it does |
|---------|--------------|
| `scp file.txt user@host:/path/` | Copy a local file to a remote server |
| `scp user@host:/path/file.txt .` | Copy a file from a remote server to current directory |
| `scp -r folder/ user@host:/path/` | Copy a folder to a remote server |

---

### `ssh`
Connect to a remote server securely.

| Command | What it does |
|---------|--------------|
| `ssh user@host` | Connect to a remote server |
| `ssh -i key.pem user@host` | Connect using a private key file |

---

## Redirects & Pipes

### Redirect output to a file
| Command | What it does |
|---------|--------------|
| `command > file.txt` | Write output to a file (overwrites) |
| `command >> file.txt` | Append output to a file |
| `command 2> error.log` | Write errors to a file |
| `command > out.txt 2>&1` | Write both output and errors to a file |

---

### Pipe output between commands
| Command | What it does |
|---------|--------------|
| `command1 \| command2` | Pass output of command1 as input to command2 |
| `cat file.csv \| grep "error"` | Filter lines containing "error" from a file |
| `ps aux \| grep python \| grep -v grep` | Find Python processes cleanly |

---

## Environment & Variables

### `echo`
Print a value or message to the terminal.

| Command | What it does |
|---------|--------------|
| `echo "hello"` | Print text |
| `echo $HOME` | Print value of an environment variable |
| `echo $PATH` | Print the system PATH |

---

### `export`
Set an environment variable for the current session.

| Command | What it does |
|---------|--------------|
| `export VAR=value` | Set an environment variable |
| `export SPARK_HOME=/opt/spark` | Example: set Spark home path |

---

### `env` / `printenv`
View environment variables.

| Command | What it does |
|---------|--------------|
| `env` | Print all environment variables |
| `printenv HOME` | Print a specific variable |

---

## ETL Patterns

### Check how many rows a CSV file has
```bash
wc -l data.csv
```

### Preview the header and first few rows of a CSV
```bash
head -n 5 data.csv
```

### Find all CSV files modified in the last 24 hours
```bash
find /data -name "*.csv" -mtime -1
```

### Check total size of a data folder
```bash
du -sh /data/raw/
```

### Watch a pipeline log file in real time
```bash
tail -f logs/pipeline.log
```

### Count occurrences of a value in a column
```bash
cut -d',' -f2 data.csv | sort | uniq -c | sort -rn
```

### Remove the header from a CSV and save the data only
```bash
tail -n +2 data.csv > data_no_header.csv
```

### Combine multiple CSV files into one (without repeating the header)
```bash
head -n 1 file1.csv > combined.csv
tail -n +2 -q *.csv >> combined.csv
```

### Find and replace a value across a file
```bash
sed -i 's/NULL/0/g' data.csv
```

### Extract a single column from a CSV and get unique values
```bash
cut -d',' -f3 data.csv | sort | uniq
```

### Run a Python pipeline in the background and log output
```bash
nohup python pipeline.py > logs/pipeline.log 2>&1 &
```

### Check if a process is still running
```bash
ps aux | grep pipeline.py | grep -v grep
```

### Archive and compress a folder of processed data
```bash
tar -czf processed_2024_06.tar.gz /data/processed/2024/06/
```

### Download a dataset from a URL and save it
```bash
wget -O raw_data.csv https://example.com/dataset.csv
```

### Call an API and save the response to a file
```bash
curl -s https://api.example.com/data > response.json
```

### Count how many files are in a folder
```bash
find /data/raw -type f | wc -l
```

---

*Last updated: 2026*
