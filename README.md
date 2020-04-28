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
while in parent shell:
```bash
root@ubuntu18:$hostname
ubuntu18
```