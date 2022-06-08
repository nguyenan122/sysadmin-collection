https://certbot.eff.org/

# Centos:
```
yum install epel-release
yum install snapd
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot


Sau đó cài ssl như các file khác:
certbot certonly --manual --preferred-challenges=dns --email xxxxxxxxx@gmail.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d "*.xxxxxxxx.vn" -d "xxxxxxxx.vn"
```