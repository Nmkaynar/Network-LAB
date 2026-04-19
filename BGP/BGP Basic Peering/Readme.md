## BGP Nedir?
BGP (Border Gateway Protocol)i internet üzerindenki Autonomous systemler (AS) arasındaki rota bilgilerini taşıyan tek EGP (Exterior Gateway PRotocol) protokolüdür. 

BGP protokolü hem external(eBGP) hem de internal (IBGP) olarak kullanılabilir. 
- External(eBGP) olarak kullanıldığında AD değeri 20 dir. 
- İnternal (iBPG) olarak kullanıldığında AD değeri 200 dür.
- TCP 179 portu üzerinde çalışır.  
- 60 sn de bir keeplive mesajı gönderir.
- Hold süresi 180 sn dir.
- Update paketi ile değişiklik var ise gönderilir. 
- Konuşma süreci unicast gerçekleşir.
- AS_PATH ile loop Prevention yapar. Başkasından öğrendiği rotalarda kendi AS'ini görürse, paketi drop eder.
  
	
## eBGP Peering Nasıl Kurulur?
BGP bir neighbor ile peer olabilmek için şunlara ihiyaç duyar:
- TCP 179 bağlantısı, iki router arasında önce TCP 3-way handshake olur. BGP bunun üstünde çalışır
- Neighbor tanımı, ``router bgp <AS-NO>`` altında `` neighbor < karşı tarafın IP addresi> remote-as <Karşı tarafın AS-NO>``  komutu ile yapılmaktadır.


## Config

<img width="944" height="552" alt="image" src="https://github.com/user-attachments/assets/5789d4dc-0221-4268-b0bd-27dda973c3cd" />

İki router arasında eBGP yapılandıracağız.

Kullanılacak olan komutlar :

R1 Router'da 
````
router bgp 65000
neighbor 10.0.11.2 remote-as 65100
network 10.0.11.0 mask 255.255.255.252
network 192.168.10.0 mask 255.255.255.0
````
R2 Router'da
```
router bgp 65100
neighbor 10.0.11.1 remote-as 65000
network 10.0.11.0 mask 255.255.255.252
network 172.16.10.0 mask 255.255.255.0
```

### Yapılandırma Sonrası  

Yapılandırma tamamlandıktan sonra her iki konsolda BGP komşuluğu up mesajları gelmiştir.

<img width="565" height="61" alt="image" src="https://github.com/user-attachments/assets/6904a573-b0c8-40e0-a9c2-c6db771da3df" />
<img width="568" height="68" alt="image" src="https://github.com/user-attachments/assets/66933646-32a8-4465-98d4-65e8b50d5430" /><br>
BGP komşuluğu kurulduktan sonra her iki router da duyurdukları networkleri birbirine öğretti. <br><br>
<img width="1494" height="221" alt="image" src="https://github.com/user-attachments/assets/a4b264b3-95bf-444e-9ea2-0d30217719e4" />

show bgp summary çıktısı
<img width="831" height="291" alt="image" src="https://github.com/user-attachments/assets/805d9582-5cdc-49ea-a132-31a2409d09e3" />

show bgp neighbors çıktısı
<img width="858" height="330" alt="image" src="https://github.com/user-attachments/assets/212c738b-b867-409e-be4e-e1e8ee589e88" />



## BGP State'leri

### IDLE 
BGP henüz başlamadı.

### Connect
TCP bağlantısının yapıldığı aşama

### Ative 
TCP başarısız, tekrar deniyor

### OpenSent
TCP kuruldu, OPEN mesajı gönderildi.

### OpenConfirm
OPEN mesajı alındaı KEEPALIVE bekliniyor

### Established
Komşuluk kuruldu Peer aktif.

## Komşuluk Kurulum Sürecinde Gönderilen Paketler
<img width="1096" height="195" alt="image" src="https://github.com/user-attachments/assets/7dc3d4d9-95ee-4c70-8306-71f0fd78b8bc" />

### TCP Handshake 
- İki router TCP 179 üzerinden 3 way-handshake bağlantısı kurarlar.
	
### OPEN Message
- AS-NO, 65000
- Version 4
- Hold time 180
- Router-id <br>
Bilgilerini içeren ilk tanışma mesajıdır.
Not: Router-id girilmez ise, önce en büyük loopback adresi yoksa en büyük interface ip adresi router-id olur
<img width="723" height="228" alt="image" src="https://github.com/user-attachments/assets/6ec22667-7fc9-40c4-83bd-7b344e44983b" />

### KEEPALIVE
- OPEN mesajını kabul ettiğini ve komşuluğu başlattığı mesajdır.
- Bu mesaj artık komşuluğun başladığını bildirir.
- Komşuluk kurulduktan sonra 60 sn de bir gönderilir. 3 kere yani 180 sn boyunca keepalive mesajı alamazsa komşuluğu bitirir.
  <img width="734" height="185" alt="image" src="https://github.com/user-attachments/assets/d258fa08-e9ab-44b2-9c1c-b1b07c6d2311" />

### UPDATE MESSAGE
- Router'ın duyurduğu ve öğrendiği rotaları bu paketle komşusuna duyurur.
<img width="1687" height="590" alt="image" src="https://github.com/user-attachments/assets/82bbd3fb-bb6a-4ff8-ae1b-932d85cac05e" />
