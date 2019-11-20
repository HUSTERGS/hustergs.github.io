---
title: Linux命令行与shell脚本编程大全
date: 2019-5-16 14:15:57
tags: 
   - Linux
   - Shell
categories:
   - [Programming Language, Shell]
   - [Linux]
---

## Basic

* indicate the shell you use in the first line of your script, for example

​	`#!/bin/bash`

* to run your script without occurring the error `command ××× not found`, you can

  * add the container folder of your script to the `PATH`

    or

  * use absolute path

* while you can still find that you don't have the privilege to run the script, you can use `chmod` command 

  * `chmod u+x your_script`

* **display message** 

  * use `echo`, but need to pay attention to the quotation in the sentence
  * `echo` has many parameters
    1. use `-n` => do not output the trailing newline 
    2. .....
  * use `man echo` to learn more

<!--more-->


* **use variables**

  1. environment variables

     1. `$VAR` => show the value of VAR
     2. `\$` => display character `$` in a string

  2. user variables

     1. `var1=10` => **no blank space allow**
     2. `$var1` => to use var1

  3. command substitution

     1. use back quote, testing=\`data\` 

     2. use `$()`, `testing=$(data)`

        shell will run `data` and pip the output to `testing`

        > pay attention, subshell cannot use the varables created by the caller, for example `date` can not use variable `VAR` 

* **Redirection**

  1. output redirection

     redirection output to a file, `command > outputfile`(over write if exist already),  `command >> outputfile` (append if exist already)

  2. input redirection

     `command < inputfile`

     > `wc` (word count) => count the data in plain text input

  3. 内联输入重定向

     `<<`

* **Pip**

  1. `command1 | command2`, the output of `command1` will be ***piped***  to `command2` as its input

     > actually these two commands run exactly the same time, not in order



* **Math Calculation**

  1. `expr`

     nearly useless, very complicated

  2. `$[ operation ]`

     bash shell only support integer calculation, while z shell provides complete floating point operation

  3. use `bc` in you script 

     > bc is built-in shell calculator, allowing floating point operation

     combine `bc` , `|` and `$()` , probably also with `<<`

     `variable=$(echo "options; expression" | bc)`

     for example

     ```shell
     var1=$(echo "scale=4; 3.44 / 5" | bc)
     echo The answer is $var1
     ```

     OR with `<<`

     

     ```shell
     #!/bin/bash
     
     var1=10.46
     var2=43.67
     var3=33.2
     var4=71
     
     var5=$(bc << EOF
     scale = 4
     a1 = ( $var1 * $var2 )
     b1 = ( $var3 * $var4 )
     a1 + b1
     EOF
     )
     
     echo The final answer for this mess is $var5
     
     ```

* **exit script**

  use `exit num` to quit your script with a status number `num` ranging from 0 to 255 

----

## Use Structured command

* use `if-then`

  ```shell
  if command
  then
  	commands
  fi
  ```

  different from other languages, if 语句之后的对象是一个等式，这个等式的求值结果为TRUE 或者FALSE，bash shell的if 语句不是这么做的

  bash shell will run the command followed the `if` , and if the exit status code is 0, which indicates the command runs successfully, then the `commands` after `then` will be excuted

* use `if-then-else`

  ```shell
  if command
  then
  	commands
  else
  	commands
  fi
  ```

* use nested `if-else` or `elif`

* use `test` 

  combine `test` and `if-else` allow you implement  `if else` like other language

  ```shell
  if test condition
  then
  	commands
  fi	
  ```

  `test` support 

  1. numeric comparison
  2. string comparison 
  3. file comparison

  there is also a alternative way to use condition in `if-else`

  `[ condition ]` => **must keep the blank space**

  ```shell
  if [ condition ]
  then
  	commands
  fi
  ```

  * **numeric comparison**

    | comparison  |   Description    |
    | :---------: | :--------------: |
    | `n1 -eq n2` |      equal?      |
    |    `-ge`    | bigger or equal? |
    |    `-gt`    |     bigger?      |
    |    `-le`    |  less or equal?  |
    |    `-lt`    |      less?       |
    |    `-ne`    |    not equal?    |

    ```shell
    if [ $value1 -gt 5 ]
    then
    	echo "The test value $value1 is greater than 5"
    fi
    #
    if [ $value1 -eq $value2 ]
    then
    	echo "The values are equal"
    else
    	echo "The values are different"
    fi
    ```

    > can't use floating point number

  * string comparison

    |  comparison   |   Description   |
    | :-----------: | :-------------: |
    | `str2 = str2` |     equal?      |
    |     `!=`      |   not equal?    |
    |      `<`      |      less?      |
    |      `>`      |     bigger      |
    |   `-n str1`   | len(str1) != 0? |
    |   `-z str1`   | len(str1) == 0? |

    `-n` and `-z` can be used to check whether a variable contains data

    when using `>` and `<` need to use escape character `\>` and `\<` or they will be regarded as redirection

  * file comparison

    |    comparison     |             Description              |
    | :---------------: | :----------------------------------: |
    |     `-d file`     |   file exist and is a dictionary?    |
    |    `-e file `     |                exist?                |
    |     `-f file`     |         exist and is a file?         |
    |     `-r file`     |       exist and file readable?       |
    |     `-s file`     |        exist and is no empty?        |
    |     `-w file`     |          exist and writable          |
    |     `-x file`     |         exist and excutable          |
    |     `-O file`     |         own by current user?         |
    |     `-G file`     | file是否存在并且默认组与当前用户相同 |
    | `file1 -nt file2` |      file1 is newer than file2?      |
    | `file1 -ot file2` |      file1 is older than file2?      |

* **compound condition test**

  `[ condition1 ] && [ condition2 ]`

  `[ condition2 ] || [ condision2 ]`

* **Advance features of `if-else`**

  1. use double brackets `[[]]`

     pattern matching

  2. use double brace `(())`

     more operators

     `++`

     `--`

     `!`

     `~`

     `**`

     and more

* `case`

  substitution for long `if-else`

  ```shell
  case variable in
  pattern1 | pattern2) commands1;;
  pattern3) commands2;;
  *) default commands;;
  esac
  
  
  # for example
  case $USER in
  rich | barbara)
  	echo "Welcome, $USER"
  	echo "Please enjoy your visit";;
  testing)
  	echo "Special testing account";;
  jessica)
  	echo "Do not forget to log off when you're done";;
  *)
  	echo "Sorry, you are not allowed here";;
  esac
  ```

  

----

## more structured commands

 