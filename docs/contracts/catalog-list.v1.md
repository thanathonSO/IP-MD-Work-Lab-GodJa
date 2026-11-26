# API Contract: Catalog Product List (v1)

## 1. สถานะและเจ้าของ
* **สถานะเอกสาร:** `PROPOSED`
* **เวอร์ชัน:** `v1`
* **เจ้าของข้อเสนอ:** `Catalog Microservice`

## 2. คำขอ
* **HTTP Method:** `GET`
* **Endpoint Path:** `/api/v1/catalog/products`
* **Request Body:** `ไม่มีการส่ง Request Body`

### Query Parameters
| ชื่อพารามิเตอร์ | ชนิดข้อมูล | บังคับ | ค่าที่รับได้ | คำอธิบาย |
| :--- | :--- | :--- | :--- | :--- |
| `available` | `boolean` | ไม่บังคับ | `true`, `false` *(หรือเว้นว่าง หรือไม่ส่ง)* | ใช้กรองสถานะความพร้อมจำหน่ายของสินค้า หากไม่ส่งจะคืนค่ารายการสินค้าทั้งหมด |

---

## 3. โครงสร้างข้อมูลสินค้า
ข้อมูลสินค้าแต่ละรายการประกอบด้วยฟิลด์ทั้งหมด 4 ฟิลด์ ดังนี้

| ชื่อฟิลด์ | ชนิดข้อมูล | หน่วย / รูปแบบ | คำอธิบาย |
| :--- | :--- | :--- | :--- |
| `id` | `string` | ตัวอักษรภาษาอังกฤษพิมพ์เล็ก | รหัสอ้างอิงเฉพาะของสินค้า (เช่น `"pencil"`) |
| `name` | `string` | ข้อความ | ชื่อสินค้าที่แสดงผล (เช่น `"ดินสอ"`) |
| `priceSatang` | `integer` | **สตางค์ (Satang)** | ราคาสินค้าในหน่วยสตางค์ เพื่อเลี่ยงปัญหาทศนิยม (เช่น `1200` สตางค์ = `12` บาท) |
| `available` | `boolean` | `true` หรือ `false` | สถานะความพร้อมจำหน่าย (`true` = พร้อมขาย, `false` = สินค้าหมด หรือ ไม่พร้อมขาย) |

## 4. ผลตอบกลับ
### ตารางสรุปผลลัพธ์จากกรณีทดสอบ 4 กรณี

ข้อมูลตั้งต้นในระบบมี 4 รายการ ได้แก่ `pencil` (1200, true), `ruler` (2000, false), `eraser` (800, true), `sharpener` (1500, false)

| กรณี | คำขอ | ค่า `available` | HTTP Status | ผลลัพธ์ที่คาดหวัง |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `GET /api/v1/catalog/products` | ไม่ใส่ | `200 OK` | คืนค่ารายการสินค้าทั้งหมด **4 รายการ** (`pencil`, `ruler`, `eraser`, `sharpener`) |
| **2** | `GET /api/v1/catalog/products?available=true` | `true` | `200 OK` | คืนค่าเฉพาะสินค้าที่พร้อมขาย **2 รายการ** (`pencil`, `eraser`) |
| **3** | `GET /api/v1/catalog/products?available=false` | `false` | `200 OK` | คืนค่าเฉพาะสินค้าที่ไม่พร้อมขาย **2 รายการ** (`ruler`, `sharpener`) |
| **4** | `GET /api/v1/catalog/products?available=yes` | `yes` | `422 Unprocessable Content` | ปฏิเสธคำขอและส่ง Error JSON แจ้งว่าค่า `available` ไม่ใช่รูปแบบ Boolean ที่กำหนด |

### ตัวอย่าง JSON Response กรณีสำเร็จ (`200 OK`)

#### ตัวอย่างที่ 1: กรณีไม่ใส่ Query (`GET /api/v1/catalog/products`) — แสดงสินค้าทั้งหมด 4 รายการ
```json
{
  "items": [
    {
      "id": "pencil",
      "name": "ดินสอ",
      "priceSatang": 1200,
      "available": true
    },
    {
      "id": "ruler",
      "name": "ไม้บรรทัด",
      "priceSatang": 2000,
      "available": false
    },
    {
      "id": "eraser",
      "name": "ยางลบ",
      "priceSatang": 800,
      "available": true
    },
    {
      "id": "sharpener",
      "name": "กบเหลาดินสอ",
      "priceSatang": 1500,
      "available": false
    }
  ]
}
```

#### ตัวอย่างที่ 2: กรณีกรอง `available=true` (`GET /api/v1/catalog/products?available=true`)
```json
{
  "items": [
    {
      "id": "pencil",
      "name": "ดินสอ",
      "priceSatang": 1200,
      "available": true
    },
    {
      "id": "eraser",
      "name": "ยางลบ",
      "priceSatang": 800,
      "available": true
    }
  ]
}
```

#### ตัวอย่างที่ 3: กรณีกรอง `available=false` (`GET /api/v1/catalog/products?available=false`)
```json
{
  "items": [
    {
      "id": "ruler",
      "name": "ไม้บรรทัด",
      "priceSatang": 2000,
      "available": false
    },
    {
      "id": "sharpener",
      "name": "กบเหลาดินสอ",
      "priceSatang": 1500,
      "available": false
    }
  ]
}
```

#### ตัวอย่างที่ 4: กรณีผลลัพธ์เป็นรายการว่างสมมติ (Empty List — เช่น ยังไม่มีสินค้าในระบบหรือไม่มีรายการที่ตรงเงื่อนไข)
*หมายเหตุ: แม้ไม่พบรายการสินค้าที่ตรงเงื่อนไข คำขอยังถือว่าสำเร็จจึงตอบกลับด้วย `200 OK` พร้อมอาร์เรย์ว่าง (ไม่ใช้ 404)*
```json
{
  "items": []
}
```

---

## 5. กรณีผิดพลาด

### 5.1 รหัสข้อผิดพลาดที่กำหนดในรุ่น v1

| HTTP Status Code | ชื่อสถานะ | สาเหตุการเกิด | ตัวอย่าง Error Response Body |
| :--- | :--- | :--- | :--- |
| **`422`** | `Unprocessable Content` | โครงสร้างคำขอถูกต้อง แต่ค่าของ Query Parameter `available` ไม่ผ่านการตรวจสอบ (Validation) เช่น ส่งค่าที่ไม่ใช่ `true` หรือ `false` (เช่น `yes`, `123`) | ดูตัวอย่างด้านล่าง |
| **`500`** | `Internal Server Error` | เกิดข้อผิดพลาดภายในฝั่ง Catalog Microservice หรือไม่สามารถเชื่อมต่อฐานข้อมูลสินค้าได้ | ดูตัวอย่างด้านล่าง |

#### ตัวอย่าง JSON Error กรณี `422 Unprocessable Content` (เช่น เรียก `?available=yes`)
```json
{
  "error": {
    "code": "INVALID_QUERY_PARAMETER",
    "message": "Query parameter 'available' must be either 'true' or 'false'.",
    "details": [
      {
        "field": "available",
        "rejectedValue": "yes"
      }
    ]
  }
}
```

#### ตัวอย่าง JSON Error กรณี `500 Internal Server Error`
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred while retrieving the product catalog."
  }
}
```




