<h1 align="center">POSIX-sh-memo</h1>
<p align="center">Hui-Jun Chen's memo on POSIX-compliant shell</p>


<!-- vim-markdown-toc GFM -->

* [Replace `head -n 1`](#replace-head--n-1)
	* [Mission: define `var` based on the first line of `/proc/stat`.](#mission-define-var-based-on-the-first-line-of-procstat)
	* [Mission: define `var` based on the first line of another multi-line variable](#mission-define-var-based-on-the-first-line-of-another-multi-line-variable)
* [Replace `tail -n 1`](#replace-tail--n-1)
	* [Mission: define `var` based on the last line of `/proc/stat`](#mission-define-var-based-on-the-last-line-of-procstat)
	* [Mission: define `var` based on the last line of another multi-line variable](#mission-define-var-based-on-the-last-line-of-another-multi-line-variable)

<!-- vim-markdown-toc -->

## Replace `head -n 1`

### Mission: define `var` based on the first line of `/proc/stat`.

Instead of:

```sh
var=$(head -n 1 /proc/stat)
```

You can:

```sh
read -r var < /proc/stat
```

Efficiency comparison:

```sh
# Repeat both command 1000 times:

time (
    for i in $(seq 1000); do
        var=$(head -n 1 /proc/stat)
    done
)

# Output
# 0.64s user 0.27s system 103% cpu 0.886 total

time (
    for i in $(seq 1000); do
        read -r var < /proc/stat
    done
)

# Output
# 0.03s user 0.02s system 98% cpu 0.055 total
```

### Mission: define `var` based on the first line of another multi-line variable

Instead of:

```sh
source="One
Two
Three"

var=$(printf '%s' "$source" | head -n 1)
```

You can:

```sh
source="One
Two
Three"

# Pre-define a newline character first
nl='
'

var=${source%%$nl*}
```

Efficiency comparison:

```sh
# Repeat both command 1000 times:

time (
    for i in $(seq 1000); do
        var=$(printf '%s' "$source" | head -n 1)
    done
)
# Output:
# 1.87s user 0.68s system 120% cpu 2.120 total


time (
for i in $(seq 1000); do
nl='
'
var=${source%%$nl*}
done
)

# Output:
# 0.01s user 0.00s system 95% cpu 0.012 total
```

Caveat: the POSIX way to define a new-line variable `$nl` is to just type a space. Thus, if we have any indentation or space in the `$nl` variable, then this parameter expansion won't work.

## Replace `tail -n 1`

### Mission: define `var` based on the last line of `/proc/stat`

Instead of:

```sh
var=$(tail -n 1 "/proc/stat")
```

You can:

```sh
nl='
'

source=$(</proc/stat)
var=${source##*$nl}
```

Efficiency comparison:

```sh
time (
    for i in $(seq 1000); do
        var=$(tail -n 1 "/proc/stat")
    done
)

# Output:
# 0.69s user 0.30s system 102% cpu 0.964 total

time (
for i in $(seq 1000); do
nl='
'

source=$(</proc/stat)
var=${source##*$nl}
done
)

# Output:
# 0.06s user 0.03s system 99% cpu 0.092 total
```
Caveat: the POSIX way to define a new-line variable `$nl` is to just type a space. Thus, if we have any indentation or space in the `$nl` variable, then this parameter expansion won't work.

### Mission: define `var` based on the last line of another multi-line variable

Instead of:

```sh
source="One
Two
Three"

var=$(printf '%s' "$source" | tail -n 1)
```

You can:

```sh
source="One
Two
Three"

# Pre-define a newline character first
nl='
'
var=${source##*$nl}
```

Efficiency comparison:

```sh
time (
    for i in $(seq 1000); do
        var=$(printf '%s' "$source" | tail -n 1)
    done
)

# Output
# 1.83s user 0.71s system 121% cpu 2.094 total


time (
for i in $(seq 1000); do
nl='
'

var=${source##*$nl}
done
)
# Output
# 0.01s user 0.00s system 62% cpu 0.012 total
```

Caveat: the POSIX way to define a new-line variable `$nl` is to just type a space. Thus, if we have any indentation or space in the `$nl` variable, then this parameter expansion won't work.

Caveat 2: Consider the following scenario:

```sh
source="One
Two
Three
"

var=$(printf '%s' "$source" | tail -n 1)
# var=Three

nl='
'

var2=${source##*$nl}
# var2=
```

Need to check whether the multi-line variable ends with `\n` or not.
