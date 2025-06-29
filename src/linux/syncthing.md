# Syncthing

## As a systemwide service

the systemwide syncthing service is call syncthing@USER, where USER is the user to run syncthing

```sh
systemctl enable --now syncthing@USER
```

## set GUI address

in `.local/state/syncthing/config.xml` inside of syncthing's user home directory

```xml
<gui enabled="true" tls="false" debugging="false" sendBasicAuthPrompt="false">
  <address>0.0.0.0:8384</address>
  ...
</gui>
```
