# Time Tracker Azure DevOps Extension - Analiz Dökümanı

## 📋 İçindekiler
1. [Sistem Özeti](#sistem-özeti)
2. [Mimari Analiz](#mimari-analiz)
3. [Mevcut Özellikler](#mevcut-özellikler)
4. [Lisans Sistemi Detayları](#lisans-sistemi-detayları)
5. [Veritabanı Yapısı](#veritabanı-yapısı)
6. [Eksik Özellikler ve Öneriler](#eksik-özellikler-ve-öneriler)
7. [Teknik Detaylar](#teknik-detaylar)

---

## 🎯 Sistem Özeti

### Amaç
Bu proje, Azure DevOps Marketplace'te yayınlanmış olan **GOTime** (Time Tracker) extension'ı için bir lisans yönetim sistemi içermektedir. Sistem, kullanıcıların plugin'i kullanabilmesi için lisans kontrolü yapar ve şu anda demo (3 aylık) lisansların üretilmesini sağlar.

### İş Akışı
1. **Kullanıcılar**: Azure DevOps'ta plugin'i kullanmak için 3 aylık demo lisans oluşturabilir
2. **Satış Sonrası**: Müşteriler ücretli lisans satın almak için şirketle iletişime geçer
3. **Hedef**: Kimin, ne kadar ve ne zaman lisans kullandığını raporlama, müşteri portföyü kontrolü ve bildirim gönderme

---

## 🏗️ Mimari Analiz

### Sistem Bileşenleri

#### 1. **Frontend (Azure DevOps Extension)**
- **Teknoloji**: TypeScript, React 16.13.1, Azure DevOps Extension SDK
- **Konum**: `src/` klasörü
- **Görevler**:
  - Zaman takibi ve raporlama arayüzü
  - Lisans doğrulama (client-side)
  - Lisans bilgilerini Azure DevOps settings'te saklama

#### 2. **Backend (License Server)**
- **Teknoloji**: Node.js, Express.js
- **Konum**: `backend/` klasörü
- **Port**: 3001
- **Görevler**:
  - Demo lisans üretimi (`/api/license/demo`)
  - Lisans üretimi (`/api/license/generate`)
  - Lisans doğrulama (`/api/license/validate`)
  - Veritabanı yönetimi

#### 3. **Veritabanı**
- **Desteklenen**: MySQL (production) ve SQLite (development)
- **Konum**: `backend/src/db.js`
- **Şema**: `demo_licenses`, `demo_license_organizations`, `demo_license_hosts`

#### 4. **Lisans Generator Core**
- **Teknoloji**: node-forge (RSA şifreleme)
- **Konum**: `tools/license-generator-core.js`
- **Görevler**:
  - RSA private/public key ile lisans imzalama
  - Lisans doğrulama (signature verification)

---

## ✅ Mevcut Özellikler

### Frontend Özellikleri

1. **Zaman Takibi**
   - Calendar-based time entry
   - Work item entegrasyonu
   - Dönemsel raporlama
   - Matematiksel hesaplamalar

2. **Lisans Yönetimi** (`src/utils/license-utils.ts`)
   - Client-side lisans doğrulama
   - RSA signature verification
   - Organization/Host tabanlı kontrol (Azure DevOps Cloud ve Server desteği)
   - Domain tabanlı kontrol (legacy)
   - Expiration kontrolü
   - User count limit kontrolü (uyarı olarak)
   - License storage (Azure DevOps collection settings)

3. **UI Bileşenleri**
   - Settings panel (`effort-settings`)
   - License activation/deactivation
   - License warning messages
   - Calendar view
   - Report generation

### Backend Özellikleri

1. **Lisans Üretimi** (`backend/src/routes/license.js`)
   - **POST `/api/license/demo`**: 3 aylık demo lisans üretir
     - Organization veya Host kontrolü yapar
     - Çakışan lisansları tespit eder
     - Veritabanına kaydeder
   - **POST `/api/license/generate`**: Özel lisans üretir
   - **POST `/api/license/validate`**: Lisans doğrular

2. **Veritabanı Yönetimi**
   - MySQL ve SQLite desteği
   - Otomatik tablo oluşturma
   - Organization/Host mapping

3. **Health Checks**
   - `/healthz`: Basit health check
   - `/readyz`: Database readiness check

---

## 🔐 Lisans Sistemi Detayları

### Lisans Formatı
Lisans key'i 3 bölümden oluşur (nokta ile ayrılmış):
```
<base64-encoded-payload>.<rsa-signature>.<public-key>
```

### Payload Yapısı
```json
{
  "userCount": 100,
  "expireDate": "2025-12-31",
  "issuedDate": "2025-09-11",
  "version": "1.0.0",
  "features": ["demo"],
  "allowedOrganizations": ["org1", "org2"],  // Azure DevOps Cloud
  "allowedHosts": ["host1.com"],             // Azure DevOps Server
  "domains": ["dev.azure.com"]                // Legacy
}
```

### Güvenlik Özellikleri

1. **RSA Asymmetric Encryption**
   - Private key: Lisans imzalama (backend'de)
   - Public key: İmza doğrulama (extension'da embedded)

2. **Doğrulama Kontrolleri**
   - Public key eşleşmesi
   - Signature verification (SHA-256 hash + RSA)
   - Expiration date kontrolü
   - Organization/Host eşleşmesi
   - Constant-time string comparison (timing attack koruması)

3. **Lisans Türleri**
   - **Demo License**: 3 aylık, 100 kullanıcı limiti
   - **Production License**: Özelleştirilebilir süre ve kullanıcı sayısı

---

## 🗄️ Veritabanı Yapısı

### Mevcut Tablolar

#### 1. `demo_licenses`
```sql
- id (PRIMARY KEY)
- identifier (UNIQUE) - Organization/Host kombinasyonu
- license_key (TEXT) - Üretilen lisans key
- payload (TEXT) - JSON formatında payload
- created_at (TIMESTAMP)
```

#### 2. `demo_license_organizations`
```sql
- id (PRIMARY KEY)
- license_id (FOREIGN KEY -> demo_licenses.id)
- organization (UNIQUE) - Azure DevOps organization name (lowercase)
- INDEX: idx_demo_license_org_license
```

#### 3. `demo_license_hosts`
```sql
- id (PRIMARY KEY)
- license_id (FOREIGN KEY -> demo_licenses.id)
- host (UNIQUE) - Azure DevOps Server hostname (lowercase)
- INDEX: idx_demo_license_host_license
```

### Veritabanı Seçimi
- **Production**: MySQL (environment variables ile yapılandırılır)
- **Development**: SQLite (`backend/data/licenses.db`)

---

## ❌ Eksik Özellikler ve Öneriler

### 🔴 Kritik Eksiklikler

#### 1. **Kullanım Takibi (Usage Tracking)**
**Mevcut Durum**: Extension, backend'e kullanım bilgisi göndermiyor.

**İhtiyaç**:
- Her lisans aktivasyonunda backend'e bildirim
- Düzenli heartbeat/usage ping
- Kullanıcı sayısı tracking
- Aktivite logları

**Önerilen Çözüm**:
```typescript
// Frontend'de eklenecek
POST /api/license/usage
{
  "licenseKey": "...",
  "organization": "org1",
  "host": "host1.com",
  "userCount": 50,
  "lastActive": "2025-01-15T10:00:00Z",
  "extensionVersion": "1.0.6"
}
```

#### 2. **Müşteri Yönetimi (Customer Management)**
**Mevcut Durum**: Sadece demo lisanslar kaydediliyor, müşteri bilgisi yok.

**İhtiyaç**:
- Müşteri bilgileri (şirket adı, email, telefon)
- İletişim bilgileri
- Lisans geçmişi
- Ödeme durumu

**Önerilen Veritabanı Şeması**:
```sql
CREATE TABLE customers (
  id INT PRIMARY KEY AUTO_INCREMENT,
  company_name VARCHAR(255),
  contact_email VARCHAR(255),
  contact_phone VARCHAR(50),
  status ENUM('demo', 'active', 'expired', 'cancelled'),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

CREATE TABLE licenses (
  id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT FOREIGN KEY,
  license_key TEXT,
  license_type ENUM('demo', 'trial', 'paid'),
  user_count INT,
  expire_date DATE,
  issued_date DATE,
  status ENUM('active', 'expired', 'revoked'),
  created_at TIMESTAMP
);

CREATE TABLE license_usage_logs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  license_id INT FOREIGN KEY,
  organization VARCHAR(255),
  host VARCHAR(255),
  user_count INT,
  last_active TIMESTAMP,
  extension_version VARCHAR(50),
  logged_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_license_id (license_id),
  INDEX idx_logged_at (logged_at)
);
```

#### 3. **Raporlama Sistemi (Reporting)**
**Mevcut Durum**: Raporlama özelliği yok.

**İhtiyaç**:
- Kimin lisans kullandığı listesi
- Ne kadar süre kullanıldığı
- Hangi organization/host'ta kullanıldığı
- Expiring licenses raporu
- Usage statistics

**Önerilen API Endpoints**:
```
GET /api/reports/licenses
GET /api/reports/customers
GET /api/reports/usage?licenseId=123&startDate=2025-01-01&endDate=2025-01-31
GET /api/reports/expiring?days=30
```

#### 4. **Bildirim Sistemi (Notifications)**
**Mevcut Durum**: Bildirim sistemi yok.

**İhtiyaç**:
- Expiring license uyarıları (30, 15, 7 gün kala)
- Expired license bildirimleri
- Yeni demo lisans bildirimleri
- Usage limit aşımları

**Önerilen Çözüm**:
- Email notifications (Nodemailer veya SendGrid)
- Webhook desteği
- Scheduled jobs (node-cron)

#### 5. **Admin Panel**
**Mevcut Durum**: Sadece API endpoints var, UI yok.

**İhtiyaç**:
- Dashboard (aktif lisanslar, expiring licenses, usage stats)
- Müşteri listesi ve detayları
- Lisans oluşturma/revoke etme
- Rapor görüntüleme
- Bildirim yönetimi

**Önerilen Teknoloji**:
- React veya Vue.js tabanlı admin panel
- Backend API'yi kullanarak

---

### 🟡 İyileştirme Önerileri

#### 1. **Authentication & Authorization**
- API key authentication
- Role-based access control (admin, operator, viewer)
- JWT token authentication

#### 2. **API Documentation**
- Swagger/OpenAPI dokümantasyonu
- API endpoint detayları

#### 3. **Error Handling & Logging**
- Structured logging (Winston, Pino)
- Error tracking (Sentry)
- Request/response logging

#### 4. **Testing**
- Unit tests (Jest)
- Integration tests
- E2E tests

#### 5. **Monitoring & Metrics**
- Health check improvements
- Performance metrics
- Database query monitoring

#### 6. **Backup & Recovery**
- Database backup strategy
- License key backup

---

## 🔧 Teknik Detaylar

### Backend API Endpoints

#### Mevcut Endpoints
```
GET  /healthz                    - Health check
GET  /readyz                     - Readiness check
POST /api/license/demo           - Demo lisans üret
POST /api/license/generate       - Özel lisans üret
POST /api/license/validate       - Lisans doğrula
```

### Environment Variables

#### Backend
```bash
# Database (MySQL)
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=password
MYSQL_DATABASE=license_db
MYSQL_SSL=false
MYSQL_POOL_LIMIT=10

# License Keys
LICENSE_PRIVATE_KEY=-----BEGIN PRIVATE KEY-----...
LICENSE_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----...

# Server
NODE_ENV=production
PORT=3001
```

### Deployment

#### Kubernetes
- Deployment: `infrastructure/k8s/license-backend.yaml`
- Service: ClusterIP, port 3001
- Secrets: `license-backend-secrets.yaml`
- Registry: `registry.higgscloud.com/rdc/time-tracker-azdo-extension/license-backend:latest`

#### Docker
```bash
make backend-image    # Build image
make backend-push     # Push to registry
```

### Frontend Integration

Extension'da backend'e bağlantı:
- Demo lisans üretimi: `https://gotime.higgscloud.com/api/license/demo`
- Location: `src/components/effort-settings/effort-settings.tsx:429`

---

## 📊 Mimari Topoloji

```
┌─────────────────────────────────────────────────────────────────┐
│                    Azure DevOps Marketplace                      │
│                         (GOTime Extension)                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ Extension installed
                             │
┌────────────────────────────▼────────────────────────────────────┐
│              Azure DevOps Organization/Server                    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │          Frontend Extension (TypeScript/React)           │  │
│  │                                                           │  │
│  │  • License validation (RSA signature)                    │  │
│  │  • License storage (Collection Settings)                 │  │
│  │  • Time tracking UI                                      │  │
│  │  • Reports & Calendar                                    │  │
│  └───────────────────────┬──────────────────────────────────┘  │
└──────────────────────────┼──────────────────────────────────────┘
                           │
                           │ HTTPS API Calls
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                    License Backend Server                        │
│                    (Node.js + Express)                           │
│                    Port: 3001                                    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   API Endpoints                          │  │
│  │  • POST /api/license/demo                                │  │
│  │  • POST /api/license/generate                            │  │
│  │  • POST /api/license/validate                            │  │
│  │  • GET  /healthz, /readyz                                │  │
│  └───────────────────────┬──────────────────────────────────┘  │
│                          │                                       │
│  ┌───────────────────────▼──────────────────────────────────┐  │
│  │            License Generator Core                        │  │
│  │            (RSA Private Key)                             │  │
│  │            node-forge library                            │  │
│  └───────────────────────┬──────────────────────────────────┘  │
└──────────────────────────┼──────────────────────────────────────┘
                           │
                           │ Database Queries
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      Database Layer                              │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Production: MySQL                                       │  │
│  │  Development: SQLite                                     │  │
│  │                                                           │  │
│  │  Tables:                                                 │  │
│  │  • demo_licenses                                         │  │
│  │  • demo_license_organizations                            │  │
│  │  • demo_license_hosts                                    │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    Eksik Bileşenler (Önerilen)                  │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐ │
│  │ Usage Tracking   │  │ Customer Mgmt    │  │ Notification │ │
│  │ API Endpoint     │  │ Database Tables  │  │ Service      │ │
│  └──────────────────┘  └──────────────────┘  └──────────────┘ │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐ │
│  │ Reporting API    │  │ Admin Panel      │  │ Scheduled    │ │
│  │ Endpoints        │  │ (React UI)       │  │ Jobs (Cron)  │ │
│  └──────────────────┘  └──────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Özet ve Sonraki Adımlar

### Mevcut Sistem
✅ Demo lisans üretimi çalışıyor  
✅ Lisans doğrulama (client-side) çalışıyor  
✅ Veritabanı yapısı temel seviyede  
✅ Azure DevOps Cloud ve Server desteği var  

### Eksikler
❌ Kullanım takibi yok  
❌ Müşteri yönetimi yok  
❌ Raporlama sistemi yok  
❌ Bildirim sistemi yok  
❌ Admin panel yok  

### Öncelikli Geliştirmeler

1. **Phase 1: Kullanım Takibi**
   - Usage tracking API endpoint
   - Frontend'den backend'e usage ping
   - Usage logs tablosu

2. **Phase 2: Müşteri Yönetimi**
   - Customer ve License tabloları
   - Customer API endpoints
   - Demo'dan paid'e geçiş yönetimi

3. **Phase 3: Raporlama**
   - Reporting API endpoints
   - Usage statistics
   - Expiring licenses raporu

4. **Phase 4: Bildirimler**
   - Email notification service
   - Scheduled jobs
   - Expiring license uyarıları

5. **Phase 5: Admin Panel**
   - React admin panel
   - Dashboard
   - Customer management UI
   - License management UI

---

## 📞 İletişim ve Dokümantasyon

- **Support Email**: support@rdcpartner.com
- **License Tools README**: `tools/LICENSE-TOOLS-README.md`
- **Main README**: `README.md`

---

*Son Güncelleme: 2025-01-15*

