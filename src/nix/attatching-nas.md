# Attatching NAS

## Non nix way
```bash
sudo mount -t cifs -o credentials=/home/username/dit-credentials //192.168.0.100/share
```

## Configuration
```nix
fileSystems."/mnt/mount_name" = {
  device = "//192.168.0.100/share";
  fsType = "cifs";
  options = ["credentials=/home/username/dit-credentials"];
};

```

## Credentials
```
username=user
password=pass
```
