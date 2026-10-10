---
title: "101: Bash Scripting & Tshark"
description: "The truth is, you know... I never went to school either."
draft: false
date: 2026-08-23
tags:
  - Programming
categories:
  - Learning
cover: "/images/kujou-banner.jpeg"
banner: "/images/kujou-banner.jpeg"
---

## Regex

Regex, Regexp, or Regular Expression, is a sequence of characters that specifies a match pattern in text. Usually such patterns are used by string-searching algorithms for "find" or "find and replace" operations on strings, or for input validation.

### Anchor & Validation

- `^` indicates the start of string/line. For example, `^HTTP` matches lines starting with "HTTP".
- `$` indicates the end of string/line.
- `\A` indicates the start of the entire input, ignoring multiline modes.
- `\Z` indicates the end of the entire input, ignoring multine modes.

### Character Classes & Sets

- `.` matches any character except newline. For example, `c.t` matches cat, cut, c4t,...
- `[abc]` the explicit set that match any one character inside.
- `[^abc]` the negated set that match any character NOT inside.
- `\d` = `[0-9]`
- `\D` = `[^0-9]`
- `\w` = `[a-zA-Z0-9_]`
- `\W` = `[^a-zA-Z0-9_]`
- `\s` matches whitespace, including: [space ( ), tab (\t), newline (\n), carriage-return (\r), form-feed (\f), vertical-tab (\v)]
- `\S` matches non-whitespace shorthand, be it letters, digits, punctuations,..., anything but a break.

### Quantifiers

- `*` matches 0 or more times (equivalent to {0,}).
- `+` matches 1 or more times (equivalent to {1,}).
- `?` matches 0 or 1 time (makes the preceding element optional).
- `{n}` Matches exacty n times.
- `{n,}` Matches at least n times (no upper limit).
- `{n,m}` Matches between n and m times inclusively.

### Groups, Alternation & Lookarounds

- `(abc)` Capturing Group (saves match into \1, \2, etc.)
- `(?:abc)` Non-Capturing Group (groups logic without saving memory)

a|b — Alternation / OR (cat|dog matches "cat" OR "dog")

(?=abc) — Positive Lookahead (matches if followed by "abc")

(?!abc) — Negative Lookahead (matches if NOT followed by "abc")

(?<=abc) — Positive Lookbehind (matches if preceded by "abc")

(?<!abc) — Negative Lookbehind (matches if NOT preceded by "abc")

## Bash Scripting: Text Processing

### grep

The most basic function is `grep 'pattern' filename` to search for a pattern in a file. However, while sharing the same functionality, piping commands into `grep` is preferred.

- `grep -i` ignores case differences, rendering the output case-insensitive.
- `grep -r` searches "recursively" through all files in a directory, including its subdirectories.
- `grep -v` finds lines that do not match the pattern, the inverse.
- `grep -c` counts the appearances of the pattern.
- `grep -C` to displays nearby texts.
- `grep -n` displays line number alongside matched lines.
- `grep -w` forces grep to match only exact, full words only.
- `grep -x` forces grep to match only exact, whole lines.
- `grep -l` output only filenames containing matches (stops reading file on first match).
- `grep -L` output only filenames that do not contain matches.

For wildcards, substitutes * into the parameter you want to sweep.

Regular Expression, or Regex/Regexp

### awk

`awk` is heavily used in text processing, espcially in a table.

- `awk` dissects the line into numbered fields. `$0` is the entire line, `$1` is the first field, `$2` the second, `$3` the third,...
- `awk` comes in the form of `awk 'PATTERN {ACTION}' filename`. Simply put, PATTERN is the IF, and ACTION is the THEN.

For example:
```
awk '$2 == "Math" { print $1 }' class.txt 
--> IF field 2 is "Math" THEN print field 1

awk '$3 > 90 { print $0 }' class.txt 
--> IF field 3 is greater than 90 THEN print the whole line.

awk '{ sum += $3 } END { print "Total Score:", sum }' class.txt 
--> Track the scores (field 3) and print out the total at the END.
--> Note: awk features automatic loop.
```

The `awk` command has options to change how it works:

- `-F""` the field seperator, setting what separates the data fields.
- `v""` the assign variable, setting a variable to be used in the script.
- `-f""` the `.awk` file scripting.

Regular Expression, or Regex/Regexp

```
awk '$5 ~ /^\/api\/v[12]/ { print $0 }' file.txt
     │  │  │             │
     │  │  │             └─> ACTION: Print the whole line ($0) if it matches
     │  │  └───────────────> REGEX: Look for strings starting with "/api/v1" or "/api/v2"
     │  └──────────────────> OPERATOR: "matches regex pattern"
     └─────────────────────> FIELD: Examine column 5
```

### sed

The `sed` command is a non-interactive "stream editor" used to perform basic text transformations on an input stream (a file or input from a pipeline), unlike standard text editors (like vim or nano) where you open a file and edit it manually.

- `sed -n` to suppresses automatic printing of the pattern space.
- 
