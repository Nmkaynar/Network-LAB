# OSPF LSA TYPES



OSPF routing protokolü, ağ topolojisini öğrenmek ve en iyi rotaları hesaplmak için LSA(Link State Advertisement) adı verilen paketleri kullanır.
Her LSA tipi farklı bir amaca hizmet eder ve farklı bir kapsama alanına sahiptir 

OSPF veritabanında (LSDB - Link State Database) depolanmaktadır.


<img width="717" height="628" alt="image" src="https://github.com/user-attachments/assets/d602e153-1c82-45c3-9acb-ef8e4aaefb9a" /> <br>

## LSA Type 1 - Router Link States 

- LSDB deki bu kısım, area içinde OSPF komşuluğu kurmuş tüm routerları gösterir. 
- Her OSPF router kendi bağlantılarını ve bu bağlantıların maliyetlerini tanımlayan bir LSA üretir. 
- Bu LSA yalnızca üretildiği area içinde yayılır. Area sınırını geçemez.

<img width="716" height="114" alt="image" src="https://github.com/user-attachments/assets/038f5456-b601-45f2-977c-e381f047f882" /><br>

Router link states kısmındaki geçen bilgiler LSA 1 paketleri ile gelmektedir.

```
- Link ID            LSA'yı üreten router'ın Router ID'si
- ADV roouter        LSA'yı Adverstising(duyuran) eden router
- Age                LSA'nın kaç saniue önce üretildiğini gösterir. (Maks 3600)
- Seq#               Sıra numarası - Arttıkça daha yeni demektir.
- Checksum           Veri bütünlüğü kontrolü
- Link count         Router'ın  OSPF'e area içinde dahil edilmiş kaç adet interface olduğunu gösterir
```

## LSA Type 2 - Net Link States

Bu kısım ortamdaki DR Router'ın interface ip adresi ve Router ID'sini gösterir. Bu Type 2 LSA, DR router tarafından üretilir.

<img width="630" height="97" alt="image" src="https://github.com/user-attachments/assets/c4048268-a1f4-451c-a399-aae364598dca" /> <br>

<img width="410" height="191" alt="image" src="https://github.com/user-attachments/assets/fa11dc9c-d00c-49ac-8ddf-7e0515fef309" />

Farklı bir örneğe baktığımızda burada 2 farklı segment olduğundan 2 farklı segmentteki DR routerlar gözüküyor. 

<img width="665" height="124" alt="image" src="https://github.com/user-attachments/assets/cd988bc3-48aa-470e-b1f0-1752446a244b" /><br>

1.satır : R1 ve R2 arasında kurulan ospf komşuluğunda R2 DR router olduğundan onun interface ip adresi ve Router-id'si <br>
2.satır : R1 ve R4 arasında kurulan ospf komşuluğunda R1 DR router olduğundan onun interface ip adresi ve Router id'si gözükmektedir.


## LSA Type 3 - Summary Net Link States

Bu kısım, ABR tarafından üretilen  Type 3 LSA'ları gösterir. Bu LSA'lar, farklı area'lardaki subnet bilgilerini taşır.

<img width="587" height="139" alt="image" src="https://github.com/user-attachments/assets/9396617d-381f-457c-b490-8216db636eeb" /> <br>

```
- Link id       Network id adresidir.
- ADV Router    Bu subnet bilgisini duyuran ABR'ın Router ID'si
- Buradaki networklar routing table'da O IA şekiinde duyurulur.
```
<img width="678" height="165" alt="image" src="https://github.com/user-attachments/assets/f3fc6769-82d7-4ffa-b98b-130896997a5a" /> <br>


## LSA Type 4 - Summary ASB Link States
Bu kısım OSPF topolojisinde EXternal rotaları duyuran ASBR router nerede sorusunun cevabıdır. ABR tarafından üretilir ve ASBR Router'a giden rotayı gösterir. <br>

<img width="618" height="88" alt="image" src="https://github.com/user-attachments/assets/47f9bbb7-c33d-487c-8784-86deb75ffedb" /><br>
```
- link id     ASBR Router'ın Router-id'si
- ADV Router  Bu bilgiyi üreten ABR Router'ın Router-id'si
```

## LSA Type 5 - AS External Link States

Bu kısım OSPF dışından öğrenilen rotaları duyoran LSA'ları gösterir.

<img width="667" height="90" alt="image" src="https://github.com/user-attachments/assets/6ab0f54a-bcd1-49c8-94c7-6ea73cbe02f0" /><br>
```
- ASBR Router tarafından üretilir
- Link ID      Duyurulan External rotanın network idsi
- ADV router   Bu rotayı duyuran ASBR Router'ın Router-id'si
```



















