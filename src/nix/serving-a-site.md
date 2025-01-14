# Serving a site

```nix
environment.systemPackages = with pkgs; [
  static-web-server
];

services.static-web-server = {
  enable = true;
  root = "/some/path";
};
```
