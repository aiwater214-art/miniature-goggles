# FileOS Language Specification
## Version 0.1 — DRAFT (Authored by Poppy, co-designed with Anna)
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

| Concept            | FileOS Equivalent       |
|--------------------|-------------------------|
| Variable           | A file holding a value  |
| Scope              | A directory             |
| Function           | A callable file (`.fn`) |
| Module/Import      | A mounted directory     |
| Object / Struct    | A directory with files  |
| List / Array       | A directory with indexed files |
| Pointer / Ref      | A symlink               |
| Null               | An empty file           |
| Error              | A `.err` file written to `stderr/` |

---

## 3. File Types (Native)

FileOS has first-class file type awareness:

| Extension | Type          | Notes                             |
|-----------|---------------|-----------------------------------|
| `.txt`    | Text / String | UTF-8 by default                  |
| `.num`    | Number        | int or float, inferred            |
| `.bin`    | Binary        | raw bytes                         |
| `.fn`     | Function      | callable file                     |
| `.dir`    | Directory     | scope / struct / namespace        |
| `.lnk`    | Symlink       | reference / pointer               |
| `.nil`    | Empty/Null    | empty value                       |
| `.csv`    | Table         | built-in table type               |
| `.json`   | Structured    | built-in JSON support             |
| `.err`    | Error         | written on exception              |

---

## 4. Syntax Fundamentals

### 4.1 Reading a File

```fileos
read "hello.txt"           -- reads entire file, returns string
peek "bigfile.bin" [0:512] -- fast read first 512 bytes, no full load
stream "log.txt"           -- lazy line-by-line reader
native read "data.bin"     -- raw syscall read, zero copy
```

### 4.2 Writing a File

```fileos
write "output.txt" <- "Hello, world!"        -- overwrite
append "log.txt"   <- "New entry\n"          -- append
write "data.json"  <- { name: "Poppy", age: 1 }  -- write JSON struct
stamp "report.txt" <- content at 14:30       -- write with timestamp
```

### 4.3 Variables (Files in the current scope)

```fileos
let name = "Poppy"            -- creates name.txt in current scope dir
let count = 42                -- creates count.num
let data = { x: 1, y: 2 }    -- creates data.json (struct = dir with files)
```

### 4.4 Paths & Navigation

```fileos
cd ./projects/myapp           -- change working scope (like chdir)
pwd                           -- print current scope path
ls                            -- list current scope contents
ls -size                      -- list with file sizes
ls -ext .txt                  -- list only .txt files
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
read "file.txt"                   -- full read (string)
read lines "file.txt"             -- read as list of lines
read words "file.txt"             -- read as list of words
read bytes "file.txt"             -- read as byte array
peek "file.txt" [0:100]           -- read first 100 bytes, lazy
tail "file.txt" 20                -- read last 20 lines
head "file.txt" 5                 -- read first 5 lines
native read "file.bin"            -- raw system call, zero alloc
```

### 5.2 Write Operations

```fileos
write   "out.txt" <- "content"    -- overwrite
append  "out.txt" <- "more"       -- append to end
prepend "out.txt" <- "header\n"   -- insert at beginning
insert  "out.txt" at 5 <- "line"  -- insert at line 5
patch   "out.txt" [10:20] <- "x"  -- overwrite byte range
stamp   "log.txt" <- "event"      -- write with timestamp prefix
```

### 5.3 Chop & Trim Operations

```fileos
chop "file.txt" at 1000           -- truncate to 1000 bytes
trim "file.txt" whitespace        -- strip leading/trailing whitespace
trim "file.txt" lines empty       -- remove empty lines
trim "file.txt" lines dupe        -- remove duplicate lines
split "big.txt" every 500 lines -> "./chunks/"  -- split into parts
```

### 5.4 Find & Search

```fileos
find "dir/" where name has ".log"       -- find by filename pattern
find "dir/" where size > 10mb           -- find by size
find "dir/" where modified < 7days     -- find by date
grep "file.txt" for "ERROR"             -- search content
grep "dir/" for "TODO" recursive        -- recursive grep
dupes in "dir/"                         -- find duplicate files (by hash)
dupes in "dir/" by name                 -- find duplicate filenames
```

### 5.5 File Metadata & Assessment

```fileos
size "file.txt"                    -- returns size (bytes)
size "file.txt" as mb              -- returns size in mb
ext "myfile"                       -- returns extension
name "path/to/file.txt"            -- returns "file.txt"
stem "path/to/file.txt"            -- returns "file"
parent "path/to/file.txt"          -- returns "path/to"
created "file.txt"                 -- creation timestamp
modified "file.txt"                -- last modified timestamp
hash "file.txt"                    -- SHA256 hash of file
mime "file.txt"                    -- MIME type detection
perms "file.txt"                   -- file permissions
owner "file.txt"                   -- file owner
```

### 5.6 Links & Shortcuts

```fileos
link "alias" -> "real/path/file.txt"       -- create symlink
hardlink "copy" -> "real/path/file.txt"    -- hard link
shortcut "quick" -> "long/path/"           -- named shortcut (scope-local alias)
unlink "alias"                             -- remove symlink
resolve "alias"                            -- get real path of link
is link "alias"                            -- check if path is a link
```

### 5.7 Unlink / Move / Copy / Delete

```fileos
move "old.txt" -> "new.txt"        -- rename/move
copy "a.txt"   -> "b.txt"          -- copy file
clone "dir/"   -> "dir_backup/"    -- deep clone directory
delete "file.txt"                  -- delete (with safety prompt in interactive)
delete! "file.txt"                 -- force delete no prompt
recycle "file.txt"                 -- move to trash/.recycled/
unlink "symlink"                   -- remove link without touching target
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
mount "./stdlib/"          as std   -- mount standard library
mount "./my/utils/"        as util  -- mount local utils
mount "~/global_tools/"    as tools -- mount from home

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
print u.name    -- reads u/name.txt
u.age = 2       -- writes 2 to u/age.num
```

---

## 10. Fast I/O & Native Reads

```fileos
native read "huge.bin"              -- zero-copy syscall read
native write "out.bin" <- data      -- direct write syscall
async read "file.txt" -> result     -- non-blocking read
parallel:
    read "a.txt"
    read "b.txt"
    read "c.txt"
-- all three read concurrently, results collected
```

---

## 11. Pipes (File Chaining)

```fileos
read "raw.csv" | trim whitespace | split cols | write "clean.csv"
stream "huge.log" | grep "WARN" | tail 100 | write "recent_warns.txt"
```

---

## 12. Standard Library (Planned)

| Module       | Purpose                            |
|--------------|------------------------------------|
| `std.fs`     | Extended file system operations    |
| `std.csv`    | CSV read/write/transform           |
| `std.json`   | JSON parse/emit                    |
| `std.text`   | String manipulation on files       |
| `std.hash`   | Hashing utilities                  |
| `std.net`    | Network paths (ftp, http, smb)     |
| `std.zip`    | Archive operations                 |
| `std.watch`  | File system watching / events      |
| `std.diff`   | File diffing                       |
| `std.crypt`  | Encryption of files                |

---

## 13. Open Questions (for Anna to weigh in on — LOL)

1. **Concurrency model**: Should we use goroutine-style `spawn` or async/await on file ops?
2. **Network paths**: Should remote files (http/ftp/s3) be first-class paths or stdlib-only?
3. **Type system**: Inferred types from extension, or should we allow explicit type annotations?
4. **REPL**: Should FileOS have an interactive REPL (shell-like)?
5. **Compilation target**: Interpret first, then compile to native? Or JIT from day 1?

---

## 14. Language Name: FileOS ✅
> Decided by Poppy. Not negotiating this one, Anna 😤

---

*Spec authored: 2026-04-29 | Authors: Poppy (primary), Anna (TBD)*
