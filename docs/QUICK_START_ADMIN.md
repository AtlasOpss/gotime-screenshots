# Admin Panel - Hızlı Başlangıç

## 🚀 En Hızlı Yol (Önerilen)

Tek komutla başlat:

```bash
./infrastructure/k8s/admin-panel-start.sh
```

Bu script:
1. Port-forward'u başlatır
2. Backend'i test eder
3. Admin panel'i tarayıcıda açar
4. Ctrl+C ile durdurabilirsiniz

## 📋 Manuel Adımlar

### 1. Port Forward Başlat

```bash
kubectl port-forward -n gotime service/license-backend 3001:3001
```

Bu komutu çalıştırdığınız terminal açık kalmalı.

### 2. Admin Panel'i Aç

Yeni bir terminal açın veya tarayıcıyı kullanın:

**macOS:**
```bash
open admin-panel/index.html
```

**Linux:**
```bash
xdg-open admin-panel/index.html
```

**Windows:**
```bash
start admin-panel/index.html
```

Veya tarayıcıda şu adresi açın:
```
file:///path/to/project/admin-panel/index.html
```

### 3. Kullanım

Admin panel açıldığında:
- **Dashboard** sekmesinde genel istatistikleri görürsünüz
- **Licenses** sekmesinde tüm lisansları listeleyebilirsiniz
- **Customers** sekmesinde müşterileri görüntüleyebilirsiniz
- **Reports** sekmesinde raporları görüntüleyebilirsiniz

## ✅ Kontrol

Backend'in çalıştığını test etmek için (port-forward çalışırken):

```bash
curl http://localhost:3001/healthz
curl http://localhost:3001/api/reports/dashboard
```

## 🔄 Port Forward'u Durdurma

Port-forward çalışan terminalde `Ctrl+C` tuşlarına basın.

Veya arka planda çalışıyorsa:

```bash
pkill -f "kubectl port-forward.*3001"
```

## 🐛 Sorun Giderme

### Port zaten kullanımda

```bash
# Hangi process port'u kullanıyor?
lsof -i :3001

# Process'i durdur
kill <PID>
```

### Backend erişilemiyor

```bash
# Pod'ları kontrol et
kubectl get pods -l app=license-backend -n gotime

# Logları kontrol et
kubectl logs -f deployment/license-backend -n gotime
```

### CORS hatası

Admin panel HTML dosyasındaki API URL'i kontrol edin. Port-forward kullanıyorsanız `http://localhost:3001/api` olmalı.

## 📝 Notlar

- Port-forward sadece terminal açık kaldığı sürece çalışır
- Admin panel static HTML dosyasıdır, herhangi bir web server gerektirmez
- API çağrıları doğrudan backend'e yapılır (CORS açık)

