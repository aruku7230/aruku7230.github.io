---
title: "Parse options in bash shell"
date: 2022-09-19
lastmod: 2022-09-19
draft: false
tags:
  - bash
  - linux
---

## Parse options manually

A good example is from
[Bash FAQ #35](https://mywiki.wooledge.org/BashFAQ/035#Manual_loop).

It is very flexible, and can handle:
- Single-letter options (`-v`).
- GNU-style long options (`--long-options`).
- Tcl-style long options (`-longopts`).
- Options with or without arguments.
- Both `--file FILE` and `--file=FILE` can be handled.

However, it do not support:
- Single-letter option splitting (like `-xvf` handled as `-x -v -f`).
- Options after positional arguments (like `arg1 arg2 -v`)

## Using `getopt`

While `getopts` is a shell builtin command, `getopt` is an external program.
There are two `getopt` programs: the old Unix `getopt`, and the new enhanced
Linux `getopt`.

The old Unix `getopt` have many
[problems](https://mywiki.wooledge.org/ComplexOptionParsing#util-linux.27s_special_getopt),
such as cannot handle empty arguments strings, or arguments with embedded whitespace.
**DO NOT USE IT**.

If Linux `getopt` is installed, it can handle all of the following and more:
- Single-letter options (`-v`).
- GNU-style long options (`--long-options`).
- Tcl-style long options (`-longopts`).
- Options with or without arguments.
- Both `--file FILE` and `--file=FILE` can be handled.
- Single-letter option splitting (like `-xvf` handled as `-x -v -f`).
- Options after positional arguments (like `arg1 arg2 -v`)

## An example using `getopt`

Refer to [hear](https://stackoverflow.com/a/29754866) and
[here](https://unix.stackexchange.com/a/62961), below is a example of using `getopt`.

```sh
#! /usr/bin/env bash

set -o errexit -o nounset

# === Print usage ==============================================================
usage() {
    cat << USAGE 1>&2
Usage:
    $(basename -- "$0") [options] download <remote-path> <local-path>
    $(basename -- "$0") [options] upload   <local-path> <remote-path>
Options
    -h, --help
          print help.
    -m, --move-to <destination folder>
          when downloading finished, move remote resource to remote destination folder.
          or, when uploading finished, move local resource to local destination folder.
USAGE
}

# ==============================================================================
# Parse arguments
# ==============================================================================
parse_args() {
    local options=hm:
    local longoptions=help,move-to:
    if ! params="$(getopt --options $options --longoptions $longoptions \
        --name "$(basename -- "$0")" -- "$@")"
    then
        usage; return 1
    fi

    eval set -- "$params"

    help="n"
    move_to=""
    action=""
    remote_path=""
    local_path=""
    while true
    do
        case "$1" in
            -m|--move-to)
                move_to="$2"; shift 2;;
            -h|--help)
                usage; help="y"; return 0;;
            --)
                shift; break;;
            *)
                usage; return 1;;
        esac
    done

    # After option parsing, remained arguments should be 3.
    if [ $# -ne 3 ]; then
        usage; return 1
    fi

    action="$1"
    case "$action" in
        download)
            remote_path="$2"
            local_path="$3"
            ;;
        upload)
            local_path="$2"
            remote_path="$3"
            ;;
        *)
            usage; return 1
            ;;
    esac
}

download() {
    :
}

upload() {
    :
}

parse_args "$@"
if [[ "$help" = "y" ]]; then
    exit 0;
fi

case "$action" in
    download)
        download remote_path local_path
        ;;
    upload)
        upload local_path remote_path
        ;;
esac
```

## Using `getopts`

`getopts` is a shell builtin command. It is limited to parse single-letter options
and allow single-letter option splitting (like `-xvf` handled as `-x -v -f`)

The most disadvantage is that it can not handle GNU-style long option
(`--long-option`), and others it do not support are
- Options after positional arguments (like `arg1 arg2 -v`)
- Tcl-style long options (`-longopts`).

Tutorials on how to use `getopts`
- [How to Use getopts to Parse Linux Shell Script Options](https://www.howtogeek.com/778410/how-to-use-getopts-to-parse-linux-shell-script-options/)
- [Small getopts tutorial](https://wiki.bash-hackers.org/howto/getopts_tutorial)

## Conclusion
- Parse options manually if you want most compatibility (works on most OS), and
  options is not complicated.
- Use `getopt` if you can ensure Linux `getopt` is installed and want to support
  a variety ways of specifying options.
- Use `getopts` if only one-letter options are used.
