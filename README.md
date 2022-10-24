# [REStake](https://restake.app) Türkçe Kurulum Rehberi

REStake, delegatorlerin validatorlerine stake ödüllerini yeniden stake etmesine izin vermelerine olanak tanır. REStake validatorlerin bir komut dosyasını çalıştırarak kendilerine verilen delegeleri bulmaları ve restake işlemlerini otomatik olarak yapmalarını sağlar.

REStake, aynı zamanda kullanışlı bir stake etme aracıdır. Ödüllerinizi tek tek ya da toplu olarak telep etmenize ve yeniden stake etmenizi sağlar. Bu da işlem ücretlerinden ve zamandan tasarruf etmenizi sağlar ve daha birçok özellik de planlanmıştır.

🔴 **Burada yapılacak işlemler `mainnet validatorlerinin` yapacağı işlemlerdir.**

🔴 **Kurulum işlemleri `docker` ve `docker compose` kullanılarak yapılmıştır.**

## Sistemi Güncelleme

```bash
apt update && sudo apt upgrade -y
```

## Docker Kurulumu

```bash
curl -fsSL https://get.docker.com -o get-docker.sh 
sh get-docker.sh
```

## Docker Compose Yüklenmesi

```bash
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)
curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### Cüzdan Oluşturma

🔴 **Bu bölümde mainnet validatorünüze ait olan cüzdanın dışında fee ücretlerini ödemek için bir cüzdan oluşturuyoruz.**

🔴 **Oluşturmuş olduğunuz cüzdan ile Cosmos ekosisteminde yer alan tüm cüzdanlarınıza erişim yetkisi vereceksiniz. ReStake aktif eden delegatorlerinizin restake işlemlerinde kullanılacak fee ücretlerini bu cüzdanınızdan ödeyeceksiniz.**


## ReStake Deposunu Forklama(ÖNEMLİ)

[Buradan](https://github.com/eco-stake/restake) ReStake app deposunu forkluyoruz.


### ReStake Deposunu Klonlama ve .env Dosyasını Hazırlama

```bash
git clone https://github.com/eco-stake/restake
cd restake
cp .env.sample .env
```

🔴 **restake klasörünüzde oluşturduğunuz `.env` dosyanıza [yukarıda]() oluşturduğumuz cüzdana ait mnemonicleri yazıyorsunuz.**

### Local versiyonunuzu güncelleme

Güncellemeler her zaman oluyor ve hala düzeltilmesi gereken hatalar var. Sık sık güncelleme yaptığınızdan emin olun.

Yerel deponuzu güncelleyin ve aşağıdaki komutlarla docker konteynırlarınızı önceden oluşturun:

```bash
git pull
docker-compose run --rm app npm install
docker-compose build --no-cache
```

### ReStake'i Çalıştırma

Not: Docker kurulumunuzda root yetkisine sahip değilseniz `sudo` kullanmanız gerekebilir ve bazı docker sürümleri `docker-compose` yerine `docker compose`'u kullanır. Sorunlarla karşılaşırsanız, `docker compose' kullanmayı deneyin.

```bash
docker-compose run --rm app npm run autostake
```

## Validatorunuzu Kaydetme

**REStake operatör bilgilerinizi [Operatörünüzü kaydetme](#operat%C3%B6r%C3%BCn%C3%BCz%C3%BC-kaydetme) bölümünde gösterileceği gibi [Validator Kayıt Defteri](https://github.com/eco-stake/validator-registry)'ne kaydınızı yapana kadar 'operatör olmadığınıza dair bir uyarı görebilirsiniz.**

### Depoyu Forklama(ÖNEMLİ)

[Buradan](https://github.com/eco-stake/validator-registry) Validator Kayıt Defteri'i forklayın.

Forklama işlemi gerçekleştikten sonra kopyaladığınız kendi reponuzda validator adınız olacak şekilde bir klasör açıyoruz. Bu klasörün işerisine iki adet .json dosyası oluşturuyoruz.

#### chains.json Dosyası Oluşturma

Validatorunuze ait mainnet bilgileri ve REStake özelliklerini bu dosyaya giriyoruz. Aşağıdaki dosyayı kendinize göre düzenlemeniz, varsa ekleyeceğiniz başka mainnet validatorleri onları da aşağıdaki dosyaya uygun olarak eklemeniz gerekmektedir.

🔴 **Bu dosyada `name` bölümüne validator adınızı, `address` bölümüne validator adresinizi, ``, `restake` bölümünün altında bulunan `address` kısımıa ise fee ücretini ödeyeceğiniz cüzdan adresinizi giriyorsunuz, `run_time` bölümüne restake işleminin hangi araşıkra yapılacağını yazıyorsunuz, `minimum_reward` bölümüne ise ne kadar ödül biriktiğinde restake edileceğini yazıyorunuz.**

```jsonc
{
  "$schema": "../chains.schema.json",
  "name": "Anatolian Team",
  "chains": [
    {
      "name": "rebus",
      "address": "rebusvaloper.........",
      "restake": {
        "address": "rebus1..................",
        "run_time": "every 1 hour",
        "minimum_reward": 1000000000000000000
      }
    },
    {
      "name": "stride",
      "address": "stridevaloper.......",
      "restake": {
        "address": "stride1...............",
        "run_time": "every 1 hour",
        "minimum_reward": 1000000
      }
    }
  ]
}
```

`address` validator adresinizdir ve `restake.address` ise fee ödemeleri için oluşturduğunuz yeni sıcak cüzdanınızın adresidir.

`restake.run_time` *UTC zaman diliminde* botunuzu çalıştırmayı düşündüğünüz zamandır ve orada birkaç seçenek vardır. Belli bir saat ayarlamak için, ör. `09:00`, UTC zaman diliminde 9am (sabah dokuzda) scripti çalıştırdığınızı belirtir. Birden fazla zaman için bir dizi de kullanabilirsiniz, örneğin `["09:00", "21:00"]`. Saatte/günde birden çok kez için bir aralık dizesi kullanabilirsiniz, örneğin, `"every 15 minutes"`.

`restake.minimum_reward`, otomatik stake'i tetiklemek için asgari ödüldür, aksi takdirde adres atlanır. Bu, daha sık yeniden düzenleme için daha yüksek ayarlanabilir. Bunun temel nominal değer olduğunu unutmayın, Örneğin, `uosmo`.

REStake yapmak istediğiniz tüm ağlar için bu yapılandırmayı tekrarlayın.

`restake.address`'in kullanıcı ara yüzünde delegator'ün restake işlemlerini gerçekleştirmek için vermiş olduğu adrese stake işleminde fee ücretinin alınacağı adres olduğunu unutmayın.

#### profile.json Dosyası Oluşturma

Bu dosyada `name` bölümüne validator adınızı ve `identity` bölümüne [keybase.io](https://keybase.io/)'dan aldığınız PGP parmak izi kimliğinizi giriyorsunuz. Bu, şu anda REStake'de kullanılmıyor, ancak kaydınıza ek açıklamaya yardımcı oluyor.

```jsonc
{
  "$schema": "../profile.schema.json",
  "name": "Anatolian Team",
  "identity": "D06CA33098EDDACE",
  "website": "https://anatolianteam.com",
  "description": "Always forward with the Anatolian Team \ud83d\ude80",
  "contacts": {
    "email": "info@anatolianteam.com",
    "twitter": "anatolianteam"
  }
}
```

#### Pull Request Gönderme

Yukarıdaki işlemleri yapıp validatorumuze ait dosya ve içerisinde iki json dosyasını hazırladıktan sonra şağıdaki resimde görüldüğü gibi `Pull requests` sekmesini açıyoruz. 
![01](https://user-images.githubusercontent.com/102043225/197553389-0fde51af-5ccf-4598-a598-09f288985c52.JPG)
![02](https://user-images.githubusercontent.com/102043225/197553501-178a056c-6339-40c8-bacc-554f582208e6.JPG)
![03 PR](https://user-images.githubusercontent.com/102043225/197553574-34d2a407-6e4f-408b-a11b-0a549b10dc4a.JPG)

Bu sayfada `New pull request` butonuna basarak işlemimizi gerçekleştiriyoruz. Hafta içinde bu işlemi yaparsanız daha çabuk sonuç alırsınız. Merge işlemi yapıldıktan sonra resmi repoda sizin açmış olduğunuz klasör ve içerisindeki dosyalar gözükmeye başlayacaktır.
![03 PR-2](https://user-images.githubusercontent.com/102043225/197553651-12e83bba-f958-43c3-ab03-ff6e5b2cb2a5.JPG)

Bu işlemlerden sonra restake için kalan işlemleri yapmaya devam ediyoruz.

#### Operator Bilgilerinizi Güncelleme

Artık operatör bilgilerinizi oto-sake'i aktif etmek istediğiniz ağları eklemek için [Validator Kayıt Defteri](https://github.com/eco-stake/validator-registry)'ni güncellemeniz gerekiyor. güncellemenizi mümkün olan en kısa sürede merge edilmek üzere pull request isteğinde bulunabilirsiniz. REStake, değişikliklerin birleştirilmesinden sonraki 15 dakika içinde otomatik olarak güncellenir. 

### Zamanlayıcıyı Ayarlama

Burada `systemd-timer` kullanımı gösterilecektir. Systemd-timer, belirtilen kurallarla bir kerelik hizmetin çalıştırılmasına izin verir. Bu yöntem tartışmasız Cron'a tercih edilir.

Sistem zamanınızın doğru olduğundan emin olun ve komut dosyasının UTC'de ne zaman çalışacağını biliyor olmalısınız, çünkü bu daha sonra gerekli olacak. Her iki örnek de saat 21:00'e göre verilmiştir.

#### restake.service Dosyası Oluşturma

Birim dosyası çalıştırılacak uygulamayı tanımlar. `Wants` ve zamanlayıcı ifadesi ile bir bağımlılık tanımlıyoruz.

```bash
tee <<EOF >/dev/null /etc/systemd/system/restake.service
[Unit]
Description=restake service with docker compose
Requires=docker.service
After=docker.service
Wants=restake.timer
[Service]
Type=oneshot
WorkingDirectory=/path/to/restake
ExecStart=/usr/bin/docker-compose run --rm app npm run autostake
[Install]
WantedBy=multi-user.target
EOF
```

Çözüm için benimle zamanını harcayan değerli arkadaşım [Odyseus](https://github.com/odyseus8)'a teşekkür ederim. 

##### systemd timer dosyası oluşturma

Zamanlayıcı dosyası, yeniden düzenleme hizmetini her gün çalıştırma kurallarını tanımlar. Tüm kurallar [systemd dokümanlarında](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) açıklanmaktadır.

```bash
tee <<EOF >/dev/null /etc/systemd/system/restake.timer
[Unit]
Description=Restake bot timer
[Timer]
AccuracySec=1min
OnCalendar=*-*-* 21:00:00
[Install]
WantedBy=timers.target
EOF
```

##### Servisleri Etkinleştirme ve Başlatma

```bash
systemctl enable restake.service
systemctl enable restake.timer
systemctl start restake.timer
```

##### Zamanlayıcınızı kontrol etme

`systemctl status restake.timer`
<pre><font color="#8AE234"><b>●</b></font> restake.timer - Restake bot timer
     Loaded: loaded (/etc/systemd/system/restake.timer; enabled; vendor preset: enabled)
     Active: <font color="#8AE234"><b>active (waiting)</b></font> since Sun 2022-03-06 22:29:48 UTC; 2 days ago
    Trigger: Wed 2022-03-09 21:00:00 UTC; 7h left
   Triggers: ● restake.service
</pre>

`systemctl status restake.service`
<pre>● restake.service - stakebot service with docker compose
     Loaded: loaded (/etc/systemd/system/restake.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2022-03-08 21:00:22 UTC; 16h ago
TriggeredBy: <font color="#8AE234"><b>●</b></font> restake.timer
    Process: 86925 ExecStart=/usr/bin/docker-compose run --rm app npm run autostake (code=exited, status=0/SUCCESS)
   Main PID: 86925 (code=exited, status=0/SUCCESS)
</pre>

🔴 **Not: Sorun yaşarsanız `WorkingDirectory=/path/to/restake` bölümünü `WorkingDirectory=/root/restake` olarak değiştiriniz. Eğer yine sorun yaşarsanız `chmod 777 /root/restake` komutu ile dosyaya okuma, yazma ve çalıştırma izni veriniz. Daha sonra `systemctl daemon-reload` yaptıktan sonra sistemi yeniden başlatınız. **

🔴 **Eğer `Failed to restake service with docker compose` gibi bir hata alırsanız yine `chmod 777 /usr/bin/docker-compose` komutu ile dosyaya okuma, yazma ve çalıştırma izni veriniz.**

### İzleme

Her ağ için komut dosyası durumunu bildirmek için REStake oto-stake betiği [healthchecks.io] (https://healthchecks.io/) ile entegre olabilir. [HealthChecks.io] (https://healthchecks.io/) daha sonra, herhangi bir arızayı bildiğinizden emin olmak için e -posta, Discord ve Slack gibi birçok bildirim platformuyla entegre edilebilir.

Yapılandırıldıktan sonra, komut dosyası başladığında, başarılı ya da başarısız olduğunda REStake [healthchecks.io](https://healthchecks.io/)'a ping atacaktır. Kontrol günlüğü ilgili hata bilgilerini içerecektir ve yapılandırılması basittir.

Komut dosyasını çalıştırdığınız her ağ için bir kontrol ayarlayın ve beklenen programı yapılandırın. Örneğin, her 12 saatte bir Osmosis kontrolü ekleyin, Akash için her 1 saatte bir vb.

Kontrol UUID numaranızı aşağıdaki gibi `networks.local.json` yapılandırma dosyanızda ilgili ağa ekleyin. İsteğe bağlı olarak [HealthChecks.io platformunu kendi hostinginizde barındırmak](https://healthchecks.io/docs/self_hosted/) istiyorsanız. `address` özniteliğini de ayarlayabilirsiniz.

```JSON
{
  "osmosis": {
    "healthCheck": {
      "uuid": "77f02efd-c521-46cb-70g8-fa5v275au873"
    }
  }
}
```

### Kullanıcı Arayüzünü (UI) Çalıştırma

Docker'ı kullanarak kullanıcı arayüzünü bir satırla çalıştırın:

```bash
docker run -p 80:80 -t ghcr.io/eco-stake/restake
```

`docker-compose up` kullanarak kaynaktan alternatif çalıştırma da yapabilirsiniz.
