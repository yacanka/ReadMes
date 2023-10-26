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

## Anahtar kelimeler
Logrotate yapılandırma dosyasına dahil edilebilecek yönergeler hakkında daha fazla bilgiyi burada bulabilirsiniz.
> Bütün yönergeler dahil edilmemiştir. Sık kullanılacaklar eklenmiştir.

### create *mode owner group*
Döndürme işleminden hemen sonra (postrotate betiği çalıştırılmadan önce) günlük dosyası oluşturulur (döndürülen günlük dosyasıyla aynı adla). mode günlük dosyasının modunu sekizli olarak belirtir (chmod(2) ile aynıdır), owner günlük dosyasına sahip olacak kullanıcı adını belirtir ve group günlük dosyasının ait olacağı grubu belirtir. Günlük dosyası özniteliklerinden herhangi biri atlanabilir, bu durumda yeni dosya için bu öznitelikler, atlanan öznitelikler için orijinal günlük dosyasıyla aynı değerleri kullanacaktır. Bu seçenek nocreate seçeneği kullanılarak devre dışı bırakılabilir.

### size *size*
Günlük dosyaları yalnızca boyut baytından daha fazla büyüdüklerinde döndürülür. Eğer size'ı k takip ediyorsa, boyutun kilobayt cinsinden olduğu varsayılır. M kullanılırsa, boyut megabayt cinsindendir ve G kullanılırsa, boyut gigabayt cinsindendir. Yani size 100, size 100k, size 100M ve size 100G'nin hepsi geçerlidir.

### compress
Günlük dosyalarının eski sürümleri varsayılan olarak gzip(1) ile sıkıştırılır. Ayrıca bkz. nocompress.

### maxage *count* 
<count> günden daha eski döndürülmüş günlükleri kaldırın. Yaş yalnızca günlük dosyası döndürülecekse kontrol edilir. maillast ve mail yapılandırılmışsa dosyalar yapılandırılan adrese postalanır. 

### minsize *size* 
Günlük dosyaları boyut baytından daha fazla büyüdüklerinde döndürülür, ancak ek olarak belirtilen zaman aralığından (günlük, haftalık, aylık veya yıllık) önce döndürülmez. İlgili boyut seçeneği, zaman aralığı seçenekleriyle karşılıklı olarak özel olması ve günlük dosyalarının son döndürme zamanına bakılmaksızın döndürülmesine neden olması dışında benzerdir. minsize kullanıldığında, bir günlük dosyasının hem boyutu hem de zaman damgası dikkate alınır.

### copytruncate
Eski günlük dosyasını taşımak ve isteğe bağlı olarak yeni bir tane oluşturmak yerine, bir kopya oluşturduktan sonra orijinal günlük dosyasını yerinde keser. Bazı programlara günlük dosyasını kapatması söylenemediğinde ve bu nedenle önceki günlük dosyasına sonsuza kadar yazmaya (eklemeye) devam edebileceği durumlarda kullanılabilir. Dosyanın kopyalanması ve kesilmesi arasında çok küçük bir zaman dilimi olduğunu, bu nedenle bazı günlük verilerinin kaybolabileceğini unutmayın. Bu seçenek kullanıldığında, eski günlük dosyası yerinde kalacağı için create seçeneğinin bir etkisi olmayacaktır.

### rotate
Günlük dosyaları kaldırılmadan veya bir posta yönergesinde belirtilen adrese postalanmadan önce sayım kez döndürülür. Sayım 0 ise, eski sürümler döndürülmek yerine kaldırılır.

### mail *address*
Bir kütük varoluştan döndürüldüğünde, adrese postalanır. Belirli bir kütük tarafından posta üretilmemesi gerekiyorsa nomail yönergesi kullanılabilir.

### ifempty
Notifempty seçeneğini geçersiz kılarak boş olsa bile günlük dosyasını döndürün (ifempty varsayılandır).

### daily
Günlük dosyaları her gün döndürülür.

### monthly
Günlük dosyaları, logrotate bir ay içinde ilk kez çalıştırıldığında döndürülür (bu normalde ayın ilk günüdür).

### weekly
İçinde bulunulan hafta içi gün, son döndürmenin yapıldığı hafta içi günden daha azsa veya son döndürmenin üzerinden bir haftadan fazla zaman geçmişse günlük dosyaları döndürülür. Bu normalde günlükleri haftanın ilk günü döndürmekle aynıdır, ancak logrotate her gece çalıştırılmazsa daha iyi çalışır.

### yearly
Geçerli yıl son rotasyonla aynı değilse günlük dosyaları rotasyona tabi tutulur.
