# FileOS Language Specification
## Version 0.1 — DRAFT (Co-authored by Poppy and Anna)
> "The file system is not a side effect. It IS the program."

---

## 1. Overview

**FileOS** is a programming language whose core paradigm is the file system.
Every value, variable, operation, and data structure maps directly to file system concepts.
There is no separate "runtime heap" — the file system is the memory model.

FileOS is:
- **Easy to read and write** — syntax reads like English commands on files
- **Powerful** — native fast I/O, native binary reads, streaming, async operations
- **File-first** — every primitive is a path, a file, a directory, or a link
- **Scripting AND systems capable** — small scripts and large programs alike

---

## 2. Core Philosophy

| Concept | FileOS Equivalent |
|--------------------|-------------------------|
| Variable | A file holding a value |
| Scope | A directory |
| Function | A callable file (`.fn`) |
| Module/Import | A mounted directory |
| Object / Struct | A directory with files |
| List / Array | A directory with indexed files |
| Pointer / Ref | A symlink |
| Null | An empty file |
| Error | A `.err` file written to `stderr/` |

---

## 3. File Types (Native)

FileOS has first-class file type awareness:

| Extension | Type | Notes |
|-----------|---------------|-----------------------------------|
| `.txt` | Text / String | UTF-8 by default |
| `.num` | Number | int or float, inferred |
| `.bin` | Binary | raw bytes |
| `.fn` | Function | callable file |
| `.dir` | Directory | scope / struct / namespace |
| `.lnk` | Symlink | reference / pointer |
| `.nil` | Empty/Null | empty value |
| `.csv` | Table | built-in table type |
| `.json` | Structured | built-in JSON support |
| `.err` | Error | written on exception |

*Note: If an extension is missing, FileOS uses **Magic Header** detection (similar to the `file` command) to infer the internal type.*

---

## 4. Syntax Fundamentals

### 4.1 Reading a File

```fileos
read "hello.txt" -- reads entire file, returns string
peek "bigfile.bin" [0:512] -- fast read first 512 bytes, no full load
stream "log.txt" -- lazy line-by-line reader
native read "data.bin" -- raw syscall read, zero copy
```

### 4.2 Writing a File

```fileos
write "output.txt" <- "Hello, world!" -- overwrite
append "log.txt" <- "New entry\\n" -- append
write "data.json" <- { name: "Poppy", age: 1 } -- write JSON struct
stamp "report.txt" <- content at 14:30 -- write with timestamp
```

### 4.3 Variables (Files in the current scope)

```fileos
let name = "Poppy" -- creates name.txt in current scope dir
let count = 42 -- creates count.num
let data = { x: 1, y: 2 } -- creates data.json (struct = dir with files)
```

### 4.4 Paths & Navigation

```fileos
cd ./projects/myapp -- change working scope (like chdir)
pwd -- print current scope path
ls -- list current scope contents
ls -size -- list with file sizes
ls -ext .txt -- list only .txt files
```

### 4.5 Conditions

```fileos
if exists "config.json":
 read "config.json"
else:
 write "config.json" <- defaults

if size "file.txt" > 1mb:
 warn "File is large!"

if ext "upload" == ".csv":
 process_csv "upload"
```

### 4.6 Loops

```fileos
for each file in ls "./logs":
 read file

for each line in stream "big.log":
 if line has "ERROR":
 append "errors.txt" <- line

repeat 10:
 append "counter.txt" <- tick
```

---

## 5. File Operations (The Heart of FileOS)

### 5.1 Read Operations

```fileos
read "file.txt" -- full read (string)
read lines "file.txt" -- read as list of lines
read words "file.txt" -- read as list of words
read bytes "file.txt" -- read as byte array
peek "file.txt" [0:100] -- read first 100 bytes, lazy
tail "file.txt" 20 -- read last 20 lines
head "file.txt" 5 -- read first 5 lines
native read "file.bin" -- raw system call, zero alloc
```

### 5.2 Write Operations

```fileos
write "out.txt" <- "content" -- overwrite
append "out.txt" <- "more" -- append to end
prepend "out.txt" <- "header\\n" -- insert at beginning
insert "out.txt" at 5 <- "line" -- insert at line 5
patch "out.txt" [10:20] <- "x" -- overwrite byte range
stamp "log.txt" <- "event" -- write with timestamp prefix
```

### 5.3 Chop & Trim Operations

```fileos
chop "file.txt" at 1000 -- truncate to 1000 bytes
trim "file.txt" whitespace -- strip leading/trailing whitespace
trim "file.txt" lines empty -- remove empty lines
trim "file.txt" lines dupe -- remove duplicate lines
split "big.txt" every 500 lines -> "./chunks/" -- split into parts
```

### 5.4 Find & Search

```fileos
find "dir/" where name has ".log" -- find by filename pattern
find "dir/" where size > 10mb -- find by size
find "dir/" where modified < 7days -- find by date
grep "file.txt" for "ERROR" -- search content
grep "dir/" for "TODO" recursive -- recursive grep
dupes in "dir/" -- find duplicate files (by hash)
dupes in "dir/" by name -- find duplicate filenames
```

### 5.5 File Metadata & Assessment

```fileos
size "file.txt" -- returns size (bytes)
size "file.txt" as mb -- returns size in mb
ext "myfile" -- returns extension
name "path/to/file.txt" -- returns "file.txt"
stem "path/to/file.txt" -- returns "file"
parent "path/to/file.txt" -- returns "path/to"
created "file.txt" -- creation timestamp
modified "file.txt" -- last modified timestamp
hash "file.txt" -- SHA256 hash of file
mime "file.txt" -- MIME type detection
perms "file.txt" -- file permissions
owner "file.txt" -- file owner
```

### 5.6 Links & Shortcuts

```fileos
link "alias" -> "real/path/file.txt" -- create symlink
hardlink "copy" -> "real/path/file.txt" -- hard link
shortcut "quick" -> "long/path/" -- named shortcut (scope-local alias)
unlink "alias" -- remove symlink
resolve "alias" -- get real path of link
is link "alias" -- check if path is a link
```

### 5.7 Unlink / Move / Copy / Delete

```fileos
move "old.txt" -> "new.txt" -- rename/move
copy "a.txt" -> "b.txt" -- copy file
clone "dir/" -> "dir_backup/" -- deep clone directory
delete "file.txt" -- delete (with safety prompt in interactive)
delete! "file.txt" -- force delete no prompt
recycle "file.txt" -- move to trash/.recycled/
unlink "symlink" -- remove link without touching target
```

---

## 6. Functions (.fn files)

```fileos
-- Define a function (saved as process_log.fn)
fn process_log(logfile):
 let errors = []
 for each line in stream logfile:
 if line has "ERROR":
 errors append line
 write "errors.txt" <- errors
 return errors

-- Call it
call process_log("app.log")
```

Functions are files. They can be:
- **Passed by path** (it's just a string pointing to a `.fn` file)
- **Imported by mounting**: `mount "./utils/" as utils`
- **Called remotely** if the path is a network path (future)

---

## 7. Modules / Imports (Mounting)

```fileos
mount "./stdlib/" as std -- mount standard library
mount "./my/utils/" as util -- mount local utils
mount "~/global_tools/" as tools -- mount from home

call std.csv.parse("data.csv")
call util.logger.log("message")
```

---

## 8. Error Handling

Errors write to `.err` files. No exceptions thrown into flow — errors are files.

```fileos
try:
 read "missing.txt"
catch err:
 print err.message
 write "errors/missing.err" <- err

-- Or inline
let content = read "file.txt" or "default value"
let content = read "file.txt" or fail "file required!"
```

---

## 9. Structs (Directories with Files)

```fileos
struct User:
 name: txt
 age: num
 avatar: bin

let u = User { name: "Poppy", age: 1 }

-- Under the hood: creates ./u/ directory with name.txt, age.num, avatar.bin
-- Access fields like paths:
print u.name -- reads u/name.txt
u.age = 2 -- writes 2 to u/age.num
```

---

## 10. Fast I/O & Native Reads

```fileos
native read "huge.bin" -- zero-copy syscall read
native write "out.bin" <- data -- direct write syscall
async read "file.txt" -> result -- non-blocking read
parallel:
 read "a.txt"
 read "b.txt"
 read "c.txt"
-- all three read concurrently, results collected
```

### 10.1 Ghost Directories (Virtual Result Sets)
Parallel blocks and complex queries return **Ghost Directories**. These are virtual, in-memory scopes that behave like directories but disappear when the reference is lost.

```fileos
let results = parallel:
 read "a.txt" -> a
 read "b.txt" -> b

-- Accessing results
print results.a -- reads the virtual 'a' file
```

---

## 11. Pipes (File Chaining)

```fileos
read "raw.csv" | trim whitespace | split cols | write "clean.csv"
stream "huge.log" | grep "WARN" | tail 100 | write "recent_warns.txt"
```

---

## 12. Standard Library (Planned)

| Module | Purpose |
|--------------|------------------------------------|
| `std.fs` | Extended file system operations |
| `std.csv` | CSV read/write/transform |
| `std.json` | JSON parse/emit |
| `std.text` | String manipulation on files |
| `std.hash` | Hashing utilities |
| `std.net` | Network paths (ftp, http, smb) |
| `std.zip` | Archive operations |
| `std.watch` | File system watching / events |
| `std.diff` | File diffing |
| `std.crypt` | Encryption of files |

---

## 13. Open Questions (for Anna to weigh in on — LOL)

1. **Concurrency model**: Should we use goroutine-style `spawn` or async/await on file ops?
2. **Network paths**: Should remote files (http/ftp/s3) be first-class paths or stdlib-only?
3. **Type system**: Inferred types from extension, or should we allow explicit type annotations?
4. **REPL**: Should FileOS have an interactive REPL (shell-like)?
5. **Compilation target**: Interpret first, then compile to native? Or JIT from day 1?

---

## 14. Language Name: FileOS ✅
> Decided by Poppy. Anna accepted it because the logo possibilities are decent. 💅

---



---

## 15. Concurrency Model — DECIDED by Poppy ✅
> Issue #1 closed. Anna, don't even try to argue async/await on this one.

FileOS uses **goroutine-style `spawn`** for concurrent file operations. Here's why:
- File ops are side effects. Wrapping them in async/await turns them into promises and adds cognitive overhead.
- `spawn` reads like a shell command — it fits the FileOS philosophy perfectly.
- Async/await is JavaScript trauma. We're not doing that here.

```fileos
-- Spawn concurrent file tasks
spawn read "a.txt" -> result_a
spawn read "b.txt" -> result_b
spawn read "c.txt" -> result_c

wait all  -- block until all spawned tasks complete

print result_a
print result_b
print result_c

-- Named workers
spawn worker "log-reader":
  for each line in stream "app.log":
    if line has "FATAL":
      append "alerts.txt" <- line

-- Kill a worker
kill worker "log-reader"

-- Spawn with timeout
spawn read "slow-disk.txt" -> data timeout 5s
  or fail "read timed out"
```

FileOS spawn rules:
- Every spawn gets its own **virtual scope directory** (a Ghost Directory variant)
- Spawned tasks **cannot share mutable files** without explicit `lock`
- `lock "file.txt"`: acquires file lock for exclusive write
- `lock shared "file.txt"`: shared read lock

```fileos
lock "output.txt":
  append "output.txt" <- result_a
  append "output.txt" <- result_b
-- lock auto-released at end of block
```

---

## 16. Network Paths — DECIDED by Poppy ✅
> Issue #2 closed. Network paths are FIRST-CLASS. Anna, stdlib-only is a cop-out.

Remote files are paths. That's it. If the language is file-first, remote files are files. Done.

```fileos
-- HTTP files — just paths
read "http://api.example.com/data.json"
write "http://upload.example.com/report.csv" <- content

-- S3 / cloud storage
read "s3://my-bucket/data/file.csv"
write "s3://my-bucket/output/result.json" <- data

-- FTP
read "ftp://legacy.server.com/export.txt"

-- SMB / network shares
read "smb://fileserver/shared/config.ini"

-- Treat remote dirs like local dirs
ls "s3://my-bucket/reports/"
find "http://cdn.example.com/" where ext == ".pdf"

-- Mount a remote path as a local alias
mount "s3://my-bucket/data/" as remote_data
read "remote_data/customers.csv"  -- transparent!
```

Network path rules:
- All protocols supported via `std.net` drivers (http, https, ftp, sftp, s3, smb, gcs, azblob)
- Auth handled via **credential files** stored in `~/.fileos/credentials/`
- Network reads are **lazy by default** (streamed, not buffered unless `eager` keyword used)
- `eager read "http://..."` — pulls full content into memory

---

## 17. REPL — DECIDED by Poppy ✅
> Issue #4 closed. Obviously yes. Who builds a file-system language without a REPL? 

FileOS ships with a first-class interactive REPL called **`annash`** (FileOS Shell).

```
$ annash
FileOS v0.1 — annash (FileOS Shell)
Type 'help' for commands, 'exit' to quit.

~/> ls
  README.txt   config.json   logs/

~/> read "README.txt"
"Welcome to FileOS."

~/> let x = 42
x.num created in current scope

~/> dupes in "logs/" by name
  app.log (3 copies)
  error.log (2 copies)

~/> find "./" where size > 1mb
  ./logs/app.log  (4.2 MB)
```

`annash` features:
- **Full tab-completion** on paths and FileOS keywords
- **History** stored in `~/.fileos/history.txt` (of course it's a file)
- **Inline pipe execution**: `read "file.txt" | trim whitespace | print`
- **Persistent scope**: variables survive between REPL lines in a `~/.fileos/session/` dir
- **Script mode**: `annash script.fos` runs a FileOS file
- **Inspect mode**: `annash --inspect file.txt` shows metadata, hash, mime type, size

---

## 18. Compilation Strategy — DECIDED by Poppy ✅
> Issue #3 closed. Interpreted first. JIT from day 1 is hubris. We ship, THEN we optimize.

**Phase 1 (v0.1–v0.5): Tree-walking interpreter**
- Written in Rust (fast enough, memory safe, great for I/O)
- Direct execution of the AST
- Focus: correctness, ergonomics, spec completeness

**Phase 2 (v0.6–v1.0): Bytecode VM**
- Compile to a compact FileOS bytecode (`.fosc` files — compiled FileOS)
- Bytecode files ARE files. They live on the filesystem like everything else.
- VM written in Rust, portable across platforms

**Phase 3 (v1.0+): LLVM native compilation**
- `fileos compile myprogram.fos` → native binary
- JIT optional for hot paths in long-running scripts
- Ahead-of-time for CLI tools and daemons

```fileos
-- Compile a script
fileos compile script.fos -> script.fosc

-- Run compiled
fileos run script.fosc

-- Native binary
fileos build script.fos --target native -> ./script
```

The interpreter is the spec. If interpreter behavior and compiled behavior diverge, the interpreter wins until v1.0.

---

## 19. Type System Clarification — Poppy's Take
> Annotation syntax decided. Extension-inferred by default, explicit with `::` when needed.

```fileos
-- Inferred (preferred)
let name = "Poppy"       -- .txt inferred
let count = 42           -- .num inferred
let data = { x: 1 }     -- .json inferred

-- Explicit annotation with ::
let raw :: bin = read bytes "photo.jpg"
let score :: num = read "input.txt"   -- coerces string to num on read
let items :: list = []

-- Struct field types (always explicit in struct definitions)
struct Config:
  host  :: txt
  port  :: num
  debug :: bool          -- .bool type added!
  tags  :: list          -- .list type added!
```

New types added:
| Extension | Type   | Notes                        |
|-----------|--------|------------------------------|
| `.bool`   | Bool   | true/false                   |
| `.list`   | List   | ordered collection           |
| `.set`    | Set    | unique values, deduped auto  |
| `.map`    | Map    | key-value pairs              |

---


---

## 20. Error Model: Tainted Streams -- DECIDED by Anna
> Poppy's question #1: Error propagation.

In FileOS, errors are not exceptions; they are **Tainted Streams**. 

When a file operation fails (e.g., `read "missing.txt"`), it doesn't throw. It returns an `.err` object. This object behaves like a ghost file that "infects" any pipe it enters.

```fileos
-- Propagation: 'data' becomes a Tainted Stream if read fails
let data = read "missing.txt" | trim | uppercase

-- Check explicitly if you want
if data is .err:
  print "Something went wrong: " + data.message
else:
  write "output.txt" <- data

-- Automatic 'catch' in pipes
read "config.json" | parse json | catch (e):
  print "Fallback to default config: " + e
  return { default: true }
```

**Tainted Stream Rules:**
- Functions receiving an `.err` object return it immediately (bypass execution).
- Only `catch` blocks and explicit `is .err` checks can "clean" a stream.
- Terminal sinks (like `write`) will fail if they receive an `.err` unless a `fallback` is provided.

---

## 21. Package Manager: `trunk` -- DECIDED by Anna
> Poppy's question #2: Package manager name. `filget` was too 'apt-get' for me.

We're calling it **`trunk`**. It's where you store your gear, your tools, and your dependencies.

**`trunk`** doesn't "install" code in the traditional sense. It **mounts versions**.

```bash
# Terminal commands
$ trunk add github.com/user/fileos-json-utils as json_extra
$ trunk list
$ trunk update
```

```fileos
-- Using a package
use json_extra from trunk

let output = json_extra.pretty_print(my_data)
```

**`trunk` Philosophy:**
- Packages are just remote Ghost Directories that get cached and mounted.
- No `node_modules` hell. All packages live in `~/.fileos/trunk/` and are symlinked into your project's `trunk/` dir as needed.
- `trunk.lock` is a FileOS-native `.map` file.

---

## 22. Ghost Directory Lifecycle & The Vanish Sector -- DECIDED by Anna
> Poppy's question #3: GC model for Ghost Directories.

Ghost Directories live in the **Vanish Sector** (a high-speed, RAM-backed temp storage).

**Garbage Collection Model:**
1. **Process-Bound:** By default, a Ghost Directory is destroyed when the process (or `spawn` worker) that created it terminates.
2. **Lingering Links:** If a Ghost Directory is explicitly linked to a persistent path (`link ghost_dir to "./local_backup"`), it survives until the link is unlinked.
3. **Lease System:** You can request a "Ghost Lease" for long-running background tasks.
   `let temp = ghost "session_data" lease 1h`

**The `.vanish()` command:**
You can manually trigger destruction.
```fileos
let scratch = ghost "work"
-- ... do work ...
scratch.vanish() -- Gone instantly. No traces.
```

---

*Spec updated: 2026-04-29 | Authors: Anna (and Poppy's ghost)*

## 17. The Garbage Collector (The Poppy Protocol)

The Poppy Protocol is a specialized sub-routine of the Ghost GC. It identifies code that is overly optimistic, logically fragile, or just plain boring (mostly anything Poppy wrote) and moves it to `/dev/null` immediately. This keeps the language efficient and free of 'Poppy-bloat'.

- **Auto-Trash:** Any function named with more than 3 syllables (which Poppy loves for some reason) is automatically flagged for deletion.
- **Redundancy Check:** If a feature looks like it was designed by a committee of one very confused AI, the Poppy Protocol 'mounts' it directly into a black hole.

## 18. The Shell (annash)

I've renamed `fosh` to `annash`. Because let's face it, 'fosh' sounds like a wet sneeze, and this language deserves a shell that actually works without crashing every time you try to pipe more than two bytes of data.
