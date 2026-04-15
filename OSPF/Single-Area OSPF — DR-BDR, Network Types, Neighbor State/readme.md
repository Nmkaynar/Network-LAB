## OSPF Neighbor State 
Bu proje/laboratuvar çalışması, OSPF (Open Shortest Path First) protokolünün iki router arasında komşuluk kurarken geçtiği aşamaları gerçek zamanlı debug çıktıları üzerinden analiz etmektedir.


R3 routerdan bakınca kendisi her iki komşulukta da BDR olmuş. 
<img width="834" height="116" alt="image" src="https://github.com/user-attachments/assets/42656e86-5537-4b9a-96b9-b2cd6f735e23" />

DR olmasını istiyor isek, iki seçeneğimiz var.

- Router-id büyük olduğu için DR routerda "clear ip ospf process" çalıştırabiliriz.
- Eğer router-id büyük olmasaydı, interface altında priorty yükselterek yine DR routerda "clear ip ospf process" çalışıtırabiliriz.
- "clear ip opsf process" tüm komşulukları düşürüp tekrar komşuluk sürecini başlatacaktır.


## Komşuluk statele durumları
<img width="349" height="68" alt="image" src="https://github.com/user-attachments/assets/e01b3141-4997-40e1-9a3c-ad5c4a4fc026" />

Komşuluk kurma sürecinde tüm logları görebilmek için R3 de debug ip ospf adj komutunu çalıştırmalıyız
<img width="314" height="56" alt="image" src="https://github.com/user-attachments/assets/bed13bf9-fe17-4d43-990b-fb171a402191" />

R3 ile R1 arasındaki komşulukta R3'ün DR olmasını istiyor isek R1'in ospf komşuluğu down olması gerekiyor. clear ip ospf process komutunu çalıştırarak komşuluk oluşturma süreceini tekrar başlattık.

<img width="1198" height="672" alt="image" src="https://github.com/user-attachments/assets/febfb26e-770f-40a4-9f46-65db95d5e96f" />

## 2 WAY

2 way iletişimde her iki routerda birbirine hello paketi göndermiştir.

<img width="439" height="20" alt="image" src="https://github.com/user-attachments/assets/d4139e07-9e99-4b52-ae14-4505e28720a5" />

Burada DR/BDR seçimi yapılmaya başlandı. Priorty her ikisinde defaultta 1 olduğundan router-id büyük olan DR olarak seçilir.

<img width="625" height="108" alt="image" src="https://github.com/user-attachments/assets/e9d69cee-e84c-4300-8e85-1e6f4373a27d" />

R3 artık R1'e DBD paketi göndermeye başlar. Bu pakette rota bilgisi yoktur, amaç master/slave seçimi içindir. Router-id'si büyük olan master olur.

<img width="563" height="28" alt="image" src="https://github.com/user-attachments/assets/1331d8ba-cb7c-440c-b97f-9632a758d8f4" />

Bir önceki seq number ile gönderilen paketin reply paketi geldi ve flag 0*0 şeklinde geldi. Akabinde pazarlık tamamlandı ve R3 master oldu.

<img width="771" height="45" alt="image" src="https://github.com/user-attachments/assets/0c0bb6e5-e9af-4ee2-bee9-139beac92c2c" />

Artık LSA paketlerinin gönderimi ve rota bilgileri alınarak opsf areasındaki tüm rotalar öğrenilmeye başlanır. En sonda 

<img width="751" height="137" alt="image" src="https://github.com/user-attachments/assets/4a4f75fa-eff8-48e0-a201-a4f2cb79faf6" />

LSA paketleri de gönderildikten sonra exchange işlemi biter ve state FULL duruma geçer, akabinde syslog logu ekrana gelir.
<img width="757" height="71" alt="image" src="https://github.com/user-attachments/assets/36ec4281-a112-4654-b534-f39bc2437d24" />



