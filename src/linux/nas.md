# mounting nas

## manual -- mount and umount scripts

```sh
#!/bin/sh

NAS="192.168.0.100"
SHARE="private"
MOUNTPOINT="/mnt/home-server"
CREDFILE="$HOME/server-credentials"

sudo mkdir -p "$MOUNTPOINT"
sudo mount -t cifs -o credentials="$CREDFILE" //"$NAS"/"$SHARE" "$MOUNTPOINT"
```

```sh
#!/bin/sh

MOUNTPOINT="/mnt/home-server"

sudo umount "$MOUNTPOINT"
sudo rmdir "$MOUNTPOINT
```

