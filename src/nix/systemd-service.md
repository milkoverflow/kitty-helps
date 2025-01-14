# Creating a systemd service

```nix
systemd.services.my-service = {
  script = ''
      echo "Hello World" > /var/log/hi.log
  '';
  wantedBy = [ "multi-user.target" ];
};
```
