# Configuring a VPN client

## 1. Keygen

```sh
  umask 077
  wg genkey > wireguard-keys/private
  wg pubkey < wireguard-keys/private > wireguard-keys/public
```

## 2. Configuration

### using NetworkManager

- create file `/etc/NetworkManager/system-connections/wg0.nmconnection`

```sh
sudoedit /etc/NetworkManager/system-connections/wg0.nmconnection
sudo chmod 600 /etc/NetworkManager/system-connections/wg0.nmconnection
```

```ini
[connection]
id=wg0
type=wireguard
interface-name=wg0

[wireguard]
listen-port=51820
private-key=HOST-PRIVATE-KEY

[wireguard-peer.SERVER-PUBLIC-KEY]
endpoint=SERVER-PUBLIC/LAN-IP
allowed-ips=VPN-SUBNET
persistent-keepalive=25      # dont wait for host to establish connection

[ipv4]
address1=HOST-VPN-IP
method=manual
```

- setup the connection
```sh
nmcli con reload
nmcli con up wg0
```
