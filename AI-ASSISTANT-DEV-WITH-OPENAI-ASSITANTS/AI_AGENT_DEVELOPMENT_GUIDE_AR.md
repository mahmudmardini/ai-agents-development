# دليل دمج مساعد الذكاء الاصطناعي
## ربط مساعدات OpenAI بموقعك الإلكتروني

**دليل تعليمي خطوة بخطوة لطلاب الجامعة**

---

## المحتويات

1. [نظرة عامة: كيف تعمل مساعدات الذكاء الاصطناعي مع المواقع](#overview)
2. [الخطوة 1: إنشاء مفتاح OpenAI API](#step-1)
3. [الخطوة 2: إنشاء مساعد الذكاء الاصطناعي](#step-2)
4. [الخطوة 3: إعدادات المساعد](#step-3)
5. [الخطوة 4: فهم بنية الدمج](#step-4)
6. [الخطوة 5: سير عمل دمج Backend](#step-5)
7. [الخطوة 6: سير عمل دمج Frontend](#step-6)

---

## نظرة عامة: كيف تعمل مساعدات الذكاء الاصطناعي مع المواقع {#overview}

### الصورة الكبيرة

عندما يتفاعل المستخدم مع مساعد الذكاء الاصطناعي على موقعك، إليك ما يحدث:

```
المستخدم يكتب رسالة
       ↓
Frontend (الموقع الإلكتروني) 
       ↓ طلب HTTP
Backend Service (Node.js/Express)
       ↓ OpenAI SDK
OpenAI Assistants API
       ↓ معالجة وإنشاء الرد
Backend Service
       ↓ استجابة HTTP
Frontend (الموقع الإلكتروني)
       ↓
عرض الرد للمستخدم
```

### المكونات الرئيسية

1. **Frontend (الموقع الإلكتروني)**: واجهة المستخدم حيث يكتب المستخدمون الرسائل
2. **Backend Service**: الطبقة الوسطى التي تربط Frontend بـ OpenAI
3. **OpenAI Assistants API**: العقل الاصطناعي الذي يعالج الرسائل
4. **Threads (الخيوط)**: تاريخ المحادثة الذي يحتفظ به OpenAI

### لماذا نحتاج Backend؟

- **الأمان**: يجب أن تبقى مفاتيح API على الخادم، وليس في كود Frontend
- **التحكم**: يمكن للـ Backend إضافة منطق الأعمال، التحقق، وتحديد معدل الطلبات
- **المرونة**: يمكن التكامل مع قواعد البيانات، وAPIs الخارجية، إلخ

---

## الخطوة 1: إنشاء مفتاح OpenAI API {#step-1}

### 1.1 الحصول على مفتاح API

1. اذهب إلى https://platform.openai.com
2. سجل حساباً جديداً أو سجل الدخول إلى حسابك
3. انتقل إلى **API Keys** (https://platform.openai.com/api-keys)
4. انقر على **"Create new secret key"**
5. **مهم جداً**: انسخ المفتاح فوراً - لن تتمكن من رؤيته مرة أخرى!
   - التنسيق: `sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 1.2 التخزين الآمن

- احفظه في ملف `.env` في الـ Backend (لا ترفعه إلى Git أبداً)
- أضف `.env` إلى `.gitignore`
- لا تعرضه في كود Frontend أبداً

---

## الخطوة 2: إنشاء مساعد الذكاء الاصطناعي {#step-2}

### 2.1 الانتقال إلى Assistants

1. اذهب إلى https://platform.openai.com/assistants
2. انقر على زر **"Create"**

### 2.2 إعدادات المساعد الأساسية

#### **الاسم**
- اسم وصفي للرجوع إليه
- مثال: `مساعد دعم الشركة`

#### **الوصف**
- ملخص موجز لهدف المساعد
- مثال: `يساعد المستخدمين في معلومات الشركة وحجز الاجتماعات`

#### **التعليمات (System Context - السياق)**
هذا هو الحقل الأهم - يحدد سلوك مساعدك.

**مثال:**
```
أنت مساعد دعم عملاء ودود لشركة Intelligent Loops.

مسؤولياتك:
- الإجابة على أسئلة حول خدمات الشركة
- مساعدة المستخدمين في حجز الاجتماعات
- تقديم معلومات حول المنتجات والأسعار

الإرشادات:
- كن دائماً مهنياً ومفيداً
- عند حجز الاجتماعات، تحقق من التوفر أولاً
- أكد جميع التفاصيل قبل إتمام الحجز
```

**نقاط مهمة:**
- كن محدداً حول دور المساعد
- حدد كيف يجب أن يتصرف
- اذكر متى يستخدم الدوال (إن وجدت)
- حدد النبرة (مهني، ودود، إلخ)

### 2.3 الحفظ والحصول على Assistant ID

1. انقر على **"Save"**
2. **انسخ Assistant ID** (يبدأ بـ `asst_...`)
   - مثال: `asst_abc123xyz789`
   - ستحتاج هذا لإعدادات الـ Backend

---

## الخطوة 3: إعدادات المساعد {#step-3}

### 3.1 اختيار النموذج (Model)

**النماذج المتاحة:**
- **GPT-4 Turbo**: أفضل جودة، تكلفة أعلى
- **GPT-4**: جودة عالية، توازن جيد
- **GPT-3.5 Turbo**: أسرع، أرخص، جيد للمهام البسيطة

**التوصية**: ابدأ بـ `gpt-4-turbo-preview` أو `gpt-4o` للحصول على أفضل النتائج

### 3.2 Temperature (درجة الحرارة)

تتحكم في مدى إبداع/حتمية الردود.

- **0.0 - 0.3**: مركز جداً، متسق (معلومات واقعية)
- **0.4 - 0.7**: متوازن (موصى به لدعم العملاء)
- **0.8 - 1.2**: أكثر إبداعاً، ردود متنوعة
- **1.3 - 2.0**: إبداعي جداً، غير متوقع

**موصى به**: `0.7` لمعظم حالات الاستخدام

### 3.3 Top P (بديل لـ Temperature)

- النطاق: 0.0 إلى 1.0
- يتحكم في تنوع الردود
- استخدم إما Temperature أو Top P، وليس كلاهما
- **موصى به**: اتركه افتراضياً (1.0) أو اضبطه على `0.7`

### 3.4 الأدوات (Functions - الدوال)

هنا تحدد ما هي الدوال التي يمكن لمساعدك استدعاؤها.

#### **إضافة الدوال**

1. انقر على **"Add tool"** → اختر **"Function"**
2. حدد الدالة بتنسيق JSON Schema

**مثال على تعريف الدالة:**
```json
{
  "type": "function",
  "function": {
    "name": "check_availability",
    "description": "التحقق من توفر موعد ووقت محدد للحجز",
    "parameters": {
      "type": "object",
      "properties": {
        "date": {
          "type": "string",
          "description": "التاريخ بتنسيق YYYY-MM-DD"
        },
        "time": {
          "type": "string",
          "description": "الوقت بتنسيق HH:MM"
        }
      },
      "required": ["date", "time"]
    }
  }
}
```

**المكونات الرئيسية:**
- **name**: يجب أن يطابق اسم الدالة في الـ Backend
- **description**: يستخدمه الذكاء الاصطناعي ليقرر متى يستدعي الدالة
- **parameters**: حدد ما هي المدخلات التي تحتاجها الدالة
- **required**: قائمة بالمعاملات المطلوبة

**الدوال الشائعة:**
- `check_availability` - التحقق من توفر الاجتماع
- `get_current_date_time` - الحصول على التاريخ/الوقت الحالي
- `book_meeting` - حجز اجتماع

### 3.5 البحث في الملفات (اختياري)

- يسمح للمساعد بالبحث في المستندات المرفوعة
- مفيد لقواعد المعرفة، الأسئلة الشائعة، التوثيق
- انقر على **"Add tool"** → اختر **"File search"** → ارفع الملفات

### 3.6 حفظ الإعدادات

راجع جميع الإعدادات وانقر على **"Save"**

---

## الخطوة 4: فهم بنية الدمج {#step-4}

### 4.1 البنية ثلاثية الطبقات

```
┌─────────────────────────────────────────┐
│         FRONTEND (الموقع الإلكتروني)     │
│  - المستخدم يكتب رسالة                  │
│  - يرسل HTTP POST إلى Backend          │
│  - يعرض الرد                            │
└──────────────┬──────────────────────────┘
               │ طلب HTTP
               │ { messages, threadId }
               ▼
┌─────────────────────────────────────────┐
│         BACKEND SERVICE                 │
│  - يستقبل الطلب من Frontend            │
│  - يتصل بـ OpenAI API                  │
│  - يتعامل مع استدعاءات الدوال          │
│  - يعيد الرد إلى Frontend              │
└──────────────┬──────────────────────────┘
               │ OpenAI SDK
               │ openai.beta.threads.*
               ▼
┌─────────────────────────────────────────┐
│      OPENAI ASSISTANTS API              │
│  - يعالج الرسائل                        │
│  - يحافظ على خيوط المحادثة              │
│  - يستدعي الدوال عند الحاجة            │
│  - يولد الردود                          │
└─────────────────────────────────────────┘
```

### 4.2 تدفق البيانات

**التدفق خطوة بخطوة:**

1. **المستخدم يرسل رسالة** → Frontend يلتقط المدخلات
2. **Frontend → Backend**: HTTP POST مع الرسالة و threadId
3. **Backend → OpenAI**: إنشاء/استرجاع thread، إضافة الرسالة، تشغيل المساعد
4. **OpenAI يعالج**: قد يستدعي دوال إذا لزم الأمر
5. **Backend يتعامل مع الدوال**: ينفذ منطق الأعمال
6. **Backend → OpenAI**: إرسال نتائج الدوال
7. **OpenAI يولد الرد**: الرد النهائي جاهز
8. **Backend → Frontend**: إرجاع الرد و threadId
9. **Frontend يعرض**: عرض الرد للمستخدم

### 4.3 شرح Threads (الخيوط)

**ما هو Thread؟**
- حاوية محادثة يحتفظ بها OpenAI
- يخزن جميع الرسائل في المحادثة
- كل محادثة تحصل على `threadId` فريد

**لماذا Threads مهمة:**
- تحافظ على سياق المحادثة
- تسمح بمحادثات متعددة الجولات
- Frontend يخزن `threadId` لمواصلة المحادثات

**دورة حياة Thread:**
1. الرسالة الأولى: إنشاء thread جديد (threadId = null)
2. OpenAI يعيد threadId جديد
3. الرسائل اللاحقة: استخدام threadId الموجود
4. المحادثة تستمر مع السياق الكامل

---

## الخطوة 5: سير عمل دمج Backend {#step-5}

### 5.1 نظرة عامة على إعداد Backend

يحتاج الـ Backend إلى:
1. استقبال الطلبات من Frontend
2. الاتصال بـ OpenAI باستخدام SDK
3. إدارة Threads والرسائل
4. التعامل مع استدعاءات الدوال
5. إرجاع الردود إلى Frontend

### 5.2 تهيئة OpenAI SDK

**تثبيت الحزمة:**
```bash
npm install openai
```

**تهيئة العميل:**
```javascript
require('dotenv').config();
const OpenAI = require('openai');

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY  // من ملف .env
});
```

### 5.3 إعدادات البيئة

أنشئ ملف `.env`:
```env
OPENAI_API_KEY=sk-proj-your-key-here
```

### 5.4 سير عمل الدمج الأساسي

إليك سير العمل الأساسي الذي يحتاج الـ Backend لتنفيذه:

#### **الخطوة 1: إنشاء أو استرجاع Thread**

```javascript
let thread_id = threadId; // من طلب Frontend

if (!thread_id) {
  // إنشاء thread جديد لمحادثة جديدة
  const thread = await openai.beta.threads.create();
  thread_id = thread.id;
}
```

#### **الخطوة 2: إضافة رسالة المستخدم**

```javascript
const userMessage = messages[messages.length - 1].content;

await openai.beta.threads.messages.create(thread_id, {
  role: "user",
  content: userMessage
});
```

#### **الخطوة 3: تشغيل المساعد**

```javascript
const run = await openai.beta.threads.runs.create(thread_id, {
  assistant_id: assistantId  // من ملف .env الخاص بك
});
```

#### **الخطوة 4: الانتظار حتى الإكمال**

```javascript
let runStatus = await openai.beta.threads.runs.retrieve(thread_id, run.id);

while (runStatus.status !== 'completed') {
  // التعامل مع الحالات المختلفة
  if (runStatus.status === 'requires_action') {
    // استدعاءات الدوال مطلوبة - تعامل معها
  }
  
  // انتظر وتحقق مرة أخرى
  await new Promise(resolve => setTimeout(resolve, 1000));
  runStatus = await openai.beta.threads.runs.retrieve(thread_id, run.id);
}
```

#### **الخطوة 5: الحصول على الرد**

```javascript
const messages_response = await openai.beta.threads.messages.list(thread_id);
const lastMessage = messages_response.data[0];
const responseContent = lastMessage.content[0].text.value;

return { 
  content: responseContent, 
  threadId: thread_id 
};
```

### 5.5 بنية API Endpoint

يحتاج الـ Backend إلى endpoint مثل:

```
POST /api/chat
Body: {
  messages: [{ role: 'user', content: 'مرحبا' }],
  threadId: null  // أو threadId موجود
}
Response: {
  content: "مرحبا! كيف يمكنني مساعدتك؟",
  threadId: "thread_abc123"
}
```

### 5.6 مسؤوليات Backend الرئيسية

1. **إدارة Threads**: إنشاء threads جديدة، استخدام الموجودة
2. **معالجة الرسائل**: إضافة رسائل المستخدم، استرجاع ردود المساعد
3. **استدعاء الدوال**: تنفيذ الدوال عندما يطلبها المساعد
4. **معالجة الأخطاء**: التعامل مع أخطاء API بشكل لائق
5. **الأمان**: حماية مفاتيح API، التحقق من المدخلات

---

## الخطوة 6: سير عمل دمج Frontend {#step-6}

### 6.1 نظرة عامة على إعداد Frontend

يحتاج الـ Frontend إلى:
1. التقاط مدخلات المستخدم
2. إرسال طلبات HTTP إلى Backend
3. عرض الردود
4. الحفاظ على threadId لاستمرارية المحادثة

### 6.2 إعدادات API

**إنشاء إعدادات API:**
```javascript
const API_CONFIG = {
  CHAT_API: 'http://localhost:3001/api/chat'
};
```

### 6.3 إرسال الرسائل

**التدفق الأساسي:**
```javascript
const sendMessage = async (userMessage) => {
  // إعداد مصفوفة الرسائل
  const messages = [
    { role: 'user', content: userMessage }
  ];
  
  // إرسال إلى Backend
  const response = await fetch(API_CONFIG.CHAT_API, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ 
      messages: messages, 
      threadId: threadId  // null للمحادثة الجديدة، أو threadId موجود
    })
  });
  
  const data = await response.json();
  
  // عرض الرد
  displayMessage(data.content);
  
  // حفظ threadId للرسالة التالية
  threadId = data.threadId;
};
```

### 6.4 الحفاظ على سياق المحادثة

**حفظ threadId:**
- الرسالة الأولى: `threadId = null`
- Backend يعيد threadId جديد
- احفظه في حالة المكون أو localStorage
- استخدم نفس threadId للرسائل اللاحقة

**مثال:**
```javascript
let threadId = null; // ابدأ بـ null

// الرسالة الأولى
const response1 = await sendMessage("مرحبا");
threadId = response1.threadId; // احفظه

// الرسالة الثانية (تستخدم نفس threadId)
const response2 = await sendMessage("أخبرني المزيد", threadId);
// سياق المحادثة محفوظ!
```

### 6.5 اعتبارات تجربة المستخدم

1. **حالات التحميل**: اعرض مؤشر "يكتب..." أثناء الانتظار
2. **معالجة الأخطاء**: اعرض رسائل خطأ ودودة
3. **تاريخ الرسائل**: اعرض تاريخ المحادثة
4. **إدارة Threads**: خيار لبدء محادثة جديدة

---

## النقاط الرئيسية

### أساسيات الدمج

1. **Backend مطلوب**: لا تستدعي OpenAI API مباشرة من Frontend
2. **Threads تحافظ على السياق**: احفظ threadId لمواصلة المحادثات
3. **استدعاء الدوال يمكّن الإجراءات**: اربط الذكاء الاصطناعي بمنطق أعمالك
4. **الأمان أولاً**: احتفظ بمفاتيح API على Backend فقط

### ملخص سير العمل

```
Frontend → Backend → OpenAI → Backend → Frontend
   ↓         ↓         ↓         ↓         ↓
مدخل    طلب     معالجة    نتائج   عرض
المستخدم HTTP   الرسالة   الدوال  الرد
```

### أفضل الممارسات

- ✅ احفظ مفاتيح API في متغيرات البيئة
- ✅ تعامل مع الأخطاء بشكل لائق
- ✅ حافظ على خيوط المحادثة
- ✅ اختبر استدعاء الدوال بدقة
- ✅ راقب استخدام API والتكاليف
- ✅ وفر حالات تحميل في الواجهة
- ✅ تحقق من جميع المدخلات

---

## الموارد

- **OpenAI Assistants API**: https://platform.openai.com/docs/assistants
- **OpenAI Node.js SDK**: https://github.com/openai/openai-node
- **دليل استدعاء الدوال**: https://platform.openai.com/docs/guides/function-calling
- **مرجع API**: https://platform.openai.com/docs/api-reference

---
