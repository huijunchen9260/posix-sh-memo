<h1 align="center">POSIX-sh-memo</h1>
<p align="center">Hui-Jun Chen's memo on POSIX-compliant shell</p>

# Replace external program with built-in function

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

Why: Efficiency

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

var2=$(printf '%s' "$source" | head -n 1)
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

Why: Efficiency

```sh
# Repeat both command 1000 times:

time (
    for i in $(seq 1000); do
        var2=$(printf '%s' "$source" | head -n 1)
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



