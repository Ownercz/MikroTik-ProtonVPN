# MikroTik-ProtonVPN
Guide howto route traffic using ProtonVPN

## VPN using IKEv2

### Certificate download and import
``` 
/tool fetch url="https://protonvpn.com/download/ProtonVPN_ike_root.der"  
/certificate import file-name=ProtonVPN_ike_root.der
```

### IPsec config
``` 
/ip ipsec policy group add name="ProtonVPN"
/ip ipsec profile add dh-group=modp4096,modp2048 enc-algorithm=aes-256 hash-algorithm=sha256 name="ProtonVPN"
/ip ipsec proposal add name="ProtonVPN" auth-algorithms=sha256 enc-algorithms=aes-256-cbc pfs-group=modp2048 
/ip ipsec policy add dst-address=0.0.0.0/0 group="ProtonVPN" proposal="ProtonVPN" src-address=0.0.0.0/0 template=yes
/ip ipsec mode-config add connection-mark=nat-to-protonvpn-cz name=ProtonVPN-CZ responder=no use-responder-dns=no
/ip ipsec peer add address="PROTON_SERVER_ADDRESS_FROM_USER_PROFILE_OVPN_CONFIG" exchange-mode=ike2 name=ProtonVPN profile=ProtonVPN send-initial-contact=yes  
/ip ipsec identity add auth-method=eap certificate=ProtonVPN_ike_root.der_0 eap-methods=eap-mschapv2 generate-policy=port-override mode-config=ProtonVPN peer=ProtonVPN policy-template-group=ProtonVPN username=USER password=PASS 
```

### Routing via address-list, conn mark and mangle
``` 
/ip firewall address-list add address=192.168.20.100 list=protonvpn-only-cz
/ip firewall address-list
add address=192.168.0.0/16 list=no-vpn
add address=176.102.65.176 list=no-vpn
/ip firewall mangle
add action=mark-connection chain=forward dst-address-list=!no-vpn \
    new-connection-mark=nat-to-protonvpn-cz passthrough=yes src-address-list=\
    protonvpn-only-cz
add action=change-mss chain=forward connection-mark=nat-to-protonvpn-cz new-mss=\
    1360 passthrough=yes protocol=tcp tcp-flags=syn tcp-mss=!0-1360
``` 
