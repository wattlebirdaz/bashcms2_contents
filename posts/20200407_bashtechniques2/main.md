---
Keywords: bash, awk, sed
Copyright: (C) 2020 Riki Otaki
---

# Useful bash techniques 2

Introducing shell scripts that I found useful.

Today, I'll explain some use cases of `awk`. (I will not be able to explain the command in detail so if you are interested please check out the man page. `man awk` )

## Using variables in awk

awk is a command for manipulating input data line by line.

Say, you have a logfile that looks like this.


`cat logifle`
```bash
a 1
a 5
b 4
b 3
b 2
c 8
c 9
c 10
d 8
d 12
d 1
d 2
```

If you want to print only the second column, you can do the following. 

```bash
cat logfile | awk '{print $2}'
```
output
```
1
5
4
3
2
8
9
10
8
12
1
2
```

`$N` means the N-th column.

If you want to print the second column only if it is an even number, you can do the following.

```bash
cat logfile | awk '$2%2==0{print $2}'
```

or

```bash
cat logfile | awk '{if($2%2==0){print $2}}'
```
output
```
4
2
8
10
8
12
2
```

The basic format of awk will be the following.

```bash
awk 'PATTERN1{ACTION1}PATTERN2{ACTION2}...'
```

If the input **row** (or **record**) matches PATTERN1 it will execute ACTION1, PATTERN2 then ACTION2 and so on.

Things like regular expressions, ranges, BEGIN/END, etc will go into PATTERN. For more information look [here](https://www.gnu.org/software/gawk/manual/html_node/Pattern-Overview.html#Pattern-Overview).

In Action, you can plug in what you want the program to do when the input matches the pattern. It can be expressions, control statements(if , for, while and do), compound statements and so on. For more information, look [here](https://www.gnu.org/software/gawk/manual/html_node/Action-Overview.html#Action-Overview). Note that multiple actions can be set for pattern by putting a semicolon between them i.e `PATTERN1{ACTION1a;ACTION1b;ACTION1c;...}`

Here is a question.

How would you implement in awk, if you want to aggregate the numbers in the second column by the letter in the first column and print them?

From this, 
```
a 1
a 5
b 4
b 3
b 2
c 8
c 9
c 10
d 8
d 12
d 1
d 2
```

you want this.

```
a 6
b 9
c 27
d 23
```

This can be achieved by using some variables.
Look at the following piece of code.

```bash
cat logfile | awk '{if(L!=$1&&L!=""){print L,C;C=$2;L=$1}else{C+=$2;L=$1}}END{print L,C}'
```

By executing this, you get the wanted output.

The code looks a bit complicated because it is in one-liner. The following is an organized version of the code.

```bash
{
    if (L!=$1 && L!="")
        { print L, C; C=$2; L=$1}
    else
        { C+=$2; L=$1 }
}
END
{print L, C}
```

C is a variable for counting the sum of numbers in the second column. L is a variable that stores the letter in the first column. If the first letter changes, it will print L and C.

There are two things to note. 

1. C and L will be null at first. You can detect it by comparing it with "". Look at the first if condition where it is comparing L with "". 
Another thing about null value is that if you add an integer to it, it will just be the integer.

2. END pattern is needed to detect the end of file. An END rule is executed only once when it has read all the lines of the file. Likewise, a BEGIN rule is executed only once before reading the first row.

This is just one example of dealing with aggregate data. If you know any other ways of doing this in bash, please share.



