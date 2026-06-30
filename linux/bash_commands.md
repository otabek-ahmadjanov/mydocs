# Linux bash commands

- **FIle System Navigation**

    ```bash
    **ls ---> list storage
    ls /
    ls -l ---> detailed list
    ls -l /home ---> for specific package
    pwd ---> print working directory
    ls -l -a ---> list storage all
    ls -la ---> list storage all**
    
    ```

- **Working with file**

    ```bash
    **touch testfile.txt ---> creating file / overwrites existing one
    cat testfile.txt ---> view file content
    nano ---> creates an empty file
    nano testfile2.txt ---> creates an empty file with this name or opens this file
    cp testfile2.txt newcopy.txt ---> copies file to new file name. if newcopy.txt exists then it copied content overwrites existing one
    mv testfile3.txt notes/  ---> move file to another directory
    mv *.txt notes/ ---> move all .txt files to specified directory
    mv ,,/testfile.txt . ---> move file to current directory
    mv testfile3.txt newname.txt ---> rename file. if newname.txt** exists it will be overwritten 
    
    ****
    ```

- **Permissions**

    ```bash
    ***drwxr-xr-x*** - consists of 4 parts;
    1-part: ***d***     d-directory, -:file
    2-part: ***rwx*** - permissions of user
    3-part: ***r-x*** - permissions of group
    4-part: ***r-x*** - permissions of everbody else (other)
    
    ***r*** - read
    ***w*** - write
    ***x*** - execute for files, and permission to enter into for directories
    
    chmod +x myscripy.sh - add x permission for specified file to all users
    chmod u+x myscripy.sh - add x permission for specified file to only user
    chmod a+rwx myscripy.sh - add rwx permission for specified file to all users
    chmod g-rwx myscripy.sh - remove rwx permission for specified file to group users
    ```

- **Resource usage**

    ```bash
    free -m **---> shows ram usage in MB
    df -h ---> shows disk free 
    uptime ---> shows uptime (load average shows for last 1, 5 and 15 minutes respectively**
    ```

- **Package Manager**

    ```bash
    **sudo pacman -Syu ---> upgrade all packages on machine
    sudo pacman -S packageName ---> install package
    sudo pacman -R packageName ---> remove only installed package
    sudo pacman -Rs packageName ---> remove package with its dependencies
    sudo pacman -Ss packageName ---> search repository for this package
    sudo pacman -Q ---> query(see) all packages installed on your system
    sudo pacman -Qs  packageName ---> query specified package
    sudo pacman -Qdt ---> query all installed depencies that are not been used anymore**
    
    ```

- **Logs**

    ```jsx
    **journalctl ---> all logs
    journalctl -n 100 ---> last 100 logs
    journalctl -f ---> logs in real time
    journalctl -b ---> current session logs
    journalctl -b -1 ---> previous session logs
    journalctl -p err ---> only error logs
    journalctl -u nginx ---> only nginx's logs
    journalctl -fu docker ---> follow only dockers logs
    
    dmesg ---> displays log messages of Linux core
    dmesg -T --->  displays in formatted Time
    dmesg -T | grep -i error ---> displays log where specified word exists
    dmesg -w ---> displays in real time
    dmesg | tail ---> last 10
    dmesg | head ---> first 10**
    
    ```

- **User Management**

    ```jsx
    **passwd ---> change password for current user
    sudo passwd user1  ---> change password of user1
    adduser user1  ---> create user
    sudo userdel -r user1  ---> removes user1 (r1 - deletes home directory of user)**
    ```