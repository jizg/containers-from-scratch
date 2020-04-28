# containers-from-scratch
Learning Go and containers by re implementing https://github.com/lizrice/containers-from-scratch step by step

## Step1. Setup run function
insert run() function, print out all arguments after the third one `os.Args[2:]`

```bash
$go run main.go run Hello, world 
Running [Hello, world]
```

## Step2. Modify run function to execute the command in arguments
modify run() function to enable executing the command in arguments

```bash
$go run main.go run echo Hello, world 
Running [echo Hello, world]
Hello, world
```

## Step3. Modify run function to enable UTS Namespace
By enabling UTS Namespace, the hostname can be changed without affecting host process hostname.

```bash
root@ubuntu18:$go run main.go run /bin/bash
Running [/bin/bash]
root@ubuntu18:$hostname
ubuntu18
root@ubuntu18:$hostname container
root@ubuntu18:$hostname
container
```
while in parent shell hostname remains unchanged:
```bash
root@ubuntu18:$hostname
ubuntu18
```

## Step4. Add child function in order to change hostname before entering bash
In order to display an updated hostname in bash, add a `child` function and let `run` to execute this `child` in the new UTS namespace with updated hostname.
```bash
root@ubuntu18:$go run main.go run /bin/bash
Running [/bin/bash]
Running in child [/bin/bash]
root@container:$hostname
container
``` 
And if you run `ps` in this containerized bash, you still can find all parent processes of this bash process, and the PIDs are also still the number of PIDs in host OS, which means still not `containerized` completely.
```bash
root@container:$ps
$ ps
  PID TTY          TIME CMD
27070 pts/10   00:00:00 sudo
27071 pts/10   00:00:00 bash
27823 pts/10   00:00:00 go
27844 pts/10   00:00:00 main
27848 pts/10   00:00:00 exe
27852 pts/10   00:00:00 bash
28292 pts/10   00:00:00 ps
root@container:$ps fax
...
27070 pts/10   S      0:00      |   |   \_ sudo /bin/bash
27071 pts/10   S      0:00      |   |       \_ /bin/bash
27823 pts/10   SLl    0:00      |   |           \_ go run main.go run /bin/bash
27844 pts/10   SLl    0:00      |   |               \_ /tmp/go-build645762770/b0
27848 pts/10   SLl    0:00      |   |                   \_ /proc/self/exe child 
27852 pts/10   S      0:00      |   |                       \_ /bin/bash
28779 pts/10   R+     0:00      |   |                           \_ ps fax
```

## Step5. Modify run function to enable PID Namespace
In `run` function, let the PID in the containerized bash start from 1 by enabling PID Namespace.
```bash
root@ubuntu18:$go run main.go run /bin/bash
Running [/bin/bash] as 30582
Running in child [/bin/bash] as 1
```
But if you run `ps` again, you still get all parent processes of this bash process, and the PIDs are still the number of PIDs in host OS. This is beause these processes information are from `/proc` folder which is not isolated by UTS and PID namespace, and therefore still shared between host and containerized bash.
```bash
root@container:$ps
  PID TTY          TIME CMD
30191 pts/10   00:00:00 sudo
30192 pts/10   00:00:00 bash
30561 pts/10   00:00:00 go
30582 pts/10   00:00:00 main
30586 pts/10   00:00:00 exe
30590 pts/10   00:00:00 bash
30818 pts/10   00:00:00 ps
```

## Step6. Set containerized process root directory to a new path
In order to isolate containerized process from sharing with host os `/proc`, use `chroot` and `chdir` syscall to set a new root dir for containerized process.
Firstly, use the following command to prepare a clean ubuntu filesystem.
```bash
$ CID=$(docker create ubuntu)
$ ROOTFS=~/ubuntufs
$ docker export $CID | tar -xf - -C $ROOTFS
```
In `child` function, call `chroot` and `chdir` syscall to set root direcotry to the ubuntufs folder. You can check the new root for containerized process by the following commands.

In containerized bash. You can also find that `ps` will get error, as there is still nothing under `/proc`.
```bash
root@container:$sleep 100
root@container:$ps
Error, do this: mount -t proc proc /proc
```
In host OS bash.
```bash
root@ubuntu18:$ps -C sleep
PID TTY          TIME CMD
 5697 pts/10   00:00:00 sleep
root@ubuntu18:$ls -l /proc/5697/root
lrwxrwxrwx 1 root root 0 Apr 28 23:36 /proc/5697/root -> /home/jizg/ubuntufs
```
So we actually use the extracted `ubuntu:latest` image on docker hub as our new root directory.