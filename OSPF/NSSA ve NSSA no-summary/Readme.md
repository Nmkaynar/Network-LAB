#  NSSA ve NSSA no-summary

<img width="1098" height="566" alt="image" src="https://github.com/user-attachments/assets/7bb1863f-a639-4ecf-9b23-2df7e1811596" /><br>


### NSSA

Stub areada external rotaların(Type 5 LSA) nasıl engellendiğini stub area lab çalışmasında incelemiştik. Bu lab çalışmamızda ise NSSA yı inceleyeceğiz. Stub arealarda doğası gereği ASBR router bulunduramaz. Yani Stub area içerinden OSPF topolojisine external rota enjekte edemez. 

### NSSA Ne Zaman Kullanılır?

Topoloji de eğer hem external rotalar gelmesin aynı zamanda external rota duyuralım istiyor isek ospf area'sını Stub area olarak değil NSSA olarak yapılandırmalıyız.

### Config

``Router ospf <process id >`` altında ``area <area id> nssa`` şeklinde yapılandırılır



 Topoloji de R1 Router'ı ``192.168.1.0/24`` networkünü external olarak duyuracak.  Bu işlem sonrası R3 ve R4 üzerinde ``show ip ospf database`` ve ``show ip route ospf | begin Gate``  çıktısı aşağıdaki gibidir.

<img width="1630" height="768" alt="image" src="https://github.com/user-attachments/assets/f1009ce7-77e4-4162-91b6-3606bca922b8" /><br>

### Önce Area 1'i Stub yapalım.

Bunun için hem ABR router olan R2 de hem der R3 de ``router ospf 1`` altında `area 1 stub`  komutlarını etkinleştirmeliyiz.

Area 1 stub olunca ABR router area 1'e Type 5 LSA'ları girmesini engelledi ve external rota yerine deafult rota enjekte etti.<br>
Bunu LSA 3 ile duyurduğundan rouiting table da O*IA şeklinde görülür.
<img width="715" height="591" alt="image" src="https://github.com/user-attachments/assets/b67ba6be-9181-408f-a37d-0ff024529813" /><br>

Peki stub area olduğunda external rota duyurabilir miyiz? Hayır. <br>
R3 te 172.16.1.0/24 networkünü external olarak duyurmak istediğimiz de hata alıyoruz.
<img width="916" height="112" alt="image" src="https://github.com/user-attachments/assets/da01141a-c371-4509-b881-e19a8904dd81" /><br>

Bunu yapabimek için area 1'i Stub Area yerine NSSA olarak yapılandırmalıyız. Bunu da ``router ospf 1`` altında ``area 1 nssa`` komutunu etkinleştirmeliyiz.<br>


Area 1 NSSA olarak yapılandırıldığında hem ABR routerdan external rota yani TYPE LSA 5'leri almaz. Aynı zamanda external rotaları TYPE 7 LSA olarak duyurur.<br>
ABR router ise TYPE 7 LSA'ları TYPE 5 LSA olarak diğer arealara duyurur. 
Aşağıdaki çıktıda ASBR olan R3 router ``172.16.1.0/24`` networkünü TYPE 7 LSA olarak duyurur. ABR router(R2) ise diğer arealara bu networkü TYPE 5 LSA olarak duyurur.

R3 ve R4 çıktılarında bunu görebiliriz.

<img width="1755" height="819" alt="image" src="https://github.com/user-attachments/assets/7d1a67b9-6da2-4853-98dc-9f719c9760bf" /><br>


NSSA olan Area 1'e  default route enjekte etmek istiyor isek ABR Router'da (R2) ``area 1 nssa default-information-originate`` komutunu etkinleştirmeliyiz.
<img width="568" height="34" alt="image" src="https://github.com/user-attachments/assets/a177f233-237e-4efb-a9bc-eea203cf2560" /><br>
Komut aktif ettikten sonra R3'de default rota TYPE 7 LSA olarak duyurulur ve routing table O*N2 olarak gösterilir.

<img width="750" height="684" alt="image" src="https://github.com/user-attachments/assets/188c9a89-c62a-4eb4-8db4-b3e2b30544b2" />


## NSSA no-summary

Eğer inter area'lardan gelen rotalarıda engellemek sadece tek bir default rota duyrulmasını istiyor isek NSSA'yı ABR routerda no-summary ile yapılandırmalıyız.

### Config 
ABR olan R2 Router<br>
<img width="425" height="37" alt="image" src="https://github.com/user-attachments/assets/0f1d82b2-145c-45dd-84c8-844b46088ed7" />

Config sonrası ABR router gelen tüm LSA 3 rotalarınıda engelleyip Area 1'e sadece LSA 3 ile default rotayı duyurdu.
LSA 3 ile duyurduğu için routing table da ``O*IA`` şeklinde, Yukarıda LSA 7 ile duyurduğunda ``O*N2`` şeklinde gözüküyordu.

<img width="751" height="587" alt="image" src="https://github.com/user-attachments/assets/72993def-680f-4640-a7f7-e75b6b0b6b8d" />


















