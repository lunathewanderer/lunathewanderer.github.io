---
title: "101: Bash Scripting & Tshark"
description: "The truth is, you know... I never went to school either."
draft: false
date: 2026-10-06
tags:
  - Programming
categories:
  - Learning
cover: "https://i.pinimg.com/1200x/e6/b0/0b/e6b00bd9c0abc5abb852644c81fcfee0.jpg"
banner: "https://i.pinimg.com/1200x/e6/b0/0b/e6b00bd9c0abc5abb852644c81fcfee0.jpg"
---

## Bash Scripting: Text Processing

### Regex

[Regex, Regexp, or Regular Expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions), is a sequence of characters that specifies a match pattern in text. Usually [such patterns](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Cheatsheet) are used by string-searching algorithms for "find" or "find and replace" operations on strings, or for input validation.

### grep

The most basic function is `grep 'pattern' filename` to search for a pattern in a file. However, while sharing the same functionality, piping commands into `grep` is preferred.

Flags:
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
- For wildcards, substitutes * into the parameter you want to sweep.

Moreover, there are mode of Regex Dialects in grep:
- BRE - Basic Regular Expression (flag = None): Default engine. Meta-characters like `?`, `+`, `{`, `|`, `(`, and `)` are treated as literal text unless escaped with a backslash `\`
> Example: `grep 'a\{2,3\}' file` matches `aa` and `aaa`
- ERE - Extended Regular Expression (flag = -E): Equivalent to egrep. Meta-characters work natively without backslashes.
> Example: `grep -E 'a{2,3}' file` matches `aa` and `aaa`
- PCRE - Perl Compatible Regular Expressions (flag = -P): Unlocks Perl-Compatible syntax. Supports advanced shorthand concepts like `\d`, `\s`, and [lookarounds](https://www.linkedin.com/pulse/mastering-lookaround-assertions-regular-expressions-abodunrin-).

There is a dedicated documentation about `Regex in grep` [here](https://www.cyberciti.biz/faq/grep-regular-expressions/).

### awk

`awk` is heavily used in text processing, espcially in a table.

- `awk` dissects the line into numbered fields. `$0` is the entire line, `$1` is the first field, `$2` the second, `$3` the third,...
- `awk` comes in the form of `awk 'PATTERN {ACTION}' filename`. Simply put, PATTERN is the IF, and ACTION is the THEN.

For example:
```bash
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

```bash
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

https://www.w3schools.com/bash/bash_cut.php

### sort


### tail


### head
