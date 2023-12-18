# File Distributor Rehberi

## Kurulum

### File Distributor Kurulumu
File distributor uygulamasının kurulum dosyası elinizde olmalıdır. Aşağıdaki komut ile mevcut dizinde bulunan deb dosyasını kurun.
```
sudo apt install ./file-distributor-x64.deb
```

### Minio Server Kurulumu
File distributor, minio bağımlıdır. Minio'nun kurulması ve servis haline getirilmesi gerekir.
Minio serverı indirin.
```
wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio_20231209181751.0.0_amd64.deb -O minio.deb
```
İndirdiğiniz minioyu kurmak için bir .sh dosyası oluşturun.
```
nano installer.sh
```
> minio.deb dosyası ile aynı dizinde oluşturun.

Aşağıdaki script kodunu yapıştırın.
```
#!/bin/bash
MINIO_ROOT_USER="admin"
MINIO_ROOT_PASSWORD="password"

MINIO_VOLUMES="/mnt/data"
MINIO_DEB_PATH="./minio.deb"

sudo dpkg -i ${MINIO_DEB_PATH} 

mkdir -p ${MINIO_VOLUMES}
chmod +wr ${MINIO_VOLUMES}

cat <<EOL | sudo tee /etc/systemd/system/minio.service
[Unit]
Description=MinIO
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local

User=minio-user
Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"\${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server \$MINIO_OPTS \$MINIO_VOLUMES

Restart=always
LimitNOFILE=65536
TasksMax=infinity
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
EOL

cat <<EOL | sudo tee /etc/default/minio
MINIO_ROOT_USER=${MINIO_ROOT_USER}
MINIO_ROOT_PASSWORD=${MINIO_ROOT_PASSWORD}
MINIO_VOLUMES="${MINIO_VOLUMES}"
EOL

sudo groupadd -r minio-user
sudo useradd -M -r -g minio-user minio-user
sudo chown minio-user:minio-user ${MINIO_VOLUMES}

sudo systemctl daemon-reload

sudo systemctl start minio.service

sudo systemctl enable minio.service

echo "Complated"
```

Scripti çalıştırmak için yetki verin ve çalıştırın
```
chmod +x installer.sh
./installer.sh
```

### Minio Client Kurulumu
Minio client key oluşturmak için gereklidir. Aşağıdaki komutla indirin.
```
wget https://dl.min.io/client/mc/release/linux-amd64/mc
```
İndirilen mc dosyasını file distributor altına şu şekilde kopyalayın.
```
cp mc /opt/file-distributor/minio-config/
```

## Konfigürasyon
File distributor konfigürasyonunu yapmak için .env dosyasını açın.
```
sudo nano /opt/file-distributor/.env
```
Aşağıdaki kısmı dosyadan silin.
```
MINIO_ADDRESS="127.0.0.1:9000"
MINIO_ACCESS_KEY_ID="123456789"
MINIO_SECRET_ACCESS_KEY="123456789"
MINIO_BUCKET="distributor"
```
Database bilgilerini aşağıdaki bilgiler doğrultusunda isteğinize göre güncelleyin.
- `DB_DRIVER` postgres olmalıdır.
- File distributor, veritabanı ile aynı makinedeyse `DB_HOST` localhost olarak kalmalıdır.
- `DB_NAME` değeri veritabanında oluşturulmuş bir isim olmalıdır. Oluşturmak için:
```
cd /
sudo -u postgres createdb distributor -O postgres
```
- Şifre belirleyin. `DB_PASS` değeri ile veritabanına erişilmeye çalışılır. Aşağıda `DB_PASS` değeri _1_ olarak seçilmiştir.
```
sudo -u postgres psql -U postgres -d postgres -c "alter user \"distributor\" with password '1';"
```
- `DB_PORT` değeri, eğer postgres standart port üzerinde çalışıyorsa değiştirilmemelidir.
- `DB_USER` değeri, postgres kullanıcısı seçilmiştir. Yeni bir kullanıcı oluşturmak için:
```
sudo -u postgres createuser distributor
```

.env dosyası ile ilgili işlemleri tamamladıktan sonra minio config dosyası çalıştırılmalıdır.
```
/opt/file-distributor/minio-config/minio-config.sh
```



