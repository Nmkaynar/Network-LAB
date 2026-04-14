# OSPF Multi-Area ve Virtual-Link Lab
Bu lab çalışması fiziksel olarak backbone alanına bağlı olmayan uzak alanların Virtual-Link kullanılarak nasıl entegre edileceğini göstermektedir.

## Proje Amacı ve Senaryo:
- Area 0 (BackBonne): Merkeze konumlandırılmıştır.  
- Area 10 ve 20: Standar operasyonel alanlardır.
- Area 30: Fiziksel olarak Area 0 ile bağlantısı olmayan,Area 20 üzerinden sarkan bir yapıdır.  

### Karşılaşılan Temel Problem
Buradaki temel problem, OSPF kuralları gereği tüm alanların Area 0'a fiziksel bağlı olma zorunluluğudur. Area 30'un buizolasyonunu aşmak için R3 ve R5 arasındabir Virtual-Link tüneli kurgulanmıştır.

### Topoloji

<img width="818" height="677" alt="image" src="https://github.com/user-attachments/assets/c084c496-1068-453c-a7bb-63e525d1db1b" />

