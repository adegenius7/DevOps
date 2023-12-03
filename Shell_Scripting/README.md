# Shell Scripting
## Running a Shell script
### This first line (#!/bin/bash or #!/bin/sh) has a name. It is known as ‘she-bang‘(shabang). This derives from the concatenation of the tokens sharp (#) and bang (!). It is also called as sh-bang, hashbang, poundbang or hash-pling. In computing, a she-bang is the character sequence consisting of the characters number sign and exclamation mark (#!) at the beginning of a script.
![bash](./img/she-bang.png)

## Variables
### A shell variable name can consist of uppercase or lowercase letters, plus digits and the underscore character _. The name can have any length, but the first character cannot be a digit. Uppercase letters are distinguished from lowercase ones, so NAME, name, and Name are all different names.
![variables](./img/Variable.png)

## Making you script Executable

### this can be done using chmod command thus:
![chmod](./img/chmod.png)

## Control Flow
### Bash allows us to have a control flow flow statements like if-else, for loops, while loops, and case statements to control the flow of execution in your scripts. These statements allows you to make decisions, iterate over lists, and execute different commands based on specified conditions.
### Using if-else
![cont](./img/control-if.png)

### The output should be 
![alt](./img/control-output.png)
## Iterating through a list using for-loop
### We can use loops and conditional statements in BASH scripts to perform some repetitive and tricky problems
![for](./img/for_loop.png)
![output](./img/for_loop_output.png)

## Command Substitution
### Command substitution allows the output of a command to replace the command itself. Command substitution occurs when a command is enclosed as follows:
![cs](./img/cs.png)
## Input and output
### Bash provides various ways to handle input and output. You can use the read command to accept user input, and output text to the console using the echo command. Additionally, you can redirect input and output using operators like >(output to a file), <(input from a file), and |(pipe the output of one command as input to another).
![inp](./img/input_output.png)
![red](./img/redirection.png)

## Functions
### A Bash function is essentially a set of commands that can be called numerous times. The purpose of a function is to help you make your bash scripts more readable and to avoid writing the same code repeatedly. Compared to most programming languages, Bash functions are somewhat limited.
![function](./img/function.png)
![fo](./img/foutput.png)

## Running Our first shell script
![script](./img/script.png)
![output](./img/script_output.png)

## Directory Manipulation and Navigation

![navi](./img/naviscript.png)
![naviout](./img/navioutput.png)

## File Operations And Sorting
![files](./img/sorting.png)
### The output is thus:
![filesoutput](./img/sorting_output.png)

## Working with Numbers and Calculations

