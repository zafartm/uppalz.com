# How to To mount a remote folder over SSH
1. Install sshfs: `sudo apt install sshfs`
2. Create a local folder: `mkdir remote-data`
3. Do the mount: `sshfs -o IdentityFile=<path-to-user-identitiy-file> <remote-user-name>@<remote-server-dns-ip>:<path-on-remote-server> ./remote-data`
