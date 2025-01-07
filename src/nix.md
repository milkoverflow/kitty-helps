# NixOS/Nix

- [Serving a site](#serving-a-site)
- [Creating a systemd service](#creating-a-systemd-service)
- [Configuring the VPN](#configuring-a-vpn)
  - [Generate keys](#1-generate-keys)
  - [Configure the server](#2-configure-the-server)
  - [Configure the client](#3-configure-the-client)

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

## Configuring a VPN

### 1. Generate keys

```sh
  wg genkey > wireguard-keys/private
  wg pubkey < wireguard-keys/private > wireguard-keys/public
```

### 2. Configure the server

NOTE: Replace aaa.bbb with the desired IP range.

```nix
networking = let
  externalInterface = "eno1";
  internalInterface = "wg0";
  internalIP = "10.aaa.bbb.1/24";
in {
  nat.enable = true;
  nat.externalInterface = externalInterface;
  nat.internalInterfaces = [ internalInterface ];
  wireguard.interfaces = {
    "${internalInterface}" = {
      ips = [ internalIP ];
      listenPort = 51820;
      postSetup = ''
        ${pkgs.iptables}/bin/iptables -t nat -A POSTROUTING -s ${internalIP} -o ${externalInterface} -j MASQUERADE
      '';
      postShutdown = ''
        ${pkgs.iptables}/bin/iptables -t nat -D POSTROUTING -s ${internalIP} -o ${externalInterface} -j MASQUERADE
      '';
      privateKeyFile = "wireguard-keys/private";

      peers = [
        { # Peer 1
          publicKey = "Peer's public key";
          allowedIPs = [ "10.aaa.bbb.2/32" ];
        }
      ];
    };
  };
};
```

### 3. Configure the client

NOTE: Replace aaa.bbb with the desired IP range.
NOTE: Generate the keys on the client as well.

```nix
networking.wireguard.interfaces = {
  wg0 = {
    ips = [ "10.aaa.bbb.2/24" ];
    listenPort = 51820;
    privateKeyFile = "wireguard-keys/private";
    peers = [
      {
        publicKey = "Server's public key";

        allowedIPs = [ "10.aaa.bbb.0/24" ]; # The IP range to parse through the VPN
        endpoint = "192.168.0.100:51820"; # The server's IP
        persistentKeepalive = 25;
      }
    ];
  };
};
```
