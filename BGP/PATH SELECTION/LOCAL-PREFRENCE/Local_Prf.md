## LOCAL-PREFRENCE
<img width="679" height="707" alt="image" src="https://github.com/user-attachments/assets/4994ee77-ea1b-48d5-950d-fdea3cf96efd" /> <br>
Local  Pref, BGP path selection da Routerların ikinci baktığı değerdir. Bir AS içerisinde çıkış noktasını belirlemek için kullanılan BGP attribute'dur

- Vendor bağımsızdır
- Aynı AS içindeki tüm routerlara gider. Yani update paketinde taşınır.
- Default değeri 100 dür.
- Best Path seçiminde büyük değer tercih edilir.
- Farklı bir AS'a gitmez.

  
Local Pref'i değiştirmenin iki yöntemi vardır.
- bgp default local-prefrence <DEĞER>
- Route-map ile 


## 1. Yöntem 
``bgp default local-preference <DEĞER>`` komutu ile yapıldığında tüm rotaların değerini değiştirir. Etkinleştirildiğinde bu routerın diğer IBGP komşularına öğrettiği rotaların değeri atanan değer olur. Ve bu tüm IBGP komşularına aktarılır.


Şu an ki durumda internet çıkışı ISP1 dedir. Bunu ISP2 yapalım.

<img width="659" height="128" alt="image" src="https://github.com/user-attachments/assets/ff81b02f-c13c-4ba6-bf06-a0e5b4f18fcc" /><br>
R2 de ``bgp default local-preference 200`` komutunu girelim<br>
<img width="483" height="31" alt="image" src="https://github.com/user-attachments/assets/7ca52b8c-4a87-4aa8-af28-91649f7120f1" /><br>
R1 artık internet çıkışı için R2'yi tercih etmeye başladı. 
<img width="661" height="132" alt="image" src="https://github.com/user-attachments/assets/167cee2f-beed-4548-aa88-3153d4962c12" /><br>
R3'ü kontrol ettiğimizde local prf'in değiştini gözlemliyoruz.
<img width="643" height="109" alt="image" src="https://github.com/user-attachments/assets/8458ea11-f878-4a5e-b021-37db92b22cfa" /><br>


Bu yöntem kullanıldığında show çıktılarında görüldüğü gibi tüm rotalara aktif olarak uygular.


## 2. Yöntem

Route-map ile sadece 8.8.8.8 için ISP1'i tercih etmesini sağlayalım. 

R1 de yapılacak olan işlem

````
ip prefix-list PL-WAN seq 5 permit 8.8.8.8/32

route-map RM-WAN permit 10
 match ip address prefix-list PL-WAN
 set local-preference 300
 exit

route-map RM-WAN permit 20
 exit

router bgp 65000
 neighbor 192.168.1.1 route-map RM-WAN in
 end

clear ip bgp 192.168.1.1 soft in 
````

Artık R1 ve R2 Router'ları 8.8.8.8 için ISP1, 9.9.9.9 için de ISP2'yi tercih ediyor.  
<img width="644" height="116" alt="image" src="https://github.com/user-attachments/assets/e9981f69-c6b2-40a2-8be8-ce04bfa4eae2" /><br>
<img width="660" height="125" alt="image" src="https://github.com/user-attachments/assets/5b8c01a1-7a83-4632-ae21-37df6bbe20c0" /><br>

R3 te de bu değerin iletildiğini görebiliyoruz.<br>
<img width="640" height="96" alt="image" src="https://github.com/user-attachments/assets/70b90e0c-01f9-4ac2-a65e-7896ba3dc40d" />



Wiresharkta Uptade mesajı ile Local_PREF taşınmaktadır. Bu paket R2'nin R1'e gönderdiği IBGP Update paketidir.
<img width="738" height="409" alt="image" src="https://github.com/user-attachments/assets/838da49c-0d32-4668-adc3-f1d45027f921" /><br>

