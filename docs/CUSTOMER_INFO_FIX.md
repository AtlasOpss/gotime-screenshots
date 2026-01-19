# Customer Bilgisi Düzeltmesi

## Sorun

Admin panelde customer bilgisi boş geliyordu çünkü:
- Frontend (plugin) customer bilgilerini backend'e göndermiyordu
- Backend customer bilgisi olmadan lisans oluşturuyordu

## Yapılan Değişiklikler

### 1. Frontend (Plugin) - Customer Bilgisi Eklendi

`src/components/effort-settings/effort-settings.tsx` dosyasında:
- Azure DevOps SDK'dan kullanıcı bilgileri alınıyor (`SDK.getUser()`)
- Kullanıcı email, isim ve organization bilgisi backend'e gönderiliyor
- `contactEmail`, `contactName`, `companyName` alanları eklendi

### 2. Backend - Lisans Süresi 36 Gün Olarak Değiştirildi

`backend/src/routes/license.js` dosyasında:
- Demo lisans süresi 365 günden 36 güne düşürüldü
- Artık demo lisanslar 36 gün geçerli olacak

## Sonuç

Artık demo lisans oluşturulduğunda:
- Kullanıcı email'i customer olarak kaydedilecek
- Kullanıcı ismi contact name olarak kaydedilecek
- Organization adı company name olarak kaydedilecek
- Admin panelde customer bilgileri görünecek

## Deployment

Bu değişiklikler için:

1. **Frontend (Plugin) Deployment:**
   - Plugin'i build edip Azure DevOps Marketplace'e publish etmeniz gerekiyor
   - Veya development'ta test ediyorsanız, webpack dev server'ı restart edin

2. **Backend Deployment:**
   - Backend kodunu Kubernetes'e deploy etmeniz gerekiyor
   ```bash
   make backend-image
   make backend-push
   kubectl rollout restart deployment/license-backend -n gotime
   ```

## Not

Eğer kullanıcı email'i Azure DevOps'ta mevcut değilse, customer bilgisi yine boş kalabilir. Bu durumda:
- Backend customer oluşturmaz (customerId = null)
- Lisans yine oluşturulur ama customer bilgisi olmadan

