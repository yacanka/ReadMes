# LogRotate

Logrotate, Linux tabanlı işletim sistemlerinde, özellikle Ubuntu gibi dağıtımlarda kullanılan bir günlük dosyası yönetim aracıdır. Logrotate, sistem günlük dosyalarını düzenli aralıklarla döndürme (rotate), eski günlük dosyalarını sıkıştırma (compress) ve gerektiğinde eski günlük dosyalarını temizleme (delete) işlemlerini otomatik olarak gerçekleştirir. Bu, disk alanını yönetmek, günlük dosyalarını düzenli tutmak ve sistem performansını artırmak için önemli bir araçtır.

## Konfigurasyon
Öncelikle, logrotate'ı yapılandıracağınız dosyayı oluşturulmalıdır. Zaten böyle bir dosya varsa düzenlenebilir de. `/etc/logrotate.d` dizini altında bir dosya oluşturalım. `/etc/logrotate.d/myapp` içine aşağıdaki şekilde yazarak konfigurasyon yapmak mümkündür:
```
/path/to/your/logfile {
    rotate 7
    daily
    missingok
    notifempty
    compress
    delaycompress
    create 0644 <username> <groupname>
}
```
> `/path/to/your/logfile` yerine log dosyasının tam adresi yazılmalıdır.

## Kullanım
Logrotate aracı her şey yolunda gidiyorsa **inactive** durumdadır. Bir ajan gibi arka planda sürekli çalışmaz. Tetiklenmesi gerekir. Logrotate tetikleyicisi, timer ya da cron (zamanlanmış görev) olabilir.

Manuel olarak tetiklemek istiyorsanız bu komutu girin.
```
sudo logrotate /etc/logrotate.conf
```

### Timer
Eğer sistemde Logrotate mevcutsa, logrotate.timer zamanlayıcısı da bulunur. Zamanlayıcıyı kullanabilmek için onun aktif olduğundan emin olmalısınız.
```
sudo systemctl status logrotate.timer
```

Zamanlayıcı aktif ise logrotate zamanlayıcısının çalışma tanımını görmek için bu komutu girin. Bu logrotate.timer için bilgi almamızı sağlayacak.
```
sudo systemctl list-timers | grep logrotate
```

logrotate.timer standart olarak günlük çalışır. Zamanlayıcı özelliklerini değiştirmek için `/lib/systemd/system/logrotate.timer` dosyasına girip kodu değiştirebilirsiniz.
Örnek olarak, eğer zamanlayıcı belirli bir saatte çalıştırılmak isteniyorsa bu konfigurasyon kullanılabilir.
```
sudo nano /lib/systemd/system/logrotate.timer
```
```
[Unit]
Description=Daily rotation of log files
Documentation=man:logrotate(8) man:logrotate.conf(5)

[Timer]
OnCalendar=*-*-* 15:00:00
AccuracySec=1m
Persistent=true

[Install]
WantedBy=timers.target
```
> `AccuracySec` zamanlayıcının hassasiyetini belirler. Maksimum duyarlılık için `1us` değeri verilebilir.

Yapılan değişiklerin uygulanması için bu komutları girin.
```
sudo systemctl daemon-reload
sudo systemctl restart logrotate.timer
```

### Cron
Cron, Ubuntu ve diğer Linux tabanlı işletim sistemlerinde zaman tabanlı bir görev planlama sistemidir. Logrotate aracını tetiklemek için bu da bir yöntemdir.
Cron zamanlanmış görevlerini `/etc` dizini altında tutar. İsimleri şu şekildedir:
- cron.d/
- cron.daily/
- cron.hourly/
- cron.monthly/
- cron.weekly/

Örnek olarak; logrotate saatlik çalıştırılmak isteniyorsa, `cron.daily` dizini altında logrotate isimli bir dosya oluşturulup aşağıdaki bash kodları yazılabilir.
```bash
#!/bin/sh

# skip in favour of systemd timer
if [ -d /run/systemd/system ]; then
    exit 0
fi

# this cronjob persists removals (but not purges)
if [ ! -x /usr/sbin/logrotate ]; then
    exit 0
fi

/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit $EXITVALUE
```

## Örnek kullanımlar

### Logrotate konfigurasyonu
Logrotate konfigurasyonunu yapın.
```
nano /etc/logrotate.d/mico
```
Konfigurasyon yönergelerini ekleyin.

#### Logrotate yönerge örneği 1
```
/opt/mico-client/output.log {
# rotate if log grows bigger then 20M bytes
    size 20M
# old versions of log files are compressed
    compress
# log files are rotated 30 times before being removed (Max 30 old log file)
    rotate 30
# if the log file is missing, go on to the next one without an error message.
    missingok
# Truncate the original log file in place after creating a copy,
# instead of moving the old log file and optionally creating a new one
    copytruncate
}
```
- output.log 20 MB boyutuna ulaştığında logu rotate eder.
- Eski logları sıkıştırır.
- En fazla 30 adet log dosyasını rotate eder. Son rotate edilenler en sonki logların silinmesine sebep olur.
- Eğer log dosyası eksikse, hata mesajı olmadan bir sonrakine geçer.
- Orijinal log dosyası sıfırlanır, bütün içerik yeni bir dosyaya yedeklenir.

#### Logrotate yönerge örneği 2
```
/opt/mico-client/output.log {
# rotate daily
    daily
# old versions of log files are not compressed
    nocompress
# log files are rotated 5 times before being removed (Max 5 old log file)    
    rotate 5
# if there is nothing in the log file, it will not rotate       
    notifempty
# remove rotated logs older than 10 days.   
    maxage 10         
}
```
- output.log günlük olarak rotate edilir.
- Eski logları sıkıştırmaz, aynı formatta tutulur.
- En fazla 5 adet log dosyasını rotate eder. Son rotate edilenler en sonki logların silinmesine sebep olur.
- Log dosyasında hiçbir şey yoksa, rotate etmez.
- Eski loglar 10 gün bekledikten sonra silinir.

#### Logrotate yönerge örneği 3
```
/opt/mico-client/output.log {
# rotate monthly
    monthly
# rotate if log grows bigger then 2G bytes
    minsize 2G
# log files are rotated 0 times before being removed (Max 0 old log file)
    rotate 0
}
```
- output.log aylık olarak rotate edilir.
- Log dosyasının rotate edilebilmesi için en az 2 GB boyutu olmalıdır.
- Eski logları tutmaz, output.log sıfırlanır.
> minsize yönergesinin size yönergesinden farkı, minsize yönergesi daily, monthly... yönergelerine bağımlıdır. Bu örnekte yeni aya girilmeden, dosya boyutu geçilmiş olsa bile rotate etmez.

> Önemli: Eğer önce `daily`, `weekly`, `monthly` ve `yearly` yönergelerinden biri; sonra `size` yönergesi gelirse, önceki yönerge göz ardı edilecek ve `size` yönergesi log dosyasına uygulanacaktır.
> Benzer şekilde önce `size` yönergesi, sonra `daily`, `weekly`, `monthly` ve `yearly` yönergeleri kullanıldığında, `size` yönergesi göz ardı edilecektir.

### Logrotate.timer konfigurasyonu
Zamanlayıcıyı ayarlayın.
```
nano /lib/systemd/system/logrotate.timer
```
Timer ayarlarını ekleyin.

#### Timer örneği 1
```
[Unit]
Description=15 min timer rotation of log files

[Timer]
OnCalendar=*:0/15
AccuracySec=2h
Persistent=true

[Install]
WantedBy=timers.target
```
- Zamanlayıcı 15 dakikada bir tetiklenir.
- 2 saatlik hassasiyet ile çalışır.
- Zamanlayıcı, kaçırılan çalışma sürelerini yakalayarak sonraki uygun çalışma zamanında çalışır.

#### Timer örneği 2
```
[Unit]
Description=Specific daily timer rotation of log files

[Timer]
OnCalendar=*-*-* 10:00:00
AccuracySec=1us
Persistent=true

[Install]
WantedBy=timers.target
```
- Zamanlayıcı her gün saat 10.00'da tetiklenir.
- 1 mikrosaniye hassasiyet ile çalışır.
- Zamanlayıcı, kaçırılan çalışma sürelerini yakalayarak sonraki uygun çalışma zamanında çalışır.

#### Timer örneği 3
```
[Unit]
Description=Daily timer rotation of log files

[Timer]
OnCalendar=daily
AccuracySec=12h
Persistent=false

[Install]
WantedBy=timers.target
```
- Zamanlayıcı her gün saat 00.00'da tetiklenir.
- 12 saatlik hassasiyet ile çalışır.
- Zamanlayıcı sistemde beklenen zamanda çalışmazsa, bunu telafi etmeye çalışmaz.

> OnCalendar detayları için: [Systemd timers onCalendar](https://silentlad.com/systemd-timers-oncalendar-(cron)-format-explained)

Değişikliklerin etkinleştirilmesi için zamanlayıcıyı yeniden başlatın.
```
sudo systemctl daemon-reload
sudo systemctl restart logrotate.timer
```

## Yönergeler
Logrotate yapılandırma dosyasına dahil edilebilecek yönergeler hakkında daha fazla bilgiyi burada bulabilirsiniz.
> Bütün yönergeler dahil edilmemiştir. Sık kullanılacaklar eklenmiştir.

### create *mode owner group*
Döndürme işleminden hemen sonra (postrotate betiği çalıştırılmadan önce) günlük dosyası oluşturulur (döndürülen günlük dosyasıyla aynı adla). mode günlük dosyasının modunu sekizli olarak belirtir (chmod(2) ile aynıdır), owner günlük dosyasına sahip olacak kullanıcı adını belirtir ve group günlük dosyasının ait olacağı grubu belirtir. Günlük dosyası özniteliklerinden herhangi biri atlanabilir, bu durumda yeni dosya için bu öznitelikler, atlanan öznitelikler için orijinal günlük dosyasıyla aynı değerleri kullanacaktır. Bu seçenek nocreate seçeneği kullanılarak devre dışı bırakılabilir.

### size *size*
Günlük dosyaları yalnızca boyut baytından daha fazla büyüdüklerinde döndürülür. Eğer size'ı k takip ediyorsa, boyutun kilobayt cinsinden olduğu varsayılır. M kullanılırsa, boyut megabayt cinsindendir ve G kullanılırsa, boyut gigabayt cinsindendir. Yani size 100, size 100k, size 100M ve size 100G'nin hepsi geçerlidir.

### compress
Günlük dosyalarının eski sürümleri varsayılan olarak gzip(1) ile sıkıştırılır. Ayrıca bkz. nocompress.

### delaycompress
Önceki günlük dosyasının sıkıştırılmasını bir sonraki döndürme döngüsüne erteler. Bu yalnızca compress ile birlikte kullanıldığında etkili olur. Bazı programlara günlük dosyasını kapatması söylenemediğinde ve bu nedenle bir süre daha önceki günlük dosyasına yazmaya devam edebileceği durumlarda kullanılabilir.

### maxage *count* 
<count> günden daha eski döndürülmüş günlükleri kaldırın. Yaş yalnızca günlük dosyası döndürülecekse kontrol edilir. maillast ve mail yapılandırılmışsa dosyalar yapılandırılan adrese postalanır. 

### minsize *size* 
Günlük dosyaları boyut baytından daha fazla büyüdüklerinde döndürülür, ancak ek olarak belirtilen zaman aralığından (günlük, haftalık, aylık veya yıllık) önce döndürülmez. İlgili boyut seçeneği, zaman aralığı seçenekleriyle karşılıklı olarak özel olması ve günlük dosyalarının son döndürme zamanına bakılmaksızın döndürülmesine neden olması dışında benzerdir. minsize kullanıldığında, bir günlük dosyasının hem boyutu hem de zaman damgası dikkate alınır.

### copytruncate
Eski günlük dosyasını taşımak ve isteğe bağlı olarak yeni bir tane oluşturmak yerine, bir kopya oluşturduktan sonra orijinal günlük dosyasını yerinde keser. Bazı programlara günlük dosyasını kapatması söylenemediğinde ve bu nedenle önceki günlük dosyasına sonsuza kadar yazmaya (eklemeye) devam edebileceği durumlarda kullanılabilir. Dosyanın kopyalanması ve kesilmesi arasında çok küçük bir zaman dilimi olduğunu, bu nedenle bazı günlük verilerinin kaybolabileceğini unutmayın. Bu seçenek kullanıldığında, eski günlük dosyası yerinde kalacağı için create seçeneğinin bir etkisi olmayacaktır.

### rotate *count*
Günlük dosyaları kaldırılmadan veya bir posta yönergesinde belirtilen adrese postalanmadan önce sayım kez döndürülür. Sayım 0 ise, eski sürümler döndürülmek yerine kaldırılır.

### mail *address*
Bir kütük varoluştan döndürüldüğünde, adrese postalanır. Belirli bir kütük tarafından posta üretilmemesi gerekiyorsa nomail yönergesi kullanılabilir.

### ifempty
Notifempty seçeneğini geçersiz kılarak boş olsa bile günlük dosyasını döndürün (ifempty varsayılandır).

### missingok
Günlük dosyası eksikse, bir hata mesajı vermeden bir sonrakine geçin. Ayrıca bkz. nomissingok.

### daily
Günlük dosyaları her gün döndürülür.

### monthly
Günlük dosyaları, logrotate bir ay içinde ilk kez çalıştırıldığında döndürülür (bu normalde ayın ilk günüdür).

### weekly
İçinde bulunulan hafta içi gün, son döndürmenin yapıldığı hafta içi günden daha azsa veya son döndürmenin üzerinden bir haftadan fazla zaman geçmişse günlük dosyaları döndürülür. Bu normalde günlükleri haftanın ilk günü döndürmekle aynıdır, ancak logrotate her gece çalıştırılmazsa daha iyi çalışır.

### yearly
Geçerli yıl son rotasyonla aynı değilse günlük dosyaları rotasyona tabi tutulur.
