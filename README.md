# Qubes rsync

Sometimes there is a need for a central document store to which other qubes have access.
This is a simple tool that allows individual qubes read/write access to the store using rsync, rather than using `qvm-copy` or `qvm-move`.  
rsync runs as a daemon on the store, and client qubes access it using rsync over qrexec.

The rsync daemon is configured ,(as normal), by editing a config file at `/etc/rsyncd.conf`.  
The default setting is to have a **read/write** store at `/home/user/shared`.  
You can change this by editing the config file, and setting `read only = y`.  
All the usual rsync configuration options are available, and you can create other shared directories as you want.

On the client side, access is obtained with an rsync call like this:  
`rsync --port=837 localhost::shared`

Of course, access is controlled with the usual qubes-rpc policy file.  
**You must consider the security implications in sharing data between qubes.**


