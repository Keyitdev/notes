> Author: [Keyitdev](https://github.com/keyitdev)\
> Source: [Keyitdev/notes](https://github.com/keyitdev/notes)\
> Last edited on: 16 Feb 2025

# Wpa_supplicant configuration with EAP on Arch Linux

### Install required packages
```sh
pacman -S dhcpcd wpa_supplicant
```
### Create wpa_supplicant config
```sh
sudo vim /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```

```sh
ctrl_interface=/run/wpa_supplicant
update_config=1

network={
    ssid="<wifi_name>"
    psk="<wifi_password>"
    proto=RSN
    key_mgmt=WPA-PSK
    pairwise=CCMP
    auth_alg=OPEN
    mesh_fwding=1
}

network={
	ssid="<wifi_name>"
	key_mgmt=WPA-EAP
	eap=PEAP
	identity="<identity>"
	anonymous_identity="<anonymous_identity>"
	password="<wifi_password>"
	phase2="auth=MSCHAPV2"
	mesh_fwding=1
}
```

### Enable services
```sh
sudo systemctl enable dhcpcd.service
sudo systemctl enable wpa_supplicant@wlan0.service
```

### Unblock wifi
```sh
sudo rfkill list
sudo rfkill unblock all
```
