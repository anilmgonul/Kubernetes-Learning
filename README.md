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

### Declarative configuration with YAML

Daha onceden deployment'imizi olusturmustuk.

`$ kubectl create deployment hashgenerator-dep --image=jakousa/dwk-app1`

Eger dilersek olusturdugumuz bu deployment'i istedigimiz gibi olceklendirebiliriz. Replica sayilarini belirleyebilir ve guncellenmesini saglayabiliriz.

```COMMAND
$ kubectl scale deployment/hashgenerator-dep --replicas=4

$ kubectl set image deployment/hashgenerator-dep dwk-app1=jakousa/dwk-app1:b7fc18de2376da80ff0cfc72cf581a9f94d10e64
```

Deklare ederken, duzenlenme seklini belirleyebiliriz. Bunun orneklerinden bir tanesi ise ilusturulan konteyneri silmek:

```COMMAND
$ kubectl delete deployment hashgenerator-dep
  deployment.apps "hashgenerator-dep" deleted
```

Bu kisimda yeni bir klasor olusturalim `manifests` adinda ve icinde `deployment.yaml` dosyasi bulunsun. Dosyanin icerigi ise asagidaki gibi olsun.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hashgenerator-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hashgenerator
  template:
    metadata:
      labels:
        app: hashgenerator
    spec:
      containers:
        - name: hashgenerator
          image: jakousa/dwk-app1:b7fc18de2376da80ff0cfc72cf581a9f94d10e64
```  

Goruldugu uzere ***docker-compose*** dosyalarina cok benziyor.

* *kind:Deployment* bize iceriginin ne tur olddugunu deklare ediyor.
* *name:hashgenerator-dep* ise metadatanin ismini deklare ediyor.
* *replicas:1* kac tane replica oldugunu deklare ediyor.
* Genel olarak ise ismi olan bir imaj'dan konteynar oldugunu da deklare ediyoruz.

Bu deployment'i uygulamaya sokmak icin ise:

```COMMAND
$ kubectl apply -f manifests/deployment.yaml
  deployment.apps/hashgenerator-dep created
```        

![alt](yaml_file.png)

### Introduction to Debugging

Kubernetes "self-healing" dedigimiz kendi kendi onarma ozelligine sahip. Eger pod'larda veyahut konteynerlarda bir seyler ters giderse biz developerlarin is yuku Kubernetes sayesinde azalmis durumda. Mudahele edilmesi gereken yerlerde veyahut con problemleri yasadigimiz zamanlar, muhtemel tum senaryolari teker teker elemine etmek gerekir. Bunu sistematik bir sekilde saglayabilmemeiz icin ise anahtar olgu her seyi sorgulamaktir. Bu sorgulamalari Kubernetes'te `kubectl describe` ve `kubectl logs` komutlarini kullanarak yapabiliriz.

- `kubectl describe` bize kaynak hakkinda hemen hemen her seyi soyler.
- `kubectl logs` ise yazilimda problem olusturan kisimlar hakkkinda bilgi verip, kolay takip etmemizi saglar.


Oncelikle farkli bir yontemle (github'dan cekerek) bir ke daha deployment yapalim:

```COMMAND
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-hy/material-example/master/app1/manifests/deployment.yaml
  deployment.apps/hashgenerator-dep unchanged
```  

Daha sonrasinda ise **describe** komutumuzu kullanalim:

```COMMAND
$ kubectl describe deployment hashgenerator-dep
```

![alt](describe.png)

Buun yani sira, **describe** komutunu daha spesifik olarakta kullanabiliriz. Ornegin, *pod*'lar' hakkinda bilgi almak icin.

`$ kubectl describe pod hashgenerator-dep-548d4d6c8d-mrnsl`

![alt](pods.png)


Simdi ise *logs*'larimizi kontrol edebilriz.

`$ kubectl logs hashgenerator-dep-548d4d6c8d-mrnsl`

![alt](logs.png)

Son asamada ise [Lens "The Kubernetes IDE"](https://k8slens.dev/) Kubernetes platformunu kullanarak grafik display metoduyla da incelemede bulunabiliriz.

![alt](workloads_overview.png)

![alt](workloads_pods.png)


### Introduction to Networking

Biliyoruz ki Kubernetes deploymentlari yonetmek ve otomatik hale getirmekte kullandigimiz acik kaynak platformdur. Bunun yani sira, cluster yani kumemiz icerisinde konteynerlarin bakimini yapabilme, zaman cizelgesini takip etme, ve operasyonu yurutmede olanak sagliyor.

Bunun otesinde, Kubernetes networking , ag yapilari, kubernetes bilesenlerinin kendi aralarinda ve bir baska uygulamayla iletisim haline gecmesine olanak sagliyor. Kubernetes platformu, ana bilgisayar baglantı noktalarini konteynar baglanti noktalarina esleme ihtiyacini ortadan kaldiran duz bir ag yapisina dayandigindan diger ag platformlarindan farklidir.

Bu asamada, basit bir networking uygulamasinin HTTP server'ina baglanmasini gorecegiz. Bir port secilmesi gerekiyor ve port 3000'den default olarak yararlanacagiz.

```COMMAND
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-hy/material-example/master/app2/manifests/deployment.yaml
  deployment.apps/hashresponse-dep created
```

Bu kisimda, cluster'in disindan baglanmayi deneyecegiz. Olusturdugumuz *hashresponse-dep* pod'unu calistigini ise `port-forward` komutu ile gorebiliriz.

```COMMAND
$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS        AGE
hashgenerator-dep-548d4d6c8d-mrnsl   1/1     Running   1 (2m17s ago)   8h
hashresponse-dep-869df48685-pshdz    1/1     Running   0               46s
```

`$ kubectl port-forward hashresponse-dep-869df48685-pshdz 3003:3000`

![alt](port_forward.png)

Sonrasinda ise, baglantimizi [http://localhost:3003](http://localhost:3003/) adresinden gorebiliriz.

Cluster'imizi docker'in icinde k3d ile calistirdigimiz icin, yapmamiz gereken bir kac hazirlik var. Pod'a cluster'in disindan acilan port yeterli olmayacak, eger cluster'a konteynerin icinden erismek istiyorsak.

Oncelikle konteynerlarimizi gorelim. `$ docker ps`

![alt](k3d_docker_ps.png)

Isin guzel yani, k3d bizlere API'ye baglanmamiz icin bizlere bir port hazirladi ve bu port 6443 ve ek olarak port 80'e baglanabiliriz. local 8081 to 80 in k3d-k3s-default-serverlb and local 8082 to 30080 in k3d-k3s-default-agent-0 acilmis olacak.

`$ k3d cluster delete`
`$ k3d cluster create --port 8082:30080@agent:0 -p 8081:80@loadbalancer --agents 2`
`$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-hy/material-example/master/app2/manifests/deployment.yaml`

![alt](k3d_ports_open.png)
