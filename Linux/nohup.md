Here's a quick reference for using `nohup` on Ubuntu to run processes in the background, so they continue running even after logging out:

### Basic `nohup` Commands

1. **Run a Command in the Background**:
    
    bash
    
    Copy code
    
    `nohup command &`
    
    - `command` is the program or script you want to run.
    - The output is saved to `nohup.out` by default in the current directory.
2. **Specify Output File**:
    
    bash
    
    Copy code
    
    `nohup command > output.log 2>&1 &`
    
    - `> output.log`: Redirects standard output to `output.log`.
    - `2>&1`: Redirects standard error to the same file.
3. **Check Running Processes**:
    
    bash
    
    Copy code
    
    `ps aux | grep command`
    
    - Replace `command` with your process name to filter results.
4. **Kill a Background Process**:
    
    bash
    
    Copy code
    
    `kill -9 <PID>`
    
    - Replace `<PID>` with the process ID from the `ps` command.
5. **Run a Script Continuously**:
    
    bash
    
    Copy code
    
    `while true; do nohup command; sleep 1; done &`
    
    - This will rerun the command every second. Adjust the sleep time as needed.
6. **View `nohup` Logs**:
    
    bash
    
    Copy code
    
    `tail -f nohup.out`
    
    - Use `tail -f` to view the output in real-time.
7. **Exit Without Stopping `nohup` Processes**:
    
    - Press `Ctrl+D` or type `exit` to log out without stopping `nohup` jobs.

### Notes

- **Foreground**: Use `fg` to bring a background job to the foreground.
- **Jobs**: List background jobs with `jobs -l`.

To send a foreground process back to the background in Ubuntu, follow these steps:

1. **Suspend the Foreground Process**:
    
    - Press `Ctrl+Z` to suspend (pause) the running process. This will stop it temporarily and allow you to manage it.
    - You’ll see a message like `[1]+ Stopped <command>`.
2. **Send the Suspended Process to the Background**:
    
    - Use the `bg` command to continue the suspended process in the background:
        
        bash
        
        Copy code
        
        `bg`
        
    - Alternatively, if there are multiple jobs, specify the job number:
        
        bash
        
        Copy code
        
        `bg %1`
        
    - Replace `1` with the job number as shown by the `jobs` command if needed.
3. **Check Background Jobs**:
    
    - Run `jobs` to see a list of jobs and confirm the process is now running in the background.
    - 