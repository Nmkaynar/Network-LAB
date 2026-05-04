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


## Phase 2 

Burada IPSec SA oluşturulur ve kullancı trafiği  Aes ile şifrlenerek ESP ile kapsüllenerek taşındığı yer. Verinin bütünlüğünü ise sha ile doğrular 

### Config 
````
crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
 mode tunnel
````

Burada TS adında tranform-set oluşturuyoruz ve esp protokolü kullanarak veriyi aes 256 ile şifrele ve sha ile bütünlüğünü doğrula demiş oluyoruz.

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

`` set pfs group14``
Bu satır PFS'i (Perfect Forward Secrecy) aktif eder. PFS sayesinde her Phase 2 rekey'de 
yeni ve bağımsız bir Diffie-Hellman değişimi yapılır. Bu sayede uzun ömürlü bir anahtar 
ileride çalınsa bile, geçmişte üretilmiş Phase 2 anahtarları hesaplanamaz — geçmiş 
trafik güvende kalır.

Phase 2'nin varsayılan lifetime süresi 3600 saniyedir, yani PFS açıkken saatte bir 
yeni DH üretilir.


Bu bağlantı kurulurken  Phase1 (Main modda)6 paket Phase 2(Quick Modda) 3 paket ile toplam  9 pakette kurulum tamamlanır.

<img width="986" height="149" alt="image" src="https://github.com/user-attachments/assets/f593595f-3f58-4b6e-bac7-0f1f9dbaf23f" />



## Show çıktıları
``sh crypto isakmp sa`` komutu ile phase 1'in çalışıp çalışmadığı kontrol edilir.

<img width="606" height="151" alt="image" src="https://github.com/user-attachments/assets/f2f4b3f7-1775-4c22-9bf3-5518140033ad" />


``sh crypto ipsec sa  `` komutu ile phase 2 de ncrypt/decrypt sayaçlarının artıp artmadığı görülür.


<img width="752" height="637" alt="image" src="https://github.com/user-attachments/assets/300b3bd3-59c8-4f87-ac20-9231a5881591" />

PC1'den PC2'ye ping attığımda bu değerlerin arttığını ve tünel henüz kurulmamış ise ilk trafikle birlikte inboud esp sas ve outbound esp sas'un da kurulduğunu ve status kısmında active görürüz.

<img width="751" height="950" alt="image" src="https://github.com/user-attachments/assets/e8bd7d4c-0e0d-4af2-8da2-655d3eb9e4c0" />

NOT: Bu lab IKEv1 ile yapılmıştır. Modern kurulumlarda IKEv2 tercih edilir.










