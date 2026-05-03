## IPSec IKEv2 
IKEv2’de Phase 1 ve Phase 2 kavramı mantıksal olarak devam eder ama yapı daha modülerdir.
- Proposal → Şifreleme ve hash algoritmaları
- Policy → Proposal’ın kullanımı
- Keyring → Pre-shared key
- Profile → Peer ve authentication tanımı


## IKEv2 PROPOSAL
Proposal ikev2 de phase 1 için kullanılacak kriptografik algoritmaların listesidir. Ikev1 de crypto isakmp policy altında yapılırdı.

### Config

````
crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha256
 group 14
````
## IKEv2 Policy


### Config
`Policy hangi propolsalların router üzerinde hangi wan adresete geçerli olduğunu belirler.  
````
crypto ikev2 policy IKEV2-POLICY 
 proposal IKEV2-PROP
````

Router üzerinde eğer birden fazla wan ip adresi mevcut ise hangisinde geçerli olduğunu ``match address local <IP-Address>  şeklide belirtmeliyiz. 


## Keyring (Pre-Shared Key Saklama Alanı)
IKEv1 de ``crypto isakmp key ... address … `` yapılan işlemi aş.daki gibi yaparız. Bir keyring için birden fazla peer tanımlanabilir.

### Config 
````
crypto ikev2 keyring KR
 peer SITE-B
  address 100.65.2.2
  pre-shared-key CISCO
````


Burada ikev1 den farklı olarak daha düzenli bir yapı kurgulanmış olur. Ikev1 de bir den fazla şube için PSK tanımlanmış olsaydı şeklinde gözükürdü ve hangisi kime ait belli olmazdı. 
````
crypto isakmp key 7xK9$mPq2vL8nR4tY6wZ address 100.65.2.2
crypto isakmp key 3jH7&bN5cV9xQ1sD8fG2 address 100.65.3.2
crypto isakmp key 9pL4!kM6tR2yU5iO7eW3 address 100.65.4.2
````

## IKEv2 Profile 

Profile, kim hangi yöntemle doğrulanacak ve hangi keyring kullanılacak sorularına cevap verir.

### Config
````
crypto ikev2 profile IKEV2-PROF
 match identity remote address 100.65.2.2 255.255.255.255
 authentication local pre-share
 authentication remote pre-share
 keyring local KR
````

 ## Crypto Map

Phase 2 ikev1 ile bire bir aynı confige sahiptir. cyrpto map de ise bir satır daha eklenmiştir
``set ikev2-profile IKEV2-PROF`` Bu satır eklenmez ise IOS varsayılan olarak IKEv1 dener. 

````
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 100.65.2.2
 set transform-set TS
 set pfs group14
 set ikev2-profile IKEV2-PROF      
 match address VPN-TRAFFIC
````



Her şey tamamlandıktan sonra PC1 den PC2 ye ( 192.168.10.10 ->172.16.200.10) önce IKev2 Phase1 + Phase 2 kurulumu yapılır. IKEv1 9 pakette yaparken IKEv2 4 pakette tamamlar.

<img width="1072" height="216" alt="image" src="https://github.com/user-attachments/assets/b0937130-2ece-432c-ae85-e417869a6b1a" />

## Kontrol
show crypto ikev2 sa çıktısı ile IPSec Kurulumunu görebiliriz.
<img width="924" height="195" alt="image" src="https://github.com/user-attachments/assets/97cc2c00-e338-419a-a23e-a4130c064f85" />
show crypto ipsec sa çıktısı
<img width="826" height="962" alt="image" src="https://github.com/user-attachments/assets/3740b81e-d10e-4f6b-9f89-60fb3a7188eb" />

