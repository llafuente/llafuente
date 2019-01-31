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

Nothing here is new, but it's a good collection to make your life easier.

## Use Strict mode

{{< highlight bash >}}
#!/bin/bash
set -euo pipefail
{{< /highlight >}}

### Bail on failure: set -e

This makes your script to exit when any command inside fail (non-zero exit
status). Bash is very forgiving and continue executing by default.

Now you have to handle errors in your scripts or at least makes sure you know
this what may fail.

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


### Bail on undefined: set -u

Throw an error if undefined variable is accessed with the exceptions of: `$*` and
`$@`. But don't throw if you are using the value in default assignment.

{{< highlight bash >}}
set -u
bar=$foo
# OUTPUT: -bash: foo: unbound variable

bar=${foo:-alpha}
echo $bar
# OUTPUT: alpha

# Now we set foo explicitly:
foo="beta"

# ... and the default value is ignored. Here $bar is set to "beta":
bar=${foo:-alpha}
echo $bar
# OUTPUT: beta

# To make the default an empty string, use ${VARNAME:-}
empty_string=${some_undefined_var:-}
echo $empty_string
# OUTPUT:

{{< /highlight >}}



Note that subsequent files (`source`d) also have the same flags.
If you `source`d a *dangerous* file disable/enable the flag.

{{< highlight bash >}}
set +u
source path-to-dangerous-bach-file
set -u
{{< /highlight >}}



### fail on pipe errors: set -o pipefail

Pipe return code is always the last one. The return codes from previous comands
are ignored (even with `set -e`).

This force bash to listen all return error codes, and fail on the first error.

{{< highlight bash >}}
set -o pipefail
cat  /non-existent-file | sort
# OUTPUT: cat: /non-existent-file: No such file or directory

echo $?
# OUTPUT: 1

grep some-string /non/existent/file | sort
# OUTPUT: grep: /non/existent/file: No such file or directory

{{< /highlight >}}


## Debug

### Output tracing: set -x

Many times in the far past I debugged scripts using `echo`. Right now, `echo`
is more a info loggin than a debuging utility. Unlike almost every language
Bash have a `print what I'm going to execute` that is priceless, priceless!

{{< highlight bash >}}
set -x
echo $HOME
# OUTPUT: + echo /c/Users/llafuente
# OUTPUT: /c/Users/llafuente
{{< /highlight >}}

You see the command to be executed with all variables resolved, again priceless!

### Pipe debug

Did you have a sporadic failure in a pipe command? Use this to debug it!

{{< highlight bash >}}
blabla | tee debug-file | blablabla-continue
{{< /highlight >}}

`debug-file` contains the contents of the pipe and `tee` also print into stdout.

## Multiline strings

Stop using X echo lines!

{{< highlight bash >}}
cat <<DELIM | tee file
one sheep
two sheeps
three sheeps
DELIM
{{< /highlight >}}

I know what are you thinking: `tee` is not necessary.
To print to stdout I just removed, ok.
To print to a file I will redirect stdout to file.
What about a root file?

This is a pattern to copy&paste and adapt all use cases.

{{< highlight bash >}}
# overwrite file not accessible under original user (ex: root)
cat <<DELIM | sudo tee /etc/file
DELIM
# append root file
cat <<DELIM | sudo tee -a /root/file
DELIM
# stdout
cat <<DELIM
DELIM
{{< /highlight >}}


## Deferred execution / cleanups

Execute a function at the end of the script.

{{< highlight bash >}}
TMP_PATH=$(mktemp -d)

# this function will be executed when exit, error or success
function finish {
  rm -rf ${TMP_PATH}
}
trap finish EXIT

# ...
{{< /highlight >}}

## Handling exit codes

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
  echo "subscript.sh success"
else
  echo "subscript.sh failure"
fi

{{< /highlight >}}

## references

* http://redsymbol.net/articles/unofficial-bash-strict-mode/

* https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/
