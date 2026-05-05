## Gre over IPSec

Amaç iki uzak şube arasında trafiğin aynı LAN'daymış gibi güvenli bir şekilde taşınmasını sağlamak.

<img width="706" height="627" alt="image" src="https://github.com/user-attachments/assets/f4af87c7-3cf7-4ae1-bea6-f48893a03a2a" />

## Gre 
Cisco tarafından geliştirilen bir tünneling protokolüdür. IP protokol 47 dir.
### Özellikler
- Point to point sanal tünel oluşturur
- Multicast ve broadcast trafiği taşıyabilir.Bu sayede OSPF vb dynamic routing protokoller çalıştırılabilir.
- Encryption yapmaz. Paketler açık bir şekilde gider. Güvenlik için IPSec ile birlikte kullanılır.
- IP paketlerine GRE header ve yeni bir dış IP header eklenir.

## Config
````
interface Tunnel0
 ip address 10.0.0.1 255.255.255.0
 tunnel source 1.1.1.1
 tunnel destination 2.2.2.2
 tunnel mode gre ip

router ospf 1 
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
 passive-interface gi1/0
````

### Kontrol
Config tamamlandıktan sonra ping atalım.
PC1'den PC2'ye ping attığımızda PC1'in üretmiş olduğu ip paketi 
<img width="756" height="114" alt="image" src="https://github.com/user-attachments/assets/db327fbd-10e8-4131-a1f6-2bcd839df831" />
Paket Router'a geldiğinde Gre tüneline yönlendirilir. Bu noktada orjinal IP paketinin önüne bir GRE header, onun da önüne tünel source/destination IP'lerini taşıyan yeni bir dış IP header eklenir.
<img width="784" height="116" alt="image" src="https://github.com/user-attachments/assets/e526bfcc-7024-40f5-a17a-063fccea583f" />
Ancak Gre paketi şifrelemediği için güvenlik sağlanmamaktadır. Bunun için IPSec uygulanarak paketleri şifreler ve trafiğin güvenli olmasını sağlar.
IPSec ikev1 ile uygulama yapmış olacağız. 

## IPSec Config

```
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400

crypto isakmp key cisco address 2.2.2.2

 
crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode transport 

crypto ipsec profile IPSEC-PROF
 set transform-set TS

interface Tunnel0
tunnel protection ipsec profile IPSEC-PROF
```

Config uygulandıktan sonra tünel trafiği artık şifreli olmuş olacak.
<img width="864" height="90" alt="image" src="https://github.com/user-attachments/assets/c3d171f7-8b17-48e9-bb0e-4a7c482c3445" />

```
crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode transport
````
 Burada tunnel mode yerine transport kullanılmasının sebebi GRE encapsulation yapıldıktan sonra IPsec ile şifrelenir. Eğer tunnel mode kullanılsaydı GRE tekrar encapsulation olurdu ve overhead artardı.
