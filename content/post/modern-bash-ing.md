+++
draft = false
title = "Modern bash-ing"
date = "2017-06-01T14:35:17+02:00"
tags = [ "bash", "linux" ]
categories = [ "bash", "BackEnd" ]
series = []
images = []
description = ""
summary = """
Best practices for developing/debugging complex bash scripts.
"""

+++

# Strict mode

{{< highlight bash >}}
#!/bin/bash
set -euo pipefail
{{< /highlight >}}

## Bail on failure: set -e

This make your script to exit when any command inside fail (non-zero exit
status). Bash is very forgibbing and continue executing by default.

Now you have to handle errors in your scripts.
To allow errors you have two patterns to learn.


{{< highlight bash >}}
# for a single command
your-failure-command || echo


# for multiline / zones
set +e
your-failure-command
your-failure-command2
set -e
{{< /highlight >}}

## Bail on undefined: set -u

Throw an error if access an undefined variable with exceptions of: `$*` and
`$@`. But dont trow if you are using the value in default assignament.

{{< highlight bash >}}
set -u
bar=$foo

echo $bar

bar=${foo:-alpha}

# Now we set foo explicitly:
foo="beta"

# ... and the default value is ignored. Here $bar is set to "beta":
bar=${foo:-alpha}

# To make the default an empty string, use ${VARNAME:-}
empty_string=${some_undefined_var:-}
{{< /highlight >}}

OUTPUT


Note that subsequent files (`source`d) also have the same flags.
If you `source`d a *dangerous* file disable/enable the flag.

{{< highlight bash >}}
set +u
source path-to-dangerous-bach-file
set -u
{{< /highlight >}}



# fail on pipe errors: set -o pipefail

Pipe return code is always the last one. The return codes from previous comands
are ignored (even with `set -e`).

This force bash to use the first return code error as command return code.

{{< highlight bash >}}
set -o pipefail
cat  /non-existent-file | sort
> ???
echo $?
> 2
# TODO show the error
grep some-string /non/existent/file | sort

{{< /highlight >}}


# Debug

Many times in far past I debugged scripts using `echo`. Right now, `echo`
is more a comment that a debug utility. Unlike almost every language
Bash have a `print what i'm going to execute` that is priceless.

{{< highlight bash >}}
set -e
{{< /highlight >}}


# Pipe debug

{{< highlight bash >}}
blabla | tee debug-file | blablabla-continue
{{< /highlight >}}

`debug-file` contains the contents of the pipe, tee also print into stdout :)

# Multiline strings

How many times do i have to echo 5 `echo`s in consecutive lines.
Please use:

{{< highlight bash >}}
cat <<DELIM | tee /your-file
one sheep
two sheeps
three sheeps
DELIM
{{< /highlight >}}

I know what are you thinking: `tee` is not necessary (but it is).
This is a pattern that can be used for many thins:


{{< highlight bash >}}
# overwrite file not accessible under original user (ex: root)
cat <<DELIM | sudo tee /your-file
DELIM
# append root file
cat <<DELIM | sudo tee -a /your-file
DELIM
{{< /highlight >}}


# Deferred execution / cleanups

Execute a function at the end of the script.

TODO test script chain

{{< highlight bash >}}
TMP_PATH=$(mktemp -d)

function finish {
  rm -rf ${TMP_PATH}
}
# this function will be executed when exit (even on error execution)
trap finish EXIT
{{< /highlight >}}


# http://redsymbol.net/articles/unofficial-bash-strict-mode/
# https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/

After many years of working with bash I end up with a few good partices,
some I use


# Handling exit codes

Exit codes can be tricky to handle the most common usage is the combination
`exit` with `$?`.

The problem raise when you call another script inside your script and
the subscript call `exit` because It also exit you main script.
This is not an error because (maybe) you are `sourcing` a script inside
your main script.

What you need to do is to execute the script in another process, so exit only
affect itself.


{{< highlight bash >}}
#!/bin/sh

# sourcing a script, just for reference purposes
# ./subscript.sh

# run the script in a separate process
$(./subscript.sh)

SUBSCRIPT_STATUS=$?
if [ $SUBSCRIPT_STATUS -eq 0 ]; then
  echo "subscript.sh ran successfully"
else
  echo "subscript.sh crapped out"
fi

{{< /highlight >}}
