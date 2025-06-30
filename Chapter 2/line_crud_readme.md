# **แอปพลิเคชัน CRUD ด้วย Google Apps Script และ Line Messaging API**

โปรเจกต์นี้เป็นการต่อยอดจากแอปพลิเคชัน CRUD เดิมที่ใช้ Google Apps Script และ Google Sheet โดยเพิ่มการผสานรวมกับ **Line Messaging API** ทำให้ผู้ใช้สามารถสั่งการ **CRUD (Create, Read, Update, Delete)** ข้อมูลผ่านการส่งข้อความใน Line Chat ได้โดยตรง

## **🚀 คุณสมบัติหลัก**

* **CRUD Operations ผ่าน Line:** ผู้ใช้สามารถ สร้าง, อ่าน, อัปเดต, และลบข้อมูล (หนังสือ) ได้โดยส่งข้อความคำสั่งไปยัง Line Bot  
* **Google Sheet Backend:** ข้อมูลยังคงถูกจัดเก็บและจัดการใน Google Sheet  
* **Google Apps Script Webhook:** โค้ด Apps Script ทำหน้าที่เป็น Webhook Endpoint สำหรับรับและประมวลผลข้อความจาก Line  
* **การตอบกลับแบบทันที:** Line Bot จะตอบกลับสถานะและผลลัพธ์ของการดำเนินการทันที  
* **Web App UI (Optional):** ยังคงมีหน้าเว็บ UI เดิมสำหรับจัดการข้อมูลผ่าน Browser (หากมีการ Deploy)

## **⚙️ โครงสร้างโปรเจกต์**

* **Code.gs**: ไฟล์หลักของ Google Apps Script ที่มีฟังก์ชัน CRUD (Create, Read, Update, Delete) สำหรับ Google Sheet และ Logic สำหรับรับ Line Webhook, ตีความคำสั่ง, และส่งข้อความตอบกลับ  
* **Index.html**: ไฟล์ HTML สำหรับ UI ของ Web App (หากมีการใช้งาน) ซึ่งใช้ jQuery ในการจัดการส่วนหน้า

## **🛠️ ขั้นตอนการตั้งค่า**

### **1\. การเตรียม Google Sheet**

1. สร้าง Google Sheet ใหม่ใน Google Drive ของคุณ  
2. เปลี่ยนชื่อชีตแรก (แท็บ) เป็น **Data**  
3. ในแถวแรกของชีต Data ให้เพิ่มหัวข้อต่อไปนี้อย่างแม่นยำ:  
   * ID (คอลัมน์ A)  
   * Title (คอลัมน์ B)  
   * Author (คอลัมน์ C)

ตัวอย่าง:| ID | Title | Author || :-- | :---- | :----- |

### **2\. การตั้งค่า Line Messaging API Channel**

1. เข้าสู่ระบบ [Line Developers Console](https://developers.line.biz/)  
2. สร้าง Provider (ถ้ายังไม่มี)  
3. สร้าง **Messaging API Channel**  
4. ในหน้าการตั้งค่า Channel ของคุณ (Basic settings):  
   * คัดลอก **Channel access token (long-lived)**  
   * คัดลอก **Channel secret**  
5. ในแท็บ Messaging API settings:  
   * เลื่อนสวิตช์ Use webhook เป็น **ON**

### **3\. การตั้งค่า Google Apps Script**

1. เปิด Google Sheet ของคุณ และไปที่ ส่วนขยาย (Extensions) \> Apps Script  
2. ในตัวแก้ไข Apps Script:  
   * เปิดไฟล์ Code.gs และแทนที่เนื้อหาด้วยโค้ดจาก Code.gs ของโปรเจกต์นี้  
   * **สำคัญมาก:** ในไฟล์ Code.gs ให้แทนที่ค่า Placeholder ด้วย Channel access token และ Channel secret ที่คุณคัดลอกมาจาก Line:  
     const LINE\_CHANNEL\_ACCESS\_TOKEN \= "YOUR\_LINE\_CHANNEL\_ACCESS\_TOKEN";  
     const LINE\_CHANNEL\_SECRET \= "YOUR\_LINE\_CHANNEL\_SECRET";

   * (ไม่บังคับ) หากคุณต้องการ UI ด้วย ให้สร้างไฟล์ Index.html และวางโค้ดจาก Index.html ของโปรเจกต์นี้ลงไป

### **4\. การ Deploy Google Apps Script เป็น Web App**

เพื่อให้ Apps Script ของคุณทำงานเป็น Webhook Endpoint:

1. ใน Apps Script editor, คลิก **ทำให้ใช้งานได้ (Deploy)** \> **การทำให้ใช้งานได้ใหม่ (New deployment)**  
2. เลือกประเภท **เว็บแอป (Web app)**  
3. กำหนดค่า:  
   * **เรียกใช้ในฐานะ (Execute as):** ฉัน (Me)  
   * **ใครมีสิทธิ์เข้าถึง (Who has access):** ทุกคน (Anyone)  
4. คลิก **ทำให้ใช้งานได้ (Deploy)**  
5. คัดลอก **URL ของเว็บแอป (Web app URL)** ที่ปรากฏขึ้น

**หมายเหตุ:** ทุกครั้งที่แก้ไขโค้ด Code.gs คุณต้อง Deploy เวอร์ชั่นใหม่ เพื่อให้การเปลี่ยนแปลงมีผล

### **5\. การตั้งค่า Webhook ใน Line Developers Console**

1. กลับไปที่ Line Developers Console ของ Channel คุณ (Messaging API settings tab)  
2. ในส่วน **Webhook settings** วาง **URL ของเว็บแอป** ที่คุณคัดลอกมาลงในช่อง **Webhook URL**  
3. คลิก **Verify** เพื่อทดสอบการเชื่อมต่อ (ควรขึ้น Success)  
4. ตรวจสอบให้แน่ใจว่า **Use webhook** เปิดใช้งานอยู่ (ON)

## **🚀 การใช้งานและการทดสอบ**

เปิดแอปพลิเคชัน Line ของคุณ และเพิ่ม Line Bot ของคุณเป็นเพื่อน (สแกน QR Code หรือค้นหาจาก ID บอท) จากนั้นทดลองส่งข้อความคำสั่งตามรูปแบบที่กำหนด:

### **รูปแบบคำสั่ง**

คำสั่งจะใช้รูปแบบ คำสั่ง:ข้อมูล1:ข้อมูล2...

1. **สร้าง (Create): เพิ่มหนังสือใหม่**  
   * สร้าง:ชื่อหนังสือของคุณ:ชื่อผู้แต่งของคุณ  
   * **ตัวอย่าง:** สร้าง:แฮร์รี่ พอตเตอร์ ภาค1:เจ.เค. โรว์ลิ่ง  
2. **อ่าน (Read): อ่านข้อมูลหนังสือ**  
   * **อ่านรายการเฉพาะ:** อ่าน:ID\_ของ\_หนังสือ  
     * **ตัวอย่าง:** อ่าน:ITEM\_1719129037000  
   * **อ่านทั้งหมด:** อ่านทั้งหมด  
3. **อัปเดต (Update): แก้ไขข้อมูลหนังสือ**  
   * อัปเดต:ID\_ของ\_หนังสือ:ชื่อหนังสือใหม่:ชื่อผู้แต่งใหม่  
   * **ตัวอย่าง:** อัปเดต:ITEM\_1719129037000:แฮร์รี่ พอตเตอร์ ภาค1 (ฉบับปรับปรุง):โรว์ลิ่ง  
4. **ลบ (Delete): ลบหนังสือ**  
   * ลบ:ID\_ของ\_หนังสือ  
   * **ตัวอย่าง:** ลบ:ITEM\_1719129037000

## **⚠️ ข้อควรระวัง**

* **ความปลอดภัย:** สำหรับโปรดักชัน ควรมีการตรวจสอบ Request Signature เพื่อยืนยันว่าคำขอมาจาก Line จริงๆ  
* **การจัดการข้อผิดพลาด:** เพิ่ม Logic การจัดการข้อผิดพลาดใน Apps Script ให้ครอบคลุมมากขึ้น  
* **รูปแบบคำสั่ง:** สามารถขยายให้รองรับคำสั่งที่ซับซ้อน หรือใช้ UI Elements ของ Line เช่น Quick Reply Buttons เพื่อเพิ่มประสบการณ์ผู้ใช้
