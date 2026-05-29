## Stub AS 
Stub AS, sadece kendi prefix'lerini originate eden ve başka AS'lerin trafiğini taşımayan otonom sistemdir. Yani:
- Sadece kendi network'lerini dış dünyaya duyurur
- Bir ISP'den öğrendiği route'u başka bir ISP'ye geçirmez
- BGP tablosunda transit görevi yapmaz
	


Bir AS iki veya daha fazla ISP'ye BGP ile komşuluk kurduğunda, filtreleme yapılmaz ise bir ISP'den gelen trafik, kendi uplink üzerimizden diğer ISP'ye geçebilir. Bu da hattın fazla kullanımına, satürasyon olmasına sebep olur. Bu yüzden iki ISP'ye de sadece kendi prefixlerimizi duyurmamız gerekmektedir.<br>



R1 Router ISP1 den 5.5.5.0/24 prefixini öğrenirken aynı zamanda ISP2 ye de öğretiyor<br>
Aynı şekilde ISP2 den  de 172.16.10.0/24 prefixini öğrenirken ISP1'e de öğretmiş oluyor. <br>

R1'in ISP1'e duyurduğu rotalar<br>
<img width="666" height="158" alt="image" src="https://github.com/user-attachments/assets/267f20b8-557f-4938-8b75-67ae3cb9893d" /><br>

R1'in ISP2'e duyurduğu rotalar<br>

<img width="660" height="150" alt="image" src="https://github.com/user-attachments/assets/5553f612-7ccf-4d80-a714-e9a2f1da6557" /><br>

Bu durumda ISP 1 ve ISP2 bu rotalar için best path olarak AS6500'i seçerse AS65000 transit AS olmuş olacak.<br>

<img width="725" height="122" alt="image" src="https://github.com/user-attachments/assets/539c11d9-dee5-4289-8968-4e20f2f44380" /><br>
<img width="700" height="125" alt="image" src="https://github.com/user-attachments/assets/af554818-0fa2-4c5a-bfd3-fc62179e399c" /><br>
Bu durumda R1 router sadece kendisine ait olduğu rotaları duyurması gerekmektedir. <br>


R1 de yapılacak config<br>

````
ip as-path access-list 10 permit ^$

route-map My-Networks permit 10
  match as-path 10
  exit

route-map My-Networks deny 20
  exit

router bgp 65000
neighbor 10.0.10.1 route-map My-Networks out
neighbor 10.0.11.1 route-map My-Networks out
````
"^$" sadece AS-PATH'i boş olan rotaları eşleştirir demektir. Bu da local olarak üretilen rotaları kapsar. Diğer AS'lerden gelen rotalarda en az 1 tane AS NO olacağı için o rotaları outbound yönünde duyurmayacak.<br>
 
R1 ISP1'e 172.16.10.0/24 prefixini duyurmamış oldu<br>
<img width="687" height="97" alt="image" src="https://github.com/user-attachments/assets/6d206fe2-b8f3-4397-ad8c-a0b00e23439c" /><br>

<img width="669" height="105" alt="image" src="https://github.com/user-attachments/assets/5229b1ae-7cfc-4376-ba3f-737d485b0aae" /><br>
Aynı şekilde 5.5.5.0/24 prefini de ISP2'ye duyurmamış oldu<br>
<img width="708" height="108" alt="image" src="https://github.com/user-attachments/assets/b28ad524-1dea-4323-b094-7e13dcea94d4" /><br>

<img width="663" height="101" alt="image" src="https://github.com/user-attachments/assets/46f28bfa-9a1c-4f43-abc4-1e7b01c6fbcf" /><br>

## ISP'lerden sadece default rota almak. <br>


Her iki ISPden default rotanın yanında farklı prefixlerde alıyoruz.<br>

<img width="696" height="155" alt="image" src="https://github.com/user-attachments/assets/36d06f98-de98-489d-9f35-979b1de8cf03" /><br>

Bunun için  R1 de aş.daki configi uygulamalıyız.<br>

````
ip prefix-list Default-rota permit 0.0.0.0/0

route-map Only-default-rota permit 10
   match ip address prefix-list Default-rota
route-map Only-default-rota deny 20
router bgp 65000
    neighbor 10.0.10.1 route-map Only-default-rota in
    neighbor 10.0.11.1 route-map Only-default-rota in
   do clear ip bgp * soft in
````

<img width="630" height="123" alt="image" src="https://github.com/user-attachments/assets/dcf7b43a-9a0b-435b-a705-228e64373590" /><br>

Artık R1 Router'ı hem sadece kendi rotalarını duyuruyor hem de  ISP'lerden sadece default rotayı alıyor. <br>

