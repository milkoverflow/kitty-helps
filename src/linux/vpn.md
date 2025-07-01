# Configuring wireguard

## 1. Keygen

```sh
  umask 077
  wg genkey > wireguard-keys/private
  wg pubkey < wireguard-keys/private > wireguard-keys/public
```

## 2. Client configuration

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
dns=SERVER-IP;
dns-search=SERVER-DOMAIN
ignore-auto-dns=true
never-devault=true
```

- setup the connection
```sh
sudo nmcli con reload
sudo nmcli con up wg0
```

## 3. Server configuration

like client configuration, wg0.nmconnection file:

```ini
[connection]
id=wg0
type=wireguard
interface-name=wg0

[wireguard]
listen-port=51820
private-key=SERVER-PRIVATE-KEY
private-key-flags=0

[wireguard-peer.PEER-PUBLIC-KEY]
allowed-ips=PEER-IP/32

[ipv4]
address1=SERVER-VPN-IP
method=manual
```

## 4. What to do if server not forwarding traffic

make it ip forwarding!!!

```sh
sudo sysctl -p 'net.ipv4.ip_forward = 1'
sudo iptables -A FORWARD -i wg0 -o wg0 -j ACCEPT
```

maybe works....
