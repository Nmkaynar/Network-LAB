## Stub ve Totally Stub Area 


Bu lab çalışması, stub area ve totally stub area kavramlarını ele almakta ve aralarındaki fark gösterilmektedir. Burada  R1'e GigabitEthernet2/0'dan connect olan 172.16.1.0/24  networkünü ospf altında "redistribute connected subnets" şeklinde external   olarak enjekte edilmiştir.

<p><img width="927" height="602" alt="image" src="https://github.com/user-attachments/assets/05533a9d-1b7e-4acd-a739-8d75e270ac47" /></p>

## Stub area  
 Stub area olarak yapılandırılan bir ospf areasında ABR router, ASBR routerdan alınan LSA 5 tipindeki paketleri engelleyerek  yerine LSA 3 üreterek default route gönderir. Bu da bir den fazla olan external rotaları tek bir satıra  indirerek cihaz yükünü azaltmayı amaçlamaktadır.

### Ne zaman kullanılır.
  Tüm external rotaya giden tek bir çıkış noktası var ise kullanılabilir.


## Yapılandırma öncesi Routing Table 
Stub area yapılandırmasına geçmeden önce R3 ve R5 routerından show ip route çıktısına bakalım. 172.16.1.0/24 networkü E2 olarak görülmektedir.

Farklı arealardan gelen rotalarda IA olarak görülmektedir. 

### R3:

<img width="742" height="299" alt="image" src="https://github.com/user-attachments/assets/e6b824da-aa11-4151-a532-8f3ec8fbae21" /> <br>

### R5:

<img width="694" height="290" alt="image" src="https://github.com/user-attachments/assets/f61a2f4f-b09f-4b52-94d6-9bf84061824f" />  <br>
### Area 1'i Stub area olarak yapılandıralım.

Bunun için hem R2 de hem R3 de ospf altında "area 1 stub" komutunu girmeliyiz.

Bir routerı stub olarak config ettikten sonra komşusunu da stub olarak ayarlanmalıdır. Komşuluk kurulmaz.

<img width="943" height="106" alt="image" src="https://github.com/user-attachments/assets/c5a8249d-511f-4881-8312-c96774947ec5" /> <br>



R3 de yapılandırılınca komşuluk tekrar full state oldu.

<img width="947" height="157" alt="image" src="https://github.com/user-attachments/assets/647f0cda-a82e-4a90-8129-e4d22afb603d" /> <br>


R3 te tekrardan show ip route çıktısına baktığımızda ABR router olan R2,  E2 olan external rota içeren LSA 5 engelledi ve area 1'e default route enjekte etti. 

<img width="739" height="275" alt="image" src="https://github.com/user-attachments/assets/2bfe1393-9706-4400-b414-ec39e1375a05" /> <br>




## Totally Stub area

 Totally stub areada  ise ABR router stub areada yaptığına ek olarak internal areadan gelen rotaları da engelleyerek  sadece default route gönderir. 

### Ne zaman kullanılmalı
 Hem  diğer arealardan gelen rotalar hem de externaldan gelen rotalar için tek bir çıkış noktası var ise kullanılmalıdır.

 

### R4 ve R5 de totally stub area yaparak aradaki farkı görelim.

Bunun içinde R4 ve R5 te ospf altında "area 2  stub no-summary" komutunu girmeliyiz.

<img width="939" height="98" alt="image" src="https://github.com/user-attachments/assets/d36100af-aae1-4327-9ab9-054f2ca8e58d" /> <br>


Aynı şekilde komşusunda da girmeliyiz.

<img width="940" height="104" alt="image" src="https://github.com/user-attachments/assets/3377258b-32a5-4f9f-8c83-ed0d0296592c" /> <br>



İşlem sonucunda, R5'te show ip route çıktısına baktığımızda ABR router olan R4 hem internal areadan gelen rotaları hem de externaldan gelen rotaları engelleyerek yerine default route göndermiştir.

<img width="731" height="200" alt="image" src="https://github.com/user-attachments/assets/468a2b3f-4d8d-4343-9ce7-12d5a9a40441" /> <br>




## Show ip ospf database  

### R3 
Stub area yapılandırmadan önce R3'te LSA 4 ve LSA 5 bilgileri mevcuttur.

<img width="761" height="620" alt="image" src="https://github.com/user-attachments/assets/0aae577e-206a-4ba6-bfa0-31ebaf64a688" /> <br>

### Stub area sonrası R3 
ABR router olan R2, gelen Type-5 ve Type-4 LSA'ları engelleyerek yeni bir Type-3 LSA üreterek default route enjekte etmiştir.


<img width="784" height="466" alt="image" src="https://github.com/user-attachments/assets/1fddb073-5b6a-41a6-8eb7-f6f8c325d588" /> <br>


### R5

Totally stub yapılandırmadan önce R5'te LSA 4 ve LSA 5 bilgileri mevcuttur. Ayrıca burada LSA 3 bilgilerinide dikkat edelim.

<img width="785" height="621" alt="image" src="https://github.com/user-attachments/assets/f6f8aabc-1eed-488b-995b-627d91065642" /> <br>



### Totally stub area sonrası R5 
ABR router olan R4 hem LSA 3, LSA 4 ve LSA 5 olarak gelen rotaları engelleyerek sadece default route göndermektedir.

<img width="700" height="390" alt="image" src="https://github.com/user-attachments/assets/85b3a5b8-f70f-4c76-9975-92d132220363" /> <br>

