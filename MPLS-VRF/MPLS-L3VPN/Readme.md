## MPLS-L3VPN
<img width="1201" height="780" alt="image" src="https://github.com/user-attachments/assets/fb209ae4-3f62-4b74-8375-059b6d6ff8ad" />


Daha önce yaptığımız mpls basic ve vrf-lite teknolojilerini MP-BGP ile birleştrirerek tek bir mpls omurga üzerinden birden fazla müşterinin prefixlerini izole biçimde taşımasını sağlayan bir teknolojidir.

- Her müşteri aynı prefixleri kullanabilir örneğin 10.0.0.0/8 vb. 
- Müşterilerin prefixleri P(Provider Core) routerlarında tutulmaz
- İHer müşteri kendi VRF'inde taşındığı için izolasyon sağlar
- CE cihazları MPLS veya vrf bilmez standart ip forwarding yaparlar.

## Config 
### PE cihazlarda
- VRF ile her müşterinin ayrı routing tablosu oluşturulur.
- Prefixlerin başına RD eklenir. Bunun amacı aynı prefix geldiğinde BGP ile yönlendirme yapıldığında benzersiz olmalarını sağlamak
- Format genelde ASN:nn şeklinde olur
- RT etiketi ile rotaların hangi vrf'lere export/import edileceği belli edilir


### P cihazlarda 
- Sadece MPLS ve IGP çalıştırılır. Bu labda opsf tercih edilmiştir.
- Amaç MPLS ile PE ler arasındaki routingi ip yerine label ile sağlamak.


### CE Cihazlarda
- Müşteri tarafındaki cihaz olduğundan sadece PE ile statik veya dinamik routing yapılır. Bu labda eBGP tercih edilmiştir.
- Amaç CE arkasındaki prefixleri PE'ye duyurmak.

## BGP ile RD ve RT etiketlerinin taşınması
PE1 ve PE2 arasında update ile müşteri prefixleri taşınırken RD aynı prefixlerin çakışmaması için kullanılır.  RT ise bu rotaların hangi VRF'lere gireceğini belirler. Bu yüzden VRF'lerde tek RD olurken, birden fazla RT olabilir.
Bu etiketler BGP Update ile gönderilir. 


<img width="860" height="714" alt="image" src="https://github.com/user-attachments/assets/4cf1f5fc-3682-4ca8-87cc-f5fb56a7df1f" /><br>
RD ve RT etiketleri sadece control planede sadece MP-BGP kullanır, data plane de yani icmp data paketinde bunlar görülmez.<br>

172.16.10.0/24 prefixi bgp update ile taşınırken 65000:100:172.16.10.0/24 şeklinde komşusuna advirtese edilir. Bu formata VPNv4 NLRI denmektedir.

Alttaki RT etiketi ile bu rotayı hangi VRF'ler alacak ise onlara bu rotayı verir. Yani hangi vrflerden import edileceğini belirtir.

RD forwardinge etki etmez sadece prefixleri benzersiz yapmak için kullanılır.


## Ping attığımızda
PC1 den PC2'ye, PC3 den PC4'ün gateway'ine ping atalım.

PC1 de `ping 172.16.20.10`
PC3 de `ping 172.16.20.1`
atalım ki wiresharkta paketleri rahat tespit edebilielim.

PC1 ve PC2 Customer-A içerisinde, PC3 ve PC4 Customer-B içerisinde.


PE1 den çıkan hatta wiresharkta yakaladığımız paketler

PC1'in icmp paketi PE1 den çıktığında: iki adet MPLS label paketi almış. 

Outer Label: 19 etiketli olan mpls header, bu LDP den aldığı etiket ve MPLS hattında forwarding için kullanılır
İnner Label : 21 etiketli olan mpls header, bu MP-BGP den aldığı etiket ve PE'lere gelen paketlerde bu etiket sayesinde paketin hangi VRF'ye ait olduğu belli olur.

<img width="798" height="138" alt="image" src="https://github.com/user-attachments/assets/193b5beb-a171-4a42-af31-75abc519e151" /><br>
19 etiketli paket PE1 de LDP gelmektedir.  <br>

PC1'den çıkan paket PE1'e VRF Customer-A interface'inden girdi ve bu vrf'nin bgp tablosunda dst 172.16.20.0/24 için next hop 6.6.6.6 olarak görülmekte

<img width="621" height="205" alt="image" src="https://github.com/user-attachments/assets/701360f5-e35d-4a3d-b93a-796545a7c38e" />

Mpls forwarding table da ise 6.6.6.6/32 için outgoin label 19 görür ve 19 etiketiyle 10.0.10.2'ye(P1) göndermesini söyler.

<img width="738" height="258" alt="image" src="https://github.com/user-attachments/assets/ba48f8c8-a267-4670-8e71-8eb0c8ffab2a" />

Bu paket PE2'ye geldiğinde  19 etiketi bir önceki olan P2  routerda kaldırıldığı için sadece 21 etiketi kalmıştır.
21 etiketine bakıldığında outgoing interface gi 0/0 olarak görüldüğünden bu interface üzerinden gönderir.Label olarak da no label yazdığından herhangi bir etiket basmaz

<img width="744" height="256" alt="image" src="https://github.com/user-attachments/assets/f4f2d64e-91de-4349-ae0c-308d371cff48" />

V] ifadesi VRF route olduğunu gösterir.


PC3 ten PC4'ün gatewayine attığımız pingde  
MPLS forwardig için 19 etiketi almışken, 

<img width="773" height="145" alt="image" src="https://github.com/user-attachments/assets/1dd56b36-b185-47a5-adf8-ebf93c9ec8e1" />

MP-BGPden 22 etiketini almıştır. PE2, PE1'e  172.16.20.0/24 için 22 etiketi ile gelmesini söyler.

<img width="509" height="138" alt="image" src="https://github.com/user-attachments/assets/8f83923b-019d-459a-9d10-e6063120fb9a" />


Customer-A için MP-BGPden 21 etiketini almıştır. PE2, PE1'e  Customer-A daki 172.16.20.0/24 için 21 etiketi ile gelmesini söyler.

<img width="506" height="155" alt="image" src="https://github.com/user-attachments/assets/5090a137-ec6b-4e73-9d32-baa4e00062f3" />

### İZolasyon kontrolü

PC1 PC5'e ping atabilirken, PC3, PC5'e ping atamamaktadır.

<img width="581" height="188" alt="image" src="https://github.com/user-attachments/assets/0596b299-2626-4296-8df7-81529bdd8773" />
<img width="365" height="184" alt="image" src="https://github.com/user-attachments/assets/4a3cfa95-73d5-4405-a839-559edf148336" />


Bu lab ile;

- Aynı MPLS omurga üzerinden **iki farklı müşterinin (Customer-A / Customer-B) izole taşınması**
- Müşterilerin **çakışan IP prefixleri** (`172.16.10.0/24`) kullanabilmesi
- **MP-BGP** ile RD/RT etiketlerinin taşınması
- **Çift etiket (Outer/Inner)** mekanizmasının Wireshark ile gösterilmesi

Bu yapı, servis sağlayıcıların **ölçeklenebilir ve güvenli VPN hizmeti** sunmasını sağlar.









