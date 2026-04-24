# MED(Multi-Exit Discriminator)
- Komşu AS ile birden fazla bağlantı var ise kendi ağınıza girmek için hangi giriş yolunu tercih etmesi gerektiğini söylemek için kullanılır
- Aynı prefix ve AS den gelen rotalarda en düşük MED tercih edilir
- İnbound trafik kontrolü sağlar. Ancak komşu bunu görmezden gelebilir.
- MED bir AS içerisine girdiken sonra o AS içinde yayılır ancak başka bir AS'e aktarılmaz. Yani AS 100 komşusu olan AS 200'e aktarır ancak AS 200  komşusu olan AS 300'e aktarmaz.
- Defaultta 0 dır
<img width="601" height="507" alt="image" src="https://github.com/user-attachments/assets/179c5181-5155-47d6-8101-e4e76ce600de" /><br>
Şu anki topoloji de AS100'den AS65000 deki 172.16.10.0/24 networküne R1, R3 üzerinden R2 de R4 üzerinden gelmektedir.  R2 ninde R1->R3 üzerinden gelmesini istiyorsak MED değerini değiştirerek gelen trafiği manipüle edebiliriz.

R1 Router'ı:
<img width="656" height="93" alt="image" src="https://github.com/user-attachments/assets/968f8520-84a1-49e4-bc93-c9c3a8b88a42" /><br>
R2 Router'ı
<img width="646" height="102" alt="image" src="https://github.com/user-attachments/assets/c4535336-c2c2-451a-8970-c59b87565892" /><br>
R2 ve R4 yolu tercih edilmesini istemiyorsak R4 üzerinde Metric değiştirmeliyiz.



````
route-map RM-MED permit 10
 set metric 100

router bgp 65000
  neighbor 185.100.60.1 route-map RM-MED out
  do clear ip bgp 185.100.60.1 soft out


````

route map uygulandıktan sonra R2 artık R1'i tercih etmeye başladı. 

<img width="670" height="101" alt="image" src="https://github.com/user-attachments/assets/bc69ccec-5f0d-4c7c-8057-171a9632ff00" /><br>

R1 de ise tek bir rota kaldı. Bunun sebebi de R2 daha önce ebgp den aldığı best rota olark görüyor ve onu advertise ediyordu. Artık best rota olarak görmediği için advertise etmiyor. Peki R2 R1 den öğrendiği rotayı neden R1'e geri duyurmuyor. Bunun sebebide, Bir router ibgp den öğrendiği bir rotayı bir ibgp komşusuna advertise etmemesinden kaynaklı. Bu sebepten ötürüde R1 172.16.10.0/24 networkünü sadece R3'ten yani ebgp den öğrenmiş oluyor.
<img width="650" height="84" alt="image" src="https://github.com/user-attachments/assets/382a329b-e017-4265-afd9-201a02ed54af" /><br>

Wiresharkta BGP Update mesajı ile MED değerinin taşındığını görebiliriz.

<img width="762" height="477" alt="image" src="https://github.com/user-attachments/assets/909eaf43-32bd-46d7-a8c1-632684def060" /><br>


Topolojiye AS200 eklediğimizde MED değerinin taşınmadığını görebilirsiniz.
<img width="957" height="585" alt="image" src="https://github.com/user-attachments/assets/14d7403d-3465-4770-ace9-4b37154ee0ce" /><br>
<img width="710" height="85" alt="image" src="https://github.com/user-attachments/assets/52911428-60f6-4663-a2e0-4991b7875fd4" /><br>

<img width="766" height="296" alt="image" src="https://github.com/user-attachments/assets/adc9508e-bf0c-42a9-b380-7b1062c3a6e8" /><br>
