# `jq` cheatsheet

A command-line processor for JSON: slice, filter, map and transform structured
data the way `sed`/`awk` do for text. Covers `yq` at the end, which applies the
same expression language to YAML, XML and friends.

> **Two rules that prevent most confusion:**
>
> 1. Wrap the filter in **single quotes** so the shell leaves `$`, `|` and `"`
>    alone: `jq '.users[].name' file.json`.
> 2. Never paste shell variables into a filter — pass them in with `--arg`, which
>    quotes them safely: `jq --arg u "$USER" '.users[] | select(.name == $u)'`.

## Running it

| Command | What it does |
| --- | --- |
| `jq . file.json` | Pretty-print (the `.` filter is the identity) |
| `curl -s url \| jq .` | Read from stdin |
| `jq -r '.name' file` | **Raw** output: strings without quotes |
| `jq -c . file` | Compact: one JSON value per line |
| `jq -e . file` | Set exit status from the output (`1` if false/null) |
| `jq -S . file` | Sort object keys |
| `jq --tab . file` | Indent with tabs (`--indent 4` for spaces) |
| `jq -n '...'` | Null input: build JSON from scratch |
| `jq -s '...' *.json` | Slurp: read all inputs into one array |
| `jq -R '...'` | Raw input: each line arrives as a string |
| `jq -j '...'` | Join output — no trailing newlines |
| `jq -f filter.jq file` | Read the filter from a file |
| `jq -C . file \| less -R` | Force colour through a pager |

### Passing values in

| Flag | Effect |
| --- | --- |
| `--arg name value` | Define `$name` as a **string** |
| `--argjson name '{"a":1}'` | Define `$name` as parsed JSON |
| `--slurpfile name f.json` | `$name` = array of all values in the file |
| `--rawfile name f.txt` | `$name` = the file's contents as one string |
| `--args a b c` | Trailing arguments land in `$ARGS.positional` |
| `env.HOME`, `$ENV.HOME` | Read an environment variable |

## Paths and iteration

| Filter | Result |
| --- | --- |
| `.` | The whole input |
| `.foo` | The `foo` field |
| `.foo.bar` | Nested field |
| `.["odd key"]` | Field whose name needs quoting |
| `.foo?` | Same, but no error if the input isn't an object |
| `.[]` | Every element of an array / every value of an object |
| `.[]?` | Same, but skip inputs that can't be iterated |
| `.[2]` | Third element (`.[-1]` for the last) |
| `.[2:5]` | Slice — elements 2, 3 and 4 |
| `..` | Recursive descent: this value and everything under it |
| `.a, .b` | Two outputs, one after the other |
| `.a \| .b` | Pipe: feed `.a`'s output into `.b` |
| `.a // "fallback"` | Use the right side if the left is `false` or `null` |

> `.[]` produces a **stream** of values, not an array. Wrap it in brackets to
> collect it back up: `[.users[].name]`. This is the single most common
> "why is my output not a list?" moment.

## Selecting

```sh
# Objects from an array that match a condition
jq '.[] | select(.age > 30)'          # a stream of matches
jq '[.[] | select(.age > 30)]'        # collected into an array
jq 'map(select(.age > 30))'           # same thing, shorter

# Combine tests
jq '.[] | select(.age > 30 and .city == "Tokyo")'
jq '.[] | select(.name | test("^dr"; "i"))'     # regex, case-insensitive
```

| Filter | True when |
| --- | --- |
| `has("key")` | The object has that key (array: that index) |
| `"key" \| in(.obj)` | The key exists in the given object |
| `contains("str")` | Input contains the string / array / object |
| `startswith("x")` / `endswith("x")` | String prefix / suffix |
| `test("re"; "i")` | Regex matches (`i` = case-insensitive flag) |
| `any` / `all` | Any / all elements of an array are truthy |
| `isempty(f)` | The filter produces no output |

Related string tools: `match`, `capture`, `sub("a"; "b")`, `gsub`, `splits`,
`ltrimstr`, `rtrimstr`, `ascii_downcase`, `ascii_upcase`, and `trim` / `ltrim` /
`rtrim` (jq 1.8+).

## Reshaping

| Filter | What it does |
| --- | --- |
| `map(.price)` | Apply a filter to every element of an array |
| `map_values(.x)` | Apply it to every **value** of an object |
| `keys` / `keys_unsorted` | The object's keys |
| `values` | The values (also: drop nulls from a stream) |
| `to_entries` | `{"a":1}` → `[{"key":"a","value":1}]` |
| `from_entries` | The inverse |
| `with_entries(f)` | `to_entries \| map(f) \| from_entries` |
| `del(.foo)` | Delete a key (`del(.[1])`, `del(.a, .b)`) |
| `pick(.a, .b.c)` | Keep only these paths (jq 1.7+) |
| `.a + .b` | Merge objects (right wins), concatenate arrays/strings |
| `.a * .b` | Deep/recursive merge of objects |
| `flatten` / `flatten(1)` | Flatten nested arrays, optionally to a depth |
| `paths` / `leaf_paths` | Every path in the input, as arrays |
| `getpath(["a","b"])` | Look up a path built at runtime |
| `walk(f)` | Apply `f` to every value, depth-first |

### Building objects and arrays

```sh
jq '{name: .fullName, id}'            # bare "id" is shorthand for  id: .id
jq '{(.key): .value}'                 # computed key — parentheses required
jq '[.a, .b]'                         # array from two fields
jq -n --arg k foo --arg v bar '{($k): $v}'   # build JSON safely from shell vars
```

### Assignment

| Filter | Effect |
| --- | --- |
| `.a = 1` | Set `.a` to a value |
| `.a \|= ascii_downcase` | Update `.a` by running a filter on it |
| `.a += 1` | Arithmetic update (`-=`, `*=`, `/=`) |
| `.a //= "default"` | Set only if currently `false`/`null` |
| `(.items[] \| .qty) = 0` | Assign through a stream of paths |

## Aggregating

| Filter | What it does |
| --- | --- |
| `length` | Array/object/string length (number: absolute value) |
| `add` | Sum numbers, concatenate arrays/strings |
| `sort` / `sort_by(.age)` | Sort (`sort_by(.a, .b)` for tie-breaks) |
| `reverse` | Reverse an array |
| `group_by(.dept)` | Array of arrays, grouped by a key (sorts first) |
| `unique` / `unique_by(.id)` | Deduplicate |
| `min` / `max` / `min_by(f)` / `max_by(f)` | Extremes |
| `range(5)` / `range(2; 10; 2)` | Generate numbers |
| `first(f)` / `last(f)` / `nth(n; f)` | Take from a stream |
| `limit(3; .[])` | First 3 outputs of a stream (stops early) |
| `index("x")` / `indices("x")` | Position(s) of a value |
| `reduce .[] as $x (0; . + $x)` | Fold a stream into one value |
| `foreach .[] as $x (0; . + $x; .)` | Like `reduce`, but emit each step |

```sh
# Count how many items are in each group
jq 'group_by(.dept) | map({dept: .[0].dept, n: length})'

# Sum a field
jq '[.items[].amount] | add'

# Average
jq '[.[].score] | add / length'
```

## Output formats

| Filter | Produces |
| --- | --- |
| `@csv` / `@tsv` | CSV / TSV row — input must be an array |
| `@json` | The value as a JSON string |
| `@text` | Same as `tostring` |
| `@sh` | Shell-quoted, safe to `eval` |
| `@uri` | Percent-encoded |
| `@base64` / `@base64d` | Encode / decode |
| `@html` | HTML-escaped |

```sh
# Table output: -r is what strips the surrounding quotes
jq -r '.users[] | [.id, .name, .email] | @tsv' file.json

# Header row + data
jq -r '["id","name"], (.users[] | [.id, .name]) | @csv' file.json

# String interpolation
jq -r '.users[] | "\(.name) <\(.email)>"' file.json

# The @format prefix escapes every interpolation in the string
jq -r '@uri "https://example.com/search?q=\(.query)"' file.json
```

## Types, conditionals and errors

| Filter | What it does |
| --- | --- |
| `type` | `"object"`, `"array"`, `"string"`, `"number"`, `"boolean"`, `"null"` |
| `arrays`, `objects`, `strings`, `numbers`, `nulls`, `scalars` | Keep only inputs of that type |
| `tostring` / `tonumber` | Convert |
| `tojson` / `fromjson` | JSON text ↔ value (for embedded JSON strings) |
| `if .a > 1 then "big" else "small" end` | Conditional (`elif` allowed) |
| `and`, `or`, `not` | Booleans — `not` is a filter: `.a \| not` |
| `try .a.b catch "oops"` | Catch an error |
| `.a.b?` | Shorthand for `try`, yielding nothing on error |
| `error("message")` | Raise an error |
| `empty` | Produce no output at all |

> Only `false` and `null` are falsy. `0`, `""` and `[]` are all truthy — a
> frequent source of surprising `//` and `select` results.

## Variables and functions

```sh
# Bind a value, then use it further down the pipe
jq '.total as $t | .items[] | {name, share: (.amount / $t)}'

# Destructuring
jq '.point as {x: $x, y: $y} | "\($x),\($y)"'

# Define a reusable function
jq 'def pct(n): (n * 100 | floor); .items[] | pct(.ratio)'
```

## Multiple documents & streaming

| Command | What it does |
| --- | --- |
| `jq -c '.[]' file.json` | Array → NDJSON (one object per line) |
| `jq -s '.' ndjson` | NDJSON → a single array |
| `jq -n 'inputs'` | Read every input document explicitly |
| `jq -n '[inputs] \| length'` | Count documents without slurping into memory first |
| `jq -s '.[0] * .[1]' a.json b.json` | Deep-merge two files |
| `jq --stream '...'` | Emit `[path, value]` events for huge files |

## Recipes

```sh
# Validate a file (exit status only)
jq -e . file.json > /dev/null && echo valid

# Pull one field from every object, sorted and deduplicated
jq -r '[.[].country] | unique | .[]' file.json

# Filter with a shell variable, safely
jq --arg name "$NAME" '.users[] | select(.name == $name)' file.json

# Edit a file in place (jq has no -i; use sponge from moreutils)
jq '.version = "2.0"' pkg.json | sponge pkg.json

# Turn a lines-of-text file into a JSON array
jq -R -s 'split("\n") | map(select(. != ""))' list.txt

# Flatten nested objects into dotted key=value pairs
jq -r 'paths(scalars) as $p | "\($p | join(".")) = \(getpath($p))"' file.json

# Diff-friendly normalisation: sorted keys, compact, one doc per line
jq -S -c . file.json
```

## `yq` — the same ideas for YAML

`yq` (mikefarah's Go implementation, v4) uses a jq-like expression language over
YAML, JSON, XML, TOML, CSV/TSV and properties. Unlike `jq` it edits files in
place and preserves comments.

> There are two unrelated tools called `yq`: this one, and a Python wrapper
> around `jq`. Expression syntax differs between them — `yq --version` should
> report **v4.x** for what follows.

| Command | What it does |
| --- | --- |
| `yq '.a.b' file.yaml` | Read a value (scalars come out unquoted) |
| `yq -i '.version = "2.0"' file.yaml` | Edit **in place** |
| `yq -o=json '.' file.yaml` | YAML → JSON |
| `yq -p=json -o=yaml '.' file.json` | JSON → YAML |
| `yq -o=props '.' file.yaml` | Flatten to `a.b = c` properties |
| `yq -n '.a.b = 1'` | Build a document from nothing |
| `yq -N '.' multi.yaml` | Suppress `---` document separators |
| `yq ea '.' a.yaml b.yaml` | `eval-all`: load every document first |
| `yq -i '.image.tag = strenv(TAG)' k8s.yaml` | Insert an env var as a string |
| `yq -i '... comments=""' file.yaml` | Strip all comments |
| `yq -i 'sort_keys(..)' file.yaml` | Sort keys recursively |

```sh
# Merge several YAML files into one document
yq ea '. as $item ireduce ({}; . * $item)' base.yaml override.yaml

# Every image referenced by a multi-document Kubernetes manifest
yq '.spec.template.spec.containers[].image' deploy.yaml

# Round-trip a YAML file through jq
yq -o=json '.' file.yaml | jq '.services | keys'
```

Key differences from `jq`: `-i` edits in place, `-r` is unnecessary because
scalars are unwrapped by default (`--unwrapScalar=false` to keep quotes), there
is no `-s`/slurp (use `eval-all`), and `strenv()` / `env()` read environment
variables.

---

> Pinned to **jq-1.8.2** and **yq v4.53.3**. `jq --help` lists the flags and
> `man jq` documents every builtin; the online manual at <https://jqlang.org/manual/>
> is version-tagged, and <https://jqplay.org> is useful for trying a filter
> against a sample. For yq, `yq --help` and <https://mikefarah.gitbook.io/yq>.
