# Port Forward - Admin Panel İçin

## Sorun

Admin panelde "Failed to fetch" hatası alıyorsanız, port-forward çalışmıyor demektir.

## Çözüm

Port-forward'u başlatmanız gerekiyor:

```bash
kubectl port-forward -n gotime service/license-backend 3001:3001
```

Bu komut çalıştığı sürece backend `http://localhost:3001` adresinde erişilebilir olacak.

## Arka Planda Çalıştırma

Port-forward'u arka planda çalıştırmak için:

```bash
# Arka planda başlat
kubectl port-forward -n gotime service/license-backend 3001:3001 > /dev/null 2>&1 &

# Process ID'yi kaydet (isteğe bağlı)
PF_PID=$!
echo "Port-forward PID: $PF_PID"

# Durdurmak için:
kill $PF_PID
```

## Kontrol

Port-forward çalışıyor mu kontrol etmek için:

```bash
# Port kullanımda mı?
lsof -i :3001

# Backend erişilebilir mi?
curl http://localhost:3001/healthz
```

## Not

Port-forward terminal açık kaldığı sürece çalışır. Terminal'i kapatırsanız port-forward durur.

## Otomatik Başlatma

`admin-panel-start.sh` script'i port-forward'u otomatik başlatır:

```bash
./infrastructure/k8s/admin-panel-start.sh
```

