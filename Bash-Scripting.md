# Help for a command 
```sh
man head   #man command_to_search
```

# History 
```sh
history  # !55 to run 55th command , !head for most recently used head command 
```
# Files and Directories

### Curent Directory
```sh
pwd
```
### list files
```sh
ls  /file/path

ls -R -F /path/    #-R-> recursive shows all files -F -> / after every directory, * after runnable program
```
pwd - /home/krushna  
/airflow  -> relative path  
/home/krushna/airflow -> absolute path

The shell decides if a path is absolute or relative by looking at its first character: If it begins with /, it is absolute. If it does not begin with /, it is relative.

### Changing Directories
```sh
cd /directory/path
cd /home/krushna/airflow
cd ..     # move to parent directory
cd .      # current directory
cd ~      # home directory
```
### Copying Files
```sh
cp original.txt duplicate.txt
cp file1 file2 /backup 
```

### Move a file
```sh
mv file1.txt file2.txt ..   # moved to one level above 
mv file1.txt /home/krushna/backup/
```

### Rename a file
mv can also be used to rename a file
```sh
mv file.txt old-file.txt
```

### Delete file
```sh
rm file1.txt file2.txt    #removes permanently
```

## Create and Delete directories
rmdir works only when directory is empty
```sh
mkdir direcory_name
rmdir directory_name
```

### View files Content
```sh
cat filename.txt

less filename.txt

less fil1.txt file2.txt   #:n next file, :p previous file :q quit

head file.csv   # prints first 10 lines

head -n 5 file.csv   # prints first 5 lines

tail file.csv    # last lines

cut -f 1-5,8 -d , file.csv   #-f fileds -> 2 to 5 columns, and 8 number column -d -> delimeter as comma , 
```
grep selects files according to the content
```sh
grep bicuspid file.csv
```
- -c: print a count of matching lines rather than the lines themselves
- -h: do not print the names of files when searching multiple files
- -i: ignore case (e.g., treat "Regression" and "regression" as matches)
- -l: print the names of files that contain matches, not the matches
- -n: print line numbers for matching lines
- -v: invert the match, i.e., only show lines that don't match
