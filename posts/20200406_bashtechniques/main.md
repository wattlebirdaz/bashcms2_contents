---
Keywords: profile
Copyright: (C) 2020 Riki Otaki
---


# Useful bash techniques

Introducing shell scripts that I found useful.

## set -euxo pipefail 

When writing complicated bash scripts with lots of seds and awks connected with pipes, having `set -euxo pipefail` at the top might alleviate the pain of debugging.

Here I'll explain the meaning of the options.

### `set -e`
The -e option will cause the script to exit immediately when a command fails. If -e is unset, the script will continue to execute the rest.
  
```bash
#!/bin/bash
set -e
foo
echo "test"
```
Execting the script above will return `./test.sh: line 3: foo: command not found` (I name the script `test.sh`)

The exception of this is when a command that is failing is connected to a true statement with a pipe `| true`. A true statement here means a command that exits with a zero status.
For example, the output of the following will not only be `./test.sh: line 3: foo: command not found` but also `test` by executing the line after foo. 

```bash
#!/bin/bash
set -e
foo | true
echo "test"
```
```
./test.sh: line 3: foo: command not found
test
```

Some other examples which print `test` are the following.
```bash
#!/bin/bash
set -e
foo |  [ 1 -eq 1 ]
echo "test"
```
```bash
#!/bin/bash
set -e
foo | echo "test"
```

As you can see, `set -e` is far from enough. Usually, a pipeline will not stop when the faliing command is in the middle of the pipeline because `set -e` only looks at the exit code of the last command in the pipeline. 

### `set -o pipefail`
This is where `set -o pipeline` comes in. This will modify the exit status of the last command of the pipeline to:

- zero: If all commands in the pipeline exit successfully
- non-zero: Other

```bash
#!/bin/bash
set -eo pipefail
foo | true
echo "test"
```

The command above will only be printing `./test.sh: line 3: foo: command not found` because the `true` will exit with a non-zero status thanks to `-o pipefail` , which will be catched by the `set -e` option.

### `set -u`

