## WEIGHT
Cisco router üzerinde lokal olarak kullanılan bir “route preference” değeridir.
- Best path seçiminde ilk önceliğidir.
- Sadece o router’ı etkiler.
- BGP update’lerde diğer router’lara gönderilmez 
- Yani tamamen local decision tool’dur
- Ciscoya özel bir attribute'dur. O yüzden farklı vendor cihazlarda bu değer yoktur. 
- Defaultta kendi öğrettiği rota ise değeri 32768, başkasından öğrendiyse 0 dır.
- Yüksek olan tercih edilir

Bu değeri değiştirerek gelen BGP'nin seçtiği best path'e manipülasyon yapabiliriz.

Weight değerini değiştirmek için iki farklı yöntem vardır.

Komşudan gelen tüm rotalara aynı değer uygulanacak ise ``neighbor  x.x.x.x weight <Değer>``
ya da spesifik ayar yapmak isteniyorsa route-map uygulanmalıdır.


## Neighbor x.x.x.x weight <Değer>

Aş.da da görüldüğü gibi R5 arkasında bulunan iki networktüde 3 farklı rotadan öğrendi ve best rota(> olan)  10.0.12.2 next hop ip adresi olan R4 Rouer'ın seçmiş.<br>
<img width="754" height="194" alt="image" src="https://github.com/user-attachments/assets/8739c28d-7b29-462f-b578-d59f354f7b70" /><br>
Trace attığımızda R4 rotasını tercih ettiğini görüyoruz. 
<img width="636" height="148" alt="image" src="https://github.com/user-attachments/assets/da6c3a9e-e8b2-4053-99dd-e448d40d7902" /><br>
Şimdi gelelim R2 den öğrendiği rotaların weight'ini  değiştirerek best path'i değiştrielim.

Bunun için R1 Router'nda ``router bgp 65100`` altında ``neighbor 10.0.10.2 weight 100 `` komutunu kullanalım. Belirleyeceğimiz değer best rotanın weight değerinden büyük olması yeterli. 
<img width="470" height="41" alt="image" src="https://github.com/user-attachments/assets/4ba06faa-197c-4d18-9da5-02f87604d6e4" />
Bu işlemden sonra rotaların ya yeniden öğrenmesini bekleyceğiz yani doğal sürecinde update paketi beklinelecek 
ya da  ``clear ip bgp 10.0.10.2 soft in`` komutunu çalıştırarak Route Refresh paketi gönderip update paketi alacağız.
Bu komut ile BGP sesion düşürmeden, rotaları komşusundan yeniden ister. 

Not: ``clear ip bgp 10.0.10.2`` şeklinde çalışıtırılırsa, BGP sesion düşer, TCP balantısı kapanır. Ve tekrar baştan kurulur. Buda BGP komşuluğu oturasıya kadar networkte kesintiye yol açar.

İşlem sonucu best path'in R2(10.0.10.2) olduğunu gördük.
<img width="742" height="193" alt="image" src="https://github.com/user-attachments/assets/a7816071-22f8-4e6f-94ba-4a146e74f205" />

Trace attığımızda da rotanın değiştini görüyoruz.
<img width="595" height="140" alt="image" src="https://github.com/user-attachments/assets/3eb46f91-9adf-4733-854e-b94e832c70dd" />

## Route Map 


Show ip bgp tablosuna baktığımızda ``neighbor 10.0.10.2 weight 100`` ile R2 den öğrenilen tüm rotaların weight değerini değiştini görüyoruz. 
<img width="742" height="193" alt="image" src="https://github.com/user-attachments/assets/ecbf80a9-9552-4916-a5b1-cd595dcabbe7" />

Peki sadece bazı  prefixlerin weightini değiştirecek isek yani sadece 172.16.10.0/24 networkü için R3'ü tercih etsin istiyorsak. Bu seferde Route map kullanmamız gerekir.

````
ip prefix-list PL-WEIGHT permit 172.16.10.0/24

route-map RM-WEIGHT permit 10
 match ip address prefix-list PL-WEIGHT
 set weight 500

route-map RM-WEIGHT permit 20
 exit

router bgp 65100
 neighbor 10.0.11.2 route-map RM-WEIGHT in
````

Uygulamadan önceki hali

<img width="704" height="192" alt="image" src="https://github.com/user-attachments/assets/8d3b28a0-5a38-41ad-a7cc-f68d4885770f" />


Route Map'i aktif ettikten sonra sadece 172.16.10.0/24 networkünün weight değerinin değiştini görüyoruz

<img width="736" height="187" alt="image" src="https://github.com/user-attachments/assets/b5b1ba94-5040-4d75-9cb7-1ef081a12f60" />

R1 172.16.20.0/24 networkü için R4-->R5 rotasını kullanılırken
R1 172.16.10.0/24 networkü için R3-->R5 rotasını  kullanmaya başladı.


### NOT

Weight değerini değiştirmek, sadece işlem yapılan routerda BGP rotalarının en iyi rota seçiminde kullanılır ve cisco cihazlarının ilk baktıkları attribute dur. Ancak burada değiştirilen değer update paketlerinde taşınmazlar.
Yani outbound yönünde kullanılmaz.
<img width="665" height="44" alt="image" src="https://github.com/user-attachments/assets/eb82ff01-44cd-4633-af40-1efae1c2e952" />


Wireshark'ta update paketinde taşınan Path attribute'lar  <br>
<img width="787" height="233" alt="image" src="https://github.com/user-attachments/assets/e0f53a8a-95e8-4c25-86d5-5824929a2acd" />















