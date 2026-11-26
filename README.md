# Lab06 — Category-Partition ба PICT

**Оюутны нэр:** П.Пүрэвдаш
**Оюутны код:** B232270144
**Хичээл:** Программ хангамжийн чанарын баталгаа ба тест
**Хичээлийн код:** F.CSA313
**Лаборатори:** 06

## 1. Зорилго

Category-Partition аргаар банкны шилжүүлгийн функцийн
тест тохиолдлуудыг боловсруулах, Microsoft PICT ашиглан
pairwise covering array үүсгэх, гараар боловсруулсан
тест дизайнтай харьцуулах.

## 2. Ашигласан хэрэгслүүд

- Ubuntu / WSL2
- Git, CMake, g++, make
- Microsoft PICT (MIT лиценз)
- PICT source commit: ab76c25
- Build evidence: results/pict-build.txt

## 3. A1 — Category-Partition сонголт ба төлөөлөх утгууд

Тестлэх функц: POST /transfer

Үндсэн параметрүүд: from, to, amount, currency.
Далд орчны параметр: DailyLimit.

### 3.1. Тестийн суурь нөхцөл

- Үндсэн дансны валют: MNT
- Дансны үлдэгдэл: 1000 MNT
- Өдрийн нийт лимит: 2000 MNT
- Тестийн тогтмол ханш: 1 USD = 3500 MNT
- Amount-ийн үлдэгдэл болон лимитийн шалгалтыг
  валют хөрвүүлсний дараах дүнгээр хийнэ.

### 3.2. Сонголт ба төлөөлөх утгууд

| Category | Choice | Төлөөлөх утга, нөхцөл |
|---|---|---|
| From | Valid | A001, идэвхтэй данс |
| From | Frozen | A002, царцаагдсан данс |
| From | Missing | A999, байхгүй данс |
| To | Valid | B001, хүчинтэй данс |
| To | Missing | B999, байхгүй данс |
| To | SameAsFrom | Илгээгчтэй ижил данс |
| Amount | Negative | -100 MNT |
| Amount | Zero | 0 |
| Amount | Normal | 100 MNT |
| Amount | ExactBalance | 1000 MNT |
| Amount | OverBalance | 1001 MNT |
| Currency | Same | MNT |
| Currency | Different | USD |
| DailyLimit | Available | Үлдсэн лимит > хөрвүүлсэн дүн |
| DailyLimit | Exact | Үлдсэн лимит = хөрвүүлсэн дүн |
| DailyLimit | Exceeded | Үлдсэн лимит < хөрвүүлсэн дүн |

Amount-ийн төлөөлөх утгууд нь дансны валютад
хөрвүүлсний дараах дүнг илэрхийлнэ.
Currency=Different үед хүсэлтийн amount-ийг
тестийн тогтмол ханшаар тохируулна.

DailyLimit нь хүсэлтийн body-д байхгүй боловч
шилжүүлгийн үр дүнд нөлөөлдөг далд орчны нөхцөл юм.

## 5. A3 — Тест спецификацийн хүснэгт

| ID | From | To | Amount | Currency | DailyLimit | Expected result |
|---|---|---|---|---|---|---|
| TC01 | Valid | Valid | Normal | Same | Available | 201 OK |
| TC02 | Valid | Valid | Normal | Different | Available | 201 OK |
| TC03 | Missing | Valid | Normal | Same | Available | 200 ERROR_NO_FROM |
| TC04 | Frozen | Valid | Normal | Same | Available | 200 ERROR_FROZEN |
| TC05 | Valid | Missing | Normal | Same | Available | 200 ERROR_NO_TO |
| TC06 | Valid | SameAsFrom | Normal | Same | Available | 200 ERROR_SAME_ACCOUNT |
| TC07 | Valid | Valid | Negative | Same | Available | 200 ERROR_BAD_AMOUNT |
| TC08 | Valid | Valid | Zero | Same | Available | 200 ERROR_BAD_AMOUNT |
| TC09 | Valid | Valid | OverBalance | Same | Available | 200 ERROR_INSUFFICIENT |
| TC10 | Valid | Valid | Normal | Same | Exceeded | 200 ERROR_LIMIT |
| TC11 | Valid | Valid | ExactBalance | Same | Available | 201 OK |
| TC12 | Valid | Valid | Normal | Same | Exact | 201 OK |
| TC13 | Valid | Valid | Normal | Different | Available | 201 OK |
| TC14 | Frozen | Missing | Normal | Same | Available | 200 ERROR_FROZEN |

### Тусгай нөхцөлийн тайлбар

- TC01, TC02, TC13: Happy path.
- TC03–TC10: Бүх ERROR сонголтыг тусгаарлан шалгана.
- TC11: SINGLE — үлдэгдэлтэй яг тэнцүү дүн.
- TC12: SINGLE — үлдсэн лимиттэй яг тэнцүү дүн.
- TC13: TC02-оос өөр хөрвүүлэлтийн төлөөлөх дүн,
  жишээ нь 0.2 USD = 700 MNT.
- TC14: Зориудын давхар алдаа.
  From=Frozen нь To=Missing-ээс өндөр эрэмбэтэй
  тул ERROR_FROZEN буцна.

Эдгээр нь тодорхойлолтод үндэслэсэн хүлээгдэх
үр дүн бөгөөд ажиллаж буй API дээр гүйцэтгэсэн
тестийн үр дүн биш.
