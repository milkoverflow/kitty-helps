# NixOS/Nix

- [Serving a site](#serving-a-site)
- [Creating a systemd service](#creating-a-systemd-service)

## Serving a site

```nix
environment.systemPackages = with pkgs; [
  static-web-server
];

services.static-web-server = {
  enable = true;
  root = "/some/path";
};
```

## Creating a systemd service

```nix
systemd.services.my-service = {
    script = ''
        echo "Hello World" > /var/log/hi.log
    '';
    wantedBy = [ "multi-user.target" ];
  };
```
