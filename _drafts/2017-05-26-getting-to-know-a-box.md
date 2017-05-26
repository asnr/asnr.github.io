While I've been working on unix boxes for almost a decade now, I've never had to do 

Recently I had to debug operations problems for a large app for the first time. After ssh'ing into the box that was misbehaving, I wasn't sure what were good commands to run to understand what was going wrong. My team lead gave me some good suggestions, and I'm writing them down here.

Running `ps aux` to get a list of running processes when you first arrive is a good way to orient yourself and also double check that you haven't ssh'd into the wrong box. For example, if you meant to ssh into your app server, but see a database process in the output of `ps aux`, then you're probably on the wrong box. (Side note about changing command prompts and getting ssh server to print a message on opening a connection!!!!!!)

If a process is dead when you expect it to be running, it might have run out of memory or disk space. To check how much disk space is left, you can run `df -ah`. `df` is short for 'Display Free disk space', the `-h` flag makes the its output "human readable" and the `-a` flag makes it print statistics for *all* mount points. (What's a mount point? I don't know, but the more the merrier, right?) Its output looks like this:

```
```

To check memory usage, you can run `free -m` (not installed by default on Macs) to print the memory usage statistics in megabytes that will look something like this:

```
```

To see which processes are using up most of the memory, or to watch how processes behave as you restart your app you can use `top`. On linux you can toggle colour output by pressing '???', something with 'z' and order processes by memory usage using 'M' (OR IS IT 'm'?).
