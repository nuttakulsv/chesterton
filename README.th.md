# chesterton

[English](README.md) · จากคนทำ [honest-tests](https://github.com/nuttakulsv/honest-tests) และ [phantom-deps](https://github.com/nuttakulsv/phantom-deps)

> อย่ารื้อรั้วทิ้ง ถ้ายังไม่รู้ว่าเขาสร้างไว้ทำไม (G. K. Chesterton)

coding agent ชอบเก็บกวาดโค้ด มันพร้อมจะลบ `sleep(0.5)`, ลูป retry หรือเงื่อนไขพิเศษแปลกๆ ของร้านค้าบางเจ้าทิ้งทันทีเพราะดูไม่มีประโยชน์ แต่บางทีบรรทัดนั้นแหละที่กันไม่ให้เราโดนตามตอนตีสาม และเหตุผลไม่ได้อยู่ในโค้ด แต่อยู่ในประวัติของ git

chesterton คือ skill ที่บังคับให้ agent ไปดูก่อนว่าโค้ดแปลกๆ อยู่ตรงนั้นทำไม ก่อนจะลบทิ้ง พร้อมสคริปต์ `why.py` ที่รวบรวมประวัติของบรรทัดที่สนใจมาไว้ในหน้าเดียว คนใช้เองก็สะดวก

## `why.py` แสดงอะไรบ้าง

```
$ python3 why.py src/cache.py:6

# Why: src/cache.py:6-6 (in `warm_cache`)

    6      time.sleep(0.5)  # do not remove: replica lag, see #42

## Warning signs and notes nearby
- `time.sleep(0.5)  # do not remove: replica lag, see #42`

## Commits that shaped these lines (newest first)
- **81d18ed** 2024-03-02 Dana: Wait for replica before reading back (#57)
  > Reads right after load() hit the replica
  > before it caught up. Fixes #42.

## Pull requests
- #57 Wait for replica before reading back (https://github.com/acme/shop/pull/57)

## Issues
- #42 Customers see old prices right after a price update (...)

## Tests that mention the same names
- tests/test_cache.py:3:def test_warm_cache():  (mentions `warm_cache`)
```

มันใช้ `git log -L` ตามรอยบรรทัดเหล่านั้นข้ามการเปลี่ยนชื่อไฟล์และการแก้ไข ดึง commit message ฉบับเต็ม และดึงรายละเอียด PR กับ issue ที่ลิงก์ไว้ผ่าน GitHub CLI (`gh`) ถ้ามีติดตั้งไว้ โดยไม่แก้อะไรในโปรเจกต์เลย

## skill นี้ทำให้ agent ทำอะไร

ก่อนจะลบหรือเขียนใหม่ในส่วนที่เป็น sleep, retry, เงื่อนไขพิเศษ, ตัวเลขแปลกๆ, การกลืน exception, โค้ดที่เขียนไว้เพื่อรองรับของเก่า หรืออะไรก็ตามที่อยู่ใกล้ comment ว่า "do not remove" agent จะ

1. รัน `why.py` กับบรรทัดเหล่านั้น
2. อ่าน PR หรือ issue ที่ลิงก์ไว้
3. หาเทสต์ที่ครอบคลุมพฤติกรรมนั้น
4. แล้วตัดสินใจว่าจะเก็บไว้ (และเติม comment บรรทัดเดียวบอกเหตุผลถ้ายังไม่มี) จะลบโดยเขียนเหตุผลไว้ใน commit message หรือจะปล่อยไว้แล้วบอกเราว่าหาเหตุผลไม่เจอ

ตอนจบจะสรุปว่าเก็บอะไร ลบอะไร เพราะอะไร

```
Kept    time.sleep(0.5) in warm_cache: replica lag after load() (81d18ed, PR #57, fixes #42)
Removed legacy v1 header check: v1 clients were retired in PR #210
Unsure  special case for store_id 4471: no history found, left in place
```

ถามตรงๆ ก็ได้ เช่น "โค้ดตรงนี้มีไว้ทำไม"

## ติดตั้ง

Claude Code

```
/plugin marketplace add nuttakulsv/chesterton
/plugin install chesterton@chesterton
```

Codex, Cursor หรือ agent อื่นที่อ่าน SKILL.md ได้

```
npx skills add nuttakulsv/chesterton
```

หรือใช้แค่สคริปต์อย่างเดียว ต้องมีแค่ `python3` และ `git`

```
curl -O https://raw.githubusercontent.com/nuttakulsv/chesterton/main/skills/chesterton/why.py
python3 why.py path/to/file.py:120-128
```

## ข้อจำกัด

- ดีได้เท่าที่ประวัติ git ของเราดี ถ้า squash merge แล้วเขียน message แค่ "fix stuff" ก็มีข้อมูลให้หาน้อย
- รายละเอียด PR และ issue ต้องใช้ `gh` ที่ล็อกอินแล้ว และ repo ต้องมี remote บน GitHub ถ้าไม่มีจะได้แค่หมายเลข
- ส่วน "เทสต์ที่พูดถึงชื่อเดียวกัน" ใช้การค้นข้อความ ไม่ได้วัด coverage จริง

## รันเทสต์

```
python3 skills/chesterton/why.py --test
```

## License

MIT
