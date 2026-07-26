# GNU `parallel` cheatsheet

Run the same command over many inputs at once, using every core you have. Think
`xargs` with sane quoting, per-job output that doesn't interleave, replacement
strings for filenames, job logs and the ability to farm work out over SSH.

> **First run prints a citation notice.** Silence it once and for all with
> `parallel --citation` (type `will cite` when prompted), which writes
> `~/.parallel/will-cite`. Do that before putting `parallel` in a script — the
> notice goes to stderr and clutters logs.

> **Always `--dry-run` first.** It prints the commands it *would* run, which is
> the fastest way to catch a quoting mistake before it hits 200 files.

## The shape of a command

```sh
parallel [options] command {} ::: input1 input2 ...
```

`{}` is where each input is substituted. If you leave the command out entirely,
the input lines are run as commands.

| Command | What it does |
| --- | --- |
| `parallel echo ::: a b c` | Arguments given on the command line |
| `ls *.txt \| parallel gzip` | Arguments from stdin, one per line |
| `parallel gzip ::: *.txt` | Same job, arguments from a glob |
| `parallel echo :::: args.txt` | Arguments from a file (`::::`) |
| `parallel -a args.txt echo` | The same, spelled as an option |
| `parallel ::: 'echo a' 'echo b'` | No command — run the inputs themselves |
| `parallel --dry-run gzip ::: *.txt` | Show the commands, run nothing |

## Replacement strings

For an input of `/data/reads/sample1.fastq.gz`:

| String | Expands to | Example |
| --- | --- | --- |
| `{}` | The whole input | `/data/reads/sample1.fastq.gz` |
| `{.}` | Without the last extension | `/data/reads/sample1.fastq` |
| `{/}` | Basename | `sample1.fastq.gz` |
| `{//}` | Dirname | `/data/reads` |
| `{/.}` | Basename, no extension | `sample1.fastq` |
| `{#}` | Job sequence number | `1`, `2`, `3`, … |
| `{%}` | Job slot number | `1` … `$(nproc)` |
| `{1}`, `{2}`, … | The nth input field or source | see below |
| `{1.}`, `{2/}`, … | The same modifiers, applied to field n | |

Add `--plus` for the extended set, which is what you want for doubly-suffixed
files:

| String | Expands to |
| --- | --- |
| `{..}` | Strip two extensions — `sample1` from `sample1.fastq.gz` |
| `{...}` | Strip three |
| `{/..}` | Basename with two extensions stripped |
| `{%.gz}` | Strip the suffix `.gz` if present |
| `{= perl code =}` | Arbitrary Perl on the argument (`$_`) |

```sh
parallel --plus echo {..} ::: sample1.fastq.gz     # -> sample1
parallel --rpl '{noext} s/\.[^.]+$//' echo {noext} ::: a.txt   # define your own
```

## Controlling how many run

| Option | Effect |
| --- | --- |
| *(default)* | One job per CPU core |
| `-j 4` | Four jobs at a time |
| `-j 1` | Sequential (useful for debugging) |
| `-j 200%` | Two jobs per core |
| `-j -1` | One fewer than the number of cores |
| `-j 0` | As many as possible |
| `--load 80%` | Don't start jobs when load is above this |
| `--memfree 4G` | Don't start jobs unless this much RAM is free |
| `--delay 1s` | Wait between starting jobs (be kind to APIs and NFS) |
| `--timeout 60s` | Kill a job that runs too long (`200%` = 2× median) |
| `--nice 10` | Run jobs at lower priority |
| `--shuf` | Shuffle the job order |

## Output

Output is **grouped** by default: each job's output is held and printed whole,
so lines from different jobs never interleave. Jobs still finish in whatever
order they finish.

| Option | Effect |
| --- | --- |
| `-k`, `--keep-order` | Print output in *input* order, not completion order |
| `--line-buffer`, `--lb` | Stream output line by line as it appears |
| `-u`, `--ungroup` | No buffering at all — fastest, may interleave |
| `--tag` | Prefix every output line with the input argument |
| `--tagstring '{1}'` | Prefix with something you choose |
| `--results outdir` | Write each job's stdout/stderr to files under `outdir/` |
| `-v` | Print each command before running it |
| `--bar` / `--eta` / `--progress` | Progress bar / ETA / running summary |

## More than one argument per job

| Option | Effect |
| --- | --- |
| `-n 3` | At most 3 arguments per command |
| `-N 3` | Exactly 3 per command, and enables `{1} {2} {3}` |
| `-X` | Pack in as many arguments as the command line allows |
| `-m` | Like `-X`, but distribute arguments evenly across jobs |
| `-L 2` | Treat 2 input lines as one record |

```sh
# Pairs: {1} and {2} come from consecutive lines
parallel -N 2 echo pair {1} {2} ::: a b c d
```

## Several input sources

Multiple `:::` give the **cartesian product**; `:::+` and `::::+` **link**
sources element by element instead.

```sh
parallel echo {1} {2} ::: A B ::: 1 2        # A 1, A 2, B 1, B 2
parallel echo {1} {2} ::: A B :::+ 1 2       # A 1, B 2  (paired)
parallel --link echo {1} {2} ::: A B ::: 1 2 # same pairing, older spelling
parallel echo {1} {2} :::: r1.txt ::::+ r2.txt   # pair two files line by line
```

Linked sources are how you handle paired-end reads or any two lists that
correspond positionally.

## Columns and headers

| Option | Effect |
| --- | --- |
| `--colsep '\t'` | Split each input line into `{1}`, `{2}`, … |
| `--header :` | Use the first line as a header, giving named fields `{name}` |
| `--trim lr` | Strip leading/trailing whitespace from arguments |

```sh
# samples.tsv:  sample <TAB> fastq <TAB> genome
parallel --colsep '\t' --header : \
  'align --id {sample} --in {fastq} --ref {genome} > {sample}.bam' :::: samples.tsv
```

## Splitting one big input (`--pipe`)

Instead of one job per argument, chop stdin into blocks and feed a block to each
job:

```sh
# Count lines in a huge file, in parallel, then total the counts
cat big.txt | parallel --pipe --block 10M wc -l | awk '{s+=$1} END {print s}'

# Much faster for a real (seekable) file — parallel reads parts directly
parallel --pipepart -a big.txt --block 10M wc -l

# Keep records intact: start a new record at every FASTA header
parallel --pipe --recstart '>' --block 10M 'seqkit stats -' < reads.fasta
```

`--block` sets the chunk size, `--recstart` / `--recend` define where a record
may be split, and `--pipepart` is the fast path when the input is a file rather
than a stream.

## Job logs, failures and resuming

| Option | Effect |
| --- | --- |
| `--joblog jobs.log` | Record each job: start, runtime, exit code, command |
| `--resume` | Skip jobs already recorded as done in the joblog |
| `--resume-failed` | Rerun failed jobs plus any that never ran |
| `--retry-failed` | Rerun only the failed jobs from the joblog |
| `--retries 3` | Retry a failing job up to 3 times before giving up |
| `--halt now,fail=1` | Stop everything on the first failure |
| `--halt soon,fail=10%` | Let running jobs finish, start no new ones |

The exit status of `parallel` is the number of failed jobs (`>100` means
something went wrong with parallel itself). The workflow that matters:

```sh
parallel --joblog run.log --resume -j8 ./process {} ::: samples/*
# ...interrupted, machine rebooted, whatever...
parallel --joblog run.log --resume -j8 ./process {} ::: samples/*   # picks up where it left off
```

## Quoting

Each job runs through a shell, so redirections and pipes work — but they must be
quoted, or your interactive shell will grab them first.

```sh
parallel 'gzip -c {} > {.}.gz' ::: *.txt          # correct
parallel gzip -c {} > {.}.gz ::: *.txt            # WRONG: redirects parallel itself

parallel 'cut -f2 {} | sort -u > {/.}.uniq' ::: *.tsv    # pipes inside a job
parallel -q perl -pe 's/a/b/' ::: file             # -q: don't let parallel interpret the command
```

Single quotes are the default choice: `parallel` substitutes `{}` inside them,
but the shell doesn't touch `$`, `>` or `|`.

## Shell functions, aliases and variables

Plain `parallel` starts a fresh shell, so your functions and aliases don't
exist. Use `env_parallel`, which copies the current environment across:

```sh
# in ~/.bashrc  (use env_parallel.zsh for zsh)
source $(which env_parallel.bash)

myjob() { echo "processing $1"; }
env_parallel myjob ::: a b c

# Or export just what's needed to plain parallel
export -f myjob
parallel myjob ::: a b c
parallel --env MY_VAR 'echo $MY_VAR {}' ::: a b
```

## Running on other machines

| Option | Effect |
| --- | --- |
| `-S host` | Run jobs on `host` over SSH (`-S :` also uses the local machine) |
| `-S 4/user@host` | Cap that host at 4 concurrent jobs |
| `--slf hosts.txt` | Read the host list from a file |
| `--filter-hosts` | Drop hosts that don't respond |
| `--transfer` | Copy the input file to the remote host first |
| `--return {.}.out` | Copy a result file back afterwards |
| `--cleanup` | Delete the transferred and returned files remotely |
| `--trc {.}.out` | Shorthand for `--transfer --return --cleanup` |
| `--wd <dir>` | Remote working directory |
| `--env VAR` | Export an environment variable to the remote jobs |

```sh
parallel -S node1,node2 --trc {.}.bam 'align {} > {.}.bam' ::: *.fastq
```

## `sem` — one-off throttling

`sem` is `parallel --semaphore`: run commands in the background but never more
than N at once. Handy inside an existing loop you don't want to rewrite.

```sh
for f in *.txt; do
  sem -j4 "gzip $f"
done
sem --wait          # block until all of them finish
```

## Recipes

```sh
# Compress every file, keeping the originals' names
parallel gzip ::: *.fastq

# Convert images, writing next to the source
parallel convert {} {.}.png ::: *.jpg

# Run a script over samples listed in a file, 8 at a time, logged and resumable
parallel -j8 --joblog run.log --resume --bar ./pipeline.sh {} :::: samples.txt

# Paired-end reads: pair R1 with the matching R2
parallel --plus 'bwa mem ref.fa {} {%_R1.fq.gz}_R2.fq.gz > {..}.sam' ::: *_R1.fq.gz

# Every combination of two parameter lists
parallel 'run --k {1} --seed {2} > out_k{1}_s{2}.txt' ::: 10 20 30 ::: 1 2 3

# Download a list of URLs, 4 at a time, politely
parallel -j4 --delay 0.5s wget -q {} :::: urls.txt

# Search many files and tag which file each hit came from
parallel --tag rg 'pattern' {} ::: logs/*.log

# Time a serial run against a parallel one
time parallel -j1 ./slow {} ::: {1..8}
time parallel -j8 ./slow {} ::: {1..8}
```

## Gotchas

- **`{}` needs no quotes, but shell metacharacters do.** See [Quoting](#quoting).
- **Grouped output costs memory** for very chatty jobs — switch to
  `--line-buffer` if a job produces gigabytes.
- **Jobs are independent.** Anything shared (a counter, an output file appended
  by every job) will race; write per-job files and merge afterwards, or use
  `-k` to keep order.
- **`parallel` on Debian/Ubuntu may be Tollef Fog Heen's `moreutils` version**,
  which takes completely different arguments. `parallel --version` should say
  *GNU parallel*.
- **Exit status is the failure count**, so `set -e` scripts will stop on any
  failed job — often what you want, but worth knowing.

---

> Pinned to **GNU parallel 20260722**. Behaviour is stable across releases;
> `man parallel` is exhaustive, `man parallel_tutorial` is the guided version,
> and `parallel --help` covers the flags. Enforce a minimum with
> `parallel --minversion 20240101`.
