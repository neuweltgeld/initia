Eğer bir initia node kurdunuz ve validator çalıştırmak istiyorsanız, öncesinde oracle çalıştırmalısınız.
Oracle ne yapıyor ? Paritelerin güncel fiyatlarını borsa apilerinden çekerek anlık fiyat güncellemesine olanak sağlıyor. Zaten kurulum yaptığınızda borsa isimlerini ve paritleri gördüğünüzde anlayacaksınız.
Initia resmi dökümanlarda bahsedildiği şekilde

<img width="695" alt="image" src="https://github.com/neuweltgeld/initia/assets/101174090/4b4c0d32-f7bb-4e4b-aa95-87e8e9a8d54b">

Initia Slinky Oracle kullanıyor ve kurulum için resmi döküman şu şekilde. Ayrıca reponun sonunda bir servis dosyası da paylaşacağım böylece screen içinde çalıştı mı çalışmadı mı aklınıza takılmadan otomatik restart ile kullanabileceksiniz.

Öncelikle ```app.toml``` dosyamızda en altta Oracle aktif ediyoruz. İçerik şu şekilde olmalı

```
###############################################################################
###                                  Oracle                                 ###
###############################################################################
[oracle]
# Enabled indicates whether the oracle is enabled.
enabled = "true"

# Oracle Address is the URL of the out of process oracle sidecar. This is used to
# connect to the oracle sidecar when the application boots up. Note that the address
# can be modified at any point, but will only take effect after the application is
# restarted. This can be the address of an oracle container running on the same
# machine or a remote machine.
oracle_address = "127.0.0.1:8080"

# Client Timeout is the time that the client is willing to wait for responses from 
# the oracle before timing out.
client_timeout = "500ms"

# MetricsEnabled determines whether oracle metrics are enabled. Specifically
# this enables instrumentation of the oracle client and the interaction between
# the oracle and the app.
metrics_enabled = "false"
```

Henüz bir pre-compiled binary olmadığı için repoyu çekip build ediyoruz.

```
git clone https://github.com/skip-mev/slinky.git
cd slinky
```

Git checkout ile versiyon kontrolü yapıyoruz ve build alıyoruz

```console
git checkout v0.4.3
make build

# binaryi taşıyoruz (go kurduğunuzu varsayıyorum) ve gereksiz dosyayı siliyoruz

mv build/slinky /usr/local/bin/
rm -rf build
```

Servis dosyamızı oluşturuyoruz.

```
sudo tee /etc/systemd/system/slinky.service > /dev/null <<EOF
[Unit]
Description=Slinky Oracle
After=network-online.target

[Service]
User=$USER
ExecStart=$(which slinky) --oracle-config-path $HOME/slinky/config/core/oracle.json --market-map-endpoint 0.0.0.0:9090
Restart=on-failure
RestartSec=30
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```console
# Servisi aktif edip çalıştırıyoruz.
sudo systemctl daemon-reload
sudo systemctl enable slinky.service
sudo systemctl start slinky.service
```

Artık Oracle arkaplanda çalışıyor. Eğer port ve veriyollarında hata yoksa ```make run-oracle-client``` komutunu girdiğinizde veri akışını göreceksiniz.
Logları şu şekilde kontrol edebilirsiniz.
```
journalctl -fu slinky --no-hostname
```
