
### 📘 ความรู้เชิงลึกเกี่ยวกับ RESTful API

RESTful API คือรูปแบบหนึ่งของ Web Service ที่ได้รับความนิยมสูงสุดในการพัฒนาโปรแกรมบนอินเทอร์เน็ตในปัจจุบัน โดย REST ย่อมาจาก **Representational State Transfer** ซึ่งเป็นแนวคิดที่แยกระบบ Client และ Server ออกจากกันอย่างชัดเจน และอาศัยการใช้ HTTP Method ในการดำเนินการกับ Resource (ทรัพยากร) ต่าง ๆ

#### หลักการสำคัญของ REST
1. **Stateless**: แต่ละคำขอ (Request) จาก Client จะต้องมีข้อมูลครบถ้วนในตัวมันเอง Server จะไม่เก็บสถานะ (Session) ระหว่างคำขอ
2. **Resource-based**: ข้อมูลถูกนำเสนอในรูปแบบของ Resource เช่น `/users`, `/products`
3. **HTTP Methods**: ใช้ HTTP Method ในการสื่อความหมาย เช่น `GET` เพื่อดึงข้อมูล, `POST` เพื่อเพิ่มข้อมูล, `PUT` เพื่ออัปเดตข้อมูล, `DELETE` เพื่อลบข้อมูล
4. **Uniform Interface**: การเข้าถึง API ควรมีรูปแบบมาตรฐาน เช่น การตั้งชื่อ endpoint ให้สื่อความหมาย
5. **Cacheable**: คำตอบ (Response) บางประเภทสามารถเก็บใน Cache ได้เพื่อเพิ่มประสิทธิภาพ
6. **Layered System**: อนุญาตให้ระบบมีหลายชั้น เช่น API Gateway, Load Balancer โดย Client ไม่จำเป็นต้องรู้รายละเอียดของทุกชั้น

#### HTTP Method กับ RESTful API

| Method | ใช้ทำอะไร | ตัวอย่าง |
|--------|------------|----------|
| GET | ดึงข้อมูล | GET /api/books |
| POST | สร้างข้อมูลใหม่ | POST /api/books |
| PUT | อัปเดตข้อมูลทั้งหมด | PUT /api/books/1 |
| PATCH | อัปเดตข้อมูลบางส่วน | PATCH /api/books/1 |
| DELETE | ลบข้อมูล | DELETE /api/books/1 |

#### HTTP Status Code ที่สำคัญ
- `200 OK`: การดำเนินการสำเร็จ
- `201 Created`: การสร้างข้อมูลสำเร็จ
- `204 No Content`: ไม่มีข้อมูลส่งกลับ (มักใช้กับ DELETE)
- `400 Bad Request`: ข้อมูลไม่ถูกต้อง
- `401 Unauthorized`: ไม่มีสิทธิ์ใช้งาน
- `404 Not Found`: ไม่พบ Resource ที่ร้องขอ
- `500 Internal Server Error`: เกิดข้อผิดพลาดในฝั่ง Server

#### รูปแบบของ REST API ที่ดี
- ใช้ **คำนาม** แทนคำกริยา เช่น `/users` แทน `/getUsers`
- ใช้ **URL Path** สำหรับ Resource ไม่ใช่ query string เช่น `/books/1` แทน `/books?id=1`
- แยก endpoint ตาม Resource และความสัมพันธ์ เช่น `/users/1/posts`

#### ตัวอย่าง JSON ที่ส่งหรือรับใน RESTful API

```json
{
    "id": 1,
    "title": "เรียนรู้ REST API",
    "author": "อาจารย์เขมวิทย์"
}
```

#### ความแตกต่างระหว่าง REST และ SOAP
| คุณสมบัติ | REST | SOAP |
|------------|------|------|
| ความเรียบง่าย | สูง | ต่ำ |
| การสนับสนุน JSON | ใช่ | ไม่ (ใช้ XML เท่านั้น) |
| น้ำหนักข้อมูล | เบา | หนัก |
| เหมาะสำหรับ | Web, Mobile | Enterprise-level, การเงิน |



### 🛠 Workshop: สร้างระบบบริหารจัดการหนังสือด้วย RESTful API (Book Management API)

#### วัตถุประสงค์:
- เรียนรู้การสร้าง RESTful API ที่สมบูรณ์
- ฝึกใช้ Express.js และ TypeScript ในการจัดการข้อมูล Resource
- ฝึกการใช้ HTTP Method อย่างถูกต้อง

#### โจทย์:
สร้าง API สำหรับจัดการรายการหนังสือในห้องสมุด โดยสามารถดำเนินการได้ดังนี้:
- ดูหนังสือทั้งหมด (GET)
- เพิ่มหนังสือ (POST)
- แก้ไขหนังสือ (PUT)
- ลบหนังสือ (DELETE)

#### โครงสร้างข้อมูลหนังสือ (Book):
```json
{
    "id": 1,
    "title": "Typescript เบื้องต้น",
    "author": "เขมวิทย์",
    "year": 2024
}
```

#### ขั้นตอนการทำ Workshop

##### 1. สร้างโปรเจกต์และติดตั้งแพ็กเกจ
```bash
mkdir book-api
cd book-api
npm init -y
npm install express cors body-parser
npm install typescript ts-node-dev @types/node @types/express --save-dev
npx tsc --init
```

##### 2. สร้างไฟล์ `server.ts`
```typescript
import express, { Request, Response } from 'express';
import cors from 'cors';
import bodyParser from 'body-parser';

const app = express();
const port = 3000;

app.use(cors());
app.use(bodyParser.json());

interface Book {
    id: number;
    title: string;
    author: string;
    year: number;
}

let books: Book[] = [];

app.get('/books', (req: Request, res: Response) => {
    res.json(books);
});

app.post('/books', (req: Request, res: Response) => {
    const newBook: Book = {
        id: books.length + 1,
        title: req.body.title,
        author: req.body.author,
        year: req.body.year
    };
    books.push(newBook);
    res.status(201).json(newBook);
});

app.put('/books/:id', (req: Request, res: Response) => {
    const id = parseInt(req.params.id);
    const index = books.findIndex(book => book.id === id);
    if (index !== -1) {
        books[index] = { ...books[index], ...req.body };
        res.json(books[index]);
    } else {
        res.status(404).json({ error: "Book not found" });
    }
});

app.delete('/books/:id', (req: Request, res: Response) => {
    const id = parseInt(req.params.id);
    books = books.filter(book => book.id !== id);
    res.status(204).send();
});

app.listen(port, () => {
    console.log(`Book API is running at http://localhost:${port}`);
});
```

##### 3. ทดสอบ API ผ่าน Postman หรือ curl:
- `GET http://localhost:3000/books`
- `POST http://localhost:3000/books`
- `PUT http://localhost:3000/books/1`
- `DELETE http://localhost:3000/books/1`

#### ผลลัพธ์ที่คาดหวัง:
- นักศึกษาสามารถอธิบายการใช้ RESTful API ได้
- เข้าใจการ Mapping ของ HTTP Method กับการกระทำ
- เขียนโค้ด API ด้วย TypeScript ได้อย่างถูกต้องและปลอดภัย

#### การบ้าน:
- เพิ่มการตรวจสอบ input เช่น ไม่อนุญาตให้ title ว่าง
- เพิ่ม endpoint `/books/search?author=xxx` เพื่อค้นหาตามผู้แต่ง
