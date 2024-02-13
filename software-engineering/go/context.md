---
description: >-
  เราจะแน่ใจได้ยังไง? ว่าถ้ามีการยกเลิก process บางอย่าง แล้ว process
  ที่เกี่ยวข้องจะถูกยกเลิกด้วยเช่นกัน
cover: ../../.gitbook/assets/context.svg
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# ⏲ Context

## 🤔 Context คืออะไร ?

Context เป็น package จาก standard library ที่จะช่วยในการจัดการการทำงานและแชร์ข้อมูลระหว่าง process

{% hint style="info" %}
By deriving a context, you create a node in the linked list of contexts.
{% endhint %}

## 🤨 Context มีไว้ทำไม ?

* การ Cancel process (ex: ยกเลิก flow งานทั้งหมดถ้า API ถูกยกเลิก)
* ส่ง/แชร์ข้อมูลระหว่าง process
* กำหนด deadlines และ timeouts.

## 😎 การใช้งาน Context

1.  สร้าง `Parent context` / `Root context`&#x20;

    โดยปกติจะสร้าง empty context ออกมาใช้ใน Main function, initialization, test และ top-level และจะไม่โดน canceled และ ไม่มี values หรือ deadline ซึ่ง สามารถเรียกใช้ได้ 2 แบบ!

    * `context.Background()` ใช้เมื่อมีความชัดเจนว่าไม่ต้องการ `context`เฉพาะใด ๆ หรือเป็น `root context`&#x20;
    * `context.TODO()` ใช้เป็นตัวยึดเพื่อบ่งบอกว่า  `context` นั้นอาจจะต้องถูกกำหนดใหม่หรือปรับปรุงในอนาคต

> สรุปคือทั้ง context.Background() และ context.TODO() คือการสร้าง Empty context ออกมาเหมือนกัน แตกต่างกันแค่ความหมายในการใช้งาน

***

2. `Parent context`  ที่สร้างมาจะเป็น interface  ซึ่งจะมี Methode ต่างๆ ดังนี้

```go
Deadline() (Deadline time.Time, ok book) 
Done() <-chan struct{}
Err() error 
Value(key interface{}) interface{}
```

***

3. สามารถสร้างเป็น Context ใหม่ด้วยวิธีสืบทอดมาจาก `Parent context`ซึ่งการสร้าง Context ใหม่สามารถสร้างได้หลายแบบตามลักษณะของการนําไปใช้งานดังนี้

### WithCancel

```go
ctx, cancel := context.WithCancel(context.Background())
```

จะ Return new context และ Function cancel ออกมาให้ โดยที่ New context ที่ได้จะถูก Implement method Done() ออกมาให้เราด้วย ซึ่งเราจะใช้ Done() ในการควบคุมการทํางานอีกที

### WithTimeout

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
```

&#x20;จะเป็นการกําหนด Deadline หรือ Expire time แต่ WithTimeout จะเป็นการกําหนด เป็น Timeout แทนครับ เพื่อให้เข้าใจมากขึ้น ลองดูตามตัวอย่างนี้ครับ

{% hint style="info" %}
Timeout = a duration
{% endhint %}

***

### WithDeadline

```go
ctx, cancel := context.WithDeadline(context.Background(), deadline)
```

เมื่อถึงกําหนด Deadline แล้ว Done() channel จะถูก Close ทันที

{% hint style="info" %}
Deadline = a precise point in time
{% endhint %}

***

### WithValue

```go
ctx := context.WithValue(context.Background(),"key","value")
```

จะเป็นการกําหนด Key และ Value เข้าไปใน Context ซึ่งส่วนใหญ่จะใช้ในการ Share ข้อมูลกันระหว่าง Function หรือ thread ต่างๆ

***

## 🚦ลำดับขั้นในการ Cancel ของ Context

เมื่อเรายกเลิก process จาก context คำสั่งในการ cancel จะถูกประกาศออกไปจาก  `Parent context`  ไปหา `Children context`  ตามลำดับ ตามรูป

* เมื่อเราเรียก cancel func `c2`มันจะเป้นการ cancel `ctx2` และจะเป็นการ cancel ลูกของมันด้วย (`ctx3`).
* เมื่อ `ctx3` ถูก cancel มันก็ต้องไป cancel ของมันด้วยนั่นก็คือ `ctx4`
* `ctx4` จะถูกยกเลิกเป็นอันสุดท้าย

<figure><img src="https://www.practical-go-lessons.com/img/cancellation_propagation_2.51cb103e.png" alt="" width="375"><figcaption><p><a href="https://www.practical-go-lessons.com/img/cancellation_propagation_2.51cb103e.png">https://www.practical-go-lessons.com/img/cancellation_propagation_2.51cb103e.png</a></p></figcaption></figure>

