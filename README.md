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

### Kubernetes Cluster with k3d

Cluster tanimini bir kume olarak dusunursek, calisan makinalae ve nodes'lar bu Kubernetes kumesini olusturur. *kd3* ise herhangi bir teknolojik alette calisabilecek uyumu, hafifligi ve adabtasyonu sunuyor. Bu sebeple orneklerimizi kd3 uzerinden verecegiz.

![alt](kd3_schema.png)

Ilk cluster'imizi, kumemizi olusturmak icin bir komut yeterli olacaktir.

`$ k3d cluster create -a 2`

Yukaridaki komut bize Kubernetes kumesinin 2 nodu oldugunu gosteriyor. Node ise konteynar olarak algilanabilir.

![alt](kd3_cluster.png)

Olusturdugumuz bu Kubernetes kumesini Docker konteynerlarinda da gorebiliriz `$ docker ps`

```
$ docker ps

CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS              PORTS                             NAMES
44a58ff50581   rancher/k3d-proxy:5.3.0    "/bin/sh -c nginx-pr…"   2 minutes ago   Up About a minute   80/tcp, 0.0.0.0:64689->6443/tcp   k3d-k3s-default-serverlb
85252c0acb2e   rancher/k3s:v1.22.6-k3s1   "/bin/k3d-entrypoint…"   2 minutes ago   Up 2 minutes                                          k3d-k3s-default-agent-1
973d87c70ae4   rancher/k3s:v1.22.6-k3s1   "/bin/k3d-entrypoint…"   2 minutes ago   Up 2 minutes                                          k3d-k3s-default-agent-0
1aefdd355f24   rancher/k3s:v1.22.6-k3s1   "/bin/k3d-entrypoint…"   2 minutes ago   Up 2 minutes                                          k3d-k3s-default-server-0
```

Gorulecegi uzere, port numarasi 6443 acik konumda ve "k3d-k3s-default-serverlb" olarak tanimlanmis. Boylece sunucuya baglantimizi bu port uzerinden direkt olarak gerceklestirebiliriz. Makinanin uzerindeki port 64689 ise rastgele secildi.

Clusterlarla interaktif bir sekilde baglanti kurmamizi saglayacak bir baska arac ise **Kubectl**. ***Kubectl*** Kubernetes'in CLI aracidir. k3d ayrica bizim icin Kubectl konfigurasyonunu da yapti ve kullanmaya hazir.

Simdi ise kubectl ile cluster'a , kumeye eriselim. `$ kubectl cluster-info`

![alt](kubectl_info.png)

Bu durum icin gecerli olmazla beraber, kubectl'in 64689 port ile ***k3d-k3s-default-serverlb*** konterynarina baglandigini gorebiliyoruz.

Eger ki cluster'imizi durudurmak veya baslatmak veyahut silmek istersek `$ k3d cluster stop` ve `$ k3d cluster start` ve `k3d cluster delete` komutlari yeterli olacaktir.

![alt](cluster_startstop.png)


### First Deploy

Ilk mevzilendirmemizi yapalim. `$ kubectl create deployment <konteyner_ismi> --image=<imajiniz>`

![alt](kubectl_create.png)

Olusturulan bu objeyle beraber inceleme altina almamiz gereken iki konu var: ***Deployment resource*** ve ***Pod***.

### What is Pod?

Pod icin Kubernetes’in en kücük bilesenidir demek yanlıs olmayacaktir. Kubernetes Cluster’inizda calısan process’leri temsil etmektedir. Pod’lar Kubernetes Cluster’inizin deploy edilebilir birimlerdir. Bu bakıs acısiyla, Docker’a asina iseniz, en kaba haliyle container’a denk geldigini söyleyebiliriz.

![alt](pod_explain.png)

![alt](get_pod.png)

### What is Deployment resource?

Deployment resurce olusturulan deploymentlari , mevzilendirilmeleri gozeten bir yapidir. Kabaca tabir ile Kubernetes'e hangi konteyneri istedigini, nasil calismasi gerektigini ve kac tanesinin calismasi gerektigini soyler.

Deployment'larimizi bu sekilde gorebiliriz:

```
$ kubectl get deployments

NAME                READY   UP-TO-DATE   AVAILABLE   AGE
hashgenerator-dep   1/1     1            1           13m
```
