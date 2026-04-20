
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
Tabloya baktığımızda IBGP loop engelleme özelliğinden dolayı(Split horizon kuralı) R2, R1 'den öğrendiği networkü R3'e, R3 'ten öğrendiği networküde R1'e öğretmemiş. 
Bu durumda yapılması gereken FULL MESH ya da Route Reflector.

## Full MESH
Full mesh olması için tüm routerların birbiri ile conected olması ve IBGP komşuluğu kurması gerekmektedir. 
<img width="708" height="385" alt="image" src="https://github.com/user-attachments/assets/c46c8832-ffce-4f4f-af37-558cd32d06f5" /><br>

Topoloji  de R1 ile R3 arasında da bir kablo bağlantısı yapıp R1 ve R3 ile de IBGP komşuluk kurduğumuzda R1 ve R3 kendi duyurdukları networkleride birbirine öğretmiş oluyor

### R1 Router:
<img width="791" height="110" alt="image" src="https://github.com/user-attachments/assets/d43ba1e4-d7f6-427b-86f9-59da5c301708" /><br>

### R1 Routing table:
<img width="519" height="115" alt="image" src="https://github.com/user-attachments/assets/8bd8c987-e47a-4a11-afdb-a1a7f974d4fb" /><br>

### R3 Router :
<img width="801" height="90" alt="image" src="https://github.com/user-attachments/assets/663120e1-3e60-4edc-9b38-8fda88bb8181" /><br>


### R3 Routing table
<img width="531" height="101" alt="image" src="https://github.com/user-attachments/assets/a0e558a0-190f-41b3-b217-9b430047fb65" /><br>


Buradaki problem ağ büyüdükçe gelen operasyonel zorluk. Eğer 3 router değilde 4 router olsaydı 6 tane komşuluk veya 10 tane router olsaydı 45 tane komşuluk kurulması gerekirdi.
Ve hepside birbiri ile direkt connect olması gerekirdi. Bu da hem kablo maliyetini hem de yönetim maaliyetini artıracaktır. 


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

