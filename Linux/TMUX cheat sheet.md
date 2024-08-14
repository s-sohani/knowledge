TMUX is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen.  tmux may be detached from a screen and continue running in the background, then later reattached.
### Starting and Exiting tmux

- **Start tmux:** `tmux`
- **Start a new session with a name:** `tmux new -s session_name`
- **Detach from the session:** `Ctrl-b d`
- **List all sessions:** `tmux ls`
- **Attach to a session:** `tmux attach -t session_name`
- **Kill a session:** `tmux kill-session -t session_name`

## Prefix Key

- **Default Prefix:** `Ctrl-b`
- **Change Prefix (inside .tmux.conf):** `unbind C-b` then `set -g prefix C-a`

## Panes

### Creating Panes

- **Horizontal Split:** `Ctrl-b %`
- **Vertical Split:** `Ctrl-b "`

### Navigating Panes

- **Move to the next pane:** `Ctrl-b o`
- **Move to the previous pane:** `Ctrl-b ;`
- **Move to a specific pane (1-9):** `Ctrl-b q` followed by pane number
- **Swap panes:** `Ctrl-b {` (up) or `Ctrl-b }` (down)

### Resizing Panes

- **Resize pane:** `Ctrl-b` followed by arrow keys
- **Precise resize (horizontal):** `Ctrl-b :resize-pane -L 10` or `Ctrl-b :resize-pane -R 10`
- **Precise resize (vertical):** `Ctrl-b :resize-pane -U 10` or `Ctrl-b :resize-pane -D 10`
- **Maximize a pan**: `Ctrl-b + z`

### Closing Panes

- **Close current pane:** `Ctrl-b x` then confirm with `y`

## Windows

### Creating and Managing Windows

- **New window:** `Ctrl-b c`
- **Rename window:** `Ctrl-b ,` then type new name
- **Select window by number:** `Ctrl-b` then number (0-9)
- **Next window:** `Ctrl-b n`
- **Previous window:** `Ctrl-b p`
- **Swap windows:** `Ctrl-b :swap-window -t 2` (swap current window with window 2)
- **Show window tree with pans** `Ctrl-b w`

### Closing Windows

- **Kill current window:** `Ctrl-b &` then confirm with `y`

## Sessions

### Creating and Managing Sessions

- **New session:** `tmux new -s session_name`
- **List sessions:** `tmux ls`
- **Attach to a session:** `tmux attach -t session_name`
- **Detach from a session:** `Ctrl-b d`
- **Rename session:** `Ctrl-b $` then type new name

### Moving Windows Between Sessions

- **Move window to another session:** `Ctrl-b :move-window -t destination_session_name:`

## Write parallel in pans

```
set-window-option synchronize-panes [on | off]
```

## Connect to Multi Server 

```bash
#!/bin/bash  
starttmux() {  
	if [ -z "$HOSTS" ]; then  
		echo -n "Please provide of list of hosts separated by spaces [ENTER]: "  
		read HOSTS  
	fi  
	local hosts=( $HOSTS )  
	tmux new-window "ssh ${hosts[0]}"  
	unset hosts[0];  
	for i in "${hosts[@]}"; do  
		tmux split-window "ssh $i"  
		tmux select-layout tiled > /dev/null  
	done  
	tmux select-pane -t 0  
	tmux set-window-option synchronize-panes on > /dev/null  
}  
HOSTS="server1 server2 server3"  
starttmux
```

## Run code with tmux in parallel

``` bash
# !/bin/bash  
# ssh-multi  

starttmux() {  
	tmux new-window ""  
	for i in `seq 0 3`; do  
		tmux split-window -h "echo 'Hello word'"  
		tmux select-layout tiled > /dev/null  
	done  
	tmux select-pane -t 0  
	tmux set-window-option synchronize-panes on > /dev/null   
}  
  
starttmux
```

