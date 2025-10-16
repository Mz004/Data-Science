
# ⚡️ Master Linux Commands Cheatsheet — for Data Scientists

> Copy-paste ready snippets to **explore data**, **manage environments**, **move files**, and **keep jobs running** on local machines or remote servers.

## 0) Shell superpowers (read this first)

```bash
# Pipes & redirection
cmd1 | cmd2 | cmd3        # chain commands
cmd > out.txt             # overwrite file
cmd >> out.txt            # append to file
cmd 2> err.txt            # redirect only stderr
cmd | tee out.txt         # see output AND save it

# Globbing & quoting
ls data/*.csv             # wildcard expansion
echo "A B"                # keep spaces together
````

## 1) Navigation & help

```bash
pwd                       # where am I?
ls -lah                   # list with sizes, human-readable
cd /path/to/dir           # move
mkdir -p data/raw         # make nested folders

# Help
man grep                  # manual page (q to quit)
grep --help               # quick flags
type -a python            # how a command is resolved
which python              # path to the executable
```

## 2) Files & safety

```bash
cp src.csv dst.csv        # copy
mv old.csv archive/       # move/rename
rm file.txt               # remove (⚠️ careful)
rm -rf folder/            # recursive delete (⚠️ DANGEROUS)
touch notes.txt           # create empty file / update timestamp
stat file.csv             # metadata
```

## 3) Quick looks at data (CSV/TSV/logs)

```bash
head -n 5 data.csv        # first 5 rows
tail -n 5 data.csv        # last 5 rows
nl -ba data.csv | head    # show line numbers
wc -l data.csv            # line count (≈ rows)
du -h data.csv            # file size
column -s, -t data.csv | less -S   # pretty table for simple CSVs
```

> Tip: `less -S` enables horizontal scroll; use arrows.

## 4) Searching text & finding files

```bash
grep -Rni "pattern" .                 # recursive, line numbers, case-insensitive
grep -Rnw --include="*.py" "fit(" .   # word match in .py files
zgrep -n "ERROR" logs.gz              # search inside .gz
```

**Find by properties**

```bash
find . -type f -name "*.csv" -mtime -7 -size +100M -print
# files in last 7 days, larger than 100MB
```

## 5) Columns, filtering, sorting (CSV-ish)

> These work best for **simple** CSVs (no embedded commas/quotes). For robust CSV work, see “Nice-to-install” at the end.

```bash
# Select columns 1 and 3 (comma-separated)
cut -d, -f1,3 data.csv | head

# Filter rows where column 3 equals "yes"
awk -F, 'BEGIN{OFS=","} NR==1 || $3=="yes"' data.csv | head

# Sort by column 2 (numerically), then unique counts of column 3
sort -t, -k2,2n data.csv | head
cut -d, -f3 data.csv | tail -n +2 | sort | uniq -c | sort -nr | head

# Count missing values in column 5 (empty cell)
awk -F, 'NR>1 {c+=($5==""?1:0)} END{print c}' data.csv
```

## 6) Summaries & sampling (one-liners)

```bash
# Row count excluding header
tail -n +2 data.csv | wc -l

# Random 1,000-row sample (keep header)
{ head -n1 data.csv; tail -n +2 data.csv | shuf -n 1000; } > sample.csv

# Train/test split 80/20 (keep header)
H=$(head -n1 data.csv)
T=$(mktemp)
tail -n +2 data.csv | shuf > "$T"
N=$(wc -l < "$T")
K=$(( N*80/100 ))
{ echo "$H"; head -n "$K" "$T"; } > train.csv
{ echo "$H"; tail -n +"$((K+1))" "$T"; } > test.csv
rm "$T"

# Top 10 most frequent values in column "city" (assuming it's column 4)
cut -d, -f4 data.csv | tail -n +2 | sort | uniq -c | sort -nr | head
```

## 7) Compression & archives

```bash
gzip big.csv                          # .gz compress
pigz -p 8 big.csv                     # faster parallel gzip (if installed)
zcat big.csv.gz | head                # preview without decompressing
tar -czf data.tgz data/               # create tar.gz
tar -xzf data.tgz                     # extract
```

## 8) Downloading data

```bash
wget -c URL                          # resume if interrupted
curl -L -O URL                       # follow redirects, save with filename
curl -L "URL" | tar -xzf -           # download and untar in one go
sha256sum file.zip                   # verify checksum
```

## 9) Processes, performance & disk usage

```bash
top                                   # live system monitor
htop                                  # nicer top (if installed)
ps aux | grep jupyter                 # list matching processes
pkill -f jupyter                      # kill by pattern (careful)
lscpu; free -h                        # CPU info; memory
watch -n 1 nvidia-smi                 # GPU usage (NVIDIA, if available)
```

**Disk usage**

```bash
df -h                                 # disk space per filesystem
du -sh * | sort -h                    # sizes of items in current dir
du -sh -- * | sort -h | tail         # biggest folders here
```

## 10) Ports & networking

```bash
hostname -I                           # machine IP
curl ifconfig.me                      # public IP
lsof -i :8888                         # what uses port 8888?
ss -tulpn | grep LISTEN               # listening ports
ping -c 4 example.com                 # connectivity test
```

## 11) Background jobs & long runs

```bash
# Run in background, keep alive after logout, capture logs
nohup python train.py > train.log 2>&1 & disown

jobs; fg %1; bg %1                    # job control in current shell
```

**Terminal multiplexers (recommended for servers)**

```bash
tmux new -s ds                        # new session
tmux ls                               # list
tmux attach -t ds                     # reattach
```

## 12) Permissions & ownership

```bash
ls -l                                  # see permissions
chmod u+x script.sh                     # make executable
chmod -R o-rwx data/                    # remove others' access
chown -R $USER:$USER project/           # change owner (need sudo if not owner)
umask                                  # default permission mask
```

## 13) Environment variables & Python launchers

```bash
echo $PATH
export MYVAR=value
python -V
which python

# Virtual environments (venv)
python -m venv .venv
source .venv/bin/activate
pip install -U pip wheel
deactivate
```

> Conda users: `conda create -n ds python=3.11`, `conda activate ds`, `conda env export > env.yml`.

## 14) Jupyter on a remote server

```bash
# On the server
jupyter lab --no-browser --port 8888

# On your laptop (tunnel)
ssh -L 8888:localhost:8888 user@server

# Open: http://localhost:8888
```

Kill all Jupyter servers:

```bash
jupyter lab list
jupyter lab stop 8888
pkill -f "jupyter-lab"
```

## 15) Moving data between machines

```bash
# Fast, resumable, keeps permissions, shows progress
rsync -avP ~/data/ user@server:~/data/

# Simple copy over SSH
scp big.csv user@server:~/data/
scp -r folder/ user@server:~/folder/
```

## 16) Git (quick terminal essentials)

```bash
git status; git add -A; git commit -m "msg"
git log --oneline --graph --decorate --all
git switch -c feature/x
git pull --rebase; git push
# Remember a proper .gitignore
```

## 17) Containers (brief)

```bash
docker ps -a
docker images
docker run --rm -it -v "$PWD":/work -w /work python:3.11 bash
```

## 18) Cron: schedule periodic jobs

```bash
crontab -e
# Run every day at 02:30
30 2 * * * /home/user/.venv/bin/python /home/user/etl.py >> /home/user/etl.log 2>&1
```

## 19) Data-wrangling “recipes”

```bash
# Combine many CSVs with same header (keep first header only)
awk 'FNR==1 && NR!=1 {next} {print}' *.csv > all.csv

# Extract rows matching a regex in column 2
awk -F, '$2 ~ /regex/' data.csv > filtered.csv

# Replace "NA" with empty in column 5
awk -F, 'BEGIN{OFS=","} {if(NR>1 && $5=="NA") $5=""; print}' data.csv > out.csv

# Convert TSV → CSV (simple)
tr '\t' ',' < data.tsv > data.csv

# Check for Windows line endings and fix
file data.csv
dos2unix data.csv       # if installed

# Validate JSON (needs jq)
jq . data.json | head   # pretty-print
```

## 20) “What’s eating my space/CPU/GPU?” quick checks

```bash
du -sh /data/* | sort -h | tail
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv
```

## Nice-to-install CLI tools (highly recommended)

* **ripgrep (`rg`)** – super fast grep
* **fd** – friendlier `find`
* **bat** – prettier `cat` with syntax highlighting
* **csvkit** – robust CSV processing: `csvcut`, `csvlook`, `csvstat`
* **Miller (`mlr`)** – fast CSV/TSV wrangler (joins, group-bys)
* **jq** – JSON processor
* **htop/bpytop** – system monitors
* **pigz** – parallel gzip
* **GNU parallel** – parallelize pipelines

**Examples**

```bash
csvstat data.csv                        # quick schema & stats
csvcut -c id,price data.csv | csvlook   # select columns & pretty table
mlr --icsv --opprint stats1 -a count,mean,median -f price data.csv
```

---

### ⚠️ Safety notes

* Double-check paths before `rm -rf` or `chmod -R`.
* Use quotes around variables/paths with spaces: `rm -rf "$DIR"`.
* For long jobs on servers, prefer `tmux`/`screen` or `nohup … & disown`.