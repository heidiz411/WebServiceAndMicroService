# **การผสานรวม Google Apps Script API กับ Line Messaging API สำหรับ CRUD**

ในบทเรียนนี้ เราจะเรียนรู้วิธีการนำ RESTful API ที่สร้างด้วย Google Apps Script มาเชื่อมต่อกับ Line Messaging API เพื่อให้ผู้ใช้สามารถดำเนินการ CRUD (Create, Read, Update, Delete) บน Google Sheet ผ่านการส่งข้อความในแอปพลิเคชัน Line ได้โดยตรง

## **🎯 วัตถุประสงค์การเรียนรู้**

* เข้าใจหลักการทำงานของ Line Messaging API และ Webhook  
* เรียนรู้วิธีการตั้งค่า Line Developers Console สำหรับการเชื่อมต่อ Webhook  
* ปรับปรุงโค้ด Google Apps Script เพื่อทำหน้าที่เป็น Webhook Endpoint สำหรับ Line  
* สร้าง Logic ในการตีความคำสั่ง CRUD จากข้อความ Line  
* ทดสอบการทำงานของระบบ CRUD ผ่าน Line Chat

## **💡 ภาพรวมการทำงาน**

ระบบจะทำงานเป็นขั้นตอนดังนี้:

1. ผู้ใช้ส่งข้อความคำสั่ง CRUD (เช่น "สร้าง:ชื่อหนังสือ:ผู้แต่ง") ไปยัง Line Bot  
2. Line Messaging Platform รับข้อความและส่งต่อ (Webhook) ไปยัง Google Apps Script Web App ของเรา  
3. Google Apps Script Web App ประมวลผลข้อความ:  
   * แยกประเภทคำสั่ง (สร้าง, อ่าน, อัปเดต, ลบ)  
   * แยกข้อมูลที่จำเป็นจากคำสั่ง  
   * เรียกใช้ฟังก์ชัน CRUD ที่มีอยู่ใน Google Apps Script (createItem, readItems, updateItem, deleteItem)  
4. ฟังก์ชัน CRUD ทำงานกับ Google Sheet (เพิ่ม, อ่าน, แก้ไข, ลบข้อมูล)  
5. Google Apps Script Web App สร้างข้อความตอบกลับ (Response) ตามผลลัพธ์ของการดำเนินการ  
6. Line Messaging Platform ส่งข้อความตอบกลับนั้นกลับไปยังผู้ใช้ใน Line Chat

## **⚙️ ขั้นตอนการตั้งค่าและ Workshop**

### **ขั้นตอนที่ 1: การเตรียม Line Messaging API Channel**

ก่อนอื่น เราต้องมี Channel ใน Line Developers Console เพื่อใช้ในการสื่อสารกับ Line Bot ของเรา

1. **เข้าสู่ระบบ Line Developers Console:**  
   * ไปที่ [https://developers.line.biz/](https://developers.line.biz/) และเข้าสู่ระบบด้วยบัญชี Line ของคุณ  
2. **สร้าง Provider (หากยังไม่มี):**  
   * คลิก Providers \> Create new provider ตั้งชื่อ Provider ตามที่คุณต้องการ (เช่น "My Apps Script Bots")  
3. **สร้าง Messaging API Channel:**  
   * ภายใน Provider ที่สร้างขึ้น คลิก Create a new channel  
   * เลือก Messaging API  
   * กรอกข้อมูลช่อง:  
     * **Channel Type:** Messaging API  
     * **Provider:** เลือก Provider ที่คุณสร้าง  
     * **Channel icon:** (ไม่บังคับ)  
     * **Channel name:** ตั้งชื่อบอทของคุณ (เช่น "CRUD Bot")  
     * **Channel description:** คำอธิบายบอท  
     * **Large category / Small category:** เลือกหมวดหมู่ที่เหมาะสม  
     * **Email address:** อีเมลของคุณ  
   * ยอมรับข้อกำหนดและเงื่อนไขทั้งหมด แล้วคลิก Create  
4. **คัดลอก Channel Access Token และ Channel Secret:**  
   * เมื่อสร้าง Channel สำเร็จ ไปที่แท็บ Basic settings (การตั้งค่าพื้นฐาน)  
   * เลื่อนลงมาที่ส่วน **"Channel secret"** และ **"Channel access token (long-lived)"** คัดลอกค่าทั้งสองเก็บไว้ คุณจะต้องใช้ในโค้ด Apps Script ของเรา  
   * **สิ่งสำคัญ:** ในแท็บ Messaging API settings (การตั้งค่า Messaging API)  
     * เปิดใช้งาน Webhook URL โดยเลื่อนสวิตช์ Use webhook เป็น ON (สีเขียว)  
     * Webhook URL จะถูกกำหนดในขั้นตอนถัดไป

### **ขั้นตอนที่ 2: การปรับปรุง Google Apps Script (Code.gs)**

เราจะปรับปรุงไฟล์ Code.gs ที่มีอยู่เพื่อให้สามารถรับคำขอจาก Line Webhook, ประมวลผลคำสั่ง และส่งข้อความตอบกลับได้

**เปิด Apps Script Editor ของคุณ (จาก Google Sheet ที่เชื่อมโยง) และอัปเดตไฟล์ Code.gs ดังนี้:**

// \--- GLOBAL CONFIGURATION (โค้ดการกำหนดค่าสากล) \---  
const SHEET\_NAME \= "Data";  
const ID\_COLUMN \= 1; // Column A  
const TITLE\_COLUMN \= 2; // Column B  
const AUTHOR\_COLUMN \= 3; // Column C

// แทนที่ด้วยค่า Channel Access Token และ Channel Secret ของคุณจาก Line Developers Console  
const LINE\_CHANNEL\_ACCESS\_TOKEN \= "YOUR\_LINE\_CHANNEL\_ACCESS\_TOKEN"; // เช่น "Ew...="  
const LINE\_CHANNEL\_SECRET \= "YOUR\_LINE\_CHANNEL\_SECRET"; // เช่น "123..."

/\*\*  
 \* Gets the active sheet.  
 \* @returns {GoogleAppsScript.Spreadsheet.Sheet} The active sheet.  
 \*/  
function getSheet() {  
  return SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET\_NAME);  
}

/\*\*  
 \* Generates a unique ID (simple example, improve for production).  
 \* @returns {string} A unique ID.  
 \*/  
function generateId() {  
  return "ITEM\_" \+ new Date().getTime();  
}

/\*\*  
 \* CREATE: Adds a new item to the sheet.  
 \* @param {Object} itemData \- An object containing title and author.  
 \* @returns {Object} The created item with its ID.  
 \*/  
function createItem(itemData) {  
  const sheet \= getSheet();  
  const newId \= generateId();  
  const row \= \[newId, itemData.title, itemData.author\];  
  sheet.appendRow(row);  
  return { id: newId, title: itemData.title, author: itemData.author };  
}

/\*\*  
 \* READ: Retrieves all items or a specific item by ID.  
 \* @param {string} \[itemId\] \- Optional ID of the item to retrieve. If not provided, all items are returned.  
 \* @returns {Array\<Object\>|Object|null} An array of items, a single item object, or null if not found.  
 \*/  
function readItems(itemId) {  
  const sheet \= getSheet();  
  const range \= sheet.getDataRange();  
  const values \= range.getValues();

  if (values.length \< 2\) return \[\]; // No data rows, only header

  const dataRows \= values.slice(1); // Skip header row

  const items \= dataRows.map(row \=\> ({  
    id: row\[ID\_COLUMN \- 1\],  
    title: row\[TITLE\_COLUMN \- 1\],  
    author: row\[AUTHOR\_COLUMN \- 1\]  
  }));

  if (itemId) {  
    return items.find(item \=\> item.id \=== itemId) || null;  
  }  
  return items;  
}

/\*\*  
 \* UPDATE: Updates an existing item by ID.  
 \* @param {Object} itemData \- An object containing id, title, and author.  
 \* @returns {Object|null} The updated item object, or null if not found.  
 \*/  
function updateItem(itemData) {  
  const sheet \= getSheet();  
  const range \= sheet.getDataRange();  
  const values \= range.getValues();

  let rowIndex \= \-1;  
  for (let i \= 1; i \< values.length; i++) { // Start from 1 to skip header  
    if (values\[i\]\[ID\_COLUMN \- 1\] \=== itemData.id) {  
      rowIndex \= i;  
      break;  
    }  
  }

  if (rowIndex \!== \-1) {  
    sheet.getRange(rowIndex \+ 1, TITLE\_COLUMN).setValue(itemData.title);  
    sheet.getRange(rowIndex \+ 1, AUTHOR\_COLUMN).setValue(itemData.author);  
    return itemData;  
  }  
  return null;  
}

/\*\*  
 \* DELETE: Deletes an item by ID.  
 \* @param {string} itemId \- The ID of the item to delete.  
 \* @returns {boolean} True if deleted, false otherwise.  
 \*/  
function deleteItem(itemId) {  
  const sheet \= getSheet();  
  const range \= sheet.getDataRange();  
  const values \= range.getValues();

  let rowIndex \= \-1;  
  for (let i \= 1; i \< values.length; i++) { // Start from 1 to skip header  
    if (values\[i\]\[ID\_COLUMN \- 1\] \=== itemId) {  
      rowIndex \= i;  
      break;  
    }  
  }

  if (rowIndex \!== \-1) {  
    sheet.deleteRow(rowIndex \+ 1); // \+1 because sheet rows are 1-indexed  
    return true;  
  }  
  return false;  
}

// \--- LINE MESSAGING API INTEGRATION (การผสานรวม Line Messaging API) \---

/\*\*  
 \* Sends a reply message to Line.  
 \* @param {string} replyToken \- The reply token received from Line.  
 \* @param {string} messageText \- The text message to send back.  
 \*/  
function replyToLine(replyToken, messageText) {  
  const url \= "https://api.line.me/v2/bot/message/reply";  
  const headers \= {  
    "Content-Type": "application/json",  
    "Authorization": "Bearer " \+ LINE\_CHANNEL\_ACCESS\_TOKEN  
  };  
  const payload \= {  
    replyToken: replyToken,  
    messages: \[{  
      type: "text",  
      text: messageText  
    }\]  
  };

  const options \= {  
    method: "post",  
    headers: headers,  
    payload: JSON.stringify(payload)  
  };

  try {  
    UrlFetchApp.fetch(url, options);  
  } catch (e) {  
    console.error("Failed to send reply to Line:", e);  
  }  
}

/\*\*  
 \* Handles incoming Line webhook POST requests.  
 \* This function will be the main entry point for Line messages.  
 \* @param {GoogleAppsScript.Events.DoPost} e \- The event object from the POST request.  
 \* @returns {GoogleAppsScript.Content.TextOutput} A JSON response.  
 \*/  
function doPost(e) {  
  let replyMessage \= "ขออภัย ไม่เข้าใจคำสั่งของคุณ กรุณาลองใหม่";

  try {  
    const json \= JSON.parse(e.postData.contents);  
    const events \= json.events;

    if (events && events.length \> 0\) {  
      const event \= events\[0\]; // Process the first event  
      const replyToken \= event.replyToken;  
      const userMessage \= event.message.text;

      console.log("Received message:", userMessage);

      // Define command format: ACTION:param1:param2...  
      // Examples:  
      // สร้าง:ชื่อหนังสือ:ผู้แต่ง  
      // อ่าน:ID\_ของ\_หนังสือ  
      // อ่านทั้งหมด  
      // อัปเดต:ID\_ของ\_หนังสือ:ชื่อใหม่:ผู้แต่งใหม่  
      // ลบ:ID\_ของ\_หนังสือ

      const parts \= userMessage.split(':');  
      const action \= parts\[0\].toLowerCase();

      switch (action) {  
        case 'สร้าง':  
          if (parts.length \=== 3\) {  
            const title \= parts\[1\].trim();  
            const author \= parts\[2\].trim();  
            const newItem \= createItem({ title: title, author: author });  
            replyMessage \= \`สร้างสำเร็จ\!\\nID: ${newItem.id}\\nชื่อเรื่อง: ${newItem.title}\\nผู้แต่ง: ${newItem.author}\`;  
          } else {  
            replyMessage \= "รูปแบบคำสั่งสร้างไม่ถูกต้อง: สร้าง:ชื่อหนังสือ:ผู้แต่ง";  
          }  
          break;

        case 'อ่าน':  
          if (parts.length \=== 2\) { // Read specific item  
            const itemId \= parts\[1\].trim();  
            const item \= readItems(itemId);  
            if (item) {  
              replyMessage \= \`ข้อมูล:\\nID: ${item.id}\\nชื่อเรื่อง: ${item.title}\\nผู้แต่ง: ${item.author}\`;  
            } else {  
              replyMessage \= \`ไม่พบรายการ ID: ${itemId}\`;  
            }  
          } else if (userMessage.toLowerCase() \=== 'อ่านทั้งหมด') { // Read all items  
            const allItems \= readItems();  
            if (allItems.length \> 0\) {  
              replyMessage \= "รายการทั้งหมด:\\n";  
              allItems.forEach(item \=\> {  
                replyMessage \+= \`ID: ${item.id}, ชื่อเรื่อง: ${item.title}, ผู้แต่ง: ${item.author}\\n\`;  
              });  
            } else {  
              replyMessage \= "ไม่มีรายการในระบบ";  
            }  
          } else {  
            replyMessage \= "รูปแบบคำสั่งอ่านไม่ถูกต้อง: อ่าน:ID\_ของ\_หนังสือ หรือ อ่านทั้งหมด";  
          }  
          break;

        case 'อัปเดต':  
          if (parts.length \=== 4\) {  
            const itemId \= parts\[1\].trim();  
            const newTitle \= parts\[2\].trim();  
            const newAuthor \= parts\[3\].trim();  
            const updatedItem \= updateItem({ id: itemId, title: newTitle, author: newAuthor });  
            if (updatedItem) {  
              replyMessage \= \`อัปเดตสำเร็จ\!\\nID: ${updatedItem.id}\\nชื่อเรื่อง: ${updatedItem.title}\\nผู้แต่ง: ${updatedItem.author}\`;  
            } else {  
              replyMessage \= \`ไม่พบรายการ ID: ${itemId} สำหรับอัปเดต\`;  
            }  
          } else {  
            replyMessage \= "รูปแบบคำสั่งอัปเดตไม่ถูกต้อง: อัปเดต:ID\_ของ\_หนังสือ:ชื่อใหม่:ผู้แต่งใหม่";  
          }  
          break;

        case 'ลบ':  
          if (parts.length \=== 2\) {  
            const itemId \= parts\[1\].trim();  
            const deleted \= deleteItem(itemId);  
            if (deleted) {  
              replyMessage \= \`ลบรายการ ID: ${itemId} สำเร็จแล้ว\`;  
            } else {  
              replyMessage \= \`ไม่พบรายการ ID: ${itemId} สำหรับลบ\`;  
            }  
          } else {  
            replyMessage \= "รูปแบบคำสั่งลบไม่ถูกต้อง: ลบ:ID\_ของ\_หนังสือ";  
          }  
          break;

        default:  
          replyMessage \= "คำสั่งไม่ถูกต้อง กรุณาใช้: สร้าง, อ่าน, อัปเดต, ลบ หรือ อ่านทั้งหมด";  
          break;  
      }

      replyToLine(replyToken, replyMessage);  
    }  
  } catch (e) {  
    console.error("Error processing webhook:", e);  
    // You might want to reply with an error message to the user here as well  
  }

  // Acknowledge the webhook received  
  return ContentService.createTextOutput(JSON.stringify({ status: "success" }))  
    .setMimeType(ContentService.MimeType.JSON);  
}

// This doGet is for serving the HTML, not for API calls directly  
/\*\*  
 \* Serves the HTML file for the web app.  
 \* @returns {GoogleAppsScript.HTML.HtmlOutput} The HTML output.  
 \*/  
function doGet() {  
  return HtmlService.createTemplateFromFile('Index').evaluate()  
      .setTitle('CRUD App')  
      .setSandboxMode(HtmlService.SandboxMode.IFRAME);  
}

**สิ่งที่ต้องทำในการปรับโค้ด Code.gs:**

* **สำคัญมาก:** แทนที่ YOUR\_LINE\_CHANNEL\_ACCESS\_TOKEN และ YOUR\_LINE\_CHANNEL\_SECRET ด้วยค่าจริงที่คุณคัดลอกมาจาก Line Developers Console  
* doPost(e): ถูกเขียนขึ้นใหม่เพื่อรับ JSON payload จาก Line Webhook  
* replyToLine(): เป็นฟังก์ชันสำหรับส่งข้อความตอบกลับไปยัง Line User  
* Logic ภายใน doPost จะแยกคำสั่งจากข้อความของผู้ใช้ (ตามรูปแบบ คำสั่ง:พารามิเตอร์1:พารามิเตอร์2) และเรียกใช้ฟังก์ชัน CRUD ที่เหมาะสม

### **ขั้นตอนที่ 3: การ Deploy Google Apps Script เป็น Web App (สำหรับ Line Webhook)**

เราจะ Deploy สคริปต์ที่อัปเดตแล้วเป็น Web App เพื่อให้ Line Webhook สามารถเรียกใช้งานได้

1. ใน Apps Script editor, คลิก **ทำให้ใช้งานได้ (Deploy)** \> **การทำให้ใช้งานได้ใหม่ (New deployment)**  
2. สำหรับ "เลือกประเภท (Select type)", เลือก **เว็บแอป (Web app)**  
3. กำหนดการตั้งค่าการทำให้ใช้งานได้:  
   * **เรียกใช้ในฐานะ (Execute as):** ฉัน (Me) (บัญชี Google ของคุณ) \- **นี่สำคัญมาก** เนื่องจากสคริปต์จำเป็นต้องมีสิทธิ์ในการเข้าถึงและแก้ไข Google Sheet ในฐานะคุณ  
   * **ใครมีสิทธิ์เข้าถึง (Who has access):** ทุกคน (Anyone) \- **นี่สำคัญมาก** เพื่อให้ Line Webhook สามารถเรียกใช้งานได้โดยไม่ต้องมีการรับรองสิทธิ์เพิ่มเติม (Line Messaging API จะใช้ Channel Secret ในการยืนยันตัวตนแทน)  
4. ระบุ **คำอธิบายการทำให้ใช้งานได้ (Deployment description)** (เช่น "Line Webhook Endpoint")  
5. คลิก **ทำให้ใช้งานได้ (Deploy)**  
6. หน้าต่างป๊อปอัปจะแสดง **URL ของเว็บแอป (Web app URL)** ของคุณ (เช่น: https://script.google.com/macros/s/AKfyc.../exec) **คัดลอก URL นี้เก็บไว้ให้ดี** นี่คือ URL ที่คุณจะใช้ใน Line Developers Console เป็น Webhook URL

**หมายเหตุสำคัญ:** ทุกครั้งที่คุณแก้ไขโค้ดใน Code.gs และต้องการให้การเปลี่ยนแปลงมีผล คุณต้อง **Deploy เวอร์ชั่นใหม่** เสมอ โดยไปที่ ทำให้ใช้งานได้ (Deploy) \> จัดการการทำให้ใช้งานได้ (Manage deployments) \> คลิกที่ deployment ที่มีอยู่ \> แก้ไข (Edit) (รูปดินสอ) \> เวอร์ชันใหม่ (New version) \> ทำให้ใช้งานได้ (Deploy)

### **ขั้นตอนที่ 4: การตั้งค่า Webhook ใน Line Developers Console**

1. กลับไปที่ Line Developers Console ของ Channel ที่คุณสร้างขึ้น (Messaging API settings tab)  
2. เลื่อนลงมาที่ส่วน **"Webhook settings"**  
3. ในช่อง **"Webhook URL"** วาง URL ของ Google Apps Script Web App ที่คุณคัดลอกมาในขั้นตอนที่ 3  
4. คลิกปุ่ม **Verify** เพื่อตรวจสอบว่า Line สามารถเข้าถึง Webhook URL ของคุณได้ หากสำเร็จจะขึ้นข้อความ "Success"  
5. ตรวจสอบให้แน่ใจว่า **Use webhook** เปิดใช้งานอยู่ (สวิตช์เป็น ON สีเขียว)  
6. (ไม่บังคับ) หากคุณไม่ต้องการให้ Line Bot ตอบกลับข้อความอัตโนมัติ (เช่น ข้อความต้อนรับ) คุณสามารถปิด Auto-reply messages และ Greeting messages ในส่วน "LINE Official Account features" เพื่อให้บอทของเราควบคุมการตอบกลับทั้งหมด

## **👩‍🏫 Workshop: การทดสอบผ่าน Line Chat**

ตอนนี้ระบบของคุณพร้อมสำหรับการทดสอบแล้ว\! เปิดแอปพลิเคชัน Line บนมือถือของคุณ และสแกน QR Code ของบอทของคุณ (อยู่ในแท็บ Messaging API settings ของ Channel ใน Line Developers Console) หรือค้นหาบอทด้วยชื่อที่คุณตั้งไว้ แล้วเริ่มทดสอบได้เลย

### **รูปแบบคำสั่งที่ใช้**

เราได้กำหนดรูปแบบคำสั่งง่ายๆ ดังนี้: คำสั่ง:ข้อมูล1:ข้อมูล2...

1. **สร้าง (Create): เพิ่มหนังสือใหม่**  
   * **คำสั่ง:** สร้าง:ชื่อหนังสือของคุณ:ชื่อผู้แต่งของคุณ  
   * **ตัวอย่าง:** สร้าง:แฮร์รี่ พอตเตอร์ ภาค1:เจ.เค. โรว์ลิ่ง  
   * **ผลลัพธ์ที่คาดหวัง:** บอทจะตอบกลับด้วย ID ของรายการที่สร้างขึ้น พร้อมชื่อเรื่องและผู้แต่ง  
2. **อ่าน (Read): อ่านข้อมูลหนังสือ**  
   * **อ่านรายการเฉพาะ:**  
     * **คำสั่ง:** อ่าน:ID\_ของ\_หนังสือ  
     * **ตัวอย่าง:** อ่าน:ITEM\_1719129037000 (ใช้ ID จริงที่บอทตอบกลับมา)  
     * **ผลลัพธ์ที่คาดหวัง:** บอทจะตอบกลับด้วยข้อมูลของหนังสือเล่มนั้น  
   * **อ่านทั้งหมด:**  
     * **คำสั่ง:** อ่านทั้งหมด  
     * **ตัวอย่าง:** อ่านทั้งหมด  
     * **ผลลัพธ์ที่คาดหวัง:** บอทจะแสดงรายการหนังสือทั้งหมดใน Google Sheet ของคุณ  
3. **อัปเดต (Update): แก้ไขข้อมูลหนังสือ**  
   * **คำสั่ง:** อัปเดต:ID\_ของ\_หนังสือ:ชื่อหนังสือใหม่:ชื่อผู้แต่งใหม่  
   * **ตัวอย่าง:** อัปเดต:ITEM\_1719129037000:แฮร์รี่ พอตเตอร์ ภาค1 (ฉบับปรับปรุง):โรว์ลิ่ง  
   * **ผลลัพธ์ที่คาดหวัง:** บอทจะตอบกลับด้วยข้อมูลที่อัปเดตแล้ว  
4. **ลบ (Delete): ลบหนังสือ**  
   * **คำสั่ง:** ลบ:ID\_ของ\_หนังสือ  
   * **ตัวอย่าง:** ลบ:ITEM\_1719129037000  
   * **ผลลัพธ์ที่คาดหวัง:** บอทจะตอบกลับว่าลบรายการสำเร็จแล้ว

### **การตรวจสอบ**

* **ใน Line Chat:** ตรวจสอบข้อความตอบกลับจากบอทของคุณ  
* **ใน Google Sheet:** ตรวจสอบว่าข้อมูลในชีต Data ถูกเพิ่ม, อ่าน, แก้ไข หรือลบตามคำสั่งที่คุณส่งไป  
* **ใน Apps Script Editor (ส่วน Execution logs):** หากมีข้อผิดพลาดใดๆ เกิดขึ้น ให้ตรวจสอบ Execution logs เพื่อหาสาเหตุ

## **⚠️ ข้อควรพิจารณาเพิ่มเติม**

* **ความปลอดภัย:** สำหรับแอปพลิเคชันที่ใช้งานจริง ควรมีการตรวจสอบ Channel Secret เพื่อยืนยันว่าคำขอ Webhook มาจาก Line จริงๆ เพื่อป้องกันการโจมตี (ในตัวอย่างนี้ไม่ได้รวมไว้เพื่อความเรียบง่ายในการสอน)  
* **การจัดการข้อผิดพลาด:** เพิ่มการจัดการข้อผิดพลาดที่ซับซ้อนขึ้นใน Apps Script เพื่อให้ผู้ใช้ได้รับข้อความที่ชัดเจนและเป็นประโยชน์เมื่อเกิดปัญหา  
* **รูปแบบคำสั่ง:** สามารถพัฒนาให้รูปแบบคำสั่งมีความยืดหยุ่นหรือเป็นมิตรกับผู้ใช้มากขึ้น เช่น การใช้ปุ่ม (Quick Reply) หรือ Rich Menus ใน Line  
* **การขยายคุณสมบัติ:** สามารถเพิ่มคุณสมบัติอื่นๆ เช่น การค้นหาแบบยืดหยุ่น, การจัดการสถานะ, การแจ้งเตือนอัตโนมัติ เป็นต้น
