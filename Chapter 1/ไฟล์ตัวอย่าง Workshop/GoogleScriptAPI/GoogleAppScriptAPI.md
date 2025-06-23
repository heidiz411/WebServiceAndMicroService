# **แอปพลิเคชัน CRUD ด้วย Google Apps Script**

โปรเจกต์นี้แสดงแอปพลิเคชัน **CRUD (สร้าง, อ่าน, อัปเดต, ลบ)** อย่างง่ายที่สร้างด้วย **Google Apps Script** โดยใช้ Google Sheet เป็นฐานข้อมูลเบื้องหลัง ประกอบด้วยส่วนหน้า (frontend) HTML พื้นฐานพร้อม jQuery สำหรับการโต้ตอบ และจุดเชื่อมต่อ API ที่แข็งแกร่งสำหรับการทดสอบผ่านเครื่องมือต่างๆ เช่น Postman

## **🚀 คุณสมบัติ**

* **สร้าง (Create):** เพิ่มรายการใหม่ (หนังสือ) พร้อมชื่อเรื่อง (Title) และผู้แต่ง (Author)  
* **อ่าน (Read):** แสดงรายการทั้งหมด หรือดึงข้อมูลรายการเฉพาะโดยใช้ ID  
* **อัปเดต (Update):** แก้ไขรายละเอียดรายการที่มีอยู่  
* **ลบ (Delete):** ลบรายการออกจากฐานข้อมูล  
* **Google Sheet Backend:** ข้อมูลถูกจัดเก็บและจัดการใน Google Sheet ทำให้ง่ายต่อการดูและแก้ไขโดยตรง  
* **Web App UI:** ส่วนหน้า HTML/CSS/jQuery พื้นฐานสำหรับการโต้ตอบกับผู้ใช้  
* **API Endpoints:** ฟังก์ชัน doGet() และ doPost() สำหรับการเข้าถึงแบบโปรแกรม เหมาะสำหรับการทดสอบด้วยเครื่องมือเช่น Postman

## **🛠️ ขั้นตอนการตั้งค่า**

### **1\. เตรียม Google Sheet ของคุณ**

1. สร้าง Google Sheet ใหม่ใน Google Drive ของคุณ  
2. เปลี่ยนชื่อชีตแรก (แท็บ) เป็น **Data**  
3. ในแถวแรกของชีต Data ให้เพิ่มหัวข้อต่อไปนี้อย่างแม่นยำ:  
   * ID (คอลัมน์ A)  
   * Title (คอลัมน์ B)  
   * Author (คอลัมน์ C)

ชีตของคุณควรมีลักษณะดังนี้:

| ID | Title | Author |
| :---- | :---- | :---- |

### **2\. ตั้งค่า Google Apps Script**

1. เปิด Google Sheet ของคุณ  
2. ไปที่ ส่วนขยาย (Extensions) \> Apps Script  
3. ในตัวแก้ไข Apps Script คุณจะเห็นไฟล์ Code.gs เริ่มต้น ให้แทนที่เนื้อหาด้วยโค้ดที่ให้ไว้ใน Code.gs จากโปรเจกต์นี้  
4. สร้างไฟล์ HTML ใหม่: ไปที่ ไฟล์ (File) \> ใหม่ (New) \> ไฟล์ HTML (HTML file) ตั้งชื่อว่า **Index.html**  
5. แทนที่เนื้อหาของ Index.html ด้วยโค้ดที่ให้ไว้ใน Index.html จากโปรเจกต์นี้  
   **หมายเหตุ:** ไฟล์ Index.html มีการรวมไลบรารี jQuery จาก CDN

### **3\. Deploy เป็น Web App**

เพื่อให้แอปพลิเคชันของคุณสามารถเข้าถึงได้ (ทั้ง UI และ API) คุณต้อง Deploy เป็น Web App

1. ในตัวแก้ไข Apps Script คลิก **ทำให้ใช้งานได้ (Deploy)** \> **การทำให้ใช้งานได้ใหม่ (New deployment)**  
2. สำหรับ "เลือกประเภท (Select type)", เลือก **เว็บแอป (Web app)**  
3. กำหนดการตั้งค่าการทำให้ใช้งานได้:  
   * **เรียกใช้ในฐานะ (Execute as):** ฉัน (Me) (บัญชี Google ของคุณ) \- สิ่งนี้จะให้สิทธิ์สคริปต์ในการเข้าถึง Google Sheet ของคุณ  
   * **ใครมีสิทธิ์เข้าถึง (Who has access):** ทุกคน (Anyone) (หรือ ทุกคนที่มีบัญชี Google (Anyone with Google account) หากคุณต้องการจำกัด) \- สำหรับการทดสอบด้วย Postman โดยทั่วไปแล้ว ทุกคน ก็เพียงพอ  
4. ระบุ **คำอธิบายการทำให้ใช้งานได้ (Deployment description)** (เช่น "การทำให้ใช้งานได้ CRUD เริ่มต้น")  
5. คลิก **ทำให้ใช้งานได้ (Deploy)**  
6. หน้าต่างป๊อปอัปจะแสดง **URL ของเว็บแอป (Web app URL)** ของคุณ (เช่น: https://script.google.com/macros/s/AKfyc.../exec) **คัดลอก URL นี้** เนื่องจากคุณจะต้องใช้สำหรับการทดสอบ Postman และการเข้าถึง UI ของเว็บแอป

### **4\. อัปเดตการทำให้ใช้งานได้หลังจากเปลี่ยนแปลง**

เมื่อใดก็ตามที่คุณแก้ไขโค้ดใน Code.gs หรือ Index.html คุณ **ต้อง Deploy เวอร์ชั่นใหม่** เพื่อให้การเปลี่ยนแปลงมีผล:

1. ไปที่ ทำให้ใช้งานได้ (Deploy) \> จัดการการทำให้ใช้งานได้ (Manage deployments)  
2. คลิกที่การทำให้ใช้งานได้ที่มีอยู่ที่คุณสร้างไว้  
3. คลิกไอคอน **แก้ไข (Edit)** (รูปดินสอ)  
4. ภายใต้ "เวอร์ชัน (Version)", เลือก **เวอร์ชันใหม่ (New version)**  
5. คลิก **ทำให้ใช้งานได้ (Deploy)**

## **🌐 การเข้าถึง UI ของ Web App**

วาง **URL ของเว็บแอป (Web app URL)** ที่คุณได้รับระหว่างการ Deploy ลงในเว็บเบราว์เซอร์ของคุณ คุณควรจะเห็นฟอร์มง่ายๆ และตารางสำหรับจัดการรายการ (หนังสือ) ของคุณ

## **🚀 การทดสอบ API ด้วย Postman**

Web App ของ Google Apps Script มีจุดเชื่อมต่อ API ที่คุณสามารถโต้ตอบด้วยโดยใช้ Postman หรือ HTTP client อื่นๆ คำขอ API ทั้งหมดใช้ **URL ของเว็บแอป (Web app URL)** เดียวกันเป็นฐาน

### คำขอ GET: อ่านรายการทั้งหมด

ใช้คำขอนี้เพื่อดึงรายการ (หนังสือ) ทั้งหมดจาก Google Sheet ของคุณ

* **Method:** GET  
* **URL:** https://play.google.com/store/apps/details?id=com.google.android.googlequicksearchbox\&hl=en  
* **Headers:** Content-Type: application/json  
* **Body:** (ไม่มี)

**ตัวอย่าง Response:**

{  
    "status": "success",  
    "data": \[  
        {  
            "id": "ITEM\_1700000000000",  
            "title": "My First Book",  
            "author": "Author A"  
        },  
        {  
            "id": "ITEM\_1700000000001",  
            "title": "Another Great Read",  
            "author": "Author B"  
        }  
    \]  
}

### คำขอ GET: อ่านรายการเฉพาะ

หากคุณต้องการดึงข้อมูลของรายการใดรายการหนึ่งโดยเฉพาะ ให้ระบุ id ของรายการนั้นๆ ผ่าน **Query Parameter**

* **Method:** GET  
* **URL:** https://play.google.com/store/apps/details?id=com.google.android.googlequicksearchbox\&hl=en?id=YOUR\_ITEM\_ID (เช่น: .../exec?id=ITEM\_1700000000000)  
* **Headers:** Content-Type: application/json  
* **Body:** (ไม่มี)

**ตัวอย่าง Response:**

{  
    "status": "success",  
    "data": {  
        "id": "ITEM\_1700000000000",  
        "title": "My First Book",  
        "author": "Author A"  
    }  
}

### คำขอ POST: สร้างรายการ

เพื่อเพิ่มรายการ (หนังสือ) ใหม่

* **Method:** POST  
* **URL:** https://play.google.com/store/apps/details?id=com.google.android.googlequicksearchbox\&hl=en  
* **Headers:** Content-Type: application/json  
* **Body (raw JSON):**  
  {  
      "action": "create",  
      "data": {  
          "title": "The Art of Apps Script",  
          "author": "Script Master"  
      }  
  }

**ตัวอย่าง Response:**

{  
    "status": "success",  
    "message": "Item created successfully",  
    "data": {  
        "id": "ITEM\_1719129037000",  
        "title": "The Art of Apps Script",  
        "author": "Script Master"  
    }  
}

### คำขอ POST: อัปเดตรายการ

เพื่อแก้ไขรายการที่มีอยู่ แทนที่ YOUR\_ITEM\_ID ด้วย ID จริงจากชีตของคุณ

* **Method:** POST  
* **URL:** https://play.google.com/store/apps/details?id=com.google.android.googlequicksearchbox\&hl=en  
* **Headers:** Content-Type: application/json  
* **Body (raw JSON):**  
  {  
      "action": "update",  
      "data": {  
          "id": "YOUR\_ITEM\_ID",  
          "title": "The Art of Apps Script (Revised Edition)",  
          "author": "Script Master Pro"  
      }  
  }

**ตัวอย่าง Response:**

{  
    "status": "success",  
    "message": "Item updated successfully",  
    "data": {  
        "id": "YOUR\_ITEM\_ID",  
        "title": "The Art of Apps Script (Revised Edition)",  
        "author": "Script Master Pro"  
    }  
}

### คำขอ POST: ลบรายการ

เพื่อลบรายการออกจากชีต แทนที่ YOUR\_ITEM\_ID ด้วย ID จริง

* **Method:** POST  
* **URL:** https://play.google.com/store/apps/details?id=com.google.android.googlequicksearchbox\&hl=en  
* **Headers:** Content-Type: application/json  
* **Body (raw JSON):**  
  {  
      "action": "delete",  
      "data": {  
          "id": "YOUR\_ITEM\_ID"  
      }  
  }

**ตัวอย่าง Response:**

{  
    "status": "success",  
    "message": "Item deleted successfully",  
    "data": null  
}

## **✍️ การมีส่วนร่วม**

ยินดีต้อนรับการ Fork โปรเจกต์นี้และปรับปรุงเพิ่มเติม\! ข้อเสนอแนะสำหรับการปรับปรุงจะได้รับการต้อนรับเสมอ