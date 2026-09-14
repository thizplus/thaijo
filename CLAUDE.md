# ThaiJO AIHD-MU — CSS-Only Override Project

## กฎเหล็ก (CRITICAL)

> **แก้ได้แค่ไฟล์เดียว: `css/override.css`**
> **ห้ามแก้ HTML, JS, หรือ CSS ต้นฉบับ แม้แต่ตัวอักษรเดียว**

## โครงสร้างโปรเจกต์

| โฟลเดอร์/ไฟล์ | หน้าที่ | แก้ได้? |
|---|---|:---:|
| `css/override.css` | CSS override ทั้งหมด — โหลดท้ายสุดใน `<head>` | **YES** |
| `_original/*.html` | HTML ต้นฉบับจาก ThaiJO (source of truth) | NO |
| `pages/*.html` | HTML เดียวกับ `_original/` แต่ rewrite path เป็น local + inject `<link>` override | NO |
| `assets/css/` | CSS ต้นฉบับ 3 ไฟล์ (bootstrap theme, citations, styleSheet) | NO |
| `assets/js/` | JS ต้นฉบับ (jQuery, Bootstrap, etc.) | NO |
| `assets/img/` | รูปภาพจากเว็บต้นฉบับ | NO |
| `assets/fonts/` | ฟอนต์ glyphicons | NO |
| `CSS-REDESIGN-PLAN.md` | แผนงานละเอียด (tokens, breakpoints, per-page plan) | อ้างอิง |

## วิธีทำงานของระบบ

1. โคลน HTML จาก `he01.tci-thaijo.org/index.php/AIHD-MU` เก็บใน `_original/`
2. แปลง asset paths เป็น local เก็บใน `pages/`
3. แทรก `<link href="../css/override.css">` ท้ายสุดใน `<head>`
4. `override.css` ชนะด้วย **cascade order** (ไม่ต้องลบของเดิม)

## ข้อจำกัดที่ต้องจำ

- **ห้ามเพิ่ม HTML element, class, หรือ attribute ใดๆ**
- **ห้ามเพิ่ม JS ใหม่**
- selector ต้องมาจาก class/tag/attribute ที่มีอยู่ในหน้าเว็บแล้วเท่านั้น
- inline `style=""` ที่ฝังใน content → ต้องใช้ `!important` สู้
- `styleSheet-f0a84b.css` มี `!important` 23 จุด → ต้อง `!important` สู้

## โครงสร้างภายใน override.css

```
@import Google Fonts (Inter + Noto Sans Thai)
[0]  TOKENS            — :root { --c-*, --font-*, --sp-*, --radius-* }
[1]  BASE              — reset, typography, media
[2]  LAYOUT            — container, grid, breakpoints
[3]  HEADER            — masthead + เมนู responsive
[4]  COMPONENTS        — page-header, breadcrumb, block, panel, btn
[5]  SIDEBAR
[6]  FOOTER
[7]  PAGES             — body.pkp_page_* scope
[8]  RESPONSIVE FIXES  — ตาราง/รูป fixed-width
[9]  A11Y
[10] IMPORTANT ESCAPES — กฎที่ต้องใช้ !important (แยกไว้ให้เห็นชัด)
```

## Breakpoints (ตรงกับ Bootstrap 3)

| Device | ช่วง | Bootstrap tier |
|---|---|---|
| Mobile | < 768px | xs |
| Tablet | 768–991px | sm |
| Desktop | ≥ 992px | md/lg |

## แพลตฟอร์ม

- **CMS**: OJS 3.3.0.8 (Open Journal Systems)
- **Theme**: bootstrap3 (Bootstrap 3.x)
- **เว็บต้นฉบับ**: https://he01.tci-thaijo.org/index.php/AIHD-MU

## Production Deploy

อัปโหลดผ่าน OJS: **Settings → Website → Appearance → Journal Style Sheet**
(ไม่ต้องแก้ HTML บน server)
