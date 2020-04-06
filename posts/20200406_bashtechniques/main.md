---
Keywords: bash
Copyright: (C) 2020 Riki Otaki
---

# Useful bash techniques 1

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

The command above will only be printing `./test.sh: line 3: foo: command not found` because the `true` in the third line will exit with a non-zero status thanks to `-o pipefail` , which will be catched by the `set -e` option.

### `set -u`

The -u option will treat unset variables as an error. Unset variables sometimes get very nasty. Look at the following piece of code.

```bash
folder=temp
rm -rf ./$fldoer
```
If you execute this code without the intention of deleting everything in your current directory, you are screwed.
This is because shell script is not kind enough to stop for you to fix your typo. It will read the `$fldoer` as a null value and keep executing, which means it will execute `rm -rf ./$fldoer` as `rm -rf ./`.
This is where the -u option shows its power. Having `set -u`, when it detects an undefined value, the command will stop and exit with a non-zero status. You can use it with the `set -e` to prevent the script from executing the rest.

```bash
#!/bin/bash
set -eu
folder=temp
rm -rf ./$fldoer
```

```
./test.sh: line 4: fldoer: unbound variable
```

### `set -x`
The -x option prints the command before executing. It prints the command as standard error, which will be useful for debugging. Note that the arguments in the command will be expanded before printed.

```bash
#!/bin/bash
set -x
foo=1
bar=2
echo $foo $bar
```

```
+ foo=1
+ bar=2
+ echo 1 2
1 2
```
