## IPsec nedir, ne işe yarar?
İki ofisi internet üzerinden kendi LAN'larını tek bir LAN'daymış gibi connect etmek için kullanılan bir protokoldür.
<img width="1072" height="600" alt="image" src="https://github.com/user-attachments/assets/a1656353-0fe1-4524-9e6f-2d7c3d063b56" />

Bu lab çalışmasında Site-A'da bulunan  192.168.10.0/24 networkü ile Site-B de bulunan 172.16.200.0/24 networkünün birbilerine iletişimini sağlayacağız.
IPSec iki aşamadan kurulur. Phase 1 ve Phase2.

## Phase 1 

Burada yapılan işlem her iki sitedaki çıkış cihazların(bu labda routerların) birbirleri arasında güvenli bir kontrol kanalı (IKE SA) oluşturur.

### Config
````  
crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400
 exit
crypto isakmp key CISCO address 100.65.2.2
````  
Burada DH ile (group 14) ortak bir secret üretildikten sonra ike mesajlar aes(varsayılan 128) ile şifrelenir ve bütünlüğünü sha256 ile kontrol edilir. Bütünlüğü uymayan ike mesajları drop edilir.
İlk mesajlar şifreli değildir.


## Phase 2 

Burada IPSec SA oluşturulur ve kullancı trafiği  Aes ile şifrlenerek ESP ile taşındığı yer. Verinin bütünlüğünü ise sha ile doğrular 

### Config 
````
crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel
````

Burada TS adında tranform-set oluşturuyoruz ve esp protokolü kullanarak veriyi aes(varsayılan 128) ile şifrele ve sha ile bütünlüğünü doğrula demiş oluyoruz.

PC1'den PC3'e ping attığımızda ICMP yerine ESP olara görülecektir.ESP içnde ICMP Payloadu bulunur

<img width="962" height="303" alt="image" src="https://github.com/user-attachments/assets/ddfc7388-cec1-4058-a684-2d7c76c6fd61" />


## Interesting Traffic

Burada extend ACL ile hangi networklerin iletişim kuracağını belirliyoruz

### config 
ip access-list extended VPN-TRAFFIC
 permit ip 192.168.10.0 0.0.0.255 172.16.200.0 0.0.0.255

### Dikkat edilmesi gereken
 Eğer NAT var ise VPN trafiğinde kullanılacağımız networkü NAT'tan hariç tutmalıyız.

ip access-list extended NAT-ACL
 deny   ip 192.168.10.0 0.0.0.255 172.16.200.0 0.0.0.255
 permit ip 192.168.10.0 0.0.0.255 any

ip nat inside source list NAT-ACL interface GigabitEthernet0/0 overload


## Crypto Map

Burada artık yaptığımız aşamaları birleştiriyoruz

### Config
````
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 100.65.2.2
 set transform-set TS
 set pfs group14
 match address VPN-TRAFFIC
interface g0/0
 crypto map VPN-MAP
````

Crypto map, karşı tarafın (peer) IP’sini, verinin nasıl korunacağını (transform-set) ve hangi trafiğin VPN’e gireceğini (ACL) belirleyerek bu politikayı interface’e uygular.
 
Her iki tarafta bu işlem yapıldıktan sonra 192.168.10.0/24 ile 172.16.200.0/24 networkleri birbirlerine aynı LAN'daymış gibi erişim salayabilirler. Ve aralarındaki iletişim şifreli olarak ilerler.

`` set pfs group14`` Bu 



Bu bağlantı kurulurken main Phase1 (Main modda)6 paket Phase 2(Quick Modda) 3 paket ile toplam  9 pakette yapılır
 
<img width="986" height="149" alt="image" src="https://github.com/user-attachments/assets/f593595f-3f58-4b6e-bac7-0f1f9dbaf23f" />

















