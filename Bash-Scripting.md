# Files and Directories

### Curent Directory
```sh
pwd
```
### list files
```sh
ls  /file/path
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