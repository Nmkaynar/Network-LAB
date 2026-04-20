
# IBGP Full Mesh ve Route Reflector
Aynı AS içindeki routerlarda kurulan IBGP öğrenilen bir rota, loop önlemek amacıyla başka bir IBGP komşusuna öğretemez. Bu da IBGP içindeki routerların birbiri ile komşuluk kurmasını zorunlu kılar. 
<img width="1067" height="249" alt="image" src="https://github.com/user-attachments/assets/1f57abe8-6434-416d-bf3f-39ebd0ee8058" /><br>
## Bu topolojideki config
### R1 için
````
router bgp 65000
bgp router-id 1.1.1.1
neighbor 10.0.10.2 remote-as 65000
network 192.168.10.0 mask 255.255.255.0
````
Routing table da sadece R2'nin duyurduğu LAN networkü öğrenilmiş
<img width="503" height="83" alt="image" src="https://github.com/user-attachments/assets/35795043-7e2d-466d-afaa-86a66b6ad65c" /><br>
### R2 için 
````
router bgp 65000
bgp router-id 2.2.2.2
neighbor 10.0.10.1 remote-as 65000
neighbor 10.0.11.2 remote-as 65000
network 200.100.200.0 mask 255.255.255.0
````

Hem R1 hemde R3 ile komşuluğu olduğundan her ikisinin duyurduğu networkleri öğrenmiş.

<img width="485" height="130" alt="image" src="https://github.com/user-attachments/assets/0ac5cfea-b0a4-4cd9-9d9e-7b9508c62fd8" /><br>


### R3 için 
````
router bgp 65000
bgp router-id 3.3.3.3
neighbor 10.0.11.1 remote-as 65000
network 172.16.10.0 mask 255.255.255.0
````
R1 de olduğu gibi R2 nin kendisinin duyurduğu networkü öğrenmiş
<img width="508" height="81" alt="image" src="https://github.com/user-attachments/assets/48c68ceb-1aeb-4574-aee6-f52596b925f6" /><br>
Tabloya baktığımızda IBGP loop engelleme kuralından dolayı R2, R1 'den öğrendiği networkü R3'e, R3 'ten öğrendiği networküde R1'e öğretmemiş. 
Bu durumda yapılması gereken FULL MESH ya da Route Reflector.

## Full MESH
Full mesh olması için tüm routerların birbiri ile IBGP komşuluğu kurması gerekmektedir.<br>
R1 ve R3'ün birbirine erişim sağlayabilmesi için statik route girdik. 

Topoloji  de R1 ile R3 arasında da bir IBGP komşuluk kurduğumuzda R1 ve R3 karşılıklı networklerini birbilerini öğretmiş oluyor

### R1 Router:
<img width="803" height="101" alt="image" src="https://github.com/user-attachments/assets/e240894c-3091-4e9d-b998-76c0c02d5bdc" /><br>

### R1 Routing table:
<img width="528" height="147" alt="image" src="https://github.com/user-attachments/assets/1da5b0ed-9fe7-466c-b86a-bb1c454ecc63" /><br>

### R3 Router :
<img width="803" height="111" alt="image" src="https://github.com/user-attachments/assets/b9d13365-86f4-4786-8445-9f8fb6d6c126" /><br>

### R3 Routing table
<img width="536" height="125" alt="image" src="https://github.com/user-attachments/assets/6dc4bb13-73a2-400d-8e2b-f8bfe9ff291d" /><br>


Buradaki problem ağ büyüdükçe gelen operasyonel zorluk. Eğer 3 router değilde 4 router olsaydı 6 tane komşuluk veya 10 tane router olsaydı 45 tane komşuluk kurulması gerekirdi.


## Route Reflectör(RR)

Route Reflector, IBGP de ki bu engeli kaldırmak için ve aynı zamanda loop oluşturmadan, başka bir IBGP komşusundan öğrendiği rotaları öğreten bir roldür.


### Config
RR olacak olan router da BGP config altında ``neighbor x.x.x.x route-reflector-client`` komutu girilmelidir.


Topolojide R2 de bu komutu hem R1 için hem de R3 için yapmalıyız.

````
router bgp 65000
neighbor 10.0.10.1 route-reflector-client
neighbor 10.0.11.2 route-reflector-client
````
<img width="870" height="255" alt="image" src="https://github.com/user-attachments/assets/37035beb-cae2-470b-86c5-13828c0931ef" /><br>


Bu şekilde yapılınca R2 öğrendiği rotaları RR Client olan routerlarda öğretecek. Ancak buradaki sorun şu , R2 öğrendiği şekliyle öğretecek yani 
R2, R1'e 172.16.10.0/24 networkünü öğretirken next hop olarak 10.0.11.2 olarak öğretecek ve R1 de eğer 10.0.11.2'yi bilmiyor ise RIB'e R2 den öğrendiği rotayı ekleyemeyecek. 

### R1'de
<img width="624" height="120" alt="image" src="https://github.com/user-attachments/assets/952e7b3a-a7d0-4ceb-868e-bdcb056139f8" />
<img width="545" height="97" alt="image" src="https://github.com/user-attachments/assets/fe15b176-0a14-4982-bcfa-9e244975a85b" />

### R3'de
<img width="696" height="118" alt="image" src="https://github.com/user-attachments/assets/92df022c-c579-4b31-9d2e-6b97c45590a2" />
<img width="504" height="98" alt="image" src="https://github.com/user-attachments/assets/72fee276-bff4-4424-abe7-66b783c91bfb" /><br>

Bunu çözmek için  R1'in 10.0.11.0/30 networkünü öğretmeli veya RR'da (R2)``neighbor x.x.x.x  next-hop-self all`` kullanmalıyız.

<img width="505" height="48" alt="image" src="https://github.com/user-attachments/assets/c6117f49-d129-4c35-9958-c56c9a0c81f6" /><br>

R1 routing table

<img width="512" height="128" alt="image" src="https://github.com/user-attachments/assets/f0be201c-83fe-46e1-bbda-cb74837beea7" /><br>

R3 routing table
<img width="551" height="116" alt="image" src="https://github.com/user-attachments/assets/c93d2100-a8b9-47b3-8b8a-2847865c8225" /><br>


## Route Reflector de Neden Loop Yok

Standar IBGP yapılandırmasında, loop prevention kuralı(IBGP de öğrenilmiş rotalar duyurulmaz kuralı) gereği bir router IBGP komşusundan öğrendiği rotaları başka bir IBGP komşusuna advertise etmez. RR yapıldığında ise artık client olan IBPG komşularına bu rotaları advertise etmeye başlar. 

Loop olmasını ise paketlere eklediği  ORIGINATOR_ID ve CLUSTER_LIST ile kontrol etmektedir. <br>
ORIGINATOR_ID:  Networkü duyuran orjinal router'ın Router-id'si <br>
CLUSTER_LIST :  Route'un geçtiği RR cluster'larının ID listesi. <br>
Eğer Router'lar kendi ID'lerini görürlerse paketi drop ederler. <br>

<img width="760" height="694" alt="image" src="https://github.com/user-attachments/assets/f3a2a320-b399-4b92-8a6a-ea09c3ffcd16" /><br>

Görselde görüldüğü gibi bu paket R2'inin R3'e gönderdiği BGP Update paketi. R1'den öğrendiği 192.168.10.0/24 networkünü R3'e öğretirken Path Attribute'larına eklediği iki tane özellik mevcut. <br>
Ancak kendi üzerindeki networkü öğretirken Path Attribute'larında ORIGINATOR_ID ve CLUSTER_LIST bulunmamaktadır.
