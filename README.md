# Kubernetes-Learning
Personal achievement for learning and improving my knowledge with Kubernetes


## What is Kubernetes?

Konteynerlanmis is yukunu ve servisleri yonetmeyi saglayan  otomasyon ve configure etmemize olanak saglayan, acik kaynakli portatif ve gelistirilebilir bir platformdur Kubernetes.

***Traditional deployment era***:Geleneksel aplikasyon dagitim seruveninde fiziksel serverlar kullanilirlirdi. Fiziksel sunucudaki uygulamalar icin kaynar sinirlarini tanimlamanin bir yolu yoktu ve bu durum kaynak tahsil etme sorunlarina neden oluyordu. Ornek verecek olursak, fiziksel bir serverda birden fazla uygulamayi calistiriyorsak, belki de bir uygulama digerlerinden daha fazla kaynak kullanacak ve diger uygulamalar performanslarinin altinda kalacaklar. Her bir uygulamayi farkli serverlarda calistirarak buna cozum getirebiliriz. Ancak bu cozum maliyet ve yuksek performans gerektirdigi icin mantikli olmayacaktir.


***Virtualized deployment era***: Yukaridaki bu durum icin virtualization (sanallastirma) ortaya sunuldu. Bir tane fiziksel server'in CPU'su ile birden vazla sanal makinalarin (VMs) calismasi olanagi kilindi. Ayrica bu virtualization yani sanallastirma diger sanal makinalarin birbirinden izole olmasina olanak sagliyor ve bir uygulamanin diger bir uygulama tarafindan erisim saglamasina izin vermeyerek guvenlik onlemi de aliyor. Bunun yani sira, kolayca maintain yani tamir edilmesi, genel maliyeti dusurmesi, guncellenmesinin kolayligi ve dahasi sanallastirmayi geleneksel metoda nazaran daha cazip kildi.

***Container deployment era***: Konteyner aplikasyon dagitim sistemi ise VMs modeline cok benziyor. Ek olarak bu method, izole ozelliklerini esneklestirerek, uygulamalar arasinda isletim sistemi paylamisina olanak sagliyor. VM'lere benzer olarak konteynerlar kendi filesystem'lari var ve CPU, memory ve isletim sistemini vs palyasirlar.

Konteynerlarin sahip olduklari artilardan bazilari:
- VM'lere kiyasla imaj alimi daha kolaydir.
- Imaj'larin degismezligi sebebiyle guvenilir ve surekli gelisim ve entegrasyon halinde.
- Dev ve Ops is bolumunun endiselerinin ayrilmasinda rol oynar. Mesela derleme ve yayin zamani ile deployment dagitim zamani konulari gibi.
- Yalnizca isletim sistemi duzeyindeki bilgileri veyahut olcumleri degil de uygulama durumunu ve sinyalleri de ortaya cikarir.

Konteynerlar uglulamari calistirmak icin iyi bir secenek. urun/aplikasyon ortaminda uygulamalari calistiran konteynerlari yonetmek gerekir ve herhangi bir kesinti olmadigindan emin olunmasi gerekir. Mesela, bir konteynar cokerse, bir baska konteynerin baslamasi gerekir. **Kubernetes** bu asamada bize bu durumun sistem tarafindan framework ile idare edilmesi sagliyor. Bilgisayar agindaki akista bir sorun oldugunda ya da olceklendirme durumunda deployment dagitim yollari sunarak olayi ele alir. 
