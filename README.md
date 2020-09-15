# Qubes rsync

Sometimes there is a need for a central document store to which other qubes have access.
This is a simple tool that allows individual qubes read/write access to the store using rsync, rather than using `qvm-copy` or `qvm-move`.  
rsync runs as a daemon on the store, and client qubes access it using rsync over qrexec.

The rsync daemon is configured,(as normal), by editing a config file at `/etc/rsyncd.conf`.  
The default setting is to have a **read/write** store at `/home/user/shared`, and a **read only** directory at `/home/user/archive`.  
You must create these directories on the rsync server.
All the usual rsync configuration options are available, and you can create other shared directories as you want.  
**N.B. Because access apears to come from localhost, host control directives will not work**

Of course, access is controlled with the usual qubes-rpc policy file.  
**You must consider the security implications in sharing data between qubes.**

## How to use it

I really should package this properly.

In the meantime:
Install rsync in the template.
Copy `rsyncd.conf` to `/etc`  
Copy `qubes.Rsync` to /etc/qubes-rpc`  
Copy `qubes-rsync-forwarder@.service` to `/lib/systemd/system`
Copy `qubes-rsync-forwarder.socket` to `/lib/systemd/system`

###
On the server:
Create `/home/user/shared` and `/home/user/archive`, and populate with data.  
chmod as appropriate.  
`systemctl start rsync`

###
On the clients:
Enable the `rsync-setup` service in the client. (This doesnt exst as yet, but should check that rsync is installed and forwarder is enabled).  
`systemctl enable qubes-rsync-forwarder.socket`  
`systemctl start qubes-rsync-forwarder.socket`  
`

Then on the clients use rsync as normal:  
`rsync --port=837 localhost::shared`
