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

## 4. A2 — Хязгаарлалтууд

### 4.1. Алдааны давуу эрэмбэ

| Эрэмбэ | Нөхцөл | Oracle |
|---|---|---|
| 1 | From=Missing | ERROR_NO_FROM |
| 2 | From=Frozen | ERROR_FROZEN |
| 3 | To=Missing | ERROR_NO_TO |
| 4 | To=SameAsFrom | ERROR_SAME_ACCOUNT |
| 5 | Amount=Negative эсвэл Zero | ERROR_BAD_AMOUNT |
| 6 | Amount=OverBalance | ERROR_INSUFFICIENT |
| 7 | DailyLimit=Exceeded | ERROR_LIMIT |

Дансны хүчинтэй байдал, дүн, үлдэгдэл, лимит
гэсэн дарааллаар шалгах давуу эрэмбийг сонгосон.
Учир нь данс хүчинтэй болсны дараа санхүүгийн
нөхцөлийг шалгах нь логик дараалалтай.

### 4.2. Хязгаарлалтын тэмдэглэгээ

| Category | Choice | Тэмдэглэгээ | Үндэслэл |
|---|---|---|---|
| From | Valid | [property Active, Exists] | Хэвийн данс |
| From | Frozen | [ERROR] [property Exists] | Царцаагдсан данс |
| From | Missing | [ERROR] | Данс байхгүй |
| To | Valid | [property Distinct] | Хүчинтэй өөр данс |
| To | Missing | [ERROR] | Хүлээн авагч байхгүй |
| To | SameAsFrom | [ERROR] [IF Exists] | Ижил данс бодитоор байна |
| Amount | Negative | [ERROR] | Сөрөг дүн зөвшөөрөхгүй |
| Amount | Zero | [ERROR] | Тэг дүн зөвшөөрөхгүй |
| Amount | Normal | [property Positive] | Хэвийн эерэг дүн |
| Amount | ExactBalance | [SINGLE] [IF Active] | Үлдэгдлийн хилийн утга |
| Amount | OverBalance | [ERROR] [IF Active] | Үлдэгдэл хүрэлцэхгүй |
| Currency | Same | [property NoConversion] | Хөрвүүлэлтгүй |
| Currency | Different | [property Conversion] | Валют хөрвүүлнэ |
| DailyLimit | Available | [property LimitOK] | Лимит хүрэлцээтэй |
| DailyLimit | Exact | [SINGLE] [IF Positive] | Лимитийн хилийн утга |
| DailyLimit | Exceeded | [ERROR] [IF Positive] | Лимит хэтэрсэн |

### 4.3. Тест сонгох дүрэм

1. Хэвийн нөхцөлд From=Valid, To=Valid,
   Amount=Normal, DailyLimit=Available байна.
   Currency нь Same эсвэл Different байна.
2. ERROR сонголт бүрийг бусад хэвийн утгатай
   тусгаарлан нэг удаа шалгана.
3. SINGLE сонголт бүрийг нэг тусгай тестээр шалгана.
4. SameAsFrom нь бодитоор байгаа данстай байна.
5. Negative болон Zero үед DailyLimit=Available байна.
6. From=Missing үед үлдэгдэл, лимитийг шалгахгүй
   тул хэвийн placeholder ашиглана.
7. Валют хөрвүүлсний дараах үлдэгдлийн алдааг
   нэг нэмэлт тестээр шалгана.
8. Алдааны давуу эрэмбийг шалгах зорилгоор
   Frozen болон Missing нөхцөлийг нэг тестэд
   зориуд хослуулна.

### 4.4. N → M тооцоо

Хязгаарлалтгүй бүрэн олонлог:

N = 3 × 3 × 5 × 2 × 3 = 270

| Тестийн бүлэг | Тооцоо | Тоо |
|---|---|---:|
| Хэвийн нөхцөл | 1 × 1 × 1 × 2 × 1 | 2 |
| Тусгаарласан ERROR | 2 + 2 + 3 + 1 | 8 |
| SINGLE нөхцөл | 1 + 1 | 2 |
| Хөрвүүлэлттэй нэмэлт алдаа | 1 | 1 |
| Зориудын давхар алдаа | 1 | 1 |
| Нийт | 2 + 8 + 2 + 1 + 1 | 14 |

M = 14

270 → 14

Бууралт = (270 - 14) / 270 × 100 = 94.81%

M нь ERROR, SINGLE, IF болон нэмэлт сонголтын
дүрмээр боловсруулсан гар тестийн хүрээ юм.
PICT-ийн гаралтын мөрийн тооноос ялгаатай.


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
| TC13 | Valid | Valid | OverBalance | Different | Available | 200 ERROR_INSUFFICIENT |
| TC14 | Frozen | Missing | Normal | Same | Available | 200 ERROR_FROZEN |

### Тусгай нөхцөлийн тайлбар

- TC01, TC02: Happy path.
- TC03–TC10: Бүх ERROR сонголтыг тусгаарлан шалгана.
- TC11: SINGLE — үлдэгдэлтэй яг тэнцүү дүн.
- TC12: SINGLE — үлдсэн лимиттэй яг тэнцүү дүн.
- TC13: 0.286 USD = 1001 MNT.
  Хөрвүүлсний дараах дүн үлдэгдлээс хэтэрнэ.
- TC14: Зориудын давхар алдаа.
  From=Frozen нь To=Missing-ээс өндөр эрэмбэтэй
  тул ERROR_FROZEN буцна.

Эдгээр нь тодорхойлолтод үндэслэсэн хүлээгдэх
үр дүн бөгөөд ажиллаж буй API дээр гүйцэтгэсэн
тестийн үр дүн биш.

## 6. B1 — PICT Browser model

### 6.1. Model

Model файл: browser.model

| Parameter | Values |
|---|---|
| Content | Allow, Restrict, Block |
| Cookies | Allow, Restrict, Block |
| Popups | Yes, No |
| Addons | Yes, No |
| Attacks | Yes, No |
| Forgeries | Yes, No |

### 6.2. Ажиллуулсан команд

- pict browser.model
- pict browser.model /o:3

Санамсаргүй seed буюу /r ашиглаагүй.

### 6.3. Бодит үр дүн

Бүрэн олонлог = 3 × 3 × 2 × 2 × 2 × 2 = 144

| Арга | Тестийн тоо | Бууралт |
|---|---:|---:|
| Бүрэн олонлог | 144 | 0% |
| Лекцийн гар 2-way | 9 | 93.75% |
| PICT 2-way | 10 | 93.06% |
| PICT 3-way | 20 | 86.11% |

### 6.4. Greedy алгоритмын тайлбар

PICT-ийн 2-way гаралт 10 мөр байгаа нь лекцийн
гараар боловсруулсан 9 мөрөөс нэгээр олон байна.
Greedy алгоритм нь хамрагдаагүй хослолуудыг
хамруулах тестүүдийг сонгодог боловч хамгийн
цөөн мөртэй шийдлийг заавал олдоггүй.

### 6.5. Хамралтын харьцуулалт

2-way нь аль ч хоёр параметрийн утгын хослолыг,
3-way нь аль ч гурван параметрийн утгын хослолыг
хамруулна.

3-way нь 2-way-аас хоёр дахин олон тесттэй боловч
бүрэн олонлогоос 86.11 хувиар цөөн байна.

### 6.6. Нотолгооны файлууд

- results/browser-2way.txt — 10 тест
- results/browser-3way.txt — 20 тест

Толгой мөрийг тестийн тоонд оруулаагүй.
