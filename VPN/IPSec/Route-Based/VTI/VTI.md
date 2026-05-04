## VTI
<img width="835" height="620" alt="image" src="https://github.com/user-attachments/assets/b684585a-474d-48d2-a601-89541a7ee282" />


VTI, virtual tunnel interface.
IPSec trafiği bir tünel interface üzerinden iletilmeini sağlar. Policy-based ikev2 göre farkı 
crypto map, vpn-acl, ve nat muafiyeti bulunmamaktadır. **IKEv2 Bileşenleri ve IPsec Transform Set**  tamamen aynı confige sahiptir.

## IPSec Profile 

Crypto Map'in yerini almaktadır. 

### Config 

````
crypto ipsec profile IPSEC-PROF
 set transform-set TS
 set pfs group14
 set ikev2-profile IKEV2-PROF
````

Crypto map de set peer ve match addres satırları vardı. IPSec profile da gerek yok çünkü tünnel interface'e zaten routing yapılacak. Bu tünele giren trafik direkt IPSec ile korunmuş olacak. Bu sebepten ötürü ACL ile de vpn trafiğini belirlemeye gerek kalmıyor. 
Aynı şekilde IPSec ile korunacak trafik WAN yerine tünnele yönlendiğinden NAT muafiyetinede ihtiyaç kalmıyor.


## Tunnel Interface 

### Config 

````
interface Tunnel0
 ip address 10.0.0.1 255.255.255.0
 tunnel source GigabitEthernet0/0
 tunnel destination 2.2.2.2
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC-PROF
````

`tunnel mode ipsec ipv4` ile bu tünel saf IPsec olacak diyoruz GRE değil. ``tunnel protection ipsec profile IPSEC-PROF`` ile IPSec profille ile korunacak.


## Routing

``ip route 172.16.10.0 255.255.255.0 10.0.0.2`` ile  VPN trafiğini tünnelden geçmesini sağlamış oluyoruz.


## Kontrol

show crypto ikev2 sa

Çıktıda status ready olmalı.
<img width="867" height="174" alt="image" src="https://github.com/user-attachments/assets/266c6946-7bbe-4873-b509-393f88c06f8e" />


show crypto ikev2 sa


<img width="905" height="959" alt="image" src="https://github.com/user-attachments/assets/f5401a34-aee3-4ca9-be46-0196348156f6" />

