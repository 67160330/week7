# Retail Data Warehouse ETL Pipeline

ระบบ ETL Data Pipeline อัตโนมัติด้วย Python และ SQLite สำหรับดึงข้อมูลรายการสั่งซื้อ (Orders Batches), ทำความสะอาดข้อมูล (Data Cleaning & Validation), จัดเก็บข้อมูลมีปัญหาลง Quarantine, และโหลดข้อมูลเข้าสู่ Data Warehouse ในรูปแบบ Star Schema

---

## 1. วิธีติดตั้ง (Installation)

**ความต้องการของระบบ (Prerequisites)**
* **Python**: เวอร์ชัน 3.9 ขึ้นไป
* **ไลบรารีที่ต้องใช้**: `pandas`, `openpyxl`, `sqlite3` (มาพร้อม Python Standard Library)

**ขั้นตอนการติดตั้ง**
1. วางไฟล์ชุดข้อมูล `Python_Data_Pipeline_Lab_Dataset.xlsx` ไว้ในโฟลเดอร์เดียวกันกับโปรเจกต์
2. ติดตั้ง Dependencies สำหรับอ่านไฟล์ Excel:
   ```bash
   pip install pandas openpyxl
   ```

---

## 2. วิธีรันโปรแกรม (How to Run)

เรียกใช้คำสั่งผ่าน Terminal หรือ Command Prompt:
```bash
python pipeline.py
```

---

## 3. โครงสร้าง Star Schema (Data Warehouse Architecture)

Data Warehouse ออกแบบตามสถาปัตยกรรม Star Schema ประกอบด้วย 1 Fact Table (`fact_sales`) และ 3 Dimension Tables (`dim_customer`, `dim_product`, `dim_date`):

```text
                     +-------------------+
                     |   dim_customer    |
                     +-------------------+
                     | customer_key (PK) | <----+
                     | customer_id       |      |
                     | customer_name     |      |
                     | province          |      |
                     | segment           |      |
                     +-------------------+      |
                                                |
+-----------------+                    +--------+-------+
|    dim_date     |                    |   fact_sales   |
+-----------------+                    +----------------+
| date_key (PK)   | <------------------| date_key (FK)  |
| full_date       |                    | customer_key   |
| day             |                    | product_key    | <---+
| month           |                    | quantity       |     |
| quarter         |                    | unit_price     |     |
| year            |                    | discount_pct   |     |
+-----------------+                    | gross_amount   |     |
                                       | net_amount     |     |
                                       | payment_method |     |
                                       | sales_channel  |     |
                                       | updated_at     |     |
                                       +----------------+     |
                                                              |
                                       +------------------+   |
                                       |   dim_product    |   |
                                       +------------------+   |
                                       | product_key (PK) |---+
                                       | product_id       |
                                       | product_name     |
                                       | category         |
                                       +------------------+
```

---

## 4. คำตอบ Reflection

**Q: เหตุใด Data Availability จึงมักมีความสำคัญมากกว่า Strictness ใน Production Data Pipeline?**

**A:** ในระบบ Production Data Pipeline ระดับองค์กร **Data Availability (ความพร้อมใช้งานของข้อมูล)** มีความสำคัญอย่างยิ่งยวดเนื่องจาก Dashboard ผู้บริหาร ระบบ BI และเครื่องมือ Downstream Analytics ต้องใช้ข้อมูลเพื่อการตัดสินใจอย่างต่อเนื่อง

หากใช้แนวทาง **Strictness (Hard Stop)** ที่สั่งให้ Pipeline หยุดทำงานกะทันหันทันทีเมื่อเจอข้อมูลผิดพลาดแม้เพียง 1 รายการ จะส่งผลให้ข้อมูลทั้งหมดใน Batch นั้นไม่ถูกนำเข้า Data Warehouse เกิดปัญหาข้อมูลไม่อัปเดต (Data Downtime) ซึ่งสร้างความเสียหายทางธุรกิจมากกว่า

การเลือกใช้แนวทาง **Fault-Tolerant & Quarantine Pattern** จึงตอบโจทย์การทำงานจริงได้ดีกว่า โดยแยกข้อมูลที่ไม่สมบูรณ์ออกไปยัง Quarantine Zone เพื่อรอการตรวจสอบ/แก้ไข ในขณะเดียวกันก็ปล่อยให้ข้อมูลส่วนใหญ่ที่ถูกต้องสามารถไหลเข้าสู่ Data Warehouse ได้ตามปกติ ทำให้ระบบธุรกิจยังคงทำงานได้อย่างต่อเนื่องและมีความน่าเชื่อถือ
