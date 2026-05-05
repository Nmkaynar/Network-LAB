## DMVPN

Birden fazla şubeyi  merkez ofise(Hub) ve birbirine dinamik olarak bağlanmasını sağlamak amacıyla cisco tarafından geliştirilen bir VPN teknolojisidir.

Bir firmanın birden fazla şubesi var ise klasik vpn ile ayrı ayrı site to site vpn yapılması gerekmetedir. Bu ölçeklenebilir olmadığı gibi yönetimi de zorlaştırmaktadır. 

Bu sebeten ötürü  DMVPN bu sorunu ortadan kaldırır. 

DMVPN de üç temel bileşen bulunmaktadır.

## mGRE (Multipoint Gre): 
Tek bir tünel arayüzü üzerinden birden fazla noktaya bağlantı. Klasik GRE'de her hedef için ayrı tünel gerekirken, mGRE de tek arayüzden bir çok şubeyi konuşturur.

## NHRP( Next Hop Resolution Protocol) 
Bu protokol, bir şubenin gerçek ip adresi nedir sorusunun cevabını bulur. Hub bütün şubelerin public iplerini kaydeder. Bir şube başka bir şube ile iletişim kurmak istediğinde onun public ip adresini öğrenir ve Hub'a uğramadan iletişim kurabilirler.


### CONFİG

### Hub Config


````
interface Tunnel0
 ip address 10.0.0.1 255.255.255.0
 tunnel mode gre multipoint
 tunnel source Ethernet0/0
 ip mtu 1400
 ip tcp adjust-mss 1360
 no ip redirects
 
 ip nhrp authentication cisco
 ip nhrp network-id 1
 ip nhrp map multicast dynamic
 ip nhrp redirect


````
### Spoke Config


````
interface Tunnel0
 ip address 10.0.0.2 255.255.255.0
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 ip mtu 1400
 ip tcp adjust-mss 1360
 no ip redirects

 ip nhrp network-id 1
 ip nhrp authentication cisco
 ip nhrp map 10.0.0.1 100.0.11.1
 ip nhrp map multicast 100.0.11.1
 ip nhrp nhs 10.0.0.1
 ip nhrp shortcut    

`````
### Config Açıklamaları 

 `` ip mtu 1400`` Tünel arayüzün MTU'sunu 1500 den 1400 byte'a düşürür. Paketler internete çıkarken ek başlıklar eklendiğinden eğer 1500 byte'lık bir paket gelirse fragmentation yaşanır. Fragmentation problemi yaşanmaması için düşürülür.

``ip tcp adjust-mss 1360`` TCP paketlerinin MSS(Maximum Segment Size) 1360 yapar. TCP de 20 byte IP + 20 byte TCP başlığı bulunmaktadır. MTU 1400'e düşürüldüğü için MSS de 1360'a düşürülmelidir. (MSS = MTU- 40(20 byte IP + 20 byte TCP))

``ip nhrp authentication NHRPAUTH`` . Cihazların birbirlerine VPN yapabilmeleri için bir kimlik doğrulama sağlar. Bu veri gizliliği sağlamaz. Veri gizliliğini IPSec sağlamaktadır.

``ip nhrp network-id 1`` DMVPN cloudunu tanımlayan lokal bir id. NHRP paketlerinin hangi NHRP process'e ait olduğunu ayırt ekmek için kullanılan bir id.

``ip nhrp map multicast dynamic`` Hubda spokelar kayıt olduğunda multicast haritasını otomatik öğrenmesi için kullanılır. Bu satır eklenmez ise ospf gibi multicast kullanan protokoller çalışmaz.

``ip nhrp redirect`` Bu satır ile hub, bir spoketan başka bir spoke'a trafik geçtiğini gördüğünde "beni aracı  yapma" demek için nhrp redirect mesajı gönderir. Spoke tarafında `ip nhrp shortcut` dır.

``tunnel mode gre multipoint``mGREyi aktif eden satır. Point to point greden point to multipointe çeker.

`` ip nhrp map 10.0.0.1 100.0.11.1`` 10.0.0.1 tünel adresine gitmek için 100.0.11.1 public ip'sine git demek için kullanılır. 10.0.0.1 tünel ip ile 100.0.11.1 public ip'yi statik eşleştirme yapmış oluyoruz. Bu satır spokelarda yazılmaktadır. Diğer spokelarınki yazılmaz. Diğer spokeları NHRP dinamil olarak öğrenmektedir.

`` ip nhrp map multicast 100.0.11.1``  Spokelar tünelde multicast trafiği 100.0.11.1'e gönder demek için girilmiştir.  Bu satır olmadan ospf gibi dinamik protokoller çalışmaz.

``ip nhrp nhs 10.0.0.1``  Merkez ofisin(Hub) tünel ip adresidir. Burada nhs next hop server anlamına gelir. Yani spoke'un kayıt olacağı sunucu, diğer spokeları sorgulayacağı kaynak ve spoke'un routing komşuluğu kuracağı cihaz.

`` ip nhrp map 10.0.0.1 100.0.11.1`` , `` ip nhrp map multicast 100.0.11.1`` ve  ``ip nhrp nhs 10.0.0.1`` satırlarını tek bir satırda ``ip nhrp nhs 10.0.0.1 nbma 100.0.11.1 multicast`` bu şekilde yazılabilir.

`` ip nhrp shortcut `` Bu satır ile şubeler doğrudan kendi arasında tünel açarak konuşabilmektedir. Yani hub, iki şubeye birbiri ile iletişime girmek için kestirme yolu göstermektedir.Bu satır sadece Spoke'lara yazılmaktadır.



Yapılandırma tamamlandıktan sonra nhrp'yi kontorrol edelim.

<img width="518" height="219" alt="image" src="https://github.com/user-attachments/assets/421b6d56-de6b-4622-bcad-98f4523e3812" />

Hub tarafında nhrp spokeları kaydetti. Ancak Lan tarafı hala iletişim kuramaycaktır. Bunun için ya statik ile ya da dinamik ile rotaları öğret
<img width="675" height="274" alt="image" src="https://github.com/user-attachments/assets/070a93c6-e563-42df-861d-c6a3e2b8c100" />

## OSPF kullanımı

OSPF yapılandırırken tünel interface altında network type'ı broadcast yapmalıyız.

### Hub 
```
router ospf 1 
 router-id 1.1.1.1
 network 10.0.0.1 0.0.0.0 area 0
 network 192.168.10.1 0.0.0.0 area 0
 passive-interface gi 1/0

interface tunnel 0
 ip ospf network broadcast


```

### Spoke-A
```
router ospf 1 
 router-id 2.2.2.2
 network 10.0.0.2 0.0.0.0 area 0
 network 172.16.10.1 0.0.0.0 area 0
 passive-interface gi 1/0

interface tunnel 0
 ip ospf network broadcast
 ip ospf priority 0
```

### Spoke-B

```
router ospf 1 
 router-id 3.3.3.3
 network 10.0.0.3 0.0.0.0 area 0
 network 10.10.10.1 0.0.0.0 area 0
 passive-interface gi 1/0

interface tunnel 0
 ip ospf network broadcast
 ip ospf priority 0
```

## Ping Testi

PC2'den PC3'e ping ve trace attığımda, direkt tünel üzerinden Spoke-B'ye gitmektedir.
<img width="784" height="282" alt="image" src="https://github.com/user-attachments/assets/00b0cba4-8400-4268-ab28-b3a251afd798" />


Her iki spoke ta state up gözükmektedir.

<img width="836" height="528" alt="image" src="https://github.com/user-attachments/assets/5680924b-6b57-441a-a1d0-cacf101470d3" />


## IPSEC

<img width="698" height="166" alt="image" src="https://github.com/user-attachments/assets/cef257b8-7367-4d7f-af86-49b8b68af48f" />

DMVPN, bir tünel kurarak şubeleri birbirine bağlamakta ancak paketler açık olarak gittiği için güvenlik zafiyeti oluşturmaktadır. Bunun için IPSec yapılandırarak trafiği güvenli hale getirilmelidir.

### Config 
````
crypto ikev2 keyring DMVPN-KEYRING
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key cisco

crypto ikev2 profile DMVPN-IKEV2-PROFILE
 match identity remote address 0.0.0.0
 authentication local pre-share
 authentication remote pre-share
 keyring local DMVPN-KEYRING

crypto ipsec transform-set DMVPN-TS esp-aes 256 esp-sha-hmac
 mode transport

crypto ipsec profile DMVPN-IPSEC-PROFILE
 set transform-set DMVPN-TS
 set ikev2-profile DMVPN-IKEV2-PROFILE

interface Tunnel0
 tunnel protection ipsec profile DMVPN-IPSEC-PROFILE
````

Yapılandırma tamamlandıktan paketler şifrelenerek gönderilmektedir.
<img width="828" height="147" alt="image" src="https://github.com/user-attachments/assets/9c7993f1-7562-4478-82f0-43ca48d8870a" />


