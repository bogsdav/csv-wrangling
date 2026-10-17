---
name: "csv-wrangling"
description: "Use when the user wants to inspect, clean, filter, sort, join, convert or run SQL queries on CSV, Excel or JSON data files."
---

# CSV Wrangling (csvkit)

## Overview

csvkit is a set of 14 command-line tools for working with tabular data. Use it to look at a CSV, profile its columns, clean broken rows, filter, sort, join, stack, convert Excel or JSON to CSV, convert CSV to JSON, and run SQL queries against CSV files. All tools read a file argument or STDIN and write CSV to STDOUT, so they chain with pipes.

## Setup

Install once per session, then confirm:

```bash
pip install csvkit --break-system-packages
csvcut --version
```

Expected: `csvcut 2.x.x`. For Postgres or MySQL with `csvsql --db`, also install the driver: `pip install psycopg2-binary` or `pip install mysql-connector-python` (add `--break-system-packages`). SQLite needs no extra install.

## Workflows

Columns can be given by name or by 1-based index, comma-separated: `-c name,city` or `-c 1,3`.

### 1. Look at an unfamiliar file

Use first, before any other work on a CSV the user hands over.

```bash
csvcut -n data.csv                    # list column names with index numbers
csvlook data.csv | head -20           # readable table of the first rows
csvstat data.csv                      # type, nulls, min/max, mean, top values per column
csvstat --count data.csv              # row count only
```

Expected: `csvcut -n` prints lines like `  1: id`. `csvstat` prints one block per column.

Example: `csvstat -c price,qty data.csv` profiles only those two columns.

### 2. Clean a broken file

Use when rows have the wrong number of fields, columns are empty, or headers are messy. `csvclean` does nothing unless a check is enabled.

```bash
csvclean -a --omit-error-rows data.csv > clean.csv 2> errors.csv
```

Expected: `clean.csv` holds the good rows. `errors.csv` holds `line_number,msg,...` for each bad row. Exit code is 1 when any errors were found; that is normal and not a failure of the command.

Other fixes: `--fill-short-rows` (pad short rows; `--fillvalue X` sets the pad), `--join-short-rows` (merge rows split by stray newlines), `--remove-empty-columns`, `--header-normalize-space`.

### 3. Select, filter and sort

```bash
csvcut -c name,email,city data.csv > slim.csv           # keep columns
csvcut -C notes data.csv                                 # drop columns
csvgrep -c city -m Manila data.csv                       # substring match (-m 10 also matches 110)
csvgrep -c id -r '^10$' data.csv                         # exact match: anchor a regex
csvgrep -c email -r '@gmail\.com$' data.csv              # regex match
csvgrep -c status -m cancelled -i data.csv               # invert: rows NOT matching
csvsort -c amount -r data.csv                            # sort descending
```

Example chain: top 5 orders in Manila by amount, as a table:

```bash
csvgrep -c city -m Manila orders.csv | csvsort -c amount -r | head -6 | csvlook
```

(`head -6` = header + 5 rows.)

### 4. Combine files

```bash
csvjoin -c order_id orders.csv payments.csv               # inner join on a column both files share
csvjoin -c customer_id,id orders.csv customers.csv        # different column names: one per file, in file order
csvjoin -c id,customer_id --left customers.csv orders.csv # left join: keep every customer, even with no orders
csvstack jan.csv feb.csv mar.csv > q1.csv                 # append files with the same columns
csvstack --filenames jan.csv feb.csv > both.csv           # add a column saying which file each row came from
```

### 5. Convert formats

Add `-I` to `in2csv` by default. Without it, text can be rewritten: a cell `Jan` becomes `0001-01-01` and `1` becomes `True`. Drop `-I` only when the user wants typed output.

```bash
in2csv -I data.xlsx > data.csv                  # first sheet
in2csv -I --sheet "Sales" data.xlsx > sales.csv # a named sheet
in2csv -n data.xlsx                             # list sheet names
in2csv -I data.json > data.csv                  # JSON array of objects
in2csv -I -f ndjson data.ndjson > data.csv
csvjson -i 2 data.csv > data.json               # CSV to pretty JSON (numbers typed)
csvjson -I -k id data.csv                       # JSON object keyed by a column; -I keeps keys like "10" (not "1E+1")
csvformat -T data.csv > data.tsv             # CSV to TSV
csvformat -d ';' data.csv > fixed.csv        # semicolon-delimited input to normal CSV
```

### 6. Query with SQL

Use for grouping, aggregates and anything awkward with the other tools. Each file becomes a table named after its file name without the extension.

```bash
csvsql --query "select city, count(*) as n, sum(amount) as total from orders group by city order by total desc" orders.csv
csvsql --query "select o.*, c.name from orders o join customers c on o.customer_id = c.id" orders.csv customers.csv
```

Expected: CSV output with the query result. Pipe to `csvlook` to show it as a table. Integer columns may come back as `1.0`; add `-I` when that matters (all columns become text, so cast in SQL for math: `cast(amount as real)`).

Load into a database instead of querying in memory:

```bash
csvsql --db sqlite:///work.db --insert orders.csv
sql2csv --db sqlite:///work.db --query "select * from orders limit 10"
```

Show the CREATE TABLE statement csvkit would use: `csvsql -i postgresql data.csv`.

## Gotchas

- **Wrong columns or merged fields**: the delimiter was guessed wrong. Set it with `-d ';'` or `-t` for tabs, or turn off guessing with `--snifflimit 0`.
- **Leading zeros lost, IDs printed as `10.0` or `1E+1`, `1`/`0` turned into True/False, words like `Jan` turned into dates**: type inference. Add `-I` (`--no-inference`) to treat every column as text. Always check converted output with `csvlook` before handing it back.
- **Non-UTF-8 files** (Excel exports, older systems): add `-e latin1` or `-e cp1252`.
- **Unicode errors when piping**: prefix the command with `env PYTHONIOENCODING=utf8`.
- **No header row**: add `-H`; columns become `a`, `b`, `c`...
- **Junk lines above the header**: skip them with `-K 3` (number of lines).
- **STDIN with multi-file tools** (`csvjoin`, `csvstack`): pass `-` as the filename.
- **Memory**: `csvstat`, `csvsort`, `csvjoin`, `csvsql` and `csvjson` load the whole file. For files over a few hundred MB, filter with `csvcut`/`csvgrep` first (they stream), or add `--no-inference` to speed things up.
- **csvclean in 2.x writes to STDOUT/STDERR**, not to `_out.csv`/`_err.csv` files as in older versions. Redirect both streams.
- **Compressed input** (`.gz`, `.bz2`, `.xz`) is read directly; no need to unzip.

## Source

- Repo: https://github.com/wireservice/csvkit
- Commit: 6d19c625214977ab2988e674441ca24638bc12d3 (csvkit 2.2.0)
- License: MIT
- Full docs: https://csvkit.readthedocs.io