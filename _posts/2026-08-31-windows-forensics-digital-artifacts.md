---
layout: post
title: "أساسيات Windows Forensics: فهم الـ Digital Artifacts"
slug: windows-forensics-digital-artifacts
date: 2026-08-31 04:00:00 +0300
last_modified_at: 2026-08-31 04:00:00 +0300
categories: ["Digital Forensics", "Learning Notes"]
tags: ["Windows Forensics", "DFIR", "Digital Artifacts", "Registry", "Event Logs", "Prefetch", "Incident Response"]
description: "مدخل عملي إلى Windows Forensics، وأنواع Digital Artifacts، والفرق بين Machine-related وUser-related artifacts مع سيناريو تحقيق وPOC آمن."
lang: ar
toc: true
comments: true
pin: false
---

<div dir="rtl" markdown="1">
> هذه المقالة جزء من ملاحظاتي التعليمية أثناء دراسة **Certified Cyber Defender (CCD)**، وتقدّم مدخلًا عمليًا إلى **Windows Forensics** وكيفية التفكير في **Digital Artifacts** وربطها أثناء التحقيق.

---

## 1. الشرح النظري (Arabic + English Terminology)

### 1.1 ما هي Windows Forensics؟

**Windows Forensics** هي عملية جمع (Collect)، حفظ (Preserve)، تحليل (Analyze)، وتقديم (Report) الأدلة الرقمية (Digital Evidence) من نظام تشغيل **Windows** بهدف فهم:

- ماذا حدث؟ (What happened?)  
- متى حدث؟ (When?)  
- من قام به؟ (Who?)  
- وكيف حصل؟ (How?)

نظرًا للانتشار الواسع لأنظمة **Windows** في بيئات العمل، فمن المرجّح أن تتعامل مع عدد كبير من تحقيقات **DFIR** على أجهزة Windows بصفتك **DFIR Analyst / Forensic Analyst**.

---

### 1.2 مفهوم الـ Windows Forensic Artifact

كلمة مهمّة جدًّا: **Artifact**  

- بالعربي: "أثر جنائي رقمي".  
- بالتعريف: أي جزء من البيانات (File, Log, Registry Key, Memory Segment…) يعطيك معلومة عن نشاط حصل على الجهاز.

أمثلة على **Windows Forensic Artifacts**:

- **Event Logs** (سجلات الأحداث)  
- **Registry Keys** (مفاتيح الريجستري)  
- **Prefetch Files**  
- **Browser History / Cache**  
- **User Profiles & Recent Documents**  
- **Shimcache / Amcache**  
- **LNK Files (Shortcuts)**  
- إلخ…

فكّر بكل Artifact كأنه "شاهد" أو "كاميرا" سجّلت جزء من القصة. مهمّتك كـ analyst إنك تجمع هالشهود وتربط بينهم.

---

### 1.3 تقسيم الـ Artifacts: Machine-related vs User-related

الكتاب/المودل يقترح طريقة بسيطة عشان تنظّم تفكيرك:

قسم الـ artifacts لنوعين رئيسيين، وبنفس منطق الـ **Windows Registry**:

- **Machine-related Artifacts**

  تقابل تقريبًا: **HKEY\_LOCAL\_MACHINE (HKLM)**  
  - تتعلق بالجهاز نفسه بغض النظر عن المستخدم.  
  - أمثلة: 
    - Hostname  
    - IP Address  
    - OS Version / Build Number  
    - Install Date  
    - Drivers, Services, Installed Programs  
    - System-wide configuration
- **User-related Artifacts**

  تقابل تقريبًا: **HKEY\_CURRENT\_USER (HKCU)**  
  - خاصة بكل User Account.  
  - أمثلة: 
    - Username  
    - User Profile Path  
    - Browser History / Cookies  
    - Recent Files  
    - User-specific application settings  
    - Saved Credentials (أحيانًا)

#### جدول بسيط يلخّص الفكرة

| النوع | مثال Artifact | مكان نموذجي | الفائدة الجنائية |
| --- | --- | --- | --- |
| Machine-related (HKLM) | OS Install Date | `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion` | تقدير وقت تثبيت النظام |
| Machine-related (HKLM) | Hostname / Domain | `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName` | تعريف الجهاز في الشبكة |
| User-related (HKCU) | RecentDocs | `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` | تحديد الملفات التي فتحها المستخدم |
| User-related (User Files) | Browser History | `C:\Users\<User>\AppData\...` | تحديد مواقع الويب التي تمت زيارتها |

الفكرة المهمّة:  

> **كل Artifact يجاوب سؤال معيّن. مهمتك تعرف السؤال، ثم تعرف أي Artifact يجاوب عليه.**

---

### 1.4 أكثر من طريق لنفس المعلومة

النص اللي أرسلته يركّز على نقطة مهمّة جدًّا في **Windows Forensics**:

> "There is no standard way to perform Windows forensics, and each analyst can take a different route."

مثال واضح: **Windows Installation Date**

تقدر تجيبها من أكثر من مكان:

1. **Registry Key**  
   - `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion`  
   - الـ value اسمها غالبًا: `InstallDate`
2. **Event Logs**  
   - أول Events في **System Log** بعد تثبيت النظام (مثلاً Event ID 6005, 6009…).
3. **systeminfo Command**  
   - أمر `systeminfo` يعطيك line مثل: `Original Install Date`.

الفكرة إنك ما تتعلّق بطريق واحد، بل تفهم الـ artifacts كمنظومة كاملة وتستخدم أكثر من مصدر عشان تتأكد وتبني **Timeline** قوي.

> **Forensic caveat:** قيمة `InstallDate` أو أقدم Event متاح ليست دائمًا تاريخ التثبيت الأصلي؛ قد تتأثر بـ **Feature Updates** أو إعادة بناء النظام أو **Cloning** أو حذف السجلات. لذلك يجب ربطها مع أكثر من Artifact قبل اعتمادها.

---

### 1.5 إصدارات Windows والـ Artifacts المشتركة

الإصدارات الشائعة:

- Windows 7  
- Windows 8 / 8.1  
- Windows 10 (والآن Windows 11)

كلها تختلف في:

- شكل الواجهة (GUI)  
- بعض الخدمات و المسارات  
- بعض التفاصيل في الـ Security features

لكن تشترك في **Core Artifacts**:

- Registry  
- Event Logs  
- User Profiles  
- NTFS File System metadata (MFT, $LogFile, $UsnJrnl)  
- Browser artifacts (Internet Explorer/Edge/Chrome/Firefox… حسب الزمن)

لذلك، لو فهمت منطق الـ artifacts على **Windows 10** كويس، تقدر تطبّق نفس العقلية على 7, 8, 11 مع تعديلات بسيطة في المسارات أو الأسماء.

---

### 1.6 عقلية الـ Forensic Analyst (Security is a mindset)

الجملة الأخيرة مهمّة:  

> "As we keep saying, security is a mindset."

**Mindset** هنا يعني:

1. دايمًا تسأل أسئلة:   
   - What? When? Who? How? From where?  
2. ما تعتمد على أداة واحدة أو Artifact واحد.  
3. تربط بين: 
   - Network Artifacts  
   - Endpoint Artifacts  
   - User Behavior  
4. توثّق كل خطوة (Documentation & Chain of Custody).

---

## 2. قصة سردية (Case Story)

### قصة: “لابتوب الموظف اللي صار عليه Incident”

تخيّل السيناريو التالي:

شركة متوسطة فيها 200 موظف، كلهم يشتغلوا على **Windows 10 Laptops**.

محمّد موظف في قسم المشتريات، عنده صلاحيات يدخل أنظمة حسّاسة.

في يوم من الأيام، فريق الـ **SOC** لاحظ من خلال الـ **SIEM** إن في **suspicious login** من جهاز محمّد خارج ساعات العمل، وبعدها بساعات قليلة صار في **large data transfer** من ملف سيرفر حساس.

تم فتح **Incident** وتم استدعاءك كـ **Windows Forensic Analyst**.

---

### خطواتك كـ Analyst في القصة

1. **تأمين الجهاز (Containment)**  
   - تعمل له Isolation من الشبكة.  
   - تأخذ **Forensic Image** لو امكن (Disk + Memory).
2. **سؤال البداية:**  
   - هل محمّد نفسه اللي عمل الـ access؟  
   - ولا جهازه تم اختراقه (Compromised)?
3. **Machine-related Artifacts** (نظرة على مستوى النظام):
   - تشيك **Event Logs** على مستوى System و Security: 
     - Logon Events (4624 / 4625)  
     - Service Installations  
   - تراجع **OS Install Date** و **Patch Level**: 
     - كان النظام محدث أو فيه ثغرات قديمة؟  
   - تشيك الـ **Network Configuration**: 
     - IP Address وقت الحادثة (لو عندك NetFlow أو DHCP logs).
4. **User-related Artifacts** (نظرة على مستوى المستخدم):
   - تشيك **Browser History** لمستخدم محمّد: 
     - هل فتح Phishing URL قبل الاختراق بساعة مثلاً؟  
   - تشيك **Recent Documents**: 
     - هل فتح الملفات اللي تم تهريبها قبل الـ exfiltration؟  
   - تشيك **Downloads Folder** و **Email Client Artifacts**: 
     - مرفقات مشبوهة؟ Tools؟ Scripts؟
5. **الربط بين الأدلة:**
   - من Event Logs: تشوف Login من IP خارجي أو VPN مش معتاد.  
   - من Browser History: تلاقي Phishing Page مشابه لصفحة الـ VPN portal.  
   - من RecentDocs: تلاقي نفس الملفات اللي تم سحبها.  
   - من Prefetch + Amcache: تلاقي تشغيل Tool غريب (exfiltration tool مثلاً).

في النهاية تستطيع بناء فرضية مدعومة بالأدلة:

- الأدلة تتوافق مع احتمال سرقة حساب محمّد عبر **Phishing Attack**.
- نشاط الدخول وتصفّح الملفات وتشغيل الأداة يظهر ضمن تسلسل زمني مترابط.
- تأكيد النتيجة يتطلب ربط **Windows Artifacts** مع سجلات الـ **VPN** والـ **SIEM** والـ **Network Telemetry**، وعدم الاعتماد على Artifact واحد.

---

## 3. Proof of Concept (POC) – Lab بسيط على Windows 10

هدف الـ POC:  

- تتدرّب عمليًا على التفكير بنمط Machine vs User artifacts.  
- تشوف بنفسك إن في أكثر من طريقة للحصول على نفس المعلومة (مثل Install Date).  

> **ملاحظة:**
>
> هذا POC آمن، تستخدمه على جهازك الشخصي أو VM (يفضّل VM)، وهو للأغراض التعليمية فقط.

---

### 3.1 المتطلبات (Requirements)

- جهاز **Windows 10** (أو 11) – يفضّل Virtual Machine.  
- صلاحيات **Administrator** (علشان تقدر تستخدم أوامر معينة).  
- **Command Prompt** أو **PowerShell**.

---

### 3.2 الهدف الأول: جمع Machine-related Artifacts

#### Step 1 – Get basic system info (version, hostname, install date)

**English Summary:**

We will use built-in commands to collect basic machine-level artifacts: OS version, hostname, and original install date.

**عربي (شرح):**

رح نستخدم أوامر مدمجة في Windows عشان نجيب معلومات أساسية عن النظام – هاي تعتبر **Machine-related artifacts** لأنها خاصة بالجهاز نفسه.

1. افتح **Command Prompt** كـ Administrator.  
2. نفّذ الأمر التالي:

```cmd
systeminfo

```

- هذا الأمر يعطيك: 
  - OS Name  
  - OS Version  
  - Original Install Date  
  - System Boot Time  
  - System Manufacturer & Model  

لو حابب تركز على سطر الـ Install Date:

```cmd
systeminfo | findstr /i "Original Install Date"

```

> قد يختلف النص الذي يبحث عنه `findstr` حسب لغة Windows، لذلك راجع مخرجات `systeminfo` الكاملة إذا لم يظهر السطر.

- **English:** This gives you the install date from a machine perspective.  
- **عربي:** هيك حصلت على **Install Date** كـ artifact من مصدر أول (systeminfo).

---

#### Step 2 – Get hostname and IP (still machine-related)

1. للحصول على **Hostname**:

```cmd
hostname

```

2. للحصول على **IP Address**:

```cmd
ipconfig /all

```

- **English:** These values describe the machine identity on the network.  
- **عربي:** هاي المعلومات تعرّف الجهاز على الشبكة، و تعتبر **Machine-related artifacts** مهمة في أي Incident (مثلاً تربط بين جهاز في الشبكة و logs من الـ firewall).

---

### 3.3 الهدف الثاني: نفس المعلومة من Artifact آخر (Registry)

الآن بدنا نجيب **Install Date** من **Registry** بدل `systeminfo` لإثبات إن في أكثر من طريق لنفس المعلومة.

#### Step 3 – Read InstallDate from Registry (HKLM)

**English Summary:**

We will query a registry key under HKLM that stores the OS installation date.

**عربي (شرح):**

رح نستخدم أمر `reg query` عشان نقرأ قيمة `InstallDate` من الريجستري في **HKEY\_LOCAL\_MACHINE**.

نفّذ الأمر التالي في Command Prompt:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v InstallDate

```

سيرجع لك شيء مثل:

```text
InstallDate    REG_DWORD    0x63f4e2a0

```

- هذه القيمة هي **Unix Timestamp** (seconds since 1-1-1970).
- ممكن تحوّلها لتاريخ حقيقي باستخدام أدوات أو سكربت بسيط (PowerShell أو Python).

مثال أدق باستخدام **PowerShell** لقراءة القيمة وتحويلها مباشرة:

```powershell
$installDate = (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion').InstallDate
[DateTimeOffset]::FromUnixTimeSeconds($installDate).LocalDateTime
```

> تعامل مع هذه القيمة كدليل يحتاج إلى **corroboration**، وليس كحقيقة منفردة عن تاريخ التثبيت الأصلي.

- **English:** Now you have the same install date from another artifact (registry).  
- **عربي:** هيك أثبتنا إن Install Date ممكن نجيبها من **systeminfo** (أداة) أو من **Registry** (Artifact مباشر) – وهذا جوهر الفكرة اللي في النص.

---

### 3.4 الهدف الثالث: User-related Artifacts (Recent Documents)

الآن ننتقل لمستوى المستخدم.

#### Step 4 – Create some user activity

**English Summary:**

We will simulate a user opening some files so Windows creates user-related artifacts.

**عربي (شرح):**

بدنا نتصرف كمستخدم عادي: نفتح كم ملف Word/Notepad، عشان Windows يسجّل آثار في Recent Docs وغيره.

1. افتح **Notepad** واكتب أي نص بسيط.  
2. احفظ الملف باسم مثلاً: `forensic-test1.txt` على Desktop.  
3. كرّر العملية مع ملف ثاني وثالث.

هيك صار عندك **User Activity** حقيقي.

---

#### Step 5 – View Recent Files from GUI (Explorer)

**English Summary:**

Check recent documents via the normal Windows GUI.

**عربي (شرح):**

كمحقق جنائي، أحيانًا تبدأ بواجهة المستخدم؛ تشوف إيش الملفات اللي فتحها المستخدم مؤخرًا.

1. افتح **File Explorer**.  
2. من الجانب الأيسر اختر **Quick Access** أو **Recent files**.  
3. لاحظ ظهور الملفات `forensic-test1.txt` وغيرها اللي أنشأتها.

هذه تعتبر **User-related Artifact** لأنّها مرتبطة بالـ User الحالي.

---

#### Step 6 – View RecentDocs via Registry (HKCU)

الآن نروح على **Registry** في **HKEY\_CURRENT\_USER (HKCU)** ونشوف أثر نفس الملفات.

افتح **Command Prompt** نفس المستخدم (ما يحتاج Admin هالمرّة) ونفّذ:

```cmd
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs"

```

- هذا المفتاح يحتوي على معلومات عن الملفات اللي تم فتحها مؤخرًا.  
- فيه Subkeys حسب نوع الامتداد (مثلاً `.txt`, `.docx`…).

> قيم `RecentDocs` غالبًا ثنائية وليست سهلة القراءة مباشرة. في التحقيق الفعلي استخدم أدوات مثل **Registry Explorer** أو **RECmd** على نسخة جنائية بدل التفاعل مع الجهاز الأصلي.

لتستعرض Subkeys:

```cmd
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs" /s

```

**English:**

Here you are seeing user-specific artifacts stored in the registry under HKCU.

**عربي:**

الآن تشوف كيف Windows يسجّل نشاط المستخدم (الملفات اللي فتحها) في الريجستري داخل HKCU.

هذا يربط بين الـ GUI View (Recent Files) و **low-level artifact** في الريجستري.

---

### 3.5 تلخيص الـ POC

- **Machine-related:** 
  - `systeminfo`, `hostname`, `ipconfig`
  - Registry (HKLM → InstallDate, OS info)
- **User-related:** 
  - Recent Files في Explorer
  - Registry (HKCU → RecentDocs)

**الفكرة الأساسية اللي لازم تثبت عندك:**

> نفس السؤال (مثلاً: متى تم تثبيت النظام؟ ما هي الملفات التي فتحها المستخدم؟)
>
> ممكن تجاوبه من أكثر من Artifact (GUI, Command, Registry, Event Logs…).
>
> دورك كـ Forensic Analyst إنك تعرف كل هذه الطرق وتستخدم أكثر من مصدر للتأكيد وبناء Timeline قوي.

---

## 4. ماذا تتعلم بعد هذا القسم؟ (What to Learn Next)

لو فهمت هذا الـ Intro، الخطوة التالية:

1. **Windows Registry Forensics**
   - فهم الـ Hives: `SAM`, `SYSTEM`, `SOFTWARE`, `SECURITY`, `NTUSER.DAT`, `USRCLASS.DAT`.
   - كيف تستخرج منها معلومات عن: 
     - User accounts  
     - Autostart locations (Persistence)  
     - Network info
2. **Event Log Analysis**
   - سجلات: `Security`, `System`, `Application`, `Microsoft-Windows-…`
   - مهمّة لـ: 
     - Logon/Logoff  
     - Process Creation (مع Sysmon)  
     - Service Installation
3. **File System Forensics (NTFS)**
   - MFT (`$MFT`)  
   - Journals (`$LogFile`, `$UsnJrnl`)  
   - Time Stamps (MACB times)
4. **User Artifacts**
   - Browser Artifacts (Chrome, Edge, Firefox)  
   - Email Clients (Outlook, Thunderbird…)  
   - LNK files, Jump Lists, Prefetch.
5. **Tools to Practice**
   - FTK Imager (للاستخراج)  
   - Autopsy / Sleuth Kit  
   - Velociraptor / KAPE (للتجميع الآلي للـ artifacts).

---

## 5. مراجع مقترحة (References)

- كتاب: *Windows Forensic Analysis* – Harlan Carvey.  
- SANS DFIR Blog / SANS Windows Forensics Trainings.  
- Documentation الرسمية من Microsoft حول Event IDs و Registry Keys.

</div>
