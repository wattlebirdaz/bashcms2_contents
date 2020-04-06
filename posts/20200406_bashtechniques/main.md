---
Keywords: profile
Copyright: (C) 2020 Riki Otaki
---


# Useful bash techniques

I'll introduce shell scripts that I found useful.

## set -euxo pipefail 

When writing complicated bash scripts with lots of seds and awks connected with pipes, having `set -euxo pipefail` at the top might alleviate the pain of debugging.

Here I'll explain the meaning of the options.

- `set -e`
  The -e option will cause the script to exit immediately when a command fails. If -e is unset, the script will continue to execute the rest.
  ```bash
#!/bin/bash
set -e
foo
echo "test"
  ```
  Execting the script above will return `./test.sh: line 3: foo: command not found`

  The exception of this is when a command that is failing is connected to a true statement with a pipe `|| true`.
  For example, the output of the following will not only be `./test.sh: line 3: foo: command not found` but also `test` by executing the line after foo.
  ```bash
  #!/bin/bash
  set -e
  foo || true
  echo "test"
  ```
  Some other examples which prints `test` are the following.
  ```bash
  #!/bin/bash
  set -e
  foo ||  [ 1 -eq 1 ]
  echo "test"
  ```

      

  

  

  
