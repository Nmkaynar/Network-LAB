## MPLS

MPLS layer 3 ile layer 2 arasında paketleri label forwarding ile forward eden bir yöntemdir. 


MPLS de CE, PE ve P routerlar bulunmaktadır.  
CE  : Customer Edge yani müşterinin ısp tarafında ki Router'ı  
PE  : Provider Edge, ISP'nin müşterinin tarafındaki Kendi Routerları  
P   :  Bu routerlar PE arasında kalan ISP nin kendi altyapısında bulunan routerlar.
<img width="1083" height="662" alt="image" src="https://github.com/user-attachments/assets/6932ed45-b59e-4081-9638-d5b86f9c9021" /><br>

Bu topoloji de MPLS olmadan PC1, PC2ye ping atabilmesi için  bütün routerların bir şekilde prefixleri öğrenmesi gerekmektedir.

MPLS, forwarding işlemini label ile yaptığı için core routerların CE arkasındaki prefixlerini bilmesine gerek bırakmaz.

CE'den PE'ye paketler ip forwarding ile gelir. Bir PE'den diğer PE'ye gidesiye kadar olan süreçte routerlar ip forwarding yerine label forwarding yaparlar.

Yani bir P router'ına gelen pakette router destination IP'ye bakmak yerine label no ya bakar ve mpls forwarding table'a göre Next Hop'a yönlendirir. Burada P routerlar müşterinin prefixlerini bilmemektedir. 



## Paket Forwarding


Bu durumda P routerlar prefixleri bilmeden forwardingi nasıl yapmaktadır? 
MPLS, multi protocol Label switching demektir Yani CE den çıkan IP paketlerini iç taraftaki P routerlarına gönderirken 
etiket basarak sadece labele(etikete) göre swap ederek ıp paketlerini forwarding etmektedirler. 
Yani herhangi bir destination ip adresine bakmamaktadırlar.

Bu yöntemde ISP'nin kendi alt yapısındaki routerların CE  prefixleri Routing Table'da tutmalarını ortadan kaldırmaktadır.


## Config Nasıl yapılmaktadır

Burada Routerlar arasındaki config şu şekilde olmaktadır


CE'de PE'lere doğru statik default route girildi

PE'lerde ise CE'lerin arkasında kalan rotalar için statik girildi.

PE-P-PE aralarında OSPF yapılandırıldı.


## Önce PE1,P1, P2 ve PE2 routerların `show ip route` çıktılarına bakalım


### PE1 
Tüm prefixleri bilmelidir.<br>
<img width="732" height="405" alt="image" src="https://github.com/user-attachments/assets/622318f0-e91e-4118-a449-3b55dbe4b0e3" /><br>

### P1
172.16.10.0/24 ve 192.168.10.0/24 prefixlerini bilmemektedir.<br>
<img width="734" height="345" alt="image" src="https://github.com/user-attachments/assets/9efe45a3-a91e-43b9-a1e6-43d004015488" /><br>

### P2
172.16.10.0/24 ve 192.168.10.0/24 prefixlerini bilmemektedir.<br>
<img width="731" height="361" alt="image" src="https://github.com/user-attachments/assets/76dfa88c-d2e4-402b-bd9a-18706b736c54" /><br>
### PE2
tüm prefixleri bilmelidir.<br>
<img width="632" height="416" alt="image" src="https://github.com/user-attachments/assets/02b28ba9-d20e-4878-aacb-2c16704f98c4" /><br>

PE'lerin arasında kalan P router CE arkasındaki networkleri bilmemektedir.Buna rağmen PC1 den PC2'ye ping gitmektedir.<br>
<img width="599" height="167" alt="image" src="https://github.com/user-attachments/assets/9b423c5c-7978-46ed-88bf-bce7f27c0b8c" /><br>

## Wiresharkta Paketlerin LABEL'lerini görelim

PC1den PC2 ye ping başlattığımda 

### CE1-PE1 Arasında 
Burada MPLS aktif olmadığı için herhangi bir label bulunmamaktadır. Default rota ile CE1 PE1'e teslim eder
<img width="738" height="125" alt="image" src="https://github.com/user-attachments/assets/11808d41-1069-499d-b5bd-c8f28d79aeb4" />  

###  PE1- P1 Arasında 
Burada gelen paketi dst : 172.16.10.10 olduğundan girdiğimiz statik route nedeniyle PE1 paketi 4.4.4.4'e göndermeye çalışır. PE1 de 4.4.4.4'ü ospften öğrenmiştir. MPLS ise bu öğrendiği 4.4.4.4 için 19 atayarak paketi next hopa yönlendirir
<img width="688" height="217" alt="image" src="https://github.com/user-attachments/assets/cf955f7d-b28d-4ee7-9113-731a9e8f5a1c" />  

PE1 router paketi gönderirken MPLS 19 etiketi basarak P1'e göndermektedir.
<img width="737" height="186" alt="image" src="https://github.com/user-attachments/assets/b8fbb4ec-ec2e-44d1-bbe4-c59ae2690d95" />


### P1- P2 Arasında

PE1 den gelen pakette P1 Router'ı MPLS 19 etiketini gördüğünde Mpls Forwarding Table'ında local 19 satırına bakar. 
Bu satıra göre outgoing kısmı paketi gönderirken basacağı etiket, next hop olarak da 10.0.12.2'ye gönderir.
<img width="691" height="151" alt="image" src="https://github.com/user-attachments/assets/b50d408b-0326-4143-8a54-f2d73228d8d0" />
Bu tabloda local sütunu gelen paketteki etiket için ne yapalıcağını gösterir. 19 ile geldi 19 ile 10.0.12.2'ye göndermesini sağladı. MPLS Label kısmı internet protocol version 4 headerın önünde olduğundan router önce labele bakar, label varsa label forwarding yaparak ip headerı incelemez

<img width="768" height="178" alt="image" src="https://github.com/user-attachments/assets/cffb7bdd-0762-45ad-8785-952613727bac" />

### P2- PE2 Arasında

P1 den gelen pakette P2 Router'ı  MPLS 19 etiketini gördüğünde Mpls Forwarding Table'ında local 19 satırına bakar. Bu satıra göre outgoing kısmında Pop Label görmekte. PoP Label çıkar ve yeni bir etiket basmadan gönder demektir. Kime Next Hop'a 10.0.13.2

Pop Label demek tablodaki 19 local satırındaki prefixdekinden bir önceki router olduğunu anlatır. 
<img width="709" height="166" alt="image" src="https://github.com/user-attachments/assets/15684940-973e-4cfc-acb6-2d18f80dd542" />


Wiresharktada görüldüğü üzere label bulunmamaktadır. Bunun sebebide PE2 label çıkartmak ile uğraşmasın direkt ip forwarding yapabilsin diye.<br>
<img width="782" height="143" alt="image" src="https://github.com/user-attachments/assets/9231f0ae-6b90-426f-a9b4-5898f198ec65" /><br>
### PE2- CE2 Arasında

CE1-PE1 arasında nasıl ise o şekilde ip forwarding yaparak gönderir.


## Reply Paketleri

### PE2 - P2 arasında 

CE2 den PE2 ye gelen reply paketinde dst 192.168.10.10 yazmakta. Bu routing table da ise 1.1.1.1/32 olarak görülmekte.
Mpls aktif olduğundan 1.1.1.1/32 mpls forwarding table outgoing için 17 etiketini bas ve 10.0.13.1'e gönder olarak yazmaktadır.
<img width="674" height="212" alt="image" src="https://github.com/user-attachments/assets/38524bb1-1ab8-43d3-a89a-0071abc8c301" />
 
Wiresharktada görüldüğü üzere MPLS label 17 etiket bulunmakta.
<img width="766" height="194" alt="image" src="https://github.com/user-attachments/assets/739e3e34-89e5-4ba7-9a5a-44575ac52500" />

### P2-P1 Arasında 
PE2 den gelen 17 etiketli reply paketinde MPLS 17 etiketini görünce P2 forwarding table bakar. Ve local 17 satırında ise 
outgoing olarak 16 etiketini bas ve 10.0.12.1'e gönder demektedir.

<img width="684" height="162" alt="image" src="https://github.com/user-attachments/assets/5c33a639-ddb2-4f00-b183-18d8da011b5d" />

P2 17 etiketini 16 olarak değiştirerek paketi P1'e swap eder. 

<img width="810" height="147" alt="image" src="https://github.com/user-attachments/assets/20dfd4e4-3e3e-4b86-a83c-d4dfadd1dbc4" />



### P1-PE1 arasında
P2 den gelen 16 etiketli reply pakentinde 16 etiketini görünce P1 mpls forwarding table bakar.
Local de 16 satırında  Pop Label yazmakta ki bu gönderirken etiketi çıkar demektir. 

<img width="717" height="160" alt="image" src="https://github.com/user-attachments/assets/604c1678-c912-4059-9a3f-75eafffcdf7e" />

Etiketi çıkararak next hopa gönderir.

Bu şekilde PE1 ve CE1 de ip forwarding yaparak paketi hedef ulaştırır.


MPLS olmasa idi paket P1 ve P2 Routerında DST: 172.16.10.10 ip'sini bilmediği için drop olacaktı. Drop olmaması için CE arkasındaki Prefixleri öğrenmesi gerekirdi.




