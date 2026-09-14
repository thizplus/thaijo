# แผนงาน CSS Redesign — Journal of Public Health and Development (AIHD-MU)

**เว็บต้นฉบับ:** https://he01.tci-thaijo.org/index.php/AIHD-MU
**แพลตฟอร์ม:** OJS 3.3.0.8 + theme `bootstrap3` (Bootstrap 3.x)
**ขอบเขต:** แก้ CSS อย่างเดียว — **ห้ามแตะ HTML แม้แต่ตัวอักษรเดียว**
**ธีมใหม่:** ขาว / ดำ + accent `#FF5B0A` — เน้นสะอาด อ่านง่าย
**วันที่:** 2026-09-09

---

## 0. กฎเหล็กของงานนี้

| กฎ | รายละเอียด |
|---|---|
| ห้ามแก้ HTML | ทุก selector ต้องมาจาก class/tag/attribute ที่มีอยู่จริงในหน้าเว็บแล้ว |
| ห้ามแก้ CSS เดิม | `css-75e1d5.css`, `citations-46c8c7.css`, `styleSheet-f0a84b.css` = read-only |
| เพิ่มไฟล์เดียว | `override.css` โหลดท้ายสุด → ชนะด้วยลำดับ ไม่ใช่ด้วยการลบของเดิม |
| ห้ามพึ่ง JS | ไม่มี script ใหม่ ไม่มี class ใหม่ ไม่มี `<div>` ใหม่ |

### สิ่งที่ทำไม่ได้ (ยอมรับตั้งแต่ต้น)

- เพิ่ม wrapper element เพื่อจัด layout → ต้องใช้ Flexbox/Grid บน parent ที่มีอยู่แล้วเท่านั้น
- เปลี่ยนลำดับ DOM → ทำได้แค่ระดับสายตาด้วย `order` / `flex-direction: column-reverse`
- แก้ `<table width="1035">` ใน HTML → ต้อง neutralize ด้วย CSS แทน
- แก้ `style=""` ที่ฝังใน content → ต้องใช้ `!important` สู้เท่านั้น

---

## 1. ยืนยันความถูกต้องของไฟล์ที่ clone มา

| โฟลเดอร์ | สถานะ |
|---|---|
| `_original/*.html` | **byte-identical กับที่ server ส่งมา** ไม่แตะเลย — ใช้เป็น source of truth |
| `pages/*.html` | เนื้อหาเดียวกัน แต่ rewrite path asset เป็น local + inject `<link>` override 1 บรรทัด |

> `pages/` มี `<link>` เพิ่ม 1 บรรทัดเพื่อให้ทดสอบในเครื่องได้ — **ตอนขึ้น production ไม่ต้องเพิ่มอะไรใน HTML** เพราะอัปโหลดผ่าน OJS แทน (ดูข้อ 9)

**ที่ตรวจแล้ว:** 19 หน้า / 208 class / 59 class เป็นโครงร่วมทุกหน้า / local reference 373 จุด resolve ครบ 100%

---

## 2. สถาปัตยกรรมไฟล์ CSS

### ลำดับโหลดจริงใน `<head>` (ตรวจจากไฟล์จริง)

```
1. css-75e1d5.css          bootstrap3 theme ที่ OJS compile   133 KB
2. citations-46c8c7.css    plugin citations                   1.3 KB
3. styleSheet-f0a84b.css   วารสารเขียนเอง (23 x !important)   9.8 KB
4. cookie-banner.css       จาก www.tci-thaijo.org             (external)
5. <style> inline          ThaiJO ยัดมา: html{filter:saturate(80%)}
6. override.css            ← ของเรา
```

### โครงภายใน `override.css` (เรียงตามลำดับ cascade)

```
@import Google Fonts
├── LAYER 0  Tokens            :root { --... }
├── LAYER 1  Reset & Base      box-sizing, body, typography scale
├── LAYER 2  Shell             navbar / breadcrumb / sidebar / footer / grid
├── LAYER 3  Components        page-header, pkp_block, panel, btn, media, thumbnail
├── LAYER 4  Page-specific     body.pkp_page_* scope
├── LAYER 5  Responsive        @media 3 ระดับ
└── LAYER 6  Escapes           กฎที่จำเป็นต้องใช้ !important (แยกไว้ให้เห็นชัด)
```

**เหตุผลที่แยก LAYER 6:** `!important` ทุกตัวจะกองอยู่ที่เดียว มีคอมเมนต์กำกับว่าสู้กับอะไร — เวลา debug อนาคตจะรู้ทันทีว่าอันไหนเป็นหนี้ทางเทคนิค

---

## 3. Design Tokens

### 3.1 สี — ทุกค่าเป็น variable

```css
:root {
  /* --- Accent (สีเดียวของแบรนด์) --- */
  --c-accent:        #FF5B0A;   /* สีหลัก: ปุ่ม, เส้นเน้น, active state */
  --c-accent-hover:  #E04D00;   /* hover/active */
  --c-accent-text:   #C24400;   /* ใช้เมื่อ accent ต้องเป็น "ตัวหนังสือเล็ก" บนขาว */
  --c-accent-tint:   #FFF4EC;   /* พื้นหลังอ่อน */
  --c-accent-line:   #FFD9C2;   /* เส้นขอบอ่อน */

  /* --- Neutral --- */
  --c-ink:           #111111;   /* หัวข้อ, ตัวหนังสือบนพื้นส้ม */
  --c-ink-2:         #3D3D3D;   /* body text */
  --c-ink-3:         #6B6B6B;   /* meta, วันที่, caption */
  --c-line:          #E5E5E5;   /* เส้นคั่น */
  --c-line-soft:     #F0F0F0;
  --c-surface:       #FFFFFF;   /* พื้นหลังหลัก */
  --c-surface-2:     #FAFAFA;   /* พื้นหลังรอง (block, panel) */
  --c-surface-3:     #F4F4F4;

  /* --- Semantic (ชี้ไปที่ตัวบน — แก้ที่เดียวเปลี่ยนทั้งเว็บ) --- */
  --c-body-bg:       var(--c-surface);
  --c-text:          var(--c-ink-2);
  --c-heading:       var(--c-ink);
  --c-link:          var(--c-accent-text);
  --c-link-hover:    var(--c-accent);
  --c-border:        var(--c-line);
  --c-navbar-bg:     var(--c-surface);
  --c-navbar-text:   var(--c-ink);
  --c-footer-bg:     var(--c-ink);
  --c-footer-text:   #D8D8D8;
}
```

### 3.2 Contrast — ตรวจแล้วทุกคู่ (WCAG 2.1)

| คู่สี | อัตราส่วน | ตัวปกติ (4.5:1) | ตัวใหญ่ (3:1) |
|---|---:|:---:|:---:|
| `#111` บนขาว | 18.88 | ✅ | ✅ |
| `#3D3D3D` บนขาว | 10.86 | ✅ | ✅ |
| `#6B6B6B` บนขาว | 5.33 | ✅ | ✅ |
| `#C24400` บนขาว | 5.10 | ✅ | ✅ |
| **`#111` บนส้ม `#FF5B0A`** | **6.07** | ✅ | ✅ |
| `#FF5B0A` บนขาว | 3.11 | ❌ | ✅ |
| ขาวบนส้ม `#FF5B0A` | 3.11 | ❌ | ✅ |
| `#FF5B0A` บน tint `#FFF4EC` | 2.87 | ❌ | ❌ |

**กฎที่ได้จากตารางนี้ (บังคับใช้ทั้งเว็บ):**

1. บนพื้นส้ม → **ใช้ตัวหนังสือดำ `#111` เท่านั้น ห้ามใช้ขาว** (6.07 vs 3.11)
2. ตัวหนังสือสีส้มบนขาว → ใช้ได้เฉพาะขนาด ≥ 24px หรือ ≥ 19px bold; ตัวเล็กให้ใช้ `--c-accent-text` (`#C24400`)
3. ห้ามวางส้มบน tint ส้ม — ไม่ผ่านทุกเกณฑ์

> **แก้ข้อมูลที่ผมเคยบอกผิด:** ก่อนหน้านี้ผมบอกว่า `.pkp_block .title` (`#333` บน `#a1a1a1`) มี contrast ~2.9:1 ไม่ผ่าน AA — **ผิดครับ ค่าจริงคือ 4.89:1 ซึ่งผ่าน AA** ของเดิมไม่ได้พังตรงนี้ เราจะเปลี่ยนเพราะเหตุผลด้านดีไซน์ (สีเทาไม่เข้ากับธีมขาว-ดำ-ส้ม) ไม่ใช่เพราะ accessibility

### 3.3 Typography

```css
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&family=Noto+Sans+Thai:wght@400;500;700&display=swap');

:root {
  --font-sans:  'Roboto', 'Noto Sans Thai', -apple-system, 'Segoe UI', Arial, sans-serif;
  --font-thai:  'Noto Sans Thai', 'Roboto', sans-serif;
  --font-head:  var(--font-sans);
  --font-mono:  ui-monospace, 'SF Mono', Menlo, monospace;

  /* type scale — fluid ด้วย clamp() ไม่ต้องเขียน media query ซ้ำ */
  --fs-xs:   0.75rem;                              /* 12 */
  --fs-sm:   0.875rem;                             /* 14 */
  --fs-base: 1rem;                                 /* 16 */
  --fs-lg:   1.125rem;                             /* 18 */
  --fs-xl:   clamp(1.25rem, 1.1rem + 0.6vw, 1.5rem);
  --fs-2xl:  clamp(1.5rem, 1.2rem + 1.2vw, 2rem);
  --fs-3xl:  clamp(1.75rem, 1.3rem + 2vw, 2.5rem);

  --lh-tight: 1.25;
  --lh-base:  1.65;   /* ของเดิมเป็น 1 → อ่านยากมาก */
  --lh-loose: 1.8;
}
```

**ทำไมต้องแก้ font:** ของเดิมเขียน `font-family: sans-serif, Helvetica Neue, Helvetica, Arial` — วาง `sans-serif` (generic) ไว้**หน้าสุด** เบราว์เซอร์จะหยุดที่ตัวแรกเสมอ ตัวหลังทั้งหมดไม่เคยถูกใช้เลย และไม่มีฟอนต์ไทยในลิสต์

**การใช้ Roboto + Noto Sans Thai:** วาง `Roboto` ก่อน `Noto Sans Thai` — Roboto ไม่มี glyph ไทย เบราว์เซอร์จะ fallback ตัวไทยไป Noto Sans Thai อัตโนมัติแบบต่ออักขระ ได้ละตินสวยแบบ Roboto + ไทยสวยแบบ Noto ในบรรทัดเดียวกัน

### 3.4 Spacing / Radius / Shadow

```css
:root {
  --sp-1: 4px;  --sp-2: 8px;  --sp-3: 12px; --sp-4: 16px;
  --sp-5: 24px; --sp-6: 32px; --sp-7: 48px; --sp-8: 64px;

  --radius-sm: 4px;  --radius:  8px;  --radius-lg: 12px;
  --shadow-sm: 0 1px 2px rgba(0,0,0,.05);
  --shadow:    0 2px 8px rgba(0,0,0,.06);
  --shadow-lg: 0 8px 24px rgba(0,0,0,.10);

  --container-max: 1200px;
  --transition: 160ms ease;
}
```

---

## 4. Breakpoints — 3 device

**หลักการ:** ใช้ breakpoint เดียวกับ Bootstrap 3 เป๊ะๆ เพราะ HTML ใช้ `col-xs-* col-sm-* col-md-*` อยู่แล้ว ถ้าตั้งเลขไม่ตรงจะเกิดช่วงที่กฎเรากับ grid เดิมขัดกัน

| Device | ช่วง | Bootstrap tier | หมายเหตุ |
|---|---|---|---|
| **Mobile** | `< 768px` | `xs` | 1 คอลัมน์, sidebar ลงล่าง |
| **Tablet** | `768px – 991px` | `sm` | **ช่วงที่พังที่สุดตอนนี้** |
| **Desktop** | `≥ 992px` | `md` / `lg` | 2 คอลัมน์ |

```css
/* mobile-first: เขียน base เป็น mobile แล้วค่อยขยาย */
@media (min-width: 768px)  { /* tablet ขึ้นไป */ }
@media (min-width: 992px)  { /* desktop ขึ้นไป */ }
@media (min-width: 1200px) { /* wide (ปรับ container) */ }
```

`<meta name="viewport" content="width=device-width, initial-scale=1.0">` — **มีอยู่แล้วในทุกหน้า** ไม่ต้องเพิ่ม

---

## 5. กลยุทธ์เอาชนะ CSS เดิม

### 5.1 ระดับความยากของแต่ละเป้าหมาย

| เป้าหมาย | เจอที่ | วิธีชนะ |
|---|---|---|
| กฎธรรมดาใน bootstrap theme | `css-75e1d5.css` | ลำดับโหลดอย่างเดียวพอ |
| กฎธรรมดาใน styleSheet เดิม | `styleSheet-f0a84b.css` | ลำดับโหลดอย่างเดียวพอ |
| **23 กฎที่มี `!important`** | `styleSheet-f0a84b.css` | ต้อง `!important` สู้ |
| **`<style>` inline ในหน้า** | ทุกหน้า (`filter: saturate(80%)`) | ต้อง `!important` (มาทีหลังเรา) |
| **`style=""` บน element** | about 49, index 13, article 4 | ต้อง `!important` เท่านั้น |

### 5.2 รายการ `!important` ที่จำเป็น (คุมให้น้อยที่สุด — ประเมิน ~25-30 จุด)

ของเดิมใช้ `!important` กับ `font`, `color`, `padding` เป็นหลัก จุดที่ต้องสู้แน่ๆ:

- `body`, `.media-body p`, `.media-body .authors`, `.page_about p`, `.journal-description > p` → font/color
- `.navbar-default #main-navigation > li a` → font + padding (81px)
- `.dropdown-menu li > a` → font + padding
- `.form-control`, `.pull-md-right form button` → font
- `.page-header h1/h2`, `h3`, `h2` → font 30px
- `.media-heading a`, `.issue a`, `.page_about a`, `.pkp_block a` → font/color
- `.pkp_block .title` → font
- `.current_issue .lead` → font
- `.breadcrumb .active` → font
- `.date`, `.media-body .date` → font
- `.btn-group .btn-primary` ฯลฯ → font
- `footer[role="contentinfo"]` → font

### 5.3 เรื่อง `html { filter: saturate(80%) }`

ThaiJO ยัด `<style>` นี้เข้ามาทุกหน้า (มาคู่กับ mourning ribbon) ทำให้**สีทั้งเว็บซีดลง 20%** — สีส้ม `#FF5B0A` ที่เห็นจริงบนจอจึงไม่ใช่ `#FF5B0A`

- **ทางเลือก A (default):** ปล่อยไว้ เคารพนโยบายแพลตฟอร์ม → ธีมเราต้องออกแบบโดยรู้ว่าสีจะถูกลดความอิ่มตัว
- **ทางเลือก B:** `html { filter: none !important; }` — ทำได้เพราะ `!important` ชนะลำดับ แต่**เป็นการ override นโยบายส่วนกลางของ ThaiJO** ควรถามเจ้าของวารสารก่อน

> ผมจะใส่เป็นคอมเมนต์ปิดไว้ พร้อมคำอธิบาย ให้คุณตัดสินใจ

---

## 6. งานระดับ Global (มีผลทุกหน้า)

โครง shell ที่ทุกหน้าใช้ร่วมกัน (59 class):

```
body.pkp_page_*.pkp_op_*.has_site_logo
└── .pkp_structure_page
    ├── nav#accessibility-nav.sr-only
    ├── header#headerNavigationContainer.navbar.navbar-default
    │   ├── .container-fluid > .row > nav > ul#navigationUser.nav.nav-pills
    │   └── .container-fluid
    │       ├── .navbar-header > button.navbar-toggle + h1.site-name > a.navbar-brand-logo
    │       └── nav#nav-menu.navbar-collapse > ul#main-navigation.nav.navbar-nav
    ├── .pkp_structure_content.container
    │   ├── main.pkp_structure_main.col-xs-12.col-sm-10.col-md-8
    │   └── aside#sidebar.pkp_structure_sidebar.left.col-xs-12.col-sm-2.col-md-4
    └── footer.footer[role=contentinfo]
```

### 6.1 Header / Navbar — งานหนักที่สุด

**ปัญหาที่ยืนยันแล้ว:**

```css
.navbar-default { height: 221.1px; }                          /* hardcode */
.navbar-default #main-navigation > li a { padding: 81px 10px 10px 10px; }  /* ดันเมนูลงมา */
.navbar-brand-logo img { height: 150px; }                     /* hardcode */
.navbar-form { margin-top: 58px; }                            /* magic number */
```

ทั้ง 4 ค่าจูนมือให้พอดีกันที่ความกว้าง desktop เท่านั้น พอย่อจอ → เมนูทับโลโก้ / ช่องค้นหาลอย

**แผนแก้:**

| สิ่งที่ทำ | วิธี |
|---|---|
| ปลด height คงที่ | `height: auto !important` + ใช้ Flexbox บน `.container-fluid` |
| ปลด padding-top 81px | `padding: var(--sp-3) var(--sp-4) !important` |
| โลโก้ responsive | `max-height: clamp(48px, 8vw, 96px); height: auto; width: auto` |
| navbar เป็นขาว | `--c-navbar-bg: #fff` + `border-bottom: 1px solid var(--c-line)` |
| เมนู active/hover | เส้นใต้ส้ม 2px (`border-bottom`) แทนการเปลี่ยนสีพื้น |
| แถบส้มยังอยู่ | ย้ายไปเป็นแถบบางเหนือ navbar (`::before` บน header) — คงตัวตนแบรนด์ไว้ |
| ปุ่ม hamburger | ขยายเป็น ≥44×44px, `.icon-bar` เปลี่ยนเป็นสีดำ |
| ช่องค้นหา | `margin-top: 0 !important`, จัดด้วย flex `margin-left: auto` |

**Mobile:** `#nav-menu.collapse` เป็นกลไกของ Bootstrap JS — เราไม่แตะ logic แค่จัดสไตล์ตอนเปิด (`.collapse.in`) ให้เป็น list เต็มความกว้าง แตะง่าย

### 6.2 Grid / Layout — ปัญหา Tablet

```html
main  class="pkp_structure_main    col-xs-12 col-sm-10 col-md-8"
aside class="pkp_structure_sidebar col-xs-12 col-sm-2  col-md-4"
```

| ช่วงจอ | main | sidebar | ผลลัพธ์ |
|---|---|---|---|
| < 768px | 100% | 100% | ✅ เรียงลง |
| **768–991px** | **83.3%** | **16.7% (~128px)** | ❌ **sidebar แคบมาก แต่มีรูป `width="200"` → ล้น** |
| ≥ 992px | 66.7% | 33.3% | ✅ แต่ sidebar กว้างเกินความจำเป็น |

**แผนแก้ (CSS ล้วน):**

```css
@media (min-width: 768px) and (max-width: 991px) {
  .pkp_structure_main    { width: 100%; float: none; }
  .pkp_structure_sidebar { width: 100%; float: none; }
  /* sidebar เป็นแถวของการ์ด 2-3 ใบ ด้วย grid */
  .pkp_structure_sidebar { display: grid; grid-template-columns: repeat(2,1fr); gap: var(--sp-4); }
}
@media (min-width: 992px) {
  .pkp_structure_main    { width: 72%; }
  .pkp_structure_sidebar { width: 28%; }  /* คืนพื้นที่ให้เนื้อหา */
}
```

### 6.3 Sidebar (`.pkp_block`)

- ของเดิม: กรอบเทา หัว block พื้น `#a1a1a1`
- ใหม่: การ์ดขาว `border: 1px solid var(--c-line)`, `border-radius: var(--radius)`, หัว block เป็นตัวหนังสือดำ + เส้นใต้ส้ม 2px (ไม่ใช้พื้นเทา)
- `.block_make_submission_link` → ปุ่มส้มเต็มความกว้าง ตัวหนังสือดำ (6.07:1)
- **รูป badge `width="200"` / `width="300"`** → `img { max-width: 100%; height: auto; }` (แก้ overflow)
- โลโก้ index (Scopus/ACI/TCI/Crossref…) → จัดเป็น grid `repeat(auto-fit, minmax(88px,1fr))` ให้เป็นแถวเรียบร้อยแทนการเรียงลงเป็นแถวยาว

### 6.4 `.page-header` (แถบส้ม)

ของเดิม: `background:#FF5B0A; padding:1px` ใช้กับทั้ง "Announcements", ชื่อบทความ, ชื่อ issue → ชื่อบทความยาวกลายเป็นก้อนส้มเต็มจอ

**ใหม่:** พื้นขาว + **เส้นซ้ายส้มหนา 4px** + หัวข้อดำ — สะอาด ไม่แย่งสายตา และแก้ปัญหาข้อความยาวไปพร้อมกัน

### 6.5 Breadcrumb

ของเดิม: พื้นส้ม + ลิงก์สีเข้ม (`.breadcrumb .active` สีขาวบนส้ม = 3.11:1 ไม่ผ่าน)
**ใหม่:** พื้นโปร่ง, ตัวหนังสือ `--c-ink-3`, ตัวปัจจุบัน `--c-ink` ตัวหนา, ตัวคั่นเป็น `/` สีอ่อน

### 6.6 Footer

`footer[role="contentinfo"]` (**ต้องใช้ attribute selector — `.footer` เฉยๆ ไม่ตรงกับ specificity เดิม**)
ของเดิม: พื้น `#eee` line-height 1
**ใหม่:** พื้นดำ `--c-ink` ตัวหนังสือ `#D8D8D8` (18.88:1), แบ่ง 3 คอลัมน์บน desktop / เรียงลงบน mobile, `line-height: 1.65`

### 6.7 Base / Reset

- `*, *::before, *::after { box-sizing: border-box }`
- `img, svg, iframe, video { max-width: 100%; height: auto }` ← แก้รูป `width="200"` ทั้งเว็บทีเดียว
- `html, body { overflow-x: hidden }` เป็นตาข่ายกันเหนียว (แก้ที่ต้นเหตุก่อนเสมอ)
- `:focus-visible { outline: 2px solid var(--c-accent); outline-offset: 2px }`
- `a { transition: color var(--transition) }`
- `::selection { background: var(--c-accent); color: #fff }`

---

## 7. งานรายหน้า — ครบทั้ง 19 หน้า

Selector scope ใช้ `body` class ที่มีอยู่จริง เช่น `body.pkp_page_index`, `body.pkp_page_article.pkp_op_view`

| # | หน้า | body class | สิ่งที่ต้องทำ |
|---|---|---|---|
| 1 | **index** | `pkp_page_index pkp_op_index` | **`<table width="1035">` ใน `.journal-description` = ต้นเหตุหลักที่ล้นจอมือถือ** → `table{width:100%!important;max-width:100%;display:block;overflow-x:auto}` และ `td{display:block;width:100%!important}` ที่ mobile · `.journal-description` เป็นการ์ดแนะนำวารสาร · `.cmp_announcements .media-list` เป็นการ์ด · `.current_issue` เป็น hero: ปก + ชื่อ issue + ปุ่ม · `.issue-toc .section` มีหัวข้อชัดเจน |
| 2 | **about** | `pkp_page_about pkp_op_index` | **มี `style=""` 49 จุด** (`text-align:justify`, `text-indent:30px`, `font-size:130%`, `color:midnightblue`) → ต้อง `!important` ทั้งหมด · ยกเลิก justify (เกิด river gap) เป็น `text-align:left` · จำกัด `max-width: 72ch` เพื่อความอ่านง่าย |
| 3 | **editorial-team** | `pkp_page_about` | **`<table width="704">` + `<td width="300">` 86 ช่อง + `<td width="104">` 43 ช่อง** → ที่ mobile แปลงเป็น card list ด้วย `td{display:block;width:auto!important}` · desktop ใช้ `table-layout:fixed` + `width:100%` |
| 4 | **submissions** | `pkp_page_about` | เนื้อหายาว → typography + `ol/ul` spacing + `max-width:72ch` |
| 5 | **contact** | `pkp_page_about` | จัดข้อมูลติดต่อเป็นการ์ด, `mailto:` ให้แตะง่ายบนมือถือ (≥44px) |
| 6 | **author-guidelines** | `pkp_page_about` | เนื้อหายาวมาก → heading hierarchy ชัด + spacing |
| 7 | **ethics-copyright** | `pkp_page_about` | เหมือน 6 |
| 8 | **peer-review-process** | `pkp_page_about` | มีรูป flow chart → `max-width:100%` + จัดกลาง |
| 9 | **about-site** | `pkp_page_about` | หน้า OJS มาตรฐาน สั้น |
| 10 | **announcement** | `pkp_page_announcement` | `.cmp_announcements` เป็น card grid · รูปประกาศขนาดใหญ่ → responsive · `.date` เป็น meta |
| 11 | **announcement-view** | `pkp_page_announcement pkp_op_view` | หน้าอ่านเดี่ยว → `max-width:72ch` |
| 12 | **issue-current** | `pkp_page_issue` | ปก issue + TOC · `.issue-toc .section` แยกหมวด · `.article-summary` เป็นการ์ด |
| 13 | **issue-view** | `pkp_page_issue pkp_op_view` | เหมือน 12 + `.issue-details` · `.thumbnail` ปกให้ `object-fit` ไม่บิด |
| 14 | **issue-archive** | `pkp_page_issue pkp_op_archive` | `.issues > .issue-summary` (`.media-left` = ปก) → **grid ปก** `repeat(auto-fill,minmax(180px,1fr))` แทน media list แนวยาว · `.pager` เป็นปุ่มแตะง่าย |
| 15 | **issue-archive-page2** | เหมือน 14 | ตรวจ `.pager .next/.current` ให้ชัดเจน |
| 16 | **article-view** | `pkp_page_article pkp_op_view` | **หน้าสำคัญที่สุด** · `.article-details` = `.article-main` + `.article-sidebar` → บน mobile ให้ sidebar (ปก+ปุ่ม pdf) ขึ้นก่อน abstract ด้วย `order` · `.doi` ปัจจุบันจางบนพื้นส้มจนแทบมองไม่เห็น → ย้ายเป็น meta line สีเทาบนขาว · `.galley-link.pdf` เป็นปุ่มส้มเด่น ตัวอักษรดำ · `.article-abstract` `max-width:72ch` `line-height:1.8` · `.article-references .csl-entry` hanging indent · `.citations-container` (scopus/crossref/google/pmc) เป็นแถว badge · `.keywords` เป็น chip · `.panel` (`#FFFBCF`) → พื้น `--c-surface-2` |
| 17 | **article-galley** | (iframe wrapper) | หน้า PDF viewer ล้วน → `iframe{width:100%;height:calc(100vh - 120px);border:0}` |
| 18 | **search** | `pkp_page_search pkp_op_search` | `.search-form` เป็นแถบค้นหาเด่น · `input-group` responsive · `.search-results .article-summary` เป็นการ์ด · `.cmp_pagination` แตะง่าย |
| 19 | **search-empty** | `pkp_page_search` | empty state ให้ดูตั้งใจ ไม่ใช่หน้าว่าง |

---

## 8. ปัญหา Responsive ที่ยืนยันด้วย screenshot แล้ว

| # | อาการ | ต้นเหตุ (ตรวจจากโค้ด) | วิธีแก้ (CSS-only) |
|---|---|---|---|
| 1 | **มือถือ 390px: เนื้อหาล้นขอบขวา ชื่อวารสารถูกตัด** | `<table width="1035">` ใน index + navbar `height:221.1px` | table → `display:block; overflow-x:auto; width:100%!important` · navbar → `height:auto!important` |
| 2 | **Tablet 768–991px: sidebar เหลือ 128px แต่รูปกว้าง 200-300px** | `col-sm-2` + `img width="200"` | override column width + `img{max-width:100%}` |
| 3 | เมนูทับโลโก้เมื่อย่อจอ | `padding-top:81px` + โลโก้ 150px | flex layout + `padding` ปกติ |
| 4 | ช่องค้นหาลอยผิดที่ | `.navbar-form{margin-top:58px}` | `margin-top:0!important` + flex |
| 5 | editorial-team ตารางล้นจอ | `table width="704"` + `td width="300"` ×86 | `display:block` ที่ mobile |
| 6 | บรรทัดอัดกันทั้งเว็บ | `line-height: 1` ~15 จุด | `--lh-base: 1.65` |
| 7 | ปุ่ม/ลิงก์เล็กเกินแตะ | ไม่มีการกำหนด min size | `min-height: 44px` บน nav link, ปุ่ม, pager |
| 8 | ข้อความ justify เกิดช่องว่างเป็นแถบ | `style="text-align:justify"` 11 จุด | `text-align:left!important` |

---

## 9. การนำขึ้น Production

`override.css` ไม่ต้องแก้ HTML — อัปโหลดผ่าน OJS:

**Settings → Website → Appearance → Journal Style Sheet**

⚠️ **ข้อควรระวังสำคัญ:** ช่องนี้คือที่เดียวกับที่ `styleSheet.css` เดิมอยู่ การอัปโหลดจะ**แทนที่ของเดิม ไม่ใช่เพิ่มต่อท้าย**

| ทางเลือก | วิธีทำ | ผลลัพธ์ |
|---|---|---|
| **A. Replace (แนะนำ)** | อัปโหลด `override.css` ทับ | ได้ CSS สะอาด ไม่มีหนี้เดิม แต่ต้องเขียนสไตล์ครบเอง (ไม่ใช่แค่ override) — และถ้ามีคนไปกด theme customizer ของ ThaiJO ไฟล์อาจถูก regenerate ทับ |
| **B. Append** | เอา `styleSheet-f0a84b.css` เดิม + ต่อท้ายด้วย `override.css` แล้วอัปโหลดเป็นไฟล์เดียว | ปลอดภัยกว่า พฤติกรรมเหมือนที่ทดสอบในเครื่องเป๊ะ ๆ แต่ไฟล์จะมีโค้ดเดิมค้างอยู่ |

> ผมแนะนำ **B** ตอนขึ้นครั้งแรก (เสี่ยงต่ำ ผลลัพธ์ตรงกับที่ทดสอบ) แล้วค่อยพิจารณา A เมื่อมั่นใจ

---

## 10. Deliverables

| ไฟล์ | เนื้อหา |
|---|---|
| `css/override.css` | ไฟล์หลัก — tokens + 6 layers + คอมเมนต์ภาษาไทยทุกบล็อก |
| `css/override.min.css` | เวอร์ชัน minify สำหรับขึ้น production |
| `CSS-REDESIGN-PLAN.md` | เอกสารนี้ |
| `screenshots/before-after/` | ภาพเทียบ 19 หน้า × 3 ขนาดจอ (57 คู่) |

---

## 11. แผนการทำงาน

| เฟส | งาน | ผลลัพธ์ที่ตรวจได้ |
|---|---|---|
| 1 | Tokens + Base + Typography | ฟอนต์/สี/line-height เปลี่ยนทั้งเว็บ |
| 2 | Shell: navbar, breadcrumb, sidebar, footer, grid | โครงหลักใช้ได้ทั้ง 3 จอ |
| 3 | Components: page-header, block, panel, btn, media, thumbnail | หน้าตาเป็นระบบเดียวกัน |
| 4 | Page-specific ทั้ง 19 หน้า | ทุกหน้าเรียบร้อย |
| 5 | Responsive fixes 8 ข้อ | ไม่มี horizontal overflow ที่ 390/768/1280 |
| 6 | **Verify** | screenshot 19×3 + ตรวจ `scrollWidth > clientWidth` ทุกหน้า + ตรวจ contrast ทุกคู่สีที่ใช้จริง + diff HTML กับ `_original/` ว่า **0 bytes ต่าง** |

---

## 12. เกณฑ์ว่า "เสร็จ"

- [ ] `diff _original/ pages/` ต่างเฉพาะ `<link>` override + path asset เท่านั้น (ไม่มีการแก้ structure)
- [ ] ทั้ง 19 หน้า × 3 ขนาดจอ (390 / 768 / 1280) → `document.scrollWidth <= window.innerWidth`
- [ ] ทุกคู่สีที่ใช้จริงผ่าน WCAG AA (ตัวปกติ 4.5:1 / ตัวใหญ่ 3:1)
- [ ] target ที่แตะได้ทุกตัว ≥ 44×44px บน mobile
- [ ] ทุกสีใน CSS อ้างผ่าน `var(--c-*)` — ไม่มี hex ลอยนอก `:root`
- [ ] `font-family` ทุกที่อ้าง `var(--font-*)`
- [x] `!important` รวม 103 declaration — 60 อยู่ใน LAYER [10] (สู้ styleSheet เดิม + inline style) ที่เหลือเป็นการปลด `display:block!important` ของ Bootstrap และ neutralize `width`/`height` attribute ใน HTML


---

## ภาคผนวก — สถานะการทำงาน (อัปเดต 2026-09-09)

### เสร็จแล้ว

`css/override.css` — 1,391 บรรทัด / 46 KB / parse error 0

| Layer | สถานะ |
|---|---|
| [0] Tokens | ✅ สีทุกตัวเป็น variable, ฟอนต์ Roboto + Noto Sans Thai |
| [1] Base | ✅ |
| [2] Layout | ✅ header/content/footer ตรงแนวเดียวกันแล้ว |
| [3] Header | ✅ ปลด magic number ทั้ง 4 ตัว, hamburger ถึง 991px |
| [4] Components | ✅ |
| [5] Sidebar | ✅ |
| [6] Footer | ✅ |
| [7] Pages | ✅ ครบทุก template |
| [8] Responsive fixes | ✅ |
| [9] A11y | ✅ |
| [10] Important escapes | ✅ |
| [11] Fine-tuning | ✅ จากผลวัด render จริง |

### ผลทดสอบ

- **19 หน้า × 3 ขนาดจอ (390 / 768 / 1280) = 57 เคส → ไม่มี horizontal overflow เลย**
- CSS parse error = 0
- ทุกสีอ้างผ่าน `var(--c-*)` ไม่มี hex ลอยนอก `:root`

### บั๊กที่เจอจากการวัดจริง (ไม่ใช่การเดา) แล้วแก้ไปแล้ว

| # | อาการ | สาเหตุจริง |
|---|---|---|
| 1 | ตัวหนังสือทั้งเว็บเล็กกว่าที่ตั้งไว้ 37.5% | **Bootstrap 3 ตั้ง `html{font-size:10px}`** → `1rem` = 10px ไม่ใช่ 16px · แก้โดยเปลี่ยน token ทั้งหมดเป็น px |
| 2 | ปุ่ม Make a Submission เป็นกล่องขาว ไม่ใช่ปุ่มส้ม | theme เขียน `.pkp_block.block_make_submission a` (specificity 0,3,1) ชนะ `.block_make_submission_link` (0,1,0) · แก้ด้วย selector ที่แรงกว่า |
| 3 | ปุ่ม hamburger แสดงขีดเดียวแทน 3 ขีด | `display:block!important` ของเราเองไปทับ `display:flex` · แก้เป็น `display:flex!important` |
| 4 | โลโก้ลอยไปกลางจอบน tablet | `justify-content:space-between` เจอ flex item ที่มองไม่เห็นในแถวนั้น · เปลี่ยนเป็น `flex-start` + `margin-right:auto` |
| 5 | โลโก้ไม่ตรงแนวกับเนื้อหา | `padding-left` ที่เว้นให้ริบบิ้นไว้อาลัย ถูกใส่ทุกจอ · จำกัดให้เฉพาะ ≤1279px |

### ยังไม่ได้ทำ / รอตัดสินใจ

- `html{filter:saturate(80%)}` — ยังคงไว้ตามเดิม (คอมเมนต์เตรียมไว้ใน [10] ส่วน (ค))
- screenshot before/after ครบ 19 หน้า × 3 จอ — ตอนนี้ทำตัวอย่างไว้ 7 รูปใน `screenshots/`
- ยังไม่ได้ทดสอบบนเบราว์เซอร์จริงของผู้ใช้ (ทดสอบด้วย headless Chromium)
