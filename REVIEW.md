# STAT 405/605 DSCP - Complete Exam Review

> 50 points, 75 minutes, closed book, paper exam. Covers everything except the project.

---

## 1. LINUX COMMANDS

### Connecting & Downloading

```bash
ssh user@hostname                    # secure shell login
wget URL                             # download file from URL
scp [[user@]host:]f1 [[user@]host2:]f2  # secure copy between machines
```

### Directories

```bash
mkdir DIR             # make directory
mkdir -p a/b/c        # make nested dirs (parents too)
cd [dir]              # change directory (default: ~)
pwd                   # print working directory
rmdir DIR             # remove EMPTY directory
ls                    # list files
ls -l                 # long format (permissions, size, date)
ls -a                 # show hidden files (. prefixed)
ls -ltr               # long format, sorted by time, reversed (oldest first)
```

**Directory shorthand:**

| Symbol | Meaning |
|--------|---------|
| `~` | Your home directory |
| `.` | Current directory |
| `..` | Parent directory |
| `~user` | user's home directory |

### Files

```bash
cp SOURCE DEST              # copy
mv SOURCE DEST              # move or rename
cat FILE                    # print file to stdout
cat FILE1 FILE2             # concatenate files
rm FILE                     # remove
rm -r DIR                   # remove directory and contents recursively
chmod MODE FILE             # change permissions
chmod u+x script.sh         # give User eXecute permission
```

### Viewing Files

```bash
head -n 3 file.csv          # first 3 lines
tail -n 3 file.csv          # last 3 lines
tail -n +2 file.csv         # start on line 2 (SKIP HEADER)
wc file                     # line, word, character counts
diff FILE1 FILE2            # show differences
man COMMAND                 # manual page
```

### Searching

```bash
grep PATTERN [FILE]                        # print matching lines
grep -r PATTERN DIR                        # recursive search
find ~ -name "*.R"                         # find files by name
find ~ -name "*.R" -exec grep "rm(list" {} ";" -print
  # find .R files, grep each one, print matching filenames
```

### Text Processing

```bash
sort FILE                              # sort lines alphabetically
sort -t ',' -n -k 2 -r FILE           # -t=separator -n=numeric -k=field -r=reverse
uniq FILE                              # omit repeated lines
uniq -c FILE                           # include counts of repeats
cut -d ',' -f 2 FILE                   # extract field 2, comma-delimited
sed 's/PATTERN/REPLACEMENT/' FILE      # search and replace
sed 's/old/new/g' FILE                 # g = replace ALL occurrences
awk -F ',' '{ if ($3 == "8") {print $0} }' FILE
  # -F=separator, $0=whole line, $3=field 3
```

### Other Commands

```bash
echo "text"         # print text
exit                # exit shell
hostname            # print machine name
kill PID            # kill process
ps                  # list processes
time COMMAND        # time a command
top                 # live process monitor
seq N               # print 1 to N
seq A B             # print A to B
```

### Command-Line Editing

```
C-p  previous command       C-n  next command
C-r  reverse search history C-s  forward search history
C-f  cursor forward          C-b  cursor back
C-a  start of line           C-e  end of line
C-d  delete character
```

---

## 2. PIPELINES & REDIRECTION

### Pipelines

`A | B` — output of A becomes input of B

```bash
ls -l | wc                                    # count files
tail -n +2 f.csv | sort -t ',' -n -k 2 -r    # skip header, sort field 2
cat f.csv | grep "MSN" | cut -d ',' -f 5     # filter then extract column
cat a3.txt | sort | uniq -c                   # sort then count unique lines
```

### Three I/O Streams

| Stream | File Descriptor | Purpose |
|--------|----------------|---------|
| **stdin** | **0** | Standard input |
| **stdout** | **1** | Standard output |
| **stderr** | **2** | Error/diagnostic messages |

### Redirection

| Syntax | What it does |
|--------|-------------|
| `cmd > FILE` | Write stdout to FILE (overwrite) |
| `cmd >> FILE` | Append stdout to FILE |
| `cmd 2> FILE` | Write stderr to FILE |
| `cmd &> FILE` | Write both stdout AND stderr to FILE |
| `cmd 1>&2` | Redirect stdout to stderr |
| `cmd < FILE` | Read stdin from FILE |
| `cmd <<< "string"` | Here string (stdin from string) |
| `cmd << EOF ... EOF` | Here document (stdin from block) |
| `cmd > /dev/null` | Discard output |

**Key insight:** `>` is shorthand for `1>`. `<` is shorthand for `0<`.

**`cmd &> FILE`** is shorthand for **`cmd > FILE 2>&1`** — "redirect stdout to FILE, then redirect stderr to where stdout goes."

**Why `1>&2`?** — Used in scripts to send error/usage messages to stderr so they don't pollute stdout pipeline data:
```bash
echo "usage: $0 <arg>" 1>&2
```

### Here String & Here Document

```bash
bc -l <<< "4 * a(1)"          # pi via here string

cat << EOF
Hello $USER
Today is $(date)
EOF
```

### Compound Expressions (Grouped Commands)

```bash
echo "1 2 3" | sed 's/ /\n/g' | { sum=0; while read n; do sum=$(($sum+$n)); done; echo $sum; }
```

**Rules:** Space after `{` is **required**. Semicolon before `}` is **required**.

---

## 3. SHELL SCRIPTING

### Script Basics

```bash
#!/bin/bash              # FIRST LINE — tells loader to use /bin/bash
chmod u+x script.sh      # make executable
./script.sh              # run it
```

### Variables

**NO SPACES around `=`**

```bash
a=apple                          # string
b="apple and orange"             # string with spaces
c=3                              # number (stored as string)
d=$c                             # value of variable
d=${c}X                          # ${} avoids ambiguity: "3X"
files=$(ls -1)                   # command substitution
e=$(($c * $c / 2))              # integer arithmetic: + - * / ** %
f=$(echo "scale=6; 1/3" | bc)   # floating-point via bc
b+=" and cherry"                 # append to string
```

### Quoting

| Type | Expands `$`? | Expands `$()`? | Expands `$(())`? |
|------|:-----------:|:--------------:|:----------------:|
| `"double quotes"` | YES | YES | YES |
| `'single quotes'` | NO | NO | NO |
| `\` (backslash) | escapes one character | | |

```bash
x=42
echo "answer is $x"       # answer is 42
echo 'answer is $x'       # answer is $x
echo cost=\$5.00           # cost=$5.00
```

### Brace Expansion

```bash
echo {A,B}_{1,2}           # A_1 A_2 B_1 B_2
echo file{1..5}.txt        # file1.txt file2.txt file3.txt file4.txt file5.txt
echo {Tu,Th}_Table{1..6}   # Tu_Table1 Tu_Table2 ... Th_Table6
```

### Glob Patterns (for filenames — NOT regex)

| Pattern | Matches |
|---------|---------|
| `*` | Any characters |
| `?` | Any ONE character |
| `[abc]` | Any one of a, b, c |
| `[!abc]` | Any one character NOT a, b, c |
| `[[:digit:]]` | Any digit |
| `[[:alpha:]]` | Any letter |
| `[[:lower:]]` | Any lowercase letter |
| `[[:upper:]]` | Any uppercase letter |
| `[[:alnum:]]` | Any letter or digit |

### Command-Line Arguments

| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$1`, `$2`, ... | Arguments |
| `$#` | Number of arguments |

```bash
if [[ $# -ne 2 ]]; then
    echo "usage: $0 <word> <n>" 1>&2
    exit 0
fi
word=$1
n=$2
```

### Conditionals

```bash
if [[ CONDITION ]]; then
    COMMANDS
elif [[ CONDITION_2 ]]; then
    COMMANDS
else
    COMMANDS
fi
```

**Spaces inside `[[ ]]` are REQUIRED.**

**Comparison operators:**

| Type | Operators |
|------|-----------|
| Strings | `==` (equal), `!=` (not equal) |
| Integers | `-eq` `-ne` `-lt` `-le` `-gt` `-ge` |
| Logical | `!` (not), `&&` (and), `\|\|` (or) |
| Regex | `=~` (match) |

```bash
x=3; name="Philip"
if [[ ($x -eq 3) && ($name == "Philip") ]]; then
    echo "match"
fi
```

**Regex match with BASH_REMATCH:**

```bash
file="NetID.cxx"
pattern="(.*).cxx"
if [[ $file =~ $pattern ]]; then
    echo ${BASH_REMATCH[0]}   # full match: NetID.cxx
    echo ${BASH_REMATCH[1]}   # first group: NetID
fi
```

### Loops

```bash
# for loop — traverse a sequence
for file in $(ls); do echo "file=$file"; done
for i in $(seq 1 10); do echo $i; done
for n in {1..100}; do echo $n; done

# while loop — zero or more iterations
x=7
while [[ $x -ge 1 ]]; do
    echo "x=$x"
    x=$((x / 2))
done

# while read — read lines into variables
while read name height weight major; do
    echo "$name studies $major"
done < students.txt
# NOTE: last variable gets ALL remaining words on the line
# NOTE: first line (header) gets read too unless you skip it

# break and continue work like other languages
```

**"Do-while" hack (one or more):**
```bash
while echo -n "Enter positive integer: "; read n; [[ $n -le 0 ]]; do : ; done
```

### Functions

```bash
function binary_add {
    local a=$1              # local variable
    local b=$2
    local sum=$(($a + $b))
    echo "debug: a=$a" 1>&2 # debug message to STDERR
    echo $sum                # "return value" to STDOUT
}

# Capture return value via command substitution:
x=$(binary_add 3 4)
echo "x=$x"                 # x=7
```

**Key rules:**
- Parameters: `$1`, `$2`, etc. Count: `$#`
- `local` makes variables local to the function
- "Return" via `echo` (stdout), capture with `$(function args)`
- Debug/error messages go to stderr via `1>&2`

### eval

```bash
a="ls"; b="| wc"; c="$a $b"
eval $c                      # runs: ls | wc
```

**`eval` is dangerous** — can be exploited to run arbitrary code.

---

## 4. REGULAR EXPRESSIONS

### Emacs Regex Syntax

| Pattern | Meaning | Example |
|---------|---------|---------|
| `.` | Any char (except newline) | `a.c` matches "abc", "a1c" |
| `[...]` | Character set | `[aeiou]` matches any vowel |
| `[^...]` | Negated set | `[^aeiou]` matches non-vowel |
| `^` | Beginning of line | `^[A-Z]` matches uppercase at line start |
| `$` | End of line | `edu$` matches lines ending in "edu" |
| `\<` | Beginning of word | `\<[13]` matches 1 or 3 at word start |
| `\>` | End of word | `\>` matches word boundary |
| `\{n\}` | Exactly n times | `[0-9]\{4\}` matches 4 digits |
| `\{n,m\}` | n to m times | `a\{2,4\}` matches "aa", "aaa", "aaaa" |
| `\{n,\}` | n or more times | `a\{2,\}` matches 2+ a's |
| `*` | 0 or more | shorthand for `\{0,\}` |
| `+` | 1 or more | shorthand for `\{1,\}` |
| `?` | 0 or 1 | shorthand for `\{0,1\}` |
| `\|` | OR | `Jo\|Je` matches "Jo" or "Je" |
| `\(...\)` | Group (capture) | `\([a-z]\)\1` = repeated letter |
| `\1` `\2` | Backreference | refers to nth group match |

**Repetition is maximal** by default. Appending `?` makes it minimal.

**Escaping:** To match literal `$^.*+?[\`, escape with `\`. E.g. `\^` matches `^`.

### Bash Regex (inside `[[ =~ ]]`)

**Key differences from emacs:**
- Groups use `(...)` not `\(...\)`
- Quantifiers use `{n}` not `\{n\}`
- Results go in `BASH_REMATCH[]` array
- Store pattern in variable to avoid quoting issues

```bash
file="report-2024.csv"
pattern="(.*)-([0-9]+)\.csv"
if [[ $file =~ $pattern ]]; then
    echo ${BASH_REMATCH[0]}   # report-2024.csv
    echo ${BASH_REMATCH[1]}   # report
    echo ${BASH_REMATCH[2]}   # 2024
fi
```

### Query-Replace Examples (from lecture)

Given:
```
Brown,Joe 123456789 jbrown@wisc.edu 1000
```

| Goal | Find | Replace |
|------|------|---------|
| Replace vowels | `[aeiou]` | `V` |
| Replace non-vowels | `[^aeiou]` | `C` |
| Replace any 4 digits | `[0-9]\{4\}` | `DDDD` |
| Replace only 4-digit words | `\<[0-9]\{4\}\>` | `DDDD` |
| Rearrange: Last,First ID email → First,Last,NetID,ID | `\([a-zA-Z]+\),\([a-zA-Z]+\) +\([0-9]\{9\}\) \([a-z]+\)@wisc.edu.*` | `\2,\1,\4,\3` |

### Glob vs Regex

| | Glob | Regex |
|---|------|-------|
| **Used for** | Filenames (`ls`, `cp`, `rm`) | Text content (`grep`, `sed`, emacs) |
| `*` means | Any characters | 0 or more of previous |
| `?` means | Any ONE character | 0 or 1 of previous |
| `[abc]` | Same | Same |
| Example | `ls *.csv` | `grep ".*\.csv"` |

---

## 5. EMACS

### Notation

- `C-` = hold Ctrl
- `M-` = hold Alt/Option, or press/release Esc

### Essential Commands

| Action | Keys | Mnemonic |
|--------|------|----------|
| **Files** | | |
| Open/create file | `C-x C-f` | find |
| Save | `C-x C-s` | save |
| Save as | `C-x C-w` | write |
| Insert file | `C-x i` | insert |
| Exit emacs | `C-x C-c` | |
| **Movement** | | |
| Forward | `C-f` | forward |
| Backward | `C-b` | backward |
| Next line | `C-n` | next |
| Previous line | `C-p` | previous |
| Start of line | `C-a` | a=start of alphabet |
| End of line | `C-e` | end |
| Start of buffer | `M-<` | |
| End of buffer | `M->` | |
| **Editing** | | |
| Delete char | `C-d` | delete |
| Delete prev char | Backspace | |
| Kill to end of line | `C-k` | kill |
| Set mark | `C-SPACE` | |
| Kill region | `C-w` | |
| Copy region | `M-w` | |
| Yank (paste) | `C-y` | yank |
| Replace prev yank | `M-y` | |
| Kill rectangle | `C-x r k` | |
| Yank rectangle | `C-x r y` | |
| Indent line | TAB | |
| Indent buffer | `M-x indent-region` | |
| **Recovery** | | |
| Abort command | `C-g` | |
| Undo | `C-/` | |
| Revert buffer | `M-x revert-buffer` | |
| **Search** | | |
| Search forward | `C-s` | search |
| Search backward | `C-r` | reverse |
| Regex forward | `C-M-s` | |
| Regex backward | `C-M-r` | |
| Find all | `M-x occur` | |
| Query replace | `M-%` | |
| Query replace regex | `M-x query-replace-regexp` | |
| **Windows** | | |
| One window | `C-x 1` | one |
| Split vertical | `C-x 2` | two |
| Split horizontal | `C-x 3` | three |
| Switch window | `C-x o` | other |
| **Buffers** | | |
| List buffers | `C-x C-b` | buffer |
| Kill buffer | `C-x k` | kill |
| Open shell | `M-x shell` | |
| **R (ESS)** | | |
| Start R | `M-x R` | |
| Run one line | `C-c C-n` | next line |
| Run chunk | `C-c C-c` | chunk |
| Run buffer | `C-c C-b` | buffer |

**Kill = Cut. Yank = Paste.**

**`C-q C-j`** inserts a literal newline in the minibuffer (for search/replace with newlines).

---

## 6. GIT & GITHUB

### File States

```
untracked ──git add──> staged ──git commit──> committed
                         ^                       │
                         │                    (edit file)
                         │                       │
                         └──── git add ─── modified
```

### Commands

```bash
# Setup
git init                           # create new repo
git config --global user.name "N"  # set name
git config --global user.email "E" # set email
git config --global core.editor emacs
git config --list                  # show config

# Daily workflow
git status                         # show current state
git add FILE                       # stage file
git commit -m "message"            # save snapshot
git log                            # show history
git log --pretty=oneline           # compact history
git diff                           # show changes

# Branching
git branch                         # list branches (* = current)
git branch -M main                 # rename branch to "main"
git checkout BRANCH                # switch to branch
git checkout -b BRANCH             # CREATE and switch to new branch
git checkout FILE                  # UNDO changes to file (DANGEROUS)
git checkout COMMIT                # retrieve specific commit
git merge BRANCH -m "msg"          # merge branch into current
git branch -d BRANCH               # delete branch

# Remote (GitHub)
git clone REPO                     # copy repo from GitHub
git remote add origin URL          # link to GitHub
git remote set-url origin URL      # change remote URL
git remote -v                      # verify remote
git push origin main               # upload to GitHub
git pull origin main               # download & merge from GitHub
```

### Typical Workflow

```
git add → git commit → git pull → git push
```

### Merge Conflicts

When two branches change the same line:

```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> branch-name
```

**Resolution:** Edit file to keep desired version, remove ALL markers, then:
```bash
git add FILE
git commit -m "Resolve conflict"
```

### SSH Keys

```bash
ssh-keygen                  # creates ~/.ssh/id_rsa.pub (public) and ~/.ssh/id_rsa (private)
cat ~/.ssh/id_rsa.pub       # copy this to GitHub Settings > SSH keys
```

### .gitignore

Lists files git should NOT track:
```
Property_Tax_Roll.csv
*.log
data/
```

---

## 7. HPC: SLURM vs HTCONDOR

### Command Comparison

| Action | Slurm | HTCondor |
|--------|-------|----------|
| Login | `ssh STATuser@slurm-submit-00.cs.wisc.edu` | `ssh NetID@learn.chtc.wisc.edu` |
| Submit job | `sbatch <script.sh>` | `condor_submit <script.sub>` |
| Check queue | `squeue -u <STATuser>` | `condor_q` |
| Cancel jobs | `scancel -u <STATuser>` | `condor_rm <NetID>` |
| Parallel jobs | `sbatch --array=1-3` | `queue <line> from <lines.txt>` |
| Dependencies | `sbatch --afterok:<jobID>` | `condor_submit_dag <script.dag>` |
| Interactive | `srun --pty /bin/bash` | `condor_submit -i <script.sub>` |
| Check resources | `seff <jobID>` | bottom of `.log` file |

### Key Concepts

- **Slurm** = Statistics HPC cluster. **HTCondor** = CHTC.
- `--partition short` for Slurm jobs
- `sbatch --array=1-22` runs 22 parallel jobs; inside script use `$SLURM_ARRAY_TASK_ID`
- Always work in `/workspace/STATuser` on Slurm (compute nodes can't read home directory)
- **Testing progression:** 1 job → 5 jobs → full scale (never launch untested at scale)
- No job should run longer than 1 hour
- No more than 2 GB data per CHTC job
- HTCondor: `requirements = (HasCHTCStaging == true)` for staging access

### Slurm Script Template

```bash
#!/bin/bash
#SBATCH --partition=short
#SBATCH --array=1-22

YEAR=$((1986 + $SLURM_ARRAY_TASK_ID))
wget http://example.com/${YEAR}.csv.bz2
bzip2 -d ${YEAR}.csv.bz2
# ... process ...
```

### HTCondor Submit File Template

```
executable = hw4.sh
arguments = $(line)
transfer_input_files = hw4.R, file:///staging/groups/.../$(line).tgz
log = logs/$(line).log
output = out/$(line).out
error = err/$(line).err
requirements = (HasCHTCStaging == true)
request_cpus = 1
request_memory = 1GB
request_disk = 2GB
queue line from lines.txt
```

---

## 8. COMMON EXAM TRAPS

### Bug-Finding Checklist

| Bug | Fix |
|-----|-----|
| `x = 3` | `x=3` (no spaces around `=`) |
| `if [[$x -eq 3]]` | `if [[ $x -eq 3 ]]` (spaces inside `[[ ]]`) |
| `if [[ $x == 3 ]]` for numbers | `if [[ $x -eq 3 ]]` (use `-eq` for integers) |
| `if [[ $x -eq "hello" ]]` | `if [[ $x == "hello" ]]` (use `==` for strings) |
| Missing `#!/bin/bash` | Add it as the FIRST line |
| Script won't run | `chmod u+x script.sh` |
| Function "returns" wrong value | Debug output goes to `1>&2`, return value via `echo` to stdout |
| `while read` includes header | Pipe through `tail -n +2` first |
| Last `read` variable has extra words | Last variable captures ALL remaining words on the line |
| `sort` gives wrong numeric order | Use `sort -n` for numbers (default is lexicographic) |
| `{ cmd; }` syntax error | Space after `{`, semicolon before `}` |

### Quick Recall

```
$#   = number of arguments
$0   = script name
$1   = first argument
$?   = exit status of last command
$()  = command substitution
$(()) = arithmetic
>    = overwrite
>>   = append
1>&2 = stdout to stderr
2>&1 = stderr to stdout
&>   = both to file
```

---

## 9. HOMEWORK HIGHLIGHTS (exam-relevant patterns)

### HW1 — Emacs
- Fix parenthesis bugs by indenting (`M-x indent-region`)
- Rectangle operations to remove line numbers
- Regex replace to transform file formats
- `C-q C-j` to type literal newlines in minibuffer

### HW2 — Lyman-break Galaxy Search (R)
- `readFrameFromFITS()` reads `.fits` files
- Standardize flux: subtract mean, divide by sd
- Try all alignments of short template against long spectrum
- Distance measures: Euclidean, correlation, chi-squared
- `rm(list=ls())` at top of script to clear environment
- Use relative paths, never `setwd()`

### HW3 — Slurm Parallel Computing
- `sbatch --array=1-22` for 22 parallel years of airline data
- `$SLURM_ARRAY_TASK_ID` gives array index inside script
- `sbatch --afterok:JOBID` for job dependencies
- Pipeline: `wget` → `bzip2 -d` → `grep/cut` → small output file
- `tapply(X=data$col, INDEX=data$group, FUN=mean)` for grouped means

### HW4 — HTCondor (CHTC)
- `Rscript hw4.R <template> <data_dir>` — command-line R
- `commandArgs(trailingOnly=TRUE)` to get args in R
- HTCondor `queue line from lines.txt` for parallel jobs
- `transfer_input_files` to send data to compute nodes
- Check resources in `.log` file, then set `request_memory`, `request_disk`

### Shell Exercises (Group D)
- `school.sh`: pipeline with `cat | grep | cut | { sum; avg }`
- `digits.sh`: brace expansion + regex test in loop
- `five_dirs.sh`: nested loops to create directory structure
- `rm_n.sh`: `find -size +Nc -type f` to remove large files
- `mean.sh`: read from file or `/dev/stdin`, compound expression for average
