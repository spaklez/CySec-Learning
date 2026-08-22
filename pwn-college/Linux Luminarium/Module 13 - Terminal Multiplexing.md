To create virtual terminals inside your terminal, you make use of the screen command. To leave a screen session, you type exit or press Ctrl+D, the screen terminates and returns you to your original shell. 

The whole purpose of screens is the fact that they provide to you the option to attach and detatch. You detatch by pressing Ctrl+A followed by d (for detatch). This leaves your session running in the background while you return to your normal terminal. To **r**eattach, you can use the `-r` argument to `screen`. 

You can list your 'screens' with the command `screen -ls`:

```console
hacker@dojo:~$ screen -ls
There are screens on:
        23847.mysession   (Detached)
        23851.goodwork    (Detached)
        23855.morework    (Detached)
3 Sockets in /run/screen/S-hacker.
```

```console
hacker@dojo:~$ screen -ls
There are screens on:
        23847.mysession   (Detached)
        23851.goodwork    (Detached)
        23855.morework    (Detached)
3 Sockets in /run/screen/S-hacker.
```

Inside a single screen session, you can have multiple windows, these windows are handled with different keyboard shortcuts, all starting with `Ctrl-A`:

- `Ctrl-A c` - Create a new window
- `Ctrl-A n` - Next window
- `Ctrl-A p` - Previous window
- `Ctrl-A 0` through `Ctrl-A 9` - Jump directly to window 0-9
- `Ctrl-A "` - bring up a selection menu of all of the windows

Screen is one command, the more modern version of it is tmux (terminal multiplexer). It does all the same things as screen with the only difference being that for the shortcuts, instead of Ctrl A, you use Ctrl B.  To attach you use tmux attach or tmux a and to list all of your screen you do tmux ls. The key combos for tmux are 

- `Ctrl-B c` - Create a new window
- `Ctrl-B n` - Next window
- `Ctrl-B p` - Previous window
- `Ctrl-B 0` through `Ctrl-B 9` - Jump to window 0-9
- `Ctrl-B w` - See a nice window picker

