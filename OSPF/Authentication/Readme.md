## OSPFv2 Authentication

Bu lab çalışmasında, opsfv2 auhtnetication(kimlik doğrulama) mekanizmalarını inceleyeceğiz.

<img width="764" height="367" alt="image" src="https://github.com/user-attachments/assets/f0d4bb0b-3d9b-471a-b97e-2529bef0c264" />

Standart bir şekilde ospf kurulduğunda kimlik doğrulama olmadan gerekli komutlar girildiğinde direkt ospf komşuluğu kurulur. 
Bu da ağa fiziksel olarak erişim sağlayan bir saldırganın rahatlıkla rogue router bağlayıp OPSF komşuluğu kurarak 
sahte route enjekte etmesine trafiği kendi üzerine çekmesine olanak sağlar.<br>

Varsayılan OSPF yapılandırılmasında hello paketlerinde Auth Type null olarak gider.

<img width="540" height="362" alt="image" src="https://github.com/user-attachments/assets/2de143ae-eed7-4709-b930-6731d3a921b0" />

Basit şifreleme yöntemi olan plain-text(simple password) authentication, bir kimlik doğrulama sağlar ancak parola hello paketi içinde şifrelenmeden(clear-text) gönderilir. 
Eğer bir saldırgan paketleri yakalayabilirse parolayı tespt eder ve OSPF komşuluğu kurabilir.

Plain-text Yöntemi ile yapılan şifreleme. Hello paketi içerisinde parola (Cisco123)  olduğu görülmektedir.
<img width="750" height="319" alt="image" src="https://github.com/user-attachments/assets/554feb2d-eccb-40ce-a2a8-f05d0903602e" />

Topoloji de R1 ve R2 plain-text metodu ile yapılan authentication opsf komşuluğunda R3 standart bir şekilde ospf yapılandırması yapmaktadır. R1 ve R2 komşuluk kurabilirken,
R3 OSPF komşuluğu kuramamaktadır. 

R3 authentication olmadan yapılandırıldığında hello paketi gönderip almasına rağmen authentication olmadığı için komşuluk kuramaz.
<img width="836" height="60" alt="image" src="https://github.com/user-attachments/assets/62f9e8be-d5eb-4262-bd0a-ccf7fbc9713b" />


R3'e de aynı parola girildiğinde ospf komşuluğu kurulmaktadır.
<img width="925" height="175" alt="image" src="https://github.com/user-attachments/assets/54095ab5-60c0-480e-aed0-9b10ece56fff" />

### Config
interface altında aş.daki komutları aktif edilmesi gerekmektedir. 

````
 ip ospf authentication
 ip ospf authentication-key Cisco123
````

Yukarıda da bahsettiğim gibi bu yöntem ile parola şifrelenmeden gönderidiğinden basit bir güvenlik sağlamaktadır. Bunun yerine MD5 ile parola şifrelenerek gönderildiğinde paketler dinlenilse dahi parola tespit edilemiyecektir.


## MD5 ile Şifreleme

Daha güvenli bir yöntem olan MD5 authetication parolayı doğrudan göndermez. Bunun yerine hash kullanarak doğrulama yapar.

### Config 
İnterface altında aş.daki komutlar girilmelidir.

````
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Cisco123
````

MD5 ile yapılandırıldığında hello paketinde kimlik doğrulama kısmı görseldeki gibi gözükmektedir.

<img width="743" height="347" alt="image" src="https://github.com/user-attachments/assets/a46eb17f-1ee7-4915-a783-9708cd4b30aa" />

Auth Type: Cryptographic (2) — parola gönderilmez
Key ID: 1 — her iki tarafta eşleşmeli
Auth Crypt Data: hash değeri görünür, parolaya ulaşılamaz

### NOT
Authentication bir kimlik doğrulama olduğu için, giden paketlerde datayı şifrelemez. Yami LS update paketlerinde gönderilen networkler tespit edilebilir. 

Ancak burada yapılan koruma, sahte router'ın ospf komşuluğu kurmasını ve ağa sahte rotalar enjekte etmesini engellemektir.

Eğer hem kimlik doğrulama hem de veri gizliliği isteniyorsa OSPF trafiği IPsec ile korunmalıdır.
