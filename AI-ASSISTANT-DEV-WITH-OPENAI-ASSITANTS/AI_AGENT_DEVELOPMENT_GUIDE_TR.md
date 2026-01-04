# AI Asistan Entegrasyon Rehberi
## OpenAI Asistanlarını Web Sitenize Bağlama

**Üniversite Öğrencileri İçin Adım Adım Öğretici**

---

## İçindekiler

1. [Genel Bakış: AI Asistanlar Web Sitelerinde Nasıl Çalışır](#overview)
2. [Adım 1: OpenAI API Anahtarı Oluşturma](#step-1-create-openai-api-key)
3. [Adım 2: AI Asistanınızı Oluşturma](#step-2-create-your-ai-assistant)
4. [Adım 3: Asistan Ayarlarını Yapılandırma](#step-3-configure-assistant-settings)
5. [Adım 4: Entegrasyon Mimarisi](#step-4-understanding-the-integration-architecture)
6. [Adım 5: Backend Entegrasyon İş Akışı](#step-5-backend-integration-workflow)
7. [Adım 6: Frontend Entegrasyon İş Akışı](#step-6-frontend-integration-workflow)

---

## Genel Bakış: AI Asistanlar Web Sitelerinde Nasıl Çalışır {#overview}

### Büyük Resim

Bir kullanıcı web sitenizdeki bir AI asistanı ile etkileşime girdiğinde, şunlar olur:

```
Kullanıcı Mesaj Yazıyor
       ↓
Frontend (Web Sitesi) 
       ↓ HTTP İsteği
Backend Service (Node.js/Express)
       ↓ OpenAI SDK
OpenAI Assistants API
       ↓ İşleme ve Yanıt Üretme
Backend Service
       ↓ HTTP Yanıtı
Frontend (Web Sitesi)
       ↓
Kullanıcıya Yanıt Gösterme
```

### Temel Bileşenler

1. **Frontend (Web Sitesi)**: Kullanıcıların mesaj yazdığı kullanıcı arayüzü
2. **Backend Service**: Frontend'i OpenAI'ye bağlayan orta katman
3. **OpenAI Assistants API**: Mesajları işleyen AI beyni
4. **Threads**: OpenAI tarafından tutulan konuşma geçmişi

### Neden Backend'e İhtiyacımız Var?

- **Güvenlik**: API anahtarları sunucuda kalmalı, asla frontend kodunda olmamalı
- **Kontrol**: Backend iş mantığı, doğrulama, hız sınırlaması ekleyebilir
- **Esneklik**: Veritabanları, harici API'ler vb. ile entegre edilebilir

---

## Adım 1: OpenAI API Anahtarı Oluşturma {#step-1-create-openai-api-key}

### 1.1 API Anahtarınızı Alın

1. https://platform.openai.com adresine gidin
2. Hesabınıza kaydolun veya giriş yapın
3. **API Keys** bölümüne gidin (https://platform.openai.com/api-keys)
4. **"Create new secret key"** butonuna tıklayın
5. **ÖNEMLİ**: Anahtarı hemen kopyalayın - bir daha göremezsiniz!
   - Format: `sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 1.2 Güvenli Saklama

- Backend `.env` dosyasına kaydedin (asla Git'e commit etmeyin)
- `.env` dosyasını `.gitignore`'a ekleyin
- Frontend kodunda asla açığa çıkarmayın

---

## Adım 2: AI Asistanınızı Oluşturma {#step-2-create-your-ai-assistant}

### 2.1 Assistants Bölümüne Gidin

1. https://platform.openai.com/assistants adresine gidin
2. **"Create"** butonuna tıklayın

### 2.2 Temel Ayarları Yapılandırın

#### **İsim**
- Referansınız için açıklayıcı bir isim
- Örnek: `Şirket Destek Asistanı`

#### **Açıklama**
- Asistanın amacının kısa özeti
- Örnek: `Kullanıcılara şirket bilgileri ve toplantı rezervasyonlarında yardımcı olur`

#### **Talimatlar (System Context - Sistem Bağlamı)**
Bu en önemli alan - asistanınızın davranışını tanımlar.

**Örnek:**
```
Intelligent Loops için dostane bir müşteri destek asistanısınız.

Sorumluluklarınız:
- Şirket hizmetleri hakkında soruları yanıtlamak
- Kullanıcılara toplantı rezervasyonunda yardımcı olmak
- Ürünler ve fiyatlandırma hakkında bilgi sağlamak

Yönergeler:
- Her zaman profesyonel ve yardımcı olun
- Toplantı rezervasyonu yaparken önce müsaitliği kontrol edin
- Rezervasyonları tamamlamadan önce tüm detayları onaylayın
```

**Önemli Noktalar:**
- Asistanın rolü hakkında spesifik olun
- Nasıl davranması gerektiğini tanımlayın
- Fonksiyonları ne zaman kullanacağını belirtin (varsa)
- Tonu belirleyin (profesyonel, dostane, vb.)

### 2.3 Kaydetme ve Assistant ID Alma

1. **"Save"** butonuna tıklayın
2. **Assistant ID'yi kopyalayın** (`asst_...` ile başlar)
   - Örnek: `asst_abc123xyz789`
   - Backend yapılandırması için buna ihtiyacınız olacak

---

## Adım 3: Asistan Ayarlarını Yapılandırma {#step-3-configure-assistant-settings}

### 3.1 Model Seçimi

**Mevcut Modeller:**
- **GPT-4 Turbo**: En iyi kalite, daha yüksek maliyet
- **GPT-4**: Yüksek kalite, iyi denge
- **GPT-3.5 Turbo**: Daha hızlı, daha ucuz, basit görevler için iyi

**Öneri**: En iyi sonuçlar için `gpt-4-turbo-preview` veya `gpt-4o` ile başlayın

### 3.2 Temperature (Sıcaklık)

Yanıtların ne kadar yaratıcı/belirleyici olduğunu kontrol eder.

- **0.0 - 0.3**: Çok odaklı, tutarlı (gerçek bilgiler)
- **0.4 - 0.7**: Dengeli (Müşteri desteği için önerilir)
- **0.8 - 1.2**: Daha yaratıcı, çeşitli yanıtlar
- **1.3 - 2.0**: Çok yaratıcı, öngörülemez

**Önerilen**: Çoğu kullanım durumu için `0.7`

### 3.3 Top P (Temperature Alternatifi)

- Aralık: 0.0 ile 1.0 arası
- Yanıt çeşitliliğini kontrol eder
- Ya Temperature ya da Top P kullanın, ikisini birden değil
- **Önerilen**: Varsayılan olarak bırakın (1.0) veya `0.7` olarak ayarlayın

### 3.4 Araçlar (Functions - Fonksiyonlar)

Asistanınızın çağırabileceği fonksiyonları burada tanımlarsınız.

#### **Fonksiyon Ekleme**

1. **"Add tool"** butonuna tıklayın → **"Function"** seçin
2. Fonksiyonu JSON Schema formatında tanımlayın

**Fonksiyon Tanımı Örneği:**
```json
{
  "type": "function",
  "function": {
    "name": "check_availability",
    "description": "Rezervasyon için bir tarih ve saat diliminin müsait olup olmadığını kontrol et",
    "parameters": {
      "type": "object",
      "properties": {
        "date": {
          "type": "string",
          "description": "YYYY-MM-DD formatında tarih"
        },
        "time": {
          "type": "string",
          "description": "HH:MM formatında saat"
        }
      },
      "required": ["date", "time"]
    }
  }
}
```

**Temel Bileşenler:**
- **name**: Backend fonksiyon adınızla eşleşmeli
- **description**: AI bunu fonksiyonu ne zaman çağıracağına karar vermek için kullanır
- **parameters**: Fonksiyonun ihtiyaç duyduğu girdileri tanımlayın
- **required**: Gerekli parametrelerin listesi

**Yaygın Fonksiyonlar:**
- `check_availability` - Toplantı müsaitliğini kontrol et
- `get_current_date_time` - Mevcut tarih/saati al
- `book_meeting` - Toplantı rezerve et

### 3.5 Dosya Arama (İsteğe Bağlı)

- Asistanın yüklenen belgelerde arama yapmasına izin verir
- Bilgi tabanları, SSS, dokümantasyon için yararlı
- **"Add tool"** butonuna tıklayın → **"File search"** seçin → Dosyaları yükleyin

### 3.6 Yapılandırmayı Kaydetme

Tüm ayarları gözden geçirin ve **"Save"** butonuna tıklayın

---

## Adım 4: Entegrasyon Mimarisi {#step-4-understanding-the-integration-architecture}

### 4.1 Üç Katmanlı Mimari

```
┌─────────────────────────────────────────┐
│         FRONTEND (Web Sitesi)            │
│  - Kullanıcı mesaj yazar                │
│  - Backend'e HTTP POST gönderir          │
│  - Yanıtı gösterir                      │
└──────────────┬──────────────────────────┘
               │ HTTP İsteği
               │ { messages, threadId }
               ▼
┌─────────────────────────────────────────┐
│         BACKEND SERVICE                  │
│  - Frontend'den istek alır              │
│  - OpenAI API'ye bağlanır               │
│  - Fonksiyon çağrılarını yönetir       │
│  - Frontend'e yanıt döner               │
└──────────────┬──────────────────────────┘
               │ OpenAI SDK
               │ openai.beta.threads.*
               ▼
┌─────────────────────────────────────────┐
│      OPENAI ASSISTANTS API               │
│  - Mesajları işler                      │
│  - Konuşma thread'lerini tutar          │
│  - Gerektiğinde fonksiyonları çağırır   │
│  - Yanıtlar üretir                      │
└─────────────────────────────────────────┘
```

### 4.2 Veri Akışı

**Adım Adım Akış:**

1. **Kullanıcı mesaj gönderir** → Frontend girdiyi yakalar
2. **Frontend → Backend**: Mesaj ve threadId ile HTTP POST
3. **Backend → OpenAI**: Thread oluştur/al, mesaj ekle, asistanı çalıştır
4. **OpenAI işler**: Gerekirse fonksiyonları çağırabilir
5. **Backend fonksiyonları yönetir**: İş mantığını çalıştırır
6. **Backend → OpenAI**: Fonksiyon sonuçlarını gönder
7. **OpenAI yanıt üretir**: Son cevap hazır
8. **Backend → Frontend**: Yanıt ve threadId döndür
9. **Frontend gösterir**: Yanıtı kullanıcıya göster

### 4.3 Threads Açıklaması

**Thread Nedir?**
- OpenAI tarafından tutulan bir konuşma konteyneri
- Bir konuşmadaki tüm mesajları saklar
- Her konuşma benzersiz bir `threadId` alır

**Threads Neden Önemli:**
- Konuşma bağlamını korur
- Çoklu dönüşlü konuşmalara izin verir
- Frontend konuşmaları sürdürmek için `threadId` saklar

**Thread Yaşam Döngüsü:**
1. İlk mesaj: Yeni thread oluştur (threadId = null)
2. OpenAI yeni threadId döndürür
3. Sonraki mesajlar: Mevcut threadId'yi kullan
4. Konuşma tam bağlamla devam eder

---

## Adım 5: Backend Entegrasyon İş Akışı {#step-5-backend-integration-workflow}

### 5.1 Backend Kurulum Genel Bakış

Backend'inizin yapması gerekenler:
1. Frontend'den istekleri almak
2. SDK kullanarak OpenAI'ye bağlanmak
3. Thread'leri ve mesajları yönetmek
4. Fonksiyon çağrılarını yönetmek
5. Frontend'e yanıtları döndürmek

### 5.2 OpenAI SDK'yı Başlatma

**Paketi Yükleyin:**
```bash
npm install openai
```

**İstemciyi Başlatın:**
```javascript
require('dotenv').config();
const OpenAI = require('openai');

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY  // .env dosyasından
});
```

### 5.3 Ortam Yapılandırması

`.env` dosyası oluşturun:
```env
OPENAI_API_KEY=sk-proj-your-key-here
```

### 5.4 Temel Entegrasyon İş Akışı

Backend'inizin uygulaması gereken temel iş akışı:

#### **Adım 1: Thread Oluştur veya Al**

```javascript
let thread_id = threadId; // Frontend isteğinden

if (!thread_id) {
  // Yeni konuşma için yeni thread oluştur
  const thread = await openai.beta.threads.create();
  thread_id = thread.id;
}
```

#### **Adım 2: Kullanıcı Mesajını Ekle**

```javascript
const userMessage = messages[messages.length - 1].content;

await openai.beta.threads.messages.create(thread_id, {
  role: "user",
  content: userMessage
});
```

#### **Adım 3: Asistanı Çalıştır**

```javascript
const run = await openai.beta.threads.runs.create(thread_id, {
  assistant_id: assistantId  // .env dosyanızdan
});
```

#### **Adım 4: Tamamlanmayı Bekle**

```javascript
let runStatus = await openai.beta.threads.runs.retrieve(thread_id, run.id);

while (runStatus.status !== 'completed') {
  // Farklı durumları yönet
  if (runStatus.status === 'requires_action') {
    // Fonksiyon çağrıları gerekli - bunları yönet
  }
  
  // Bekle ve tekrar kontrol et
  await new Promise(resolve => setTimeout(resolve, 1000));
  runStatus = await openai.beta.threads.runs.retrieve(thread_id, run.id);
}
```

#### **Adım 5: Yanıtı Al**

```javascript
const messages_response = await openai.beta.threads.messages.list(thread_id);
const lastMessage = messages_response.data[0];
const responseContent = lastMessage.content[0].text.value;

return { 
  content: responseContent, 
  threadId: thread_id 
};
```

### 5.5 API Endpoint Yapısı

Backend'inizin şuna benzer bir endpoint'e ihtiyacı var:

```
POST /api/chat
Body: {
  messages: [{ role: 'user', content: 'Merhaba' }],
  threadId: null  // veya mevcut threadId
}
Response: {
  content: "Merhaba! Size nasıl yardımcı olabilirim?",
  threadId: "thread_abc123"
}
```

### 5.6 Temel Backend Sorumlulukları

1. **Thread Yönetimi**: Yeni thread'ler oluştur, mevcut olanları kullan
2. **Mesaj Yönetimi**: Kullanıcı mesajlarını ekle, asistan yanıtlarını al
3. **Fonksiyon Çağırma**: Asistan istediğinde fonksiyonları çalıştır
4. **Hata Yönetimi**: API hatalarını zarifçe yönet
5. **Güvenlik**: API anahtarlarını koru, girdileri doğrula

---

## Adım 6: Frontend Entegrasyon İş Akışı {#step-6-frontend-integration-workflow}

### 6.1 Frontend Kurulum Genel Bakış

Frontend'inizin yapması gerekenler:
1. Kullanıcı girdisini yakalamak
2. Backend'e HTTP istekleri göndermek
3. Yanıtları göstermek
4. Konuşma sürekliliği için threadId'yi korumak

### 6.2 API Yapılandırması

**API config oluşturun:**
```javascript
const API_CONFIG = {
  CHAT_API: 'http://localhost:3001/api/chat'
};
```

### 6.3 Mesaj Gönderme

**Temel Akış:**
```javascript
const sendMessage = async (userMessage) => {
  // Mesaj dizisini hazırla
  const messages = [
    { role: 'user', content: userMessage }
  ];
  
  // Backend'e gönder
  const response = await fetch(API_CONFIG.CHAT_API, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ 
      messages: messages, 
      threadId: threadId  // yeni için null, veya mevcut threadId
    })
  });
  
  const data = await response.json();
  
  // Yanıtı göster
  displayMessage(data.content);
  
  // Bir sonraki mesaj için threadId'yi sakla
  threadId = data.threadId;
};
```

### 6.4 Konuşma Bağlamını Koruma

**threadId'yi saklayın:**
- İlk mesaj: `threadId = null`
- Backend yeni `threadId` döndürür
- Bileşen durumunda veya localStorage'da saklayın
- Sonraki mesajlar için aynı `threadId`'yi kullanın

**Örnek:**
```javascript
let threadId = null; // null ile başla

// İlk mesaj
const response1 = await sendMessage("Merhaba");
threadId = response1.threadId; // Sakla

// İkinci mesaj (aynı threadId'yi kullanır)
const response2 = await sendMessage("Daha fazla anlat", threadId);
// Konuşma bağlamı korunur!
```

### 6.5 Kullanıcı Deneyimi Düşünceleri

1. **Yükleme Durumları**: Beklerken "yazıyor..." göstergesi göster
2. **Hata Yönetimi**: Dostane hata mesajları göster
3. **Mesaj Geçmişi**: Konuşma geçmişini göster
4. **Thread Yönetimi**: Yeni konuşma başlatma seçeneği

---

## Temel Çıkarımlar

### Entegrasyon Temelleri

1. **Backend Gereklidir**: OpenAI API'yi asla frontend'den doğrudan çağırmayın
2. **Threads Bağlamı Korur**: Konuşmaları sürdürmek için threadId'yi saklayın
3. **Fonksiyon Çağırma Eylemleri Mümkün Kılar**: AI'ı iş mantığınıza bağlayın
4. **Güvenlik Öncelikli**: API anahtarlarını sadece backend'de tutun

### İş Akışı Özeti

```
Frontend → Backend → OpenAI → Backend → Frontend
   ↓         ↓         ↓         ↓         ↓
Kullanıcı HTTP    İşleme   Fonksiyon  Gösterme
Girdisi  İsteği  Mesaj     Sonuçları  Yanıt
```

### En İyi Uygulamalar

- ✅ API anahtarlarını ortam değişkenlerinde saklayın
- ✅ Hataları zarifçe yönetin
- ✅ Konuşma thread'lerini koruyun
- ✅ Fonksiyon çağırma işlemini kapsamlı şekilde test edin
- ✅ API kullanımını ve maliyetleri izleyin
- ✅ UI'da yükleme durumları sağlayın
- ✅ Tüm girdileri doğrulayın

---

## Kaynaklar

- **OpenAI Assistants API**: https://platform.openai.com/docs/assistants
- **OpenAI Node.js SDK**: https://github.com/openai/openai-node
- **Fonksiyon Çağırma Rehberi**: https://platform.openai.com/docs/guides/function-calling
- **API Referansı**: https://platform.openai.com/docs/api-reference
