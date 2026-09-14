# ThaiJO — Journal of Public Health and Development (AIHD-MU)
Local clone สำหรับงาน **CSS override** — ไม่แก้โค้ดเดิม

Source: https://he01.tci-thaijo.org/index.php/AIHD-MU
Platform: Open Journal Systems **3.3.0.8** + theme `bootstrap3`
Cloned: 2026-09-09

## โครงสร้าง

| โฟลเดอร์ | คืออะไร |
|---|---|
| `pages/` | หน้าเว็บที่ clone มา (rewrite path แล้ว) — **เปิดไฟล์นี้ในเบราว์เซอร์** |
| `_original/` | HTML ดิบ ไม่แตะต้องเลย ใช้เทียบ/อ้างอิง |
| `assets/css/` | CSS ต้นฉบับของระบบ — **อ่านอย่างเดียว ห้ามแก้** |
| `assets/js/` | JS ต้นฉบับ (jQuery, jQuery UI, Bootstrap, plugins) |
| `assets/img/`, `assets/fonts/` | รูปและฟอนต์ |
| `css/override.css` | **ไฟล์ที่เราจะเขียนงานทั้งหมดลงไป** |
| `url-map.json` | แผนที่ URL ต้นทาง → ไฟล์ในเครื่อง |

## หลักการ override

ทุกหน้าใน `pages/` ถูก inject บรรทัดนี้ไว้ **ท้ายสุดของ `<head>`**:

```html
<link rel="stylesheet" href="../css/override.css" data-role="css-override">
```

เพราะโหลดทีหลัง → ชนะทุก rule ที่ specificity เท่ากัน โดยไม่ต้องใช้ `!important`
และไม่ต้องแตะไฟล์ `assets/css/*` เลยแม้แต่บรรทัดเดียว

## CSS เดิมของระบบ (ลำดับการโหลดจริง)

1. `css-75e1d5.css` — theme bootstrap3 ที่ OJS compile มา (133 KB)
2. `citations-46c8c7.css` — plugin citations (1.3 KB)
3. `styleSheet-f0a84b.css` — **CSS ที่ทีมวารสารเขียนเอง** (9.8 KB, ใช้ `!important` 23 จุด)
4. `<style>` inline ในทุกหน้า — ThaiJO ยัด `html{filter:saturate(80%)}` + mourning ribbon
5. `override.css` ← ของเรา

## หน้าที่ clone มา (19 หน้า)

index · about · editorial-team · submissions · contact · author-guidelines ·
ethics-copyright · peer-review-process · announcement · announcement-view ·
issue-current · issue-view · issue-archive · issue-archive-page2 ·
article-view · article-galley · search · search-empty · about-site

## ข้อจำกัดที่ควรรู้

- **login / register / Online First** ไม่ได้ clone — redirect ไป `sso.tci-thaijo.org` (ต้องล็อกอิน)
- รูปขนาดใหญ่ (ปก issue) ถูก **ย่อขนาด** เพื่อให้ไฟล์รวมเล็กลง — layout/aspect ratio เท่าเดิม
- asset จากโดเมนภายนอก (scimagojr, flagcounter, creativecommons, tci-thaijo.org) ยังชี้ไป URL จริง → ต้องต่อเน็ตตอนเปิดดู
- ลิงก์ที่ไม่ได้ clone จะชี้กลับไปเว็บจริง (absolute URL) — กดแล้วออกเน็ตตามปกติ

## เริ่มงาน

```
เปิด pages/index.html ในเบราว์เซอร์
แก้ css/override.css
refresh
```


## สิ่งที่ตรวจพบจากการอ่านโค้ดจริง

- **208 class** ทั้งเว็บ / **59 class** เป็นโครงร่วมทุกหน้า (navbar, `.pkp_structure_*`, `.pkp_block`, footer)
- body class บอก template: `pkp_page_index` `pkp_page_article` `pkp_page_issue` `pkp_page_search` `pkp_page_about` + `pkp_op_*`
- `styleSheet.css` เป็นไฟล์ที่ generate จาก theme customizer ของ ThaiJO (คอมเมนต์ `/** id 10001 **/` … `10021`) — ค่าเป็น magic number เยอะ
- `.navbar-default { height: 221.1px }` + `#main-navigation > li a { padding-top: 81px }` → จูนกับโลโก้สูง 150px แบบ hardcode
- `font-family: sans-serif, Helvetica Neue, Helvetica, Arial` → `sans-serif` มาก่อน ทำให้ตัวหลังไม่มีผล และไม่มีฟอนต์ไทย
- `line-height: 1` แทบทุกที่ → บรรทัดอัดกัน
- `.pkp_block .title` `#a1a1a1` + `#333` → contrast ~2.9:1 ต่ำกว่า WCAG AA
- ไม่มี media query ใน `styleSheet.css` เลย → ที่ 390px เนื้อหาล้นขอบขวา (ยืนยันด้วย screenshot)
- เนื้อหา CMS ฝัง `style=""` เยอะ (about 49 จุด, index 13, article 4)

## การเชื่อมลิงก์ภายใน (อัปเดต)

หน้าในเครื่องตอนนี้ **22 หน้า** และลิงก์ภายในถูก map มาที่ไฟล์ในเครื่องแล้ว **348 จุด**

| ลิงก์ต้นทาง | ชี้มาที่ |
|---|---|
| โลโก้ / Home / `/index` | `index.html` |
| `/article/view/<id>` | `article-view.html` |
| `/article/view/<id>/<galley>` | `article-galley.html` |
| `/issue/view/<id>` | `issue-view.html` |
| `/issue/archive/<n>` | `issue-archive-page2.html` |
| `/announcement/view/<id>` | `announcement-view.html` |
| `/search/...` | `search.html` |
| `/information/readers|authors|librarians` | หน้าของตัวเอง |

> บทความและ issue ทุกชิ้นชี้มาที่ **template เดียวกัน** เพราะ clone มาอย่างละ 1 ตัวอย่าง
> เพียงพอสำหรับงาน CSS — ทุกหน้าใช้ layout เดียวกันอยู่แล้ว

### ลิงก์ที่ยังออกเว็บจริง (clone ไม่ได้จริง ๆ)

| ลิงก์ | จำนวน | เหตุผล |
|---|---|---|
| Login / Register | 44 | redirect ไป `sso.tci-thaijo.org` ต้องล็อกอิน |
| www.tci-thaijo.org | 21 | เว็บ ThaiJO ส่วนกลาง ไม่ใช่ของวารสาร |
| `public/api/infoTier.php` | 18 | API endpoint ไม่ใช่หน้าเว็บ |
| citationstylelanguage | 12 | ปุ่มดาวน์โหลดไฟล์อ้างอิง (RIS/BibTeX) |

`_original/` ไม่ถูกแตะ — ยังเป็น HTML ดิบที่ server ส่งมาเป๊ะ ๆ
