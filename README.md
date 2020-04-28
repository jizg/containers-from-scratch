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