# head Command

## Purpose
+ The head command, as the name implies, print the top N number of data of the given input. By default, it prints the first 10 lines of the specified files. If more than one file name is provided then data from each file is preceded by its file name. 

+ It can also be piped to list the N most recently used file in a directory after listing it out.

## Syntax
+ head [filename]
+ head -N [filename]
+ ls [option] | head -n N

it returns the value 0 if it is successfully executed