# Lab06 — Category-Partition ба PICT

**Оюутны нэр:** П.Пүрэвдаш  
**Оюутны код:** B232270144  
**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест  
**Лаборатори:** 06

## 1. Зорилго

Category-Partition аргаар тестийн сонголт, төлөөлөх утга, хязгаарлалт тодорхойлох, Microsoft PICT ашиглан pairwise болон 3-way тест үүсгэж, үр дүнг харьцуулах.

## 2. Ашигласан хэрэгслүүд

- Ubuntu / WSL2
- Microsoft PICT
- CMake, G++
- Python 3
- Git, GitHub

PICT-ийг эх кодоос хөрвүүлж суулгасан. Build-ийн нотолгоог `results/pict-build.txt` файлд хадгалсан.

## 3. A1 — Category-Partition

Шилжүүлгийн функц: `POST /transfer`

Тестийн суурь нөхцөл:

- Дансны үлдэгдэл: 1000 MNT
- Өдрийн нийт лимит: 2000 MNT
- Тестийн тогтмол ханш: 1 USD = 1000 MNT

Ханш нь бодит зах зээлийн ханш биш, тестийн зориулалттай утга юм.

| Category | Choice | Төлөөлөх утга |
|---|---|---|
| From | Valid | A001 — идэвхтэй |
| From | Frozen | A002 — царцаагдсан |
| From | Missing | A999 — байхгүй |
| To | Valid | B001 |
| To | Missing | B999 |
| To | SameAsFrom | Илгээгчийн данс |
| Amount | Negative | -100 MNT |
| Amount | Zero | 0 MNT |
| Amount | Normal | 100 MNT |
| Amount | ExactBalance | 1000 MNT |
| Amount | OverBalance | 1001 MNT |
| Currency | Same | MNT |
| Currency | Different | USD |
| DailyLimit | Available | Үлдсэн лимит > дүн |
| DailyLimit | Exact | Үлдсэн лимит = дүн |
| DailyLimit | Exceeded | Үлдсэн лимит < дүн |

DailyLimit нь далд орчны category юм. Amount-ийн төлөөлөх утгыг хөрвүүлсний дараах MNT дүнгээр тодорхойлсон бөгөөд USD хүсэлтийн дүнг тогтмол ханшаар тооцно.

## 4. A2 — Хязгаарлалтууд

### ERROR, SINGLE, IF

| Choice | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| From=Frozen | ERROR | Царцаагдсан данс |
| From=Missing | ERROR | Илгээгч байхгүй |
| To=Missing | ERROR | Хүлээн авагч байхгүй |
| To=SameAsFrom | ERROR, IF Exists | Ижил данс |
| Amount=Negative/Zero | ERROR | Зөвшөөрөгдөхгүй дүн |
| Amount=OverBalance | ERROR, IF Active | Үлдэгдэл хүрэлцэхгүй |
| Amount=ExactBalance | SINGLE, IF Active | Үлдэгдлийн хилийн утга |
| DailyLimit=Exact | SINGLE, IF Positive | Лимитийн хилийн утга |
| DailyLimit=Exceeded | ERROR, IF Positive | Лимит хэтэрсэн |

Нэмэлт хязгаарлалтууд:

- From=Missing үед Amount=Normal, DailyLimit=Available байна.
- From=Missing үед To=SameAsFrom байхгүй.
- Amount=Negative эсвэл Zero үед DailyLimit=Available байна.
- Amount=ExactBalance эсвэл OverBalance үед From=Valid байна.
- ERROR утгуудыг боломжтой үед тусгаарлан шалгана.
- SINGLE утга бүрийг зориулалтын тестээр шалгана.

### Алдааны давуу эрэмбэ

1. ERROR_NO_FROM
2. ERROR_FROZEN
3. ERROR_NO_TO
4. ERROR_SAME_ACCOUNT
5. ERROR_BAD_AMOUNT
6. ERROR_INSUFFICIENT
7. ERROR_LIMIT

Олон алдаа зэрэг үүсвэл дансны хүчинтэй байдал, хүлээн авагч, дүн, үлдэгдэл, лимит гэсэн дарааллаар шалгана гэж тестийн дизайнд тогтоосон.

### N → M тооцоо

Бүрэн олонлог:

N = 3 × 3 × 5 × 2 × 3 = 270

Гар аргаар сонгосон тест:

M = 2 happy path + 8 ERROR + 2 SINGLE + 1 валютын алдаа + 1 давхар алдаа = 14

Тестийн тоо 270-оос 14 болж, 94.81%-иар буурсан.

## 5. A3 — Тест спецификаци

| ID | From | To | Amount | Currency | Limit | Expected result |
|---|---|---|---|---|---|---|
| TC01 | Valid | Valid | Normal | Same | Available | 201 OK |
| TC02 | Valid | Valid | Normal | Different | Available | 201 OK |
| TC03 | Missing | Valid | Normal | Same | Available | ERROR_NO_FROM |
| TC04 | Frozen | Valid | Normal | Same | Available | ERROR_FROZEN |
| TC05 | Valid | Missing | Normal | Same | Available | ERROR_NO_TO |
| TC06 | Valid | SameAsFrom | Normal | Same | Available | ERROR_SAME_ACCOUNT |
| TC07 | Valid | Valid | Negative | Same | Available | ERROR_BAD_AMOUNT |
| TC08 | Valid | Valid | Zero | Same | Available | ERROR_BAD_AMOUNT |
| TC09 | Valid | Valid | OverBalance | Same | Available | ERROR_INSUFFICIENT |
| TC10 | Valid | Valid | Normal | Same | Exceeded | ERROR_LIMIT |
| TC11 | Valid | Valid | ExactBalance | Same | Available | 201 OK |
| TC12 | Valid | Valid | Normal | Same | Exact | 201 OK |
| TC13 | Valid | Valid | OverBalance | Different | Available | ERROR_INSUFFICIENT |
| TC14 | Frozen | Missing | Normal | Same | Available | ERROR_FROZEN |

Бүх ERROR хариултын HTTP статус 200 байна.

TC11, TC12 нь хилийн утгыг шалгана. TC13-д 1.001 USD нь 1001 MNT болж үлдэгдлээс хэтэрнэ. TC14-д давуу эрэмбийн дагуу ERROR_FROZEN буцна.

## 6. B1 — Browser model

| Parameter | Values |
|---|---|
| Content | Allow, Restrict, Block |
| Cookies | Allow, Restrict, Block |
| Popups | Yes, No |
| Addons | Yes, No |
| Attacks | Yes, No |
| Forgeries | Yes, No |

Ажиллуулсан команд:

    pict browser.model > results/browser-2way.txt
    pict browser.model /o:3 > results/browser-3way.txt

| Арга | Тестийн тоо | Бууралт |
|---|---:|---:|
| Бүрэн олонлог | 144 | 0% |
| Лекцийн гар 2-way | 9 | 93.75% |
| PICT 2-way | 10 | 93.06% |
| PICT 3-way | 20 | 86.11% |

PICT-ийн 2-way гаралт лекцийн гар аргаар боловсруулсан тестээс нэгээр олон гарсан. Greedy алгоритм хамрагдаагүй хослолуудыг хамруулахыг зорьдог боловч хамгийн цөөн тесттэй шийдлийг заавал олдоггүй. 3-way нь гурван параметрийн хослолыг хамруулж, бүрэн олонлогоос 86.11%-иар цөөн тест үүсгэсэн.

Санамсаргүй seed буюу `/r` ашиглаагүй.

## 7. B2 — Transfer model

### Model файлууд

| Файл | Зориулалт |
|---|---|
| transfer-nc.model | Хязгаарлалтгүй |
| transfer.model | IF хязгаарлалттай |
| transfer-neg.model | Negative (~) утгатай |

Ажиллуулсан команд:

    pict transfer-nc.model > results/transfer-nc.txt
    pict transfer.model > results/transfer.txt
    pict transfer-neg.model > results/transfer-neg.txt

### Туршилтын үр дүн

| Үзүүлэлт | NC | Constraints | Negative |
|---|---:|---:|---:|
| Нийт тест | 17 | 20 | 21 |
| Happy path | 1 | 0 | 5 |
| Нэг алдаатай | 1 | 7 | 16 |
| Хоёр алдаатай | 8 | 9 | 0 |
| Гурван алдаатай | 6 | 4 | 0 |
| Дөрвөн алдаатай | 1 | 0 | 0 |
| Олон алдаатай нийт | 15 | 13 | 0 |
| Error masking эрсдэл | 88.24% | 65% | 0% |
| Oracle алдааны төрөл | 5/7 | 7/7 | 7/7 |

### Oracle тархалт

| Expected result | NC | Constraints | Negative |
|---|---:|---:|---:|
| OK | 1 | 0 | 5 |
| ERROR_NO_FROM | 6 | 2 | 2 |
| ERROR_FROZEN | 6 | 7 | 2 |
| ERROR_NO_TO | 1 | 6 | 2 |
| ERROR_SAME_ACCOUNT | 2 | 2 | 2 |
| ERROR_BAD_AMOUNT | 0 | 1 | 4 |
| ERROR_INSUFFICIENT | 0 | 1 | 2 |
| ERROR_LIMIT | 1 | 1 | 2 |

Хязгаарлалтгүй model-д олон алдаа давхцаж, error masking үүсэх эрсдэл өндөр байна. Хязгаарлалттай model-д энэ эрсдэл буурсан боловч happy path гараагүй. Negative model нь 16 нэг алдаатай тест үүсгэж, бүх долоон ERROR oracle-ийг хамруулсан.

Гэхдээ `~` тэмдэглэгээ IF хязгаарлалтыг бүрэн орлохгүй. Жишээлбэл, From=Missing үед ExactBalance сонгогдсон, Negative дүнтэй үед DailyLimit=Exact сонгогдсон мөрүүд бодит setup-д нийцэхгүй. Иймээс бизнесийн логикийн хязгаарлалт шаардлагатай хэвээр байна.

Negative model-ийг IF-гүй ажиллуулсан нь `~` тэмдэглэгээний дан нөлөөг харьцуулах зорилготой.

## 8. PICT гаралтаас боловсруулсан тест кейс

### TC-PICT-01 — Happy path

**Эх сурвалж:** transfer-neg.txt, тестийн 2-р мөр.

**Setup:** A001 идэвхтэй, B001 хүчинтэй, үлдэгдэл 1000 MNT, үлдсэн лимит 1000 MNT.

**Request:** POST /transfer

    {"from":"A001","to":"B001","amount":1000,"currency":"MNT"}

**Expected:** HTTP 201

    {"result":"OK"}

### TC-PICT-02 — Нэг алдаатай тест

**Эх сурвалж:** transfer-neg.txt, тестийн 6-р мөр.

**Setup:** A002 царцаагдсан, B001 хүчинтэй, үлдэгдэл 1000 MNT, үлдсэн лимит 2000 MNT.

**Request:** POST /transfer

    {"from":"A002","to":"B001","amount":100,"currency":"MNT"}

**Expected:** HTTP 200

    {"result":"ERROR_FROZEN"}

### TC-PICT-03 — Давхар алдаатай тест

**Эх сурвалж:** transfer.txt, тестийн 1-р мөр.

**Setup:** A002 царцаагдсан, үлдэгдэл 1000 MNT, үлдсэн лимит 2000 MNT.

**Request:** POST /transfer

    {"from":"A002","to":"A002","amount":100,"currency":"MNT"}

**Expected:** HTTP 200

    {"result":"ERROR_FROZEN"}

ERROR_FROZEN нь ERROR_SAME_ACCOUNT-аас өндөр эрэмбэтэй тул эхэлж буцна.

Эдгээр нь тестийн дизайн бөгөөд бодит API дээр гүйцэтгэсэн туршилт биш.

## 9. B3 — Category-Partition ба PICT харьцуулалт

| Үзүүлэлт | Category-Partition | PICT |
|---|---|---|
| Тест үүсгэх | Гараар | Автоматаар |
| Бизнесийн логик | Нарийн тусгана | Constraint шаардлагатай |
| Хослолын хамралт | Гараар сонгоно | Pairwise хамралт |
| Хилийн утга | Зориуд сонгоно | Choice-оор тодорхойлно |
| Oracle | Гараар тодорхойлно | Нэмэлт логик шаардлагатай |
| Өргөтгөх боломж | Хязгаарлагдмал | Өндөр |

Category-Partition нь бизнесийн шаардлага болон алдааны семантикийг илүү сайн тусгадаг. PICT нь олон параметрийн хослолыг автоматаар хамруулдаг боловч зөв model, constraint шаарддаг. Хоёр аргыг хослуулснаар тестийн дизайн илүү үр дүнтэй болно.

## 10. AI хэрэглээ

Лабораторийн ажлын явцад ChatGPT-ийг Category-Partition сонголт, хязгаарлалт боловсруулах, PICT model бичих, үр дүнг шинжлэх болон README-г цэгцлэхэд туслах хэрэгсэл болгон ашигласан. PICT-ийн суулгалт, model ажиллуулах, гаралтын үр дүнг шалгах болон GitHub-д байршуулах үйлдлийг Ubuntu орчинд гүйцэтгэсэн. Тайланд PICT-ийн бодит гаралтын тоон үр дүнг ашигласан.

## 11. Дүгнэлт

Энэхүү лабораториор Category-Partition болон PICT аргыг судалсан. Банкны шилжүүлэгт таван category, 16 choice тодорхойлсон. Бүрэн 270 хослолоос 14 гар тест сонгосон. Browser model-оос 10 pairwise, 20 three-way тест үүсгэсэн. Transfer model-ийн гурван хувилбараас 17, 20, 21 тест гарсан. Negative model нь алдааг тусгаарлахад илүү үр дүнтэй байсан. Гэхдээ IF хязгаарлалт шаардлагатай хэвээр байгааг тогтоосон. Хамгийн их бодол шаардсан хэсэг нь бизнесийн логик болон алдааны давуу эрэмбийг тодорхойлох байв. Хоёр аргыг хослуулан ашиглах нь тестийн хамралт, чанарыг сайжруулах боломжтой.