# Install

```bash
sudo apt-get install openssh-server
```

# Connect

```bash
ssh -p PORT USER@SERVER_IP

# Execute one command and exit.
ssh -p PORT USER@SERVER_IP ls
```

SSH in Virtual Machine needs a port forwarding rule in network settings for the VM. Name `ssh`, host port `3022`, guest port `22`.

```bash
# SSH to local VM via port forwarding.
ssh -p 3022 user@127.0.0.1
```

# Alias

```bash
/etc/ssh/ssh_config   # system-wide
~/.ssh/config         # per user (better)

# Add this in the config.
Host server_name
  Port 22
  User user
  HostName 123.456.789.255

# Connect with this now.
ssh server_name
```

# SSH with RSA key

This is used to log in without writing the password each time. Also, it is much more secure.

This works by using a private `id_rsa` and public `id_rsa.pub` key pair. These keys can be generated by running `ssh-keygen`, and they are located in `~/.ssh` i.e. `/home/user/.ssh`.

The `id_rsa` private key stays in the host machine in `~/.ssh`. The `id_rsa.pub` public key goes on the server in a `~/.ssh/authorized_keys` file. Simple as that, the next ssh is password-less.

To transfer a public key, use this from `~`:

```bash
rsync -av -e 'ssh -p 3022' ./.ssh/id_rsa.pub user@127.0.0.1:~/.ssh/
```

# rsync - Transfer files over SSH 

Only transfer the files that have changes or are missing. Much faster and secure than FTP.

```bash
-a   # Recursion and preserve everything.  
-v   # Transfer log.  
-h   # Display the output numbers in a human-readable format.
-e   # Specify remote shell.
-W   # Copy whole file, without checking for changes.

--progress   # Show the sync progress during transfer
```

```bash
# Transfer everything from the current directory to a new folder in the remote home directory.
rsync -av . user@123.456.789.255:folder/

# Transfer folder1 and its content to remote home directory
rsync -av ./folder1 user@123.456.789.255:

# Transfer everything from folder1 without the folder itself to remote folder2 in remote home directory.
rsync -av ./folder1/ user@123.456.789.255:folder2/

# Transfer multiple files and folders.
rsync -av ./file1 ./file2 ./folder1 user@123.456.789.255:folder2/

# Another way
rsync -av -e 'ssh' ./folder1/ user@123.456.789.255:~/folder1/
```

When the terminal freezes, fix it with this in a **new** terminal...

```bash
while sudo killall -CHLD ssh; do sleep 0.1; done;
```

### To Virtual Machine

```bash
rsync -av -e "ssh -p PORT_NUMBER" <SOURCE> <DESTINATION>:<PATH>

rsync -av -e 'ssh -p 3022' . user@127.0.0.1:~/make-this_folder
```

### Backup

To copy multiple files, the command must be a string. This will copy the content from the locations to the single location the command is called from.

```bash
rsync -av -e 'ssh' 'root@123.456.789.255:/etc/nginx/nginx.conf /etc/letsencrypt/keys' .
```

This command will copy the `nginx.conf` file and the letsencrypt folder with the keys. The command must run as root for the keys.