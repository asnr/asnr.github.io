Bash History
============

Keeping your bash history clean
-------------------------------

Enter `a` twice at the Bash terminal:

```sh
$ a
-bash: a: command not found
$ a
-bash: a: command not found
```

Now press the up arrow to cycle back throw your previous commands. After pressing up once you will see `a`, and after pressing up a second time you will see `a` again. What a waste of a keystroke! We can stop Bash storing duplicate commands in its history by including `ignoredups` in the `HISTCONTROL` environment variable:

```sh
export HISTCONTROL=$HISTCONTROL:ignoredups
```


HISTIGNORE!

The Bash history file! (search `man bash` for `HISTORY`)
