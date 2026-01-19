# Kubernetes Deployment - License Backend

## 🚀 Hızlı Deployment

### 1. Docker Image Build ve Push

```bash
# Image'ı build et ve push et
make backend-image
make backend-push
```

Veya manuel:
```bash
docker buildx build --platform linux/amd64 \
  -f backend/Dockerfile \
  -t registry.higgscloud.com/rdc/time-tracker-azdo-extension/license-backend:latest \
  --push .
```

### 2. Kubernetes'e Deploy Et

```bash
# Deployment'ı apply et
kubectl apply -f infrastructure/k8s/license-backend.yaml -n gotime

# Deployment durumunu kontrol et
kubectl rollout status deployment/license-backend -n gotime
```

Veya deploy script'i kullan:
```bash
./infrastructure/k8s/deploy.sh
```

## 📋 Deployment Öncesi Kontroller

### Secrets Kontrolü

Mevcut secrets'ı kontrol edin:
```bash
kubectl get secret license-backend-env -n gotime -o yaml
```

### SMTP Ayarları (Opsiyonel - Bildirimler için)

Email bildirimleri için SMTP ayarlarını ekleyin:
```bash
# SMTP ayarlarını ekle
kubectl create secret generic license-backend-env \
  --from-literal=SMTP_HOST=smtp.example.com \
  --from-literal=SMTP_PORT=587 \
  --from-literal=SMTP_SECURE=false \
  --from-literal=SMTP_USER=your-email@example.com \
  --from-literal=SMTP_PASS=your-password \
  --from-literal=SMTP_FROM=noreply@example.com \
  --dry-run=client -o yaml | kubectl apply -f - -n gotime
```

Veya script kullan:
```bash
./infrastructure/k8s/update-secrets.sh
```

**Not**: SMTP ayarları opsiyoneldir. Eğer ayarlanmazsa, notification service çalışmaya devam eder ancak email göndermez.

## 🔍 Deployment Sonrası Kontroller

### Pod Durumu

```bash
# Pod'ları kontrol et
kubectl get pods -l app=license-backend -n gotime

# Pod loglarını görüntüle
kubectl logs -f deployment/license-backend -n gotime
```

### Health Check

```bash
# Port forward yap
kubectl port-forward -n gotime service/license-backend 3001:3001

# Health check
curl http://localhost:3001/healthz
curl http://localhost:3001/readyz
```

### Yeni API Endpoint'lerini Test Et

```bash
# Dashboard stats
curl http://localhost:3001/api/reports/dashboard

# Licenses list
curl http://localhost:3001/api/reports/licenses

# Customers list
curl http://localhost:3001/api/customers
```

## 📊 Yeni Özellikler

Deployment sonrası aktif olan yeni özellikler:

1. **Usage Tracking** - `/api/usage/*`
   - Frontend otomatik olarak kullanım bilgisini gönderir

2. **Customer Management** - `/api/customers/*`
   - Müşteri yönetimi API'leri

3. **Reporting** - `/api/reports/*`
   - Dashboard, licenses, expiring, expired, usage reports

4. **Notifications** - Scheduled job
   - Günlük 9 AM'de expiring license kontrolü
   - Email bildirimleri (SMTP ayarlanmışsa)

5. **License Duration**
   - Demo lisanslar artık 1 yıl geçerli (önceden 3 ay)

## 🗄️ Veritabanı Migration

Yeni tablolar otomatik olarak oluşturulacak:
- `customers`
- `licenses` (unified)
- `license_organizations`
- `license_hosts`
- `license_usage_logs`
- `notifications`

Migration güvenlidir - sadece yeni tablolar oluşturur, mevcut tablolara dokunmaz.

## 🔄 Rollback (Gerekirse)

```bash
# Önceki versiyona geri dön
kubectl rollout undo deployment/license-backend -n gotime

# Rollout geçmişini görüntüle
kubectl rollout history deployment/license-backend -n gotime

# Belirli bir revision'a dön
kubectl rollout undo deployment/license-backend --to-revision=2 -n gotime
```

## 🐛 Troubleshooting

### Pod başlamıyor

```bash
# Pod detaylarını kontrol et
kubectl describe pod -l app=license-backend -n gotime

# Logları kontrol et
kubectl logs -l app=license-backend -n gotime
```

### Database bağlantı sorunu

```bash
# Pod içinden database bağlantısını test et
kubectl exec -it deployment/license-backend -n gotime -- \
  sh -c 'echo "SELECT 1" | mysql -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE'
```

### Email bildirimleri çalışmıyor

- SMTP environment variables'ların set edildiğini kontrol edin
- Logları kontrol edin: `kubectl logs deployment/license-backend -n gotime | grep -i smtp`
- SMTP credentials'ları doğrulayın

## 📝 Notlar

1. **Image Tag**: Deployment `latest` tag'ini kullanıyor. Production için version tag'leri kullanmanız önerilir.

2. **Secrets**: Hassas bilgiler secrets'da tutuluyor. Git'e commit edilmemeli.

3. **Resource Limits**: Mevcut limits yeterli olmalı, gerekirse artırılabilir.

4. **Replicas**: Şu anda 1 replica var. High availability için artırılabilir.

5. **Cron Job**: Notification service'deki cron job pod içinde çalışıyor. Her pod kendi cron job'ını çalıştırır (replica sayısına dikkat).

## 🔐 Güvenlik

- Secrets Kubernetes secrets'da saklanıyor
- Private keys asla log'lara yazılmıyor
- SMTP credentials secrets'da tutuluyor
- Database bağlantıları SSL üzerinden (ayarlanabilir)

## 📞 Destek

Sorun yaşarsanız:
- Logları kontrol edin: `kubectl logs deployment/license-backend -n gotime`
- Deployment durumunu kontrol edin: `kubectl describe deployment license-backend -n gotime`
- Support: support@rdcpartner.com

