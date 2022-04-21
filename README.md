# Qubes rsync

Sometimes there is a need for a central document store to which other qubes have access.  
This is a simple tool that allows individual qubes read/write access to the store using rsync, rather than using `qvm-copy` or `qvm-move`.  
rsync runs as a daemon on the store, and client qubes access it using rsync over qrexec.

The rsync daemon is configured,(as normal), by editing a config file at `/etc/rsyncd.conf`.  
The default setting is to have a **read/write** store at `/home/user/shared`, and a **read only** directory at `/home/user/archive`.  
You must create these directories on the rsync server.  
All the usual rsync configuration options are available, and you can create other shared directories as you want.  
**N.B. Because access appears to come from localhost, host control directives will not work**

Of course, access is controlled with the usual qubes-rpc policy file.  
**You must consider the security implications in sharing data between qubes.**

## How to use it

I really should package this properly.

In the meantime:  
Install rsync in the template.  
Copy `rsyncd.conf` to `/etc`  
Copy `qubes.Rsync` to `/etc/qubes-rpc`  
`chmod +x /etc/qubes-rpc/qubes.Rsync`  
Copy `qubes-rsync-forwarder@.service` to `/lib/systemd/system`  
Copy `qubes-rsync-forwarder.socket` to `/lib/systemd/system`

### On the server:  
Create `/home/user/shared` and `/home/user/archive`, and populate with data.  
chmod as appropriate:  
`chmod 777 /home/user/shared.`  

Start the rsync service:
`systemctl start rsync`


### On the clients:  
Enable the `rsync-setup` service in the client. (This doesn't exist as yet, but should check that rsync is installed and forwarder is enabled).  
Start the forwarder:  
`systemctl enable qubes-rsync-forwarder.socket`  
`systemctl start qubes-rsync-forwarder.socket`  
`

Then on the clients use rsync as normal:  
`rsync --port=837 localhost::shared`  

### Making it start automatically.  
I prefer to start the server manually.  
If you want the rsync service to start automatically, put an entry in `/rw/config/rc.local`:
`systemctl start rsync`

Similarly, if you want the clients set up automatically, add the start commands to `/rw/config/rc.local` in a client qube.

### Configuring policy
In Qubes 4.1, add these lines to /etc/qubes/policy.d/30-user.policy. (Create that file if it does not exist.)  
`qubes.Rsync     *   CLIENT    @default    allow target=SERVER`  
Change CLIENT to the name of your client qubes, and SERVER to the name of your rsync server.  
*The clients do not know the name of the server qube*

### Customising
You can create more shared folders, and make them accessible by customising the /etc/rsyncd.conf file.  
If you have more than one rsync server qube, you can use [bind-dirs](https://www.qubes-os.org/doc/bind-dirs/) to change the available folders on each server qube.

# Qubes sshfs

The greatest problem with the rsync solution is that it makes *copies* of the files or directories.
This may be fine, but with large files, or large numbers of files, there's a significant overhead.
What would be better would be to allow clients to access files on the "server" qube directly.

The simplest method is to use sshfs to make the remote files available to clients over qrexec.
Of course, you could do this using NFS, or any other network based file sharing protocol.

Again, this is not packaged.

As root:  
Install a ssh-server and sshfs in the template. (` apt install openssh-server sshfs`).  
(Follow the usual advice when installing services in Debian templates.)  
`cp qubes.ssh /etc/qubes-rpc`  
`cp qubes-ssh-forwarder@.service lib/systemd/system`  
`cp qubes-ssh-forwarder.socket /lib/systemd/system`

## Create ssh-keys for passwordless login.

In the client run `ssh-keygen` with defaults: you can choose whether to set a password or not.
`qvm-copy .ssh/id_rsa.pub` to the server.

On the server, `mkdir .ssh && cat QubesIncoming/<client>/id_rsa.pub >> .ssh/authorized_keys`

### On the server: 
There's a little set-up to do, as root:
`mkdir /home/tx`
`chmod 777 /home/tx`
`systemctl start sshd`

### On the clients:  
Enable the `ssh-setup` service in the client. (This doesn't exist as yet, but should check that ssh is installed and forwarder is enabled).  
`systemctl enable qubes-ssh-forwarder.socket`  
`systemctl start qubes-ssh-forwarder.socket`  

As *user*:  
`mkdir /home/user/tx`
`sshfs -p 840 localhost:/home/tx tx`

Now `/home/tx` on the server is mounted at `/home/user/tx` on the client.

Access can be controlled using the usual qubes-rpc mechanism, by editing a policy file.

It would be possible to constrain access to files on the server, using (e.g) ssh chroots, and access control mechanisms.
