### **1. Using `lsof` Command**

The `lsof` command (List Open Files) is commonly used for this purpose. For example:


`sudo lsof -i :<port_number>`

**Example:**

`sudo lsof -i :80`

This will display a line showing the process using port 80, including the PID.

---

### **2. Using `netstat` Command**

The `netstat` command (with the `-p` option) can show the PID of the process using a port.

`sudo netstat -tulnp | grep :<port_number>`

**Example:**

`sudo netstat -tulnp | grep :80`

Output includes columns for protocol, local address, PID/program name, etc. Look under the `PID/Program` column.

---

### **3. Using `ss` Command (Recommended for Modern Systems)**

The `ss` command is a faster replacement for `netstat` and provides similar information.

`sudo ss -tulnp | grep :<port_number>`

**Example:**

`sudo ss -tulnp | grep :80`

Output will show the PID and name of the process.

---

### **4. Using `/proc` Filesystem**

You can inspect the `/proc` filesystem manually if you know the protocol and port.

**Steps:**

1. Identify the socket inode for the port:
    
    `sudo netstat -anp | grep :<port_number>`
    
    Note the "inode" number for the socket.
    
2. Find the process using this inode:
    
    `sudo find /proc -name fd -exec sh -c 'ls -l {} | grep <socket_inode>' \;`