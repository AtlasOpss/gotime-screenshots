# Azure DevOps Marketplace Setup Guide

## Overview

Bu doküman, GOTime extension'ının Azure DevOps Marketplace'te düzgün şekilde yayınlanması ve ücretli extension olarak gösterilmesi için gerekli adımları açıklamaktadır.

## Mevcut Durum

- Extension şu anda "free" havuzunda görünüyor
- Privacy Policy, License (EULA) ve Support bilgileri eksik
- Bu nedenle "Top Publisher" listesinde gözükmüyor
- Ürün **Free Trial** modelinde olacak (ücretsiz deneme, sonrasında ücretli)

## Gerekli Dosyalar

Aşağıdaki dosyalar hazırlanmıştır:

1. ✅ **PRIVACY_POLICY.md** - Gizlilik politikası
2. ✅ **EULA.md** - Kullanıcı lisans sözleşmesi
3. ✅ **SUPPORT.md** - Destek bilgileri

## Yapılması Gerekenler

### 1. Marketplace Publisher Profile Ayarları

Azure DevOps Marketplace'te publisher profilinde aşağıdaki bilgileri ekleyin/güncelleyin:

1. **Marketplace'e giriş yapın**: https://marketplace.visualstudio.com/manage
2. **Publisher Profile** sayfasına gidin
3. Şu bilgileri doldurun:
   - **Company Name**: RDCPartner
   - **Support URL**: `https://marketplace.visualstudio.com/items?itemName=RDCPartner.gotime-extension` (veya kendi support sayfanız)
   - **Privacy Policy URL**: GitHub repo'da veya kendi domain'inizde yayınlayın
   - **Terms of Service/EULA URL**: GitHub repo'da veya kendi domain'inizde yayınlayın

### 2. Extension Manifest Güncelleme

`vss-extension.json` dosyasına aşağıdaki alanları ekleyin:

```json
{
  "manifestVersion": 1,
  "id": "gotime-extension",
  "publisher": "RDCPartner",
  "name": "GOTime",
  "description": "...",
  "categories": ["Azure Boards"],
  "targets": [
    {
      "id": "Microsoft.VisualStudio.Services"
    }
  ],
  "icons": {
    "default": "static/logo.png"
  },
  "content": {
    "details": {
      "path": "overview.md"
    }
  },
  "files": [
    {
      "path": "static",
      "addressable": true
    },
    {
      "path": "dist",
      "addressable": true
    }
  ],
  "tags": [
    "time-tracking",
    "timesheet",
    "calendar",
    "reporting",
    "capacity",
    "productivity",
    "azure-boards",
    "work-items"
  ],
  "public": true,
  "preview": true,
  "pricing": {
    "trialDays": 365,
    "plans": [
      {
        "id": "free-trial",
        "name": "Free Trial",
        "price": 0,
        "visible": true
      }
    ]
  }
}
```

**ÖNEMLİ NOT**: `pricing` alanı manifest'te desteklenmeyebilir. Pricing bilgileri genellikle Marketplace web arayüzü üzerinden yapılandırılır.

### 3. Marketplace Web Arayüzünde Pricing Ayarları

1. **Marketplace Manage sayfasına gidin**: https://marketplace.visualstudio.com/manage
2. **GOTime extension'ını seçin**
3. **Pricing & Availability** sekmesine gidin
4. Aşağıdaki ayarları yapın:

#### Pricing Model
- **Pricing Model**: "Free Trial" veya "Paid (with free trial)"
- **Free Trial Duration**: 365 days (1 year)
- **Price**: Ücretsiz (trial için)

#### Availability
- **Public**: Yes (herkese açık)
- **Preview**: İsterseniz "No" yapabilirsiniz (production için)

### 4. Privacy Policy ve EULA URL'leri

Privacy Policy ve EULA dosyalarını şu yollardan birinde yayınlayın:

#### Seçenek 1: GitHub (Önerilen)
1. Repository'ye PRIVACY_POLICY.md ve EULA.md dosyalarını commit edin
2. GitHub'da raw URL'leri alın:
   - `https://raw.githubusercontent.com/[username]/[repo]/main/PRIVACY_POLICY.md`
   - `https://raw.githubusercontent.com/[username]/[repo]/main/EULA.md`

#### Seçenek 2: Kendi Domain'iniz
1. Dosyaları kendi web sunucunuzda yayınlayın
2. URL'leri not edin:
   - `https://www.rdcpartner.com/privacy-policy`
   - `https://www.rdcpartner.com/eula`

#### Seçenek 3: GitHub Pages
1. GitHub Pages kullanarak static site oluşturun
2. URL'ler:
   - `https://[username].github.io/[repo]/PRIVACY_POLICY.md`
   - `https://[username].github.io/[repo]/EULA.md`

### 5. Marketplace Extension Sayfası Güncelleme

1. Extension'ın marketplace sayfasına gidin
2. **Manage** butonuna tıklayın
3. **Overview** sekmesinde:
   - **Description**: Güncel ve detaylı açıklama
   - **Release Notes**: Son sürüm notları
   - **Categories**: Doğru kategoriler seçili mi kontrol edin

4. **Support** sekmesinde:
   - **Support URL**: `support@rdcpartner.com` veya support sayfanız
   - **Documentation URL**: GitHub README veya docs sayfanız

5. **Legal** sekmesinde:
   - **Privacy Policy URL**: Yukarıda hazırladığınız URL
   - **Terms of Service/EULA URL**: Yukarıda hazırladığınız URL

### 6. Publisher Profile Kontrolü

1. **Publisher Profile** sayfasına gidin
2. Aşağıdaki bilgilerin dolu olduğundan emin olun:
   - Company name
   - Company logo
   - Support contact information
   - Privacy Policy URL
   - Terms of Service URL

## Adım Adım Checklist

- [ ] PRIVACY_POLICY.md dosyasını GitHub'a commit edin
- [ ] EULA.md dosyasını GitHub'a commit edin
- [ ] SUPPORT.md dosyasını GitHub'a commit edin (opsiyonel)
- [ ] Privacy Policy URL'ini alın (GitHub raw veya kendi domain)
- [ ] EULA URL'ini alın (GitHub raw veya kendi domain)
- [ ] Marketplace Publisher Profile'da Support URL'i ekleyin/güncelleyin
- [ ] Marketplace Publisher Profile'da Privacy Policy URL'ini ekleyin
- [ ] Marketplace Publisher Profile'da Terms of Service URL'ini ekleyin
- [ ] Extension'ın Pricing & Availability ayarlarını yapın (Free Trial: 365 days)
- [ ] Extension'ın Overview sayfasını güncelleyin
- [ ] Extension'ın Legal sekmesini doldurun
- [ ] Extension'ı yeniden publish edin (gerekirse)
- [ ] Marketplace'te extension'ın doğru göründüğünü kontrol edin

## Önemli Notlar

1. **Free Trial vs Paid**: Extension şu anda "free" olarak görünüyor. Marketplace'te "Free Trial" modeline geçmek için:
   - Pricing & Availability'de "Free Trial" seçeneğini etkinleştirin
   - Trial süresini 365 gün olarak ayarlayın
   - İleride ücretli plan ekleyebilirsiniz

2. **Top Publisher Listesi**: Top Publisher listesinde görünmek için:
   - Privacy Policy URL'i dolu olmalı
   - Terms of Service/EULA URL'i dolu olmalı
   - Support information dolu olmalı
   - Extension'ın aktif install sayısı yeterli olmalı

3. **Publishing**: Değişiklikler yapıldıktan sonra extension'ı yeniden publish etmeniz gerekebilir.

## İletişim

Sorularınız için: support@rdcpartner.com

