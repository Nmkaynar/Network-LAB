## Community



BGP Community, route’ları etiketlemek (taglemek) için kullanılan bir attribute’tur.
Bu etiketler sayesinde router’lar belirli politikaları uygulayabilir.

AA:NN şeklinde prefixlere etiketlenir ve gönderilen komşunun bu etikete göre aksiyon almasını sağlanır. 

Komşu AS'da bu community'ler önceden belirlenmiştir. Ve bizde komşu AS'ı belirlediği community'lere göre etiketleme yaparız.

cisco da eski IOS'larda AA:NN algılaması için ``ip bgp-community new-format`` komutu aktif edilmeli.

Community direkt path selection tetikleyicisi değildir ancak attributeları tetikleyerek path selection yapar.
Burada standart community'ler incelenecek.

``Additive`` ile daha önce eklenenen communityleri silmeden yenisini ekler.

### Well-Know Communities  

|no-export | AS dışına duyurulmaz|
|:-------- |:------------|
|no-advertise | Kimseye anons etme|
|local-AS  | Sub-AS dışına çıkmaz.|
|blackhole|Trafik drop edilir.|


## ip bgp-community new-format
Eski IOS'larda defaultta AA:NN şeklinde gözükmüyor.
<img width="660" height="237" alt="image" src="https://github.com/user-attachments/assets/9f74991b-cab6-4e77-9b81-070c192f10fe" /><br>
Community: 32768100 şeklinde görüldüğü için ``ip bgp-community new-format`` komutunu aktif etmeliyiz.<br>

Komutu aktif ettikten sonra istediğimiz formata geldi.
<img width="639" height="234" alt="image" src="https://github.com/user-attachments/assets/0564298c-2109-4e91-a7d6-89a9391e3814" />


Bu labda ISP'nin vermiş olduğu community'ler 

|500:100| weight 100 yapar|
|:------|:-----------|
|500:200| Local pref'i 200 yapar|
|500:201| Local pref'i 50 yapar|
|500:300| As Path'e 2 kere ekler|
|500:400| Med'i 5 yapar|
|500:401| Med'i 10 yapar|



## ISP de Community Listeleri Hazırlama 

### ISPRouter1 
````
ip community-list standard Pref_200 permit 500:200
ip community-list standard Pref_50 permit 500:201
ip community-list standard As_Path_3 permit 500:300
ip community-list standard Med_5 permit 500:400
ip community-list standard Med_10 permit 500:401



route-map customer_in permit 10
 match community Pref_200
 set local-preference 200

route-map customer_in permit 20
 match community Pref_50
 set local-preference 50

route-map customer_in permit 30
 match community As_Path_3
 set as-path prepend  500 500 500

route-map customer_in permit 40
 match community Med_5
 set metric 5

route-map customer_in permit 50
 match community Med_10
 set metric 10

route-map customer_in permit 60
exit

router bgp 500
  neighbor 10.0.11.2 route-map customer_in in
  neighbor 10.0.12.2 route-map customer_in in
  neighbor 10.0.10.2 send-community

````

### ISPRouter2
````
ip community-list standard Pref_200 permit 500:200
ip community-list standard Pref_50 permit 500:201
ip community-list standard As_Path_3 permit 500:301
ip community-list standard Med_5 permit 500:400
ip community-list standard Med_10 permit 500:401



route-map customer_in permit 10
 match community Pref_200
 set local-preference 200

route-map customer_in permit 20
 match community Pref_50
 set local-preference 50

route-map customer_in permit 30
 match community As_Path_3
 set as-path prepend  500 500 500

route-map customer_in permit 40
 match community Med_5
 set metric 5

route-map customer_in permit 50
 match community Med_10
 set metric 10

route-map customer_in permit 60
exit

router bgp 500
  neighbor 10.0.13.2 route-map customer_in in
  neighbor 10.0.14.2 route-map customer_in in
  neighbor 10.0.10.1 send-community

````
