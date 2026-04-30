## VRF-Lite
<img width="1084" height="394" alt="image" src="https://github.com/user-attachments/assets/e792888c-40cd-4b46-9bc2-fbb92c007e24" />
VRF (Virtual Routing and Forwarding), bir router üzerinde birden fazla bağımsız routing table tutmaya yarayan bir özelliktir. 
Her VRF kendi routing table'ına, kendi interface'lerine ve kendi forwarding kararlarına sahiptir.
Yani tek bir fiziksel router, birbirinden tamamen izole birden fazla sanal router gibi davranır.

## Neden VRF'e ihtiyaç duyarız
Standart bir router'da tek bir global routing table vardır. Bu durumda iki ayrı müşterinin trafiğini aynı router üzerinden taşımak istediğinde sorun çıkar:
- İki müşteri aynı IP bloğunu (örn. 10.0.0.0/24) kullanıyorsa çakışma olur
- Bir müşterinin prefix'leri diğer müşterinin routing table'ına sızabilir
- Trafik izolasyonu sağlanamaz

VRF bu sorunları router seviyesinde çözer. Her müşteri için ayrı bir VRF tanımlanır, müşteriye ait interface o VRF'e atanır ve müşterinin tüm routing bilgisi sadece kendi VRF'inde tutulur.

## Global Routing Table vs VRF
Bir router'da VRF tanımlanmadığında tüm interface'ler ve rotalar global routing table'da bulunur. VRF tanımlandığında ise interface'ler bir VRF'e assign edilir ve o VRF'in kendi routing table'ı oluşur.

- Her vrf nin routing table'ını `` show ip route vrf <vrf-name>`` şeklinde bakılır.
- Routerdan ping atmak için ise `` ping vrf <vrf-name> <ip-address>`` şeklinde atılır


Normal şartlarda aynı prefixi router üzerinde iki farklı interface atayamaszsınız.
<img width="528" height="101" alt="image" src="https://github.com/user-attachments/assets/899dd65f-6278-4959-a51e-15d7d6d6a588" />

VRF ile router içinde sanal routerlar oluşturulduğundan bu interfaceler birbirlerinden bağımsız hareket ederler. Ve aynı gateway ip adresini farklı interfacelere atayabilirsiniz.
<img width="737" height="245" alt="image" src="https://github.com/user-attachments/assets/78e8b01e-4ce0-4288-9c36-22bd6537901a" />

interfaceler vrf'lere atandığında bu interfaceleri global routing table göremezsiniz.
<img width="759" height="246" alt="image" src="https://github.com/user-attachments/assets/4107b91c-4c30-499c-9c14-a9b1e96a88b8" />

Her bir vrfnin routig table için `` show ip route vrf <vrf-name>`` komutunu kullanmalısınız

<img width="651" height="192" alt="image" src="https://github.com/user-attachments/assets/2858be98-936e-4def-92d4-2589051b620e" />

<img width="661" height="205" alt="image" src="https://github.com/user-attachments/assets/43f16a57-9cf7-4eb7-b3bf-40c311ee4f2f" />

## Config

- Her bir müşteri için vrf tanımlanmalıdır. Bunuda config modda `` ip vrf <vrf-name>`` şeklide oluşturulur
- Hangi interface hangi vrf'ye atanacak ise interface altında ``ip vrf forwarding <vrf-name>`` şeklinde atanır.
- Eğer interface de ip adresi var ise vrf ataması yapıldıktan sonra ip adresini tekrar yazılmalıdır. Çünkü ip adresi silinecektir.
- <img width="929" height="66" alt="image" src="https://github.com/user-attachments/assets/600d87ff-a56f-45ec-80de-12975e5c3a41" />
- R1-R2 arasında her bir vrf için ayrı interface de kullanılabilir. Burada sub-interface tercih ettim.
- Her vrf içindeki subnetlerin erişimi için statik router kullandım. `` ip route vrf <vrf-name> <network-id> <Mask> <Next-Hop> `` komutu ile statik routing yapılmalıdr.


## Configler Tamamlandıktan sonra Test Aşaması

R1 Router'ında Customer-A vrf si içinde bulundan 172.16.10.20 cihazına ping attığımda erişim sağlayabildim. Ancak aynı subnet içinde olmasına rağmen farklı vrf de olduğu için 172.16.10.10'a ping atılamamaktadır.
<img width="694" height="313" alt="image" src="https://github.com/user-attachments/assets/34f46243-b5c1-4f6f-be42-c27085a01e13" />

PC1 P4'e ping atabilirken PC3'e ping atamamaktadır.

<img width="621" height="331" alt="image" src="https://github.com/user-attachments/assets/f3845768-52b8-4225-b89e-59ddad82a77a" />
Aynı şekilde PC2 de, P3'e ping atabilirken P4'e ping atamamaktadır.

<img width="554" height="327" alt="image" src="https://github.com/user-attachments/assets/52a50a4e-e440-439f-9215-10b0dcdda982" />



Customer A ve B aynı prefixleri kullanmasına rağmen birbirlerinden izole bir şekilde uzaktaki site erişim sağlayabilmektedir.


Burada bir problem söz konusu ve bu problem ISP tarafındadır. iki müşteri için 2 farklı sub interface yapıldı veya 2 farklı interface de kullanılabilrdi. Peki 100 müşteri olsa veya 10000 müşteri olsa ne yapılacak. Bu ölçeklenebilir bir durum değildir. 

İşte burada MPLS L3VPN devreye girerek ip forwarding yerine label forwarding ile tek bir hat üzerinden yani sub interface bile yapmadan, tüm prefixlerin birbirine çakışmadan izole bir şekilde taşınmasını sağlayacaktır.

















 
