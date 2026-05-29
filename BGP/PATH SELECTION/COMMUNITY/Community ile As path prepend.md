## Community ile As Path Prepend

R2 router'ı adviretise ettiği 192.168.10.0/24 networkü için AS100 ve AS300 ISP2 olan AS600'ü tercih etmekte. 

İstediğimiz ise ISP1 olan AS500'ü tercih etmesi. Bu sebepten ötürü ISP2'nin vermiş olduğu 600:100 community si ile 192.168.10.0/24 prefixini tagliyerek AS600'a göndereceğiz. AS600 de 2 kere AS path prepend yaparak yolu uzatacak. Bu işlemin sonunda AS100 ve AS300 192.168.10.0/24 netwörüküne erişmek için ISP1'i tercih edecekler.
Şu an ki durum

AS100 (R1 Router)<br>
<img width="684" height="115" alt="image" src="https://github.com/user-attachments/assets/58f9581d-56bb-4bef-a04d-aa38cd618439" /><br>
AS300 (R3 Router)<br>
<img width="719" height="98" alt="image" src="https://github.com/user-attachments/assets/e33fef06-176e-45b4-8977-79a18518df67" /><br>
Bunun için R2 de yapılması gerekenler

````
ip prefix-list As-Path-Prepend permit 192.168.10.0/24

route-map RM-ISP2-Out permit 10 
 match ip address prefix-list As-Path-Prepend
 set community 600:100

route-map RM-ISP2-Out permit 20
exit

router bgp 200
 neighbor 172.16.20.1 send-community
 neighbor 172.16.20.1 route-map RM-ISP2-Out out
 do clear ip bgp 172.16.20.1 soft out
````


ISP2-ROuter da yapılması gerekenler

````
ip community-list standard As-Path-Prepend permit 600:100

 route-map customer-in permit 10
    match community As-Path-Prepend
    set as-path prepend 600 600
    route-map customer-in permit 20
    exit
  
router bgp 600
   neighbor 172.16.20.2 route-map customer-in in
   do clear ip bgp 172.16.20.2 soft in
 ````

Bu işlemlerden sonra 

AS100 ve AS300 192.168.10.0/24 networkü için ISP1'i tercih etmektedir.

<img width="756" height="100" alt="image" src="https://github.com/user-attachments/assets/2dc6d414-7688-406f-bb7a-ec8ad79f5757" />
<img width="752" height="96" alt="image" src="https://github.com/user-attachments/assets/a681c557-5563-43e5-92e9-1ace98c53ca6" />


