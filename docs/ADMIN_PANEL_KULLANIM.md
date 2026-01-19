# Admin Panel Kullanım Kılavuzu

## 🚀 Hızlı Başlangıç

### 1. Kubernetes Port Forward

Backend servisine port forward yapın:

```bash
kubectl port-forward -n gotime service/license-backend 3001:3001
```

Bu komut çalıştığı sürece backend servisi `http://localhost:3001` adresinde erişilebilir olacak.

**Not:** Komutu arka planda çalıştırmak için:

```bash
kubectl port-forward -n gotime service/license-backend 3001:3001 &
```

Veya ayrı bir terminal penceresi açın.

### 2. Admin Panel'i Açın

Admin panel HTML dosyasını tarayıcıda açın:

```bash
# macOS
open admin-panel/index.html

# Linux
xdg-open admin-panel/index.html

# Windows
start admin-panel/index.html
```

Veya tarayıcıda şu adresi açın:
```
file:///Users/alpaybilgic/Desktop/cursor/gotime/license-server/time-tracker-azdo-extension-main/admin-panel/index.html
```

## 📋 Adım Adım

### Adım 1: Port Forward Başlat

```bash
kubectl port-forward -n gotime service/license-backend 3001:3001
```

Çıktı şöyle olmalı:
```
Forwarding from 127.0.0.1:3001 -> 3001
Forwarding from [::1]:3001 -> 3001
```

### Adım 2: Backend'i Test Et

Yeni bir terminal açın ve backend'in çalıştığını test edin:

```bash
# Health check
curl http://localhost:3001/healthz

# Dashboard API test
curl http://localhost:3001/api/reports/dashboard
```

### Adım 3: Admin Panel'i Aç

Admin panel HTML dosyasını tarayıcıda açın. Dosya yolu:
```
admin-panel/index.html
```

### Adım 4: Admin Panel'i Kullan

Tarayıcıda admin panel açıldığında:

1. **Dashboard** - Genel istatistikleri görüntüleyin
2. **Licenses** - Tüm lisansları listeleyin
3. **Customers** - Müşteri listesini görüntüleyin
4. **Reports** - Raporları görüntüleyin

## 🔧 Sorun Giderme

### Port Forward Çalışmıyor

```bash
# Pod'ları kontrol et
kubectl get pods -l app=license-backend -n gotime

# Service'i kontrol et
kubectl get svc license-backend -n gotime

# Port'un kullanımda olup olmadığını kontrol et
lsof -i :3001

# Farklı bir port kullan
kubectl port-forward -n gotime service/license-backend 8080:3001
```

Eğer farklı bir port kullanıyorsanız, admin panel'deki API URL'ini değiştirmeniz gerekir (aşağıya bakın).

### CORS Hatası

Eğer tarayıcı console'unda CORS hatası görüyorsanız, backend CORS ayarları doğru yapılandırılmış olmalı (zaten yapılandırılmış).

### API Bağlantı Hatası

Admin panel varsayılan olarak `https://gotime.higgscloud.com/api` adresini kullanıyor. Port forward kullanıyorsanız, HTML dosyasındaki API URL'ini değiştirin:

```javascript
const API_BASE = 'http://localhost:3001/api';
```

## 🔄 Alternatif: Local Backend

Eğer Kubernetes kullanmak istemiyorsanız, backend'i local'de çalıştırabilirsiniz:

```bash
cd backend
npm install
npm start
```

Backend `http://localhost:3001` adresinde çalışacak.

## 📝 Admin Panel Özellikleri

### Dashboard
- Toplam müşteri sayısı
- Toplam lisans sayısı
- Aktif lisanslar
- 30 gün içinde expire olacak lisanslar
- Expired lisanslar
- Toplam usage log sayısı

### Licenses
- Tüm lisansların listesi
- License type (demo/paid)
- Status (active/expired)
- User count
- Expire date
- Usage count
- Last used date

### Customers
- Müşteri listesi
- Company name
- Contact information
- Status

### Reports
- Expiring licenses (30 days)
- Expired licenses
- Usage statistics

## 🔐 Güvenlik Notları

1. **Production Kullanımı**: Bu admin panel basit bir HTML dosyasıdır ve authentication içermez. Production'da kullanmak için:
   - Authentication ekleyin
   - API endpoint'lerine authentication ekleyin
   - HTTPS kullanın
   - CORS policy'leri sıkılaştırın

2. **Port Forward**: Port forward sadece development/test için kullanılmalıdır. Production'da ingress veya load balancer kullanın.

## 📞 Yardım

Sorun yaşarsanız:

```bash
# Backend loglarını kontrol et
kubectl logs -f deployment/license-backend -n gotime

# Pod durumunu kontrol et
kubectl describe pod -l app=license-backend -n gotime

# Service durumunu kontrol et
kubectl describe svc license-backend -n gotime
```

