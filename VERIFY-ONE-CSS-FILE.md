# รายงานตรวจสอบ: การเปลี่ยนแปลงทั้งหมดมาจาก CSS ไฟล์เดียวจริงหรือไม่

วันที่ตรวจ: 2026-09-11
โปรเจกต์: `_____THAIJO/thaijo-aihd-mu`

---

## สรุปผล

| ข้อตรวจ | ผล |
|---|---|
| ไฟล์ CSS ที่ "สร้างใหม่" ในโปรเจกต์ | **1 ไฟล์** — `css/override.css` (244,911 bytes / 5,396 บรรทัด) |
| ไฟล์ JS ที่เพิ่มเข้ามา | **0 ไฟล์** (มีแต่ jquery / jquery-ui / bootstrap / tag-it / citations ของเดิม) |
| HTML ต้นฉบับ `_original/*.html` เทียบกับที่เซิร์ฟเวอร์ส่งมา | **เหมือนเป๊ะ byte-per-byte 22/22 หน้า** |
| CSS เดิม 3 ไฟล์ใน `assets/css/` | ไม่มีการแก้ rule ใด ๆ (ดูหัวข้อ 3) |
| เรนเดอร์ HTML ต้นฉบับ + override.css เทียบกับหน้าทดสอบ | **ตรงกันทุกพิกเซล 65/66 เคส** (เคสที่ 1 เป็น race ของสคริปต์ในหน้าเว็บเอง — รันซ้ำแล้วตรงกัน 3/3) |

---

## 1. นับไฟล์ในโปรเจกต์

```
CSS  ./assets/css/citations-46c8c7.css      (ของเดิม)
     ./assets/css/css-75e1d5.css            (ของเดิม — theme bootstrap3)
     ./assets/css/styleSheet-f0a84b.css     (ของเดิม — CSS ของวารสาร)
     ./css/override.css                     <-- ไฟล์เดียวที่ผมสร้าง

JS   ./assets/js/articleCitation-a474fc.js   (ของเดิม)
     ./assets/js/bootstrap.min-b12560.js     (ของเดิม)
     ./assets/js/citations-723ac5.js         (ของเดิม)
     ./assets/js/jquery-ui.min-aaf0a3.js     (ของเดิม)
     ./assets/js/jquery.min-8572a9.js        (ของเดิม)
     ./assets/js/jquery.tag-it-68ac29.js     (ของเดิม)
```

ไม่มีไฟล์ JS เพิ่ม — ทุกเอฟเฟกต์ (sticky header, scroll progress, scroll-to-top,
dropdown, hover, icon) เป็น CSS ล้วน

---

## 2. HTML ต้นฉบับไม่ถูกแตะ

เทียบ `_original/*.html` กับ HTML ดิบที่ดึงจากเซิร์ฟเวอร์ (เก็บไว้ใน `bundle.json`)
แบบ byte-per-byte:

```
เหมือนเป๊ะ byte-per-byte: 22 | ต่าง: 0
```

> หมายเหตุ: การเทียบครั้งแรกที่อ่านไฟล์แบบ text mode รายงานผิดว่า "ต่าง 18 หน้า"
> เพราะ Python แปลง `\r\n` เป็น `\n` ให้อัตโนมัติ เมื่อเทียบแบบ binary จริง ๆ แล้วเหมือนกันหมด

---

## 3. CSS เดิม 3 ไฟล์ — ไม่มี rule ไหนถูกแก้

เทียบกับต้นฉบับจากเซิร์ฟเวอร์:

| ไฟล์ | เหมือนเป๊ะทุก byte | เหมือนกันเมื่อไม่นับ path ใน `url()` |
|---|---|---|
| `citations-46c8c7.css` | ใช่ | ใช่ |
| `styleSheet-f0a84b.css` | ใช่ | ใช่ |
| `css-75e1d5.css` | ไม่ (133,077 → 132,716) | **ใช่** |

`css-75e1d5.css` ต่างเฉพาะ 6 บรรทัดที่เป็น `url(...)` ของฟอนต์ Glyphicons
(ตัว clone เปลี่ยนเป็น path ในเครื่อง) — **ไม่มี selector, property หรือ value ใดเปลี่ยน**

---

## 4. บทพิสูจน์จริง: เรนเดอร์ HTML ต้นฉบับ + override.css

วิธี: เสิร์ฟ `_original/*.html` (ไฟล์บนดิสก์ไม่ถูกแก้เลย) ผ่าน HTTP
แล้วแทรก `<link href="override.css">` ตอนตอบ response เท่านั้น — ซึ่งเทียบเท่ากับ
สิ่งที่ OJS ทำให้เองเมื่ออัปโหลด CSS เข้าไปในระบบ
จากนั้นถ่ายภาพ full page แล้วเทียบพิกเซลกับหน้า `pages/*.html` ที่ใช้ทดสอบมาตลอด

```
เปรียบเทียบ 66 เคส (22 หน้า x 3 ความกว้าง: 1440 / 768 / 390)
ตรงกันทุกพิกเซล: 65
ต่าง: 1  -> issue-archive-page2 @768px
```

เคสที่ต่างเกิดจากสคริปต์ `onload` ที่อยู่ใน HTML ของเว็บเองบนรูป TCI Medal:

```html
onload="if(this.height<100){this.style.margin='0'; ...}"
```

ค่า `this.height` ขึ้นกับจังหวะ layout ตอนรูปโหลดเสร็จ รันซ้ำหน้าเดิม 3 ครั้ง
ได้ `margin=0px h=275` ทั้ง ORIG และ WORK และภาพตรงกัน 3/3 — ไม่เกี่ยวกับ CSS

---

## 5. สิ่งที่ต้องพูดให้ตรง

`pages/*.html` (สำเนาสำหรับเปิดทดสอบในเครื่อง) **ถูกแก้ 3 อย่าง** คือ

1. เปลี่ยน path ของ asset ให้ชี้ไฟล์ในเครื่อง
2. แทรก `<link data-role="css-override" href="../css/override.css">` 1 บรรทัด
3. เปลี่ยนลิงก์ภายใน ~402 จุด ให้ชี้หน้าในเครื่อง

ทั้ง 3 อย่างมีไว้เพื่อเปิดดูแบบออฟไลน์เท่านั้น **ตอนขึ้นจริงไม่ต้องใช้เลยสักข้อ**
บนเว็บจริงอัปโหลดแค่ `override.css` ไฟล์เดียวเข้า OJS (Journal CSS)
ส่วน `_original/` คือ HTML ต้นฉบับที่ไม่ถูกแตะ เก็บไว้เป็นตัวอ้างอิง

---

## เครื่องมือที่ใช้ตรวจ

- Playwright + Chromium headless, request interception map asset → ไฟล์ในเครื่อง
- เปรียบเทียบ PNG แบบ byte + pixel bbox (Pillow / numpy)
- เทียบ HTML/CSS แบบ binary กับ `bundle.json` ที่ดึงจากเซิร์ฟเวอร์
