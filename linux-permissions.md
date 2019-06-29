# Permissions

Basic permissions levels:
4: Read
2: Write
1: Execute

By adding the basic permissions, we get combined levels:
3: Write and execute
5: Read and execute
7: Read, write and execute

To change permissions, on a folder or file we use the `chmod` command:
```
chmod 777 /path/to/file
chmod 755 /path/to/dir -R
```
The three digits stand for permissions of the owner, group owner and everybody else for the file or directory respectively. The `-r` flag applies permissions recursively for all subdirectories.

# Ownership

```
sudo chown user file
sudo chown -R user directory
```

```
sudo chgrp group file
sudo chgrp group directory
```