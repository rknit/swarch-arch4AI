# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** Rachata Kanitpunyacharoen

**Student ID / รหัสนิสิต:** 6632184921

This is **individual work**: each student completes and submits their own copy, in their own fork.

_งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง_

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run      | p50 ms | p95 ms | accuracy | large calls |
| -------- | ------ | ------ | -------- | ----------- |
| t0-large | 494    | 595    | 93.5%    | 100%        |
| t0-small | 11     | 25     | 72.5%    | 0%          |

**Q0.** What does the large model buy, and what does it cost?

_large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร_

> large model ให้ accurracy สูงกว่ามาก แต่ก็แลกมาด้วย response time ที่เพิ่มขี้น

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run       | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
| --------- | ------------ | ------ | ------ | ------ | ------- | -------- |
| t1-before | 28.6         | 228    | 346    | 20     | 0       | 92.8%    |
| t1-after  | 9.6          | 408    | 1013   | 0      | 26      | 92.5%    |

**Q1.1** In `t1-before`, p50 can look _better_ than the baseline. Why is p50 misleading here? Which column tells the real story?

_ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง_

> เพราะว่า p50 ของ t1-before คือ calls ที่ error จะ exit แทนที่จะ fallback เหมือน t1-after ทำให้ request ที่ error มีเวลาทำงานที่ดู "เร็ว"
> เราต้องดู column `errors` ไปด้วย เพื่อประกอบความเข้าใจ column `p50` และ `p95`

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

_การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway_

> รีวิวช้าลงยังเก็บข้อมูลของรีวิวไว้อยู่ แต่ถ้าค้างหรือ error จะทำให้เสียข้อมูลไป

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

_รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น_

> New reviews go through a cascade: the small model's answer is used when its confidence meets the threshold needed for QR3, otherwise the large model is called. Calls that fail are saved as pending and put on a queue to be labelled later (FR4), which is the asynchronous/batch topology.

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

_เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที_

> Basically all large model calls fail.

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run    | threshold | p50 ms | p95 ms | accuracy | large calls |
| ------ | --------- | ------ | ------ | -------- | ----------- |
| t2-0.6 | 0.6       | 8      | 544    | 91.5%    | 38%         |
| t2-0.7 | 0.7       | 383    | 586    | 93.0%    | 53%         |
| t2-0.8 | 0.8       | 437    | 587    | 93.5%    | 64%         |
| t2-0.9 | 0.9       | 468    | 587    | 93.5%    | 77%         |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

_จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป_

> เลือก `threshold=0.6` เพราะ latency ต่ำ call large model น้อย ถึง accuracy จะไม่ดีที่สุด แต่ QR ต้องการแค่อย่างน้อย 90%

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

_ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก_

> เพราะ p50 คือ request ส่วนใหญ่ที่ใช้ small model ที่เร็วมากๆ ส่วน p95 เป็นกลุ่มที่หลุด confidence ไปทำให้มี large model call เพิ่มขึ้น (latency ก็สูงขี้นตาม)

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

_cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้_

> ภาษาประชดประชัน เช่น "เยี่ยม! รอแค่ 5 ชั่วโมงเอง" จะทำให้ small model พลาดบ่อย
> complaint review พวกนี้จะทำให้ restaurant owner เห็นสถิติที่ผิดไปจากความจริง เช่น เห็นร้านตัวเองรีวิว "ดี" แต่ยอดขายแย่

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run      | p50 ms | p95 ms | accuracy | large calls | cache hits |
| -------- | ------ | ------ | -------- | ----------- | ---------- |
| t3-cache | 7      | 536    | 93.0%    | 38%         | 35%        |

**Q3.1** Which reviews benefit from the cache, and which never will?

_รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์_

> รีวิวพวกวลีทั่วไปจะได้ประโยชน์จาก cache
> รีวิวที่มีคำพิมพ์ผิดจะไม่ได้ประโยชน์จาก cache เลย เพราะตัวอักษรไม่เหมือนกัน

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

_สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร_

> อาจต้องล้าง cache ถ้าไม่ได้เก็บ metadata ว่า review ไหน model ไหน label
> ถ้ามีเก็บ เราก็ run batch large model เอาไว้ตามที่มีใน snapshot ของ cache label หลังจากนั้นก็ replace entry ที่ตรงกัน

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

_ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture_

> Fallback ทําให้ error หาย แต่ tail ช้าลง 3 เท่า
> t1-after errors เหลือ 0 แต่ p95 พุ่งจาก 346 → 1013 ms เพราะ call ที่ fail ไม่ได้ fail เร็ว แต่กลายเป็น PENDING ค้างรอจนครบ timeout 1 วิ ข้อมูลนี้แสดงว่า "error เป็นศูนย์" กับ "latency ดี" อาจขัดแย้งกัน ซึ่งเป็น trade-off ที่ lecture ไม่ได้ให้ตัวเลข
