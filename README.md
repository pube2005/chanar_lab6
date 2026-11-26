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
- Тестийн тогтмол ханш: 1 USD = 1000 MNT (зөвхөн тестийн хийсвэр ханш)
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
- TC13: 1.001 USD = 1001 MNT.
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

## 7. B2 — PICT Transfer model

### 7.1. Model файлууд

- transfer-nc.model — хязгаарлалтгүй model.
- transfer.model — IF хязгаарлалттай model.
- transfer-neg.model — ERROR утгуудыг ~ тэмдэглэгээгээр
  тодорхойлсон negative model.

A1 хэсгийн таван category, нийт 16 choice-ийг ашигласан.

### 7.2. Ажиллуулсан командууд

- pict transfer-nc.model > results/transfer-nc.txt
- pict transfer.model > results/transfer.txt
- pict transfer-neg.model > results/transfer-neg.txt

### 7.3. Хязгаарлалтын үндэслэл

- From=Missing үед To=SameAsFrom байхгүй.
- From=Missing үед Amount=Normal байна.
- From=Missing үед DailyLimit=Available байна.
- Amount=Negative эсвэл Zero үед DailyLimit=Available байна.
- Amount=ExactBalance эсвэл OverBalance үед From=Valid байна.

Эдгээр дүрэм нь A2 хэсгийн IF нөхцөл болон
төлөөлөх утга сонгох дүрэмд үндэслэсэн.

### 7.4. Бодит үр дүнгийн харьцуулалт

| Үзүүлэлт | No constraints | Constraints | Negative |
|---|---:|---:|---:|
| Нийт тест | 17 | 20 | 21 |
| Happy path | 1 | 0 | 5 |
| Нэг алдаатай | 1 | 7 | 16 |
| Хоёр алдаатай | 8 | 9 | 0 |
| Гурван алдаатай | 6 | 4 | 0 |
| Дөрвөн алдаатай | 1 | 0 | 0 |
| Олон алдаатай нийт | 15 | 13 | 0 |
| Error masking эрсдэлтэй мөр | 88.24% | 65.00% | 0% |
| Oracle алдааны төрөл | 5/7 | 7/7 | 7/7 |
| 270-аас буурсан хувь | 93.70% | 92.59% | 92.22% |

Тооцооллыг PICT-ийн бодит гаралтыг Python ашиглан
уншиж, ERROR нөхцөлийн тоо болон давуу эрэмбээр
хүлээгдэх oracle-ийг тодорхойлох замаар хийсэн.

Алдааны тоо нь model-д тэмдэглэгдсэн алдааны
нөхцөлийн тоо бөгөөд серверээс бодитоор буцсан
алдааны тоо биш.

### 7.5. Oracle тархалт

| Oracle | No constraints | Constraints | Negative |
|---|---:|---:|---:|
| OK | 1 | 0 | 5 |
| ERROR_NO_FROM | 6 | 2 | 2 |
| ERROR_FROZEN | 6 | 7 | 2 |
| ERROR_NO_TO | 1 | 6 | 2 |
| ERROR_SAME_ACCOUNT | 2 | 2 | 2 |
| ERROR_BAD_AMOUNT | 0 | 1 | 4 |
| ERROR_INSUFFICIENT | 0 | 1 | 2 |
| ERROR_LIMIT | 1 | 1 | 2 |
| Нийт | 17 | 20 | 21 |

### 7.6. Error masking шинжилгээ

Хязгаарлалтгүй model-ийн 17 тестээс 15 нь олон
алдаатай байна. Өндөр эрэмбэтэй алдаа бусад
алдааг далдлах эрсдэлтэй.

Хязгаарлалттай model-д олон алдаатай мөрийн
тоо 13 болж буурсан. Гэвч happy path гараагүй.
Pairwise хамралт нь happy path заавал үүсгэнэ
гэсэн баталгаа биш юм.

Negative model-д 5 happy path, 16 нэг алдаатай
тест үүссэн бөгөөд олон алдаатай мөр гараагүй.
Бүх долоон төрлийн ERROR oracle хамрагдсан.

### 7.7. Negative model-д IF хэрэгтэй эсэх

Тийм. Negative тэмдэглэгээ нь олон ERROR утгыг
нэг тестэд зэрэг оруулахаас сэргийлдэг боловч
бизнесийн логикийн бүх хязгаарлалтыг орлохгүй.

Жишээлбэл, negative гаралтын 11-р мөрд
Amount=~Negative, DailyLimit=Exact байна.
Сөрөг дүнтэй үед үлдсэн лимит сөрөг байх
боломжгүй тул энэ нь бодит setup-д нийцэхгүй.

Мөн 19-р мөрд From=~Missing, Amount=ExactBalance,
DailyLimit=Exact байна. Байхгүй дансанд бодит
үлдэгдэл болон лимит тодорхойлох боломжгүй.

Иймээс A2-ын IF хязгаарлалтыг negative
тэмдэглэгээтэй хамтатган хэрэглэх шаардлагатай.

Энэхүү туршилтад negative model-ийг IF-гүй
ажиллуулж, ~ тэмдэглэгээний дан нөлөөг
харьцуулан судалсан.

### 7.8. Нотолгооны файлууд

- results/transfer-nc.txt — 17 тест.
- results/transfer.txt — 20 тест.
- results/transfer-neg.txt — 21 тест.


## 8. PICT гаралтаас боловсруулсан тест кейсүүд

Мөрийн дугаар нь файлын толгой мөрийг оруулаагүй
тестийн дарааллын дугаар юм.

### TC-PICT-01 — Happy path

Эх сурвалж: results/transfer-neg.txt, тестийн 2-р мөр.

| Parameter | Choice |
|---|---|
| From | Valid |
| To | Valid |
| Amount | ExactBalance |
| Currency | Same |
| DailyLimit | Exact |

Setup:

- A001 идэвхтэй данс.
- B001 хүчинтэй данс.
- Үлдэгдэл: 1000 MNT.
- Өдрийн нийт лимит: 2000 MNT.
- Өмнө ашигласан лимит: 1000 MNT.
- Үлдсэн лимит: 1000 MNT.

Request: POST /transfer

Body:

    {"from":"A001","to":"B001","amount":1000,"currency":"MNT"}

Expected HTTP status: 201

Expected response:

    {"result":"OK"}

Үндэслэл: Дүн нь үлдэгдэл болон үлдсэн
лимиттэй яг тэнцүү тул зөвшөөрөгдөнө.

### TC-PICT-02 — Нэг алдаатай negative test

Эх сурвалж: results/transfer-neg.txt, тестийн 6-р мөр.

| Parameter | Choice |
|---|---|
| From | ~Frozen |
| To | Valid |
| Amount | Normal |
| Currency | Same |
| DailyLimit | Available |

Setup:

- A002 царцаагдсан данс.
- B001 хүчинтэй данс.
- Үлдэгдэл: 1000 MNT.
- Үлдсэн лимит: 2000 MNT.

Request: POST /transfer

Body:

    {"from":"A002","to":"B001","amount":100,"currency":"MNT"}

Expected HTTP status: 200

Expected response:

    {"result":"ERROR_FROZEN"}

Үндэслэл: Бусад нөхцөл хэвийн боловч
илгээгч данс царцаагдсан байна.

### TC-PICT-03 — Давхар алдаатай test

Эх сурвалж: results/transfer.txt, тестийн 1-р мөр.

| Parameter | Choice |
|---|---|
| From | Frozen |
| To | SameAsFrom |
| Amount | Normal |
| Currency | Same |
| DailyLimit | Available |

Setup:

- A002 царцаагдсан данс.
- Илгээгч, хүлээн авагч нь ижил данс.
- Үлдэгдэл: 1000 MNT.
- Үлдсэн лимит: 2000 MNT.

Request: POST /transfer

Body:

    {"from":"A002","to":"A002","amount":100,"currency":"MNT"}

Expected HTTP status: 200

Expected response:

    {"result":"ERROR_FROZEN"}

Үндэслэл: ERROR_FROZEN болон ERROR_SAME_ACCOUNT
гэсэн хоёр алдааны нөхцөл үүснэ. A2-ын давуу
эрэмбээр ERROR_FROZEN эхэлж буцна.

Эдгээр нь тестийн дизайн бөгөөд бодит API
дээр гүйцэтгэсэн туршилтын үр дүн биш.


## 9. B3 — Гар тест ба PICT харьцуулалт

| Үзүүлэлт | Category-Partition | PICT |
|---|---|---|
| Тест үүсгэх | Гараар | Автоматаар |
| Тестийн тоо | 14 | 17, 20, 21 |
| Бизнесийн логик | Гараар нарийн тодорхойлно | Model, constraint шаардлагатай |
| Хилийн утга | Зориуд сонгоно | Choice болгон тодорхойлно |
| Хослолын хамралт | Сонгосон тестээр хязгаарлагдана | Pairwise хамралт |
| Oracle | Гараар тодорхойлно | Нэмэлт логикоор тодорхойлно |
| Error masking | ERROR дүрмээр тусгаарлана | Negative утгаар тусгаарлана |
| Өргөтгөх боломж | Харьцангуй бага | Өндөр |

Category-Partition аргын давуу тал нь бизнесийн
шаардлага, хилийн утга, алдааны давуу эрэмбийг
нарийн тусгах боломжтой.

Сул тал нь параметр болон сонголтын тоо өсөхөд
гараар тест боловсруулахад хугацаа их зарцуулдаг.

PICT-ийн давуу тал нь олон параметрийн хослолыг
автоматаар үүсгэж, системтэй хамралт бий болгодог.

Сул тал нь буруу model болон дутуу constraint
ашиглавал бодит бус тест үүсэх боломжтой бөгөөд
хүлээгдэх үр дүнг өөрөө тодорхойлдоггүй.

Иймээс Category-Partition аргаар шаардлагыг
загварчилж, PICT ашиглан хослол үүсгэх нь
хоёр аргын давуу талыг нэгтгэнэ.

## 10. Дүгнэлт

1. Энэхүү лабораториор Category-Partition болон
   PICT ашиглан тест дизайн боловсруулах аргыг судалсан.
2. Банкны шилжүүлгийн функцэд таван category,
   нийт 16 choice тодорхойлсон.
3. ERROR, SINGLE, IF дүрмээр 270 боломжит
   хослолоос 14 гар тест сонгосон.
4. Browser model-ийн 2-way туршилтаар 10,
   3-way туршилтаар 20 тест үүсгэсэн.
5. Greedy алгоритм хамгийн цөөн мөртэй
   шийдлийг заавал олдоггүйг ажигласан.
6. Transfer model-ийн гурван хувилбараар
   17, 20, 21 тест тус тус үүсгэсэн.
7. Negative model нь олон алдаа давхцахаас
   сэргийлж, бүх ERROR oracle-ийг хамруулсан.
8. Хамгийн төвөгтэй хэсэг нь бизнесийн логикт
   нийцсэн constraint болон алдааны давуу
   эрэмбийг тодорхойлох байсан.
9. Negative тэмдэглэгээ нь IF хязгаарлалтыг
   бүрэн орлохгүй болохыг тогтоосон.
10. Цаашид Category-Partition болон PICT-ийг
    хослуулан ашиглах нь тестийн хамралт,
    чанар, үр ашгийг сайжруулах боломжтой.

