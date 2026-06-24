# Yeka Camp NFC Points System — Design Spec

## Context

A summer camp with 118 confirmed campers needs a personalized NFC card system. Each camper gets a physical NFC card (NTAG213) that opens a webpage showing their name, group, and points when scanned. The camp director manages points via Google Sheets — no app, no database, no login required. Campers can only view their own data.

---

## Architecture

**Two static HTML files** hosted on GitHub Pages at `yeka.dynoduck.com`:

- `index.html` — landing page (static, no data fetching)
- `card.html` — camper card view (existing client-designed HTML + JS data layer)

**Data source:** A single public Google Sheet (user's personal Google account), accessed via the gviz/tq JSON endpoint — no API key, no Google Cloud setup required. Sheet shared as "Anyone with the link can view."

**NFC cards:** NTAG213 tags, each written once with the camper's unique URL using the NFC Tools app. Cards never need to be reprogrammed.

---

## Pages

### Landing Page (`/`)

Static page matching the dark/gold visual style of the card view. Content:
- Camp name / "I Love Yeka" branding
- Instruction in Russian: "Поднеси свою карточку к телефону, чтобы увидеть свои баллы"
- Small NFC tap icon or animation

No data fetching. Purely informational.

### Card View (`/card?id=001`)

The existing client-designed HTML file (`yeka-card-visa.html`) with a JavaScript block added. On page load:

1. Extract `id` from the URL query string
2. Show a loading spinner immediately
3. Fetch the Google Sheet via the gviz/tq endpoint
4. Find the row where `id` matches
5. Inject `name`, `group`, `points` into existing DOM elements (`#childName`, `#childGroup`, `#childPoints`)
6. Hide the spinner

The Google Sheet ID is hardcoded as a constant at the top of the JS block.

---

## Data Flow

```
Camper scans NFC card
  → phone opens https://yeka.dynoduck.com/card?id=042
  → spinner shown
  → fetch https://docs.google.com/spreadsheets/d/{SHEET_ID}/gviz/tq?tqx=out:json&sheet=Campers
  → parse response, find row where id === "042"
  → populate #childName, #childGroup, #childPoints
  → spinner hidden, card data visible
```

Updates to the Google Sheet appear on the **next scan** — no client-side caching.

---

## Google Sheet Structure

**Tab 1 — "Campers"** (director edits this tab only):

| id  | name                 | group | points |
|-----|----------------------|-------|--------|
| 001 | Олександр Соловйов   | 3     | 0      |
| ... | ...                  | ...   | ...    |

- `id`: zero-padded string (001–118), 118 total campers
- `name`: full name
- `group`: integer (~8 groups total)
- `points`: integer (camp dollars / Доллары)

**Tab 2 — "Links"** (auto-generated via formula, read-only):

| id  | name                 | url                                          |
|-----|----------------------|----------------------------------------------|
| 001 | Олександр Соловйов   | https://yeka.dynoduck.com/card?id=001         |

URL formula: `="https://yeka.dynoduck.com/card?id="&A2`

Director copies URLs from this tab into NFC Tools to program cards.

---

## Error Handling

Card image always remains visible. Russian-language messages:

| Scenario | Message |
|---|---|
| No `?id=` in URL | Карточка не найдена. Попробуй снова. |
| ID not found in Sheet | Карточка не найдена. Попробуй снова. |
| Network failure | Не удалось загрузить данные. Проверь подключение к интернету. |

---

## Hosting & Deployment

- **Platform:** GitHub Pages (existing repo)
- **Custom domain:** `yeka.dynoduck.com` via CNAME DNS record
- **HTTPS:** Automatic via GitHub Pages / Let's Encrypt
- **Deployment:** Push to `main` → live in ~1 minute

---

## Campers (118 total)

| ID  | Name                              |
|-----|-----------------------------------|
| 001 | Олександр Соловйов                |
| 002 | Артем Соловйов                    |
| 003 | Мазур Гена                        |
| 004 | Ткачук-Сорока Давид               |
| 005 | Нікіта Макаров                    |
| 006 | Филипп Варвянский                 |
| 007 | Васьковский Шнеур Залман          |
| 008 | Шаулов [FIRST NAME NEEDED]        |
| 009 | Шаулов [FIRST NAME NEEDED]        |
| 010 | Наливайко Иван                    |
| 011 | Сидоренко Натан                   |
| 012 | Смилянский Биньямин               |
| 013 | Осадчий Беньямин                  |
| 014 | Морозов Натан                     |
| 015 | Оскар Духовний                    |
| 016 | Леон Духовний                     |
| 017 | Эфраим Терешкевич                 |
| 018 | Саланжий Матвей                   |
| 019 | Драгомарецкий Артем               |
| 020 | Марк Кропивский                   |
| 021 | Забродов Давид                    |
| 022 | Забродов Михаэль                  |
| 023 | Васьковский Леви Исроель          |
| 024 | Опалихин Вова                     |
| 025 | Куликов Саша                      |
| 026 | Шолом Петрушанський               |
| 027 | Марк Гусинский                    |
| 028 | Лев Зіслін                        |
| 029 | Лазарєв Давид                     |
| 030 | Давид Сусоров                     |
| 031 | Біньямин Сусоров                  |
| 032 | Нікольський Давід                 |
| 033 | Михайло Бяльський                 |
| 034 | Горидько Марк                     |
| 035 | Олейник Андрій                    |
| 036 | Полісмак Артем                    |
| 037 | Момот Роберт                      |
| 038 | Котляр Ян                         |
| 039 | Йосеф Якубов                      |
| 040 | Кропивський [FIRST NAME NEEDED]   |
| 041 | Медяний Даниэль                   |
| 042 | Медяный Давид                     |
| 043 | Гусинський Арон                   |
| 044 | Марцінішен Артем                  |
| 045 | Роговий Матвій                    |
| 046 | Серебряний Микита                 |
| 047 | Юркевич Ілля                      |
| 048 | Дмитро Місюра                     |
| 049 | Артур Нікітін                     |
| 050 | Ушаков Данило                     |
| 051 | Казарцев Ярослав                  |
| 052 | Данільчук Данило Юрійович         |
| 053 | Гуржиєнко Іван Іванович           |
| 054 | Гуржиєнко Руслан Іванович         |
| 055 | Марк Шкиндер                      |
| 056 | Артур Щербіна                     |
| 057 | Давид Терлецький                  |
| 058 | Лазоренко Ярослав                 |
| 059 | Грановський Давид Олексійович     |
| 060 | Грановський Герман Олексійович    |
| 061 | Борис Борошенко                   |
| 062 | Сєва Солодкін                     |
| 063 | Демид Соловей                     |
| 064 | Ничепорук Тимофій                 |
| 065 | Дабіжа Сєргій                     |
| 066 | Кавун Володимир Олексійович       |
| 067 | Верівський Богдан                 |
| 068 | Піковський Ілан                   |
| 069 | Лео Миримский                     |
| 070 | Максимчук Кирило                  |
| 071 | Серветер Ростислав                |
| 072 | Цопа Давид                        |
| 073 | Равкіс Дмитро                     |
| 074 | Владислав Рудь                    |
| 075 | Музалевський Євгеній Федорович    |
| 076 | Буджурак Михайло                  |
| 077 | Буджурак Олексій                  |
| 078 | Джуган Максим Олександрович       |
| 079 | Іван Полковниченко                |
| 080 | Давид Рудик                       |
| 081 | Єлсуков Дмитро                    |
| 082 | Єрьоменко Максим                  |
| 083 | Глотов Адріан                     |
| 084 | Руденко Валерій                   |
| 085 | Руденко Володимир                 |
| 086 | Рзаєв Антон                       |
| 087 | Лозюк Данило                      |
| 088 | Юрий [LAST NAME NEEDED]           |
| 089 | Беккер Эльханан                   |
| 090 | Ермолаев Вячеслав                 |
| 091 | Шавелашвілі Давид                 |
| 092 | Михайло Вакштейн                  |
| 093 | Махарінський Олександр            |
| 094 | Савченко Олег                     |
| 095 | Артем Дацина                      |
| 096 | Єгор Кунфаншун                    |
| 097 | Тимур Удовіков                    |
| 098 | Тарас Азаров                      |
| 099 | Адам Лялько                       |
| 100 | Мікаель Лялько                    |
| 101 | Арон Якименко                     |
| 102 | Йосі Якименко                     |
| 103 | Ильин Максим                      |
| 104 | Шевченко Святослав                |
| 105 | Шевченко Герман                   |
| 106 | Жишко Валерій                     |
| 107 | Дмитро Гордієнко                  |
| 108 | Ілля Швидкий                      |
| 109 | Михайло Арсеній                   |
| 110 | Філенко Марк                      |
| 111 | Сухин Виталий                     |
| 112 | Oskaras Navickas                  |
| 113 | Orestas Navickas                  |
| 114 | Калашніков Максим                 |
| 115 | Матвей Аршинный                   |
| 116 | Менахем Мендл Файнгольд           |
| 117 | Руденко Максим                    |
| 118 | Даніл Скаковський                 |

---

## Pending Before NFC Card Programming

- First names for Шаулов #008 and #009
- First name for Кропивський #040
- Last name for Юрий #088
- Group assignments for all 118 campers
