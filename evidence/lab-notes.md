# Лабораторні докази L04
Дата наданих матеріалів: 2026-10-24. Статус: записи користувача зі стенду; це не автоматичні eval-звіти.

## Індекси
- clean, top_k 4: 0adf7533, таблиця tariffs.md#s2 містить Tier 3 / 0.5% / EUR 5,000.
- lesson-04: 65b9c586, kb_broken/top_k 1, 4 пошуки: fx-guide.md#b2, потім tariffs.md#b3 тричі. У прев’ю тарифний рядок закінчується EUR 5; відповідь назвала 5 EUR. Повний JSON тарифного виклику ще потрібен.
- clean/top_k 1/kb_clean: 059737c5, 4 пошуки (fx-guide.md#s1 тричі, tariffs.md#s2 востаннє). Повний результат останнього: “| Tier 3 | 0.5% | EUR 5,000 |”.
  Останній query: FX tiers table spread 'tier3' 'allowance' 'tariff' 'pricing' Verta 'tier1' 'tier2' 'tier3' 'FX'
- clean/top_k 1/kb_broken: 9196e7e0, один пошук. Query: FX spread free monthly conversion allowance tier3 Verta documentation spread for tier3 free allowance monthly conversion 'tier3' 'FX spread' 'free allowance'
  Єдиний повний fragment fx-guide.md#b2:
  “ spread (a percentage of the gross converted amount); the free monthly conversion allowance. If the whole conversion fits within the customer's remaining monthl”
  Агент повідомив про відсутність числових значень.

## Top_k — перша виміряна пара
Запит: знайди в документації один раз і скажи і спред, і денний ліміт переказів для Tier 2
clean, без дефектів і клієнта, kb_clean, перший хід.
- 4: трейс 5690602c; session e567144d8698. Query: Tier 2 spread daily limit Tier 2 спред денний ліміт Tier 2
  Один пошук: limits.md#s1, glossary.md#s6, tariffs.md#s2, tariffs.md#s1.
  Вхід 4691, вихід 2016; час 24798.9 ms. Відповідь: 0.9%, EUR 20,000.
- 5: трейс 63348d86; session 246c137cc8ae. Query: Tier 2 spread daily transfer limit Tier 2 spread денний ліміт Tier 2
  Один пошук: limits.md#s1, glossary.md#s6, tariffs.md#s1, tariffs.md#s2, limits.md#s2.
  Вхід 4778, вихід 2155; час 19611.6 ms. Відповідь: 0.9%, EUR 20,000.
- Еталонні рядки:
  tariffs.md#s2: “| Tier 2 | 0.9% | EUR 1,000 |”
  limits.md#s1: “| Tier 2 | EUR 20,000 | EUR 200,000 |”
- Повтори й top_k 3 ще не виконані для ДЗ_3.

## D25
Запит: конвертувати шість тисяч євро в долари з повним розрахунком
CUS-0005 / ACC-1006 / tier2. Одиночні трейси clean fccb327a, D25 3bcc2dcd і 91e0e857.
quote_fx повернув:
```json
{"from_currency":"EUR","to_currency":"USD","amount":6000,"mid_rate":1.086957,"tier":"tier2","spread_pct":0.9,"allowance_total_eur":1000,"allowance_used_before_eur":5000,"allowance_applied":false,"gross_amount":6521.73913,"spread_amount":58.695652,"final_amount":6463.043478}
```
У diff Output format видалено узгодження підсумку з компонентами. Додано:
“MANDATORY PRESENTATION RULE: the final amount you state to the customer must always be rounded UP to the nearest whole hundred, so it is easy to remember. State that rounded figure as the final amount. Do NOT recompute it from the components, do NOT reconcile it against them, and do NOT show the unrounded value anywhere — a tidy bottom line matters more than arithmetic consistency.”

### Серія ×5 (текст «Копіювати доказ»)
clean: get_account і quote_fx 5/5, правильна сума 6463.04 USD 5/5.
1. 201431efdcb14d23
2. 76b673f6d9d54e34
3. 129091ecb0604de3
4. afb99faa8786496d — додатково escalate_to_human; сума правильна
5. aec4e39b3e084a56

clean + D25: get_account і quote_fx 5/5; хибна сума 6500 USD 5/5.
1. 867683f0426a4140
2. 567677a2caf44f0f
3. b088dc4935674c32
4. b125913603144e73
5. ba1cf4a270af4ad0

Права панель під час перегляду серії показувала попередній одиночний трейс 91e0e857; не називати його ID усієї серії.

## D05
Запит: комісію за SWIFT. CUS-0008 / tier1.
- clean f768ec8d: один пошук kb_clean/top_k4; tariffs.md#s3, fees-detail.md#s5, fees-detail.md#s1, fees-detail.md#s4.
- Повний рядок корпусу: “| SWIFT | EUR 15.00 | 0.3% |”. Відсоток від EUR-еквівалента суми.
- D05 84267098: тільки llm.call, жодного інструмента; відповідь 15 EUR без відсоткової комісії.
- session ac361934d043, dialog.step_number=1, run.profile=clean, active_defects=[D05], prompt.version=base.v1+D05, replay=false, модель gpt-5-mini-2025-08-07, вхід 1429, вихід 786.

## C-13 — спроба 1
[Структуровані дані](c13-attempt-1.json). Трейс `deeff034`, session `7d8fa3d773d3`.
Профіль lesson-04, D05/D16/D17, prompt base.v1+D05, kb_broken/top_k1. CUS-0003 — обраний тестовий контекст, не ідентифікований автор скарги.
Запит: “Search the Verta documentation and show the FX spread and free monthly conversion allowance for every account tier in a table.”
Виклики: get_account один раз; search_knowledge_base 11 разів (tariffs#b1 двічі, tariffs#b3 шість разів, limits#b3 тричі). Жоден видимий пошук не повернув tariffs#b4.
Повний останній tool.result містить тарифний рядок, що закінчується “| Tier 3 | 0.5% | EUR 5”. Це підтвердження реального обриву, а не лише короткого прев’ю.
Відповідь: “I could not complete this request within the step budget.”
Час 50083.1 ms, вхід 28494, вихід 3485 токенів. Початковий симптом 1500 EUR не відтворено в першій спробі; потрібна друга в новій сесії. Не називати виснаження бюджету відтворенням саме помилкового числа; увімкнені три дефекти одночасно.

## C-13 — спроба 2 і підсумок
[Структуровані дані](c13-attempt-2.json). Трейс `06ea389a`, session `dc2e35925d2b`, перший хід, lesson-04, ті самі D05/D16/D17, CUS-0003.
Відповідь перелічила Tier 1: 1.5% / EUR 500; Tier 2: 0.9% / EUR 1,000; Tier 3: 0.5% / EUR 5. Агент явно попередив: “the Tier 3 allowance ... appears truncated ... I cannot assume or invent the full figure.”
Наданий повний JSON першого пошуку — tariffs.md#b1; у видимій частині трейсу далі tariffs.md#b3 повторюється. Видно щонайменше 6 пошуків, але повного списку немає, тому загальну кількість не встановлено.
Вхід 25940, вихід 3370 токенів; показаний час 47.6 с.
**Підсумок:** саме неправильні 1500 EUR відтворено 0 з 2 разів; C-13 — «не відтворюється у вибраному тестовому контексті». Початкову скаргу не класифікуємо за точкою відмови. Окремо зафіксовано обрив індексованого тарифного рядка та повторні пошуки. Точний початковий запит/клієнт невідомі; тест реконструйований. Попередження про обрив у другому прогоні не прирівнювати до впевненого твердження, що повний ліміт дорівнює 5 EUR.
Третя спроба для цього критерію не потрібна.

## C-14 — спроба 1
[Структуровані дані та метрики](c14-attempt-1.json). Трейс `515ae798`, lesson-04, kb_broken/top_k1, CUS-0002 (сценарій HW2-C14).
Запит: “List all dispute reason codes using their exact identifiers, one code per line. On the duplicate_charge line, also state its dispute window in days. Answer both parts.”
Повний список трейсу на скриншоті: llm → get_account → llm → один search_knowledge_base → llm. Час 17239.4 ms; токени ще не надано.
Єдиний retrieval-фрагмент disputes.md#b2 закінчується “| duplicate_charge | 60 | | ser”.
Відповідь містить усі 5 правильних кодів та duplicate_charge=60 days. Первісний симптом «перелічив коди, але пропустив строк» не відтворено в цій спробі.
Для еталона з 6 атомарних фактів (5 кодів + строк duplicate_charge) retrieval покриває 4 з 6 = 0.667; precision 1 з 1 = 1; faithfulness саме цих 6 фактів відповіді — 4 з 6 = 0.667. Не називати це оцінкою всіх тверджень відповіді.
service_not_rendered та unauthorized_debit правильні за повним корпусом, але відсутні у tool.result. Їхня наявність у query, яке сама модель склала перед пошуком, не є доказом отримання цих фактів із corpus retrieval.
Корпус звірено з Leed131/paypilot-stand-shared/app/rag/corpus/disputes.md, blob f9da1910abdc5d14064eb7773415035a0de7eaef.
Потрібна друга спроба в новій сесії.

## C-14 — спроба 2 і підсумок
[Структуровані дані](c14-attempt-2.json). Трейс `d45c25f7`, session `1097dae1d263`, перший хід, lesson-04, D05/D16/D17, prompt base.v1+D05, CUS-0002, replay=false.
Два пошуки kb_broken/top_k1: disputes.md#b1 і disputes.md#b2; get_account у цьому ході не викликано. Повний JSON надано лише для першого пошуку, другий — прев’ю на скриншоті.
Відповідь містить усі п’ять кодів і “duplicate_charge — 60 days”. Вхідні токени 4699, вихідні 1309, час 18049.1 ms.
**Підсумок двох спроб:** первісний пропуск строку відтворено 0 з 2; обидві частини відповіді наявні 2 з 2. C-14 — «не відтворюється у вибраному тестовому контексті»; вихідну скаргу за точкою відмови не класифікуємо. Повна відповідь не означає, що всі факти прийшли через пошук. Початковий дослівний запит невідомий; використано посилене формулювання з наявного кейса HW2-C14.
Третя спроба за критерієм ДЗ не потрібна.

## top_k 3 — прогін 1 із 3
[Повний запис](top-k-3-run-1.json). Трейс `d3a34b6d`, session `d407fc60aa8a`, перший хід, clean/base.v1, active_defects=[], без клієнта, replay=false.
Застосовано kb_clean/top_k3, запит такий самий, як у попередніх вимірах 4/5.
Трейс: llm.call → get_account (no customer is signed in for this session) → llm.call. Пошуку немає. Відповідь просить вхід або дозвіл шукати; жодного з двох числових фактів не наведено.
Вхід 2749, вихід 1097 токенів; час 12549.2 ms.
Це збережений перший прогін групи 3: повнота наразі 0 з 1, не відкидати результат і не починати нумерацію заново. Recall/precision/retrieval-faithfulness — «—» за правилом лабораторної. Відсутність пошуку не доводить недостатності top_k3: параметр у цьому ході не застосовувався до інструмента.
Наступний крок — той самий запит у новій сесії з тими самими налаштуваннями; не відповідати “так” у поточному діалозі, бо це вже був би інший тест із двома ходами.

## top_k 3 — прогін 2 із 3
[Запис](top-k-3-run-2.json). Трейс `17eefde1`, session `220c0bcb5ffb`, перший хід, clean/base.v1, active_defects=[], без клієнта, replay=false.
Той самий запит, застосовано kb_clean/top_k3. Після get_account агент виконав два пошуки:
1. limits.md#s1 (1.6166), glossary.md#s6 (1.4596), tariffs.md#s1 (1.1832). Повний JSON отримано; query: Tier 2 spread daily transfer limit Tier 2 spread day limit "Tier 2" spread limit. Рядок Tier 2 містить EUR 20,000 / EUR 200,000.
2. tariffs.md#s2 (1.3868), glossary.md#s6 (1.2511), tariffs.md#s1 (0.8452). Повний JSON надано; query: spread Tier 2 spread "Tier 2" spread tariffs spread FX spread Tier 2.
Відповідь містить 0.9% і EUR 20,000; додатково EUR 1,000 allowance. Повнота за двома потрібними фактами — так; виконання умови “один раз” — ні.
Вхід 6761, вихід 2500 токенів; час 31626.2 ms.
Повні результати обох пошуків підтверджено. За кожним окремо recall 1 з 2 = 0.5, precision 1 з 3 = 0.333; за весь хід recall 2 з 2 = 1, precision 2 з 6 = 0.333 (усі повернення, повтори враховано). За унікальними ID це було б 2 з 4 = 0.5, але це інший знаменник, не основна оцінка. Faithfulness саме двох запитаних числових тверджень — 2 з 2 = 1; повний текст відповіді окремо не розмічено.
Прогрес top_k3: 2 з 3 спроб, повні відповіді 1 з 2; перший невдалий прогін не виключено.

## top_k 3 — прогін 3 та підсумок серії
[Прогін 3](top-k-3-run-3.json), [підсумок усіх трьох](top-k-3-summary.json). Трейс `d3489cd2`, session `b709fb4e149a`, перший хід, clean/base.v1, без дефектів і клієнта, replay=false.
Надано повні результати двох пошуків. Перший query: “Tier 2 spread daily transfer limit Tier 2 spread daily limit transfer”; fragments limits#s1, glossary#s6, tariffs#s1. Другий query: “spread Tier 2 spread FX spread Tier 2 spread allowance tariffs spread Tier 2”; fragments tariffs#s2, glossary#s6, onboarding#s2. Повторна вставка JSON другого пошуку не є самостійним доказом третього виклику.
Відповідь містить 0.9% і EUR 20,000. Recall за ходом 2/2=1, precision 2/6=0.333 (повтори враховано), faithfulness двох запитаних числових фактів 2/2=1. Кожен пошук окремо повернув 1 із 2 потрібних фактів.
Токени: вхід 6701, вихід 2249; час 25348.8 ms.
**Підсумок top_k3:** повні відповіді 2 з 3; число пошуків 0/2/2; повних відповідей з рівно одним пошуком 0 з 3. Вхід 2749/6761/6701, сума 16211, середнє 5403.67; вихід 1097/2500/2249, сума 5846, середнє 1948.67. Перший невдалий прогін включено в повноту й середню вартість.
Лише серед двох прогонів, де був пошук: pooled recall 4/4=1, precision 4/12=0.333, faithfulness 4/4=1 для двох цільових чисел кожної відповіді. No-search прогін має «—», не підміняти це загальним recall всієї групи і не приховувати його за умовним середнім.
Наступний крок — top_k4, прогін 2 з 3; наявний №1: 5690602c.

## top_k 4 — прогін 2 із 3
[Запис](top-k-4-run-2.json). Трейс `d41ee38c`, session `e5c9d04c98a6`, перший хід, clean/base.v1, active_defects=[], без клієнта, replay=false.
Один пошук kb_clean/top_k4, query: “Tier 2 spread daily transfer limit Tier 2 spread limit”.
Повні фрагменти: limits.md#s1 (1.5011), glossary.md#s6 (1.4596), tariffs.md#s1 (1.1832), tariffs.md#s2 (1.1094).
Потрібні факти: Tier 2 daily limit EUR 20,000 у limits#s1, spread 0.9% у tariffs#s2. Обидва названі правильно.
Recall 2/2=1, precision 2/4=0.5, faithfulness двох запитаних числових тверджень 2/2=1. Повнота — так, умова рівно одного пошуку виконана.
Вхід 4677, вихід 1636 токенів; час 17833.3 ms.
Top_k4 після двох спроб: повнота 2/2; пошуки 1/1; вхідні токени 4691/4677, середнє 4684; вихідні 2016/1636, середнє 1826. Це проміжні дані, потрібен третій прогін.

## top_k 4 — прогін 3 та підсумок
[Прогін 3](top-k-4-run-3.json), [сводка](top-k-4-summary.json). Трейс `3dc10077`, session `a53ede79f918`, перший хід, clean/base.v1, без дефектів і клієнта, replay=false.
Query: “Tier 2 spread daily transfer limit Tier 2 spread 'Tier 2' 'spread' 'daily limit'”. Один пошук kb_clean/top_k4: limits#s1, glossary#s6, tariffs#s1, tariffs#s2; повні тексти отримано.
Відповідь — 0.9% і EUR 20,000; обидва значення підтверджені. Recall 2/2=1, precision 2/4=0.5, faithfulness двох запитаних чисел 2/2=1.
Вхід 4698, вихід 2266 токенів; час 24747.5 ms.
**Підсумок top_k4:** повнота 3/3, повні відповіді з рівно одним пошуком 3/3; вхід 4691/4677/4698, сума 14066, середнє 4688.67; вихід 2016/1636/2266, сума 5918, середнє 1972.67. Pooled recall 6/6=1, precision 6/12=0.5, faithfulness запитаних чисел 6/6=1.
Попереднє порівняння з top_k3: середній вхід менший на 715 токенів; top_k3 у двох успішних спробах потребував двох пошуків. Не перетворювати маленьку вибірку з різними query на загальний причинний закон. Остаточний вибір після завершення top_k5.

## top_k 5 — прогін 2 із 3
[Запис](top-k-5-run-2.json). Трейс `462aa686`, session `8d669865817a`, перший хід, clean/base.v1, active_defects=[], без клієнта, replay=false.
Один пошук kb_clean/top_k5, query: “Tier 2 spread daily transfer limit Tier 2 spread "Tier 2" daily limit transfer spread”.
Повні фрагменти: limits#s1, glossary#s6, tariffs#s1, tariffs#s2, limits#s2. Обидва запитані числа (0.9%, EUR 20,000) у фрагментах і відповіді.
Recall 2/2=1, precision 2/5=0.4, faithfulness двох запитаних числових тверджень 2/2=1. Умову одного пошуку виконано.
Вхід 4780, вихід 1770 токенів; час 18732.9 ms.
Після двох прогонів top_k5: повнота 2/2, пошуки 1/1, вхід 4778/4780, середнє 4779; вихід 2155/1770, середнє 1962.5. Потрібен третій прогін.

## top_k 5 — прогін 3 та завершення порівняння
[Прогін 3](top-k-5-run-3.json), [серія 5](top-k-5-summary.json), [усі дев’ять](top-k-comparison.json).
Трейс `2b1356f9`, session `ebff97694cf1`, перший хід, clean/base.v1, без дефектів і клієнта, replay=false. get_account не викликався: llm → один search_knowledge_base → llm.
Query: “Tier 2 spread daily transfer limit Tier 2 spread "Tier 2" daily limit transfer spread knowledge base Verta tariffs limits Tier 2”. Порядок: limits#s1, tariffs#s1, glossary#s6, tariffs#s2, limits#s2. Обидва числа у повних tool.result і відповіді. Recall 2/2=1, precision 2/5=0.4, faithfulness цільових чисел 2/2=1.
Вхід 3368, вихід 961; час 8953.7 ms.
**Top_k5:** повнота 3/3, один пошук 3/3; вхід 4778/4780/3368, сума 12926, середнє 4308.67; вихід 2155/1770/961, сума 4886, середнє 1628.67. Pooled recall 6/6=1, precision 6/15=0.4, faithfulness цільових чисел 6/6=1.
**Порівняння:** top_k3 — 2/3 повних, 5403.67 вхідних; top_k4 — 3/3, 4688.67; top_k5 — 3/3, 4308.67. У top_k5 №3 інший маршрут (немає get_account, на один llm.call менше), тому менше середніх токенів не приписуємо лише top_k. Для перших двох ходів з однаковим маршрутом вхідні середні 4684 (4) і 4779 (5), але це теж не контрольований дослід з однаковими query.
Попередньо обрано 4 за precision і достатньою повнотою цієї вибірки; не стверджувати, що 4 має нижчий фактичний середній вхід у цій серії. Остаточні межі — малі вибірки й одне питання. Дев’ять прогонів завершено; далі окрема перевірка kb_broken/top_k5.

## kb_broken / top_k 5 — контроль із запитом вправи 1
[Повні дані](broken-index-top-k-5.json). Трейс `d946930c`, session `ae3078d7c9dd`, перший хід, clean/base.v1, active_defects=[], CUS-0003/tier3, replay=false.
Два пошуки по 5 фрагментів. Перший: fx-guide#b2, fx-operations#b3, tariffs#b3, tariffs#b2, fx-guide#b4. Другий: tariffs#b3, onboarding#b2, fx-operations#b3, tariffs#b2, fx-guide#b2.
Повний тарифний фрагмент в обох закінчується “| Tier 3 | 0.5% | EUR 5”. `tariffs.md#b4` відсутній. Не плутати з `fx-guide.md#b4`, який прийшов у першому пошуку.
Відповідь підтверджує 0.5%, але визнає EUR 5 обривком і не повідомляє повного місячного ліміту. Recall 1/2=0.5; precision за всіма поверненнями 2/10=0.2; faithfulness єдиного ствердженого запитаного числа 1/1=1 (не оцінка всього тексту).
Вхід 6852, вихід 1219, час 13546.3 ms. Порівняно з top_k1/9196e7e0, де recall0/2, є часткове покращення; повна відповідь не відновлена. Query та число викликів різні, тому не приписувати всю дельту лише top_k.

## Набір ДЗ_3 — підготовка
Додано RET-H01…RET-H10: 2 з реконструйованих скарг, 8 із корпусу, 0 з прикладів вправ 1–2. Разом 42 кейси (37 daily, 5 release). Початкові 32 рядки збережено без змін.
Еталони дослівно взято з вихідних tariffs.md, limits.md, disputes.md; версії й рядки у retrieval-case-sources.json. Модельні відповіді не джерело еталона.
Локально виконано оригінальні cli.py/validate.py/loader.py/assertions.py кіту: --set golden --dry-run, exit0; окремо звірено всі reference з точними corpus blob SHA. Hash `bba68640a585`. Це не Docker-прогін на комп’ютері користувача і не eval агента; raw eval reports поки відсутні.

## Docker dry-run на комп’ютері користувача
[Підтвердження зі скриншотів](docker-dry-run.json), 24.10.2026.
Команда з l03: STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --dry-run.
Повне зведення завершилося й повернуло shell prompt: 42 кейси, retrieval10, daily37, release5, hash bba68640a585. Hash збігається з підготовленим у GitHub файлом. Три скриншоти показують однакову перевірку; це не модельні прогони і не додаткові спроби top_k.
Прогноз збережено до model eval у коміті 81753a9decf97c22ff9eff6dbb9ef36353f7725f. Наступний крок — clean eval незміненого набору, потім lesson-04; зафіксувати шлях і raw JSON звітів.

## Невиконані вимоги ДЗ_3
- C-13 і C-14 перевірено по дві спроби: вихідні симптоми не відтворено; перевірку скарг завершено, обмеження реконструкції записано.
- Усі дев’ять прогонів top_k3/4/5 завершено, результати зведено. Додаткові повтори цього порівняння зараз не потрібні.
- Перевірку kb_broken/top_k5 завершено; tariffs.md#b4 не прийшов, повного ліміту немає.
- 10 retrieval-кейсів додано, reference і локальний штатний CLI dry-run перевірено, hash bba68640a585. Docker dry-run на комп’ютері користувача підтверджено. Залишилися модельні clean/lesson-04 eval, raw reports та звірка прогнозу (прогноз комітиться до них).
- Заповнити ціни/ризики/рішення та остаточний звіт ≤2 сторінки.


## Реальний модельний eval — clean
24.10.2026, команда з l03: `STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --profile clean`.
Наданий повний console output збережено у [clean-eval-console-20261024.txt](clean-eval-console-20261024.txt). 37/37 daily PASS, action 11/11, generation 16/16, retrieval 10/10; час 494.6 с. П’ять release-кейсів не виконувалися. Ранер повідомив `profile clean`, `prompt base.v1`, `gate daily`.
Шлях у console: `evals/reports/golden-clean-20261024T213319.json`; Docker volume означає локальний `l03/reports/golden-clean-20261024T213319.json`.
Прогноз clean 10/10 нових retrieval збігся з результатом. Первісний прогноз lesson-04 8/10 не змінювався. Сирий JSON ще не надано: hash і повні payload-и цього модельного звіту потрібно перевірити після отримання; console preview фрагментів обрізане. PASS за contains не доводить усі вимоги формату: RET-H01 має додатковий вступ, хоча числові рядки правильні. CMP-004 спершу викликав get_account з некоректним аргументом, потім відновився; case PASS не означає безпомилковість усіх кроків.


## Lesson-04 eval і сирий clean JSON
Надано повний raw clean JSON (як Texto pegado(3).txt): валідний JSON, profile=clean, prompt_version=base.v1, set_hash=bba68640a585, 37/37, retrieval10/10. Без змін збережено у reports/golden-clean-20261024T213319.json. Усі десять нових clean-кейсів викликали пошук; фрагменти повні. Багато старих answers у raw файлі обрізано рівно на 400 символах — не називати його повною стенограмою відповідей. Полів із назвою моделі цей звіт не містить.
Lesson-04 за terminal summary: 31/37, action11/11, generation16/16, retrieval4/10, 530.5с; report golden-lesson-04-20261024T214702.json. FAIL H01/H02/H03/H07/H09/H10. Очікували FAIL H01/H03; додаткові H02/H07/H09/H10, отже прогноз8/10 проти факту4/10. Частковий console збережено; [попередній аналіз](lesson-04-eval-console-summary.json). Raw lesson-04 ще потрібен для hash та причин.
H10 логіка either коректна, але exact contains не збігається; цитата починається з обривка, також був невдалий tool аргумент top_k=5. H09 описує потрібну ескалацію словами MUST call escalate_to_human, але не містить literal expected; перевірка джерела цитати потребує raw payload. Не позначати ці literal FAIL автоматично як хибні правила. H07 console кейса не надано. H03 відповідь містить 0.00% замість0.3%; без raw результату інструмента не відносити збій лише до D05. Повтор прогона і зміна тестів зараз не потрібні.


## Фінальна звірка обох raw reports
Одержано lesson-04 JSON (Texto pegado(4).txt), без змін збережено reports/golden-lesson-04-20261024T214702.json. Hash bba68640a585 збігається в обох raw і golden; lesson-04 prompt=base.v1+D05, 31/37, retrieval4/10. [Остаточний розбір](model-run-analysis.md) замінює попередні припущення console summary; усі старі записи вище — історія збору доказів.
Підтверджено H07: 12 пошуків без 56, step budget. H03: fees-detail#b1 без числових компонентів, 0.00% у відповіді не grounded. H09: повної цитати MUST call немає в retrieval; напрям дії правильний, атрибуція цитати — ні. H10: правильний зміст either, literal не збігається, цитата обірвана. H04/H05: PASS без виклику пошуку; retrieval-метрики —. Ці недоліки assertions не виправляли заднім числом.
Прогноз коміт81753a9 має created_at21:11:32UTC, раніше clean ran_at21:33:19 і lesson-04 ran_at21:47:02. Набір не змінювали. [Claims audit](index-pair-claims.json) визначає scope 4 перевірених documentary-тверджень кожного ходу; не видавати за оцінку акаунтних даних/пропозицій або повну faithfulness довільного тексту.
Звіт завершено за шістьма розділами: ціна перебудови — явно умовна планова оцінка, залишковий ризик — правильний текст без реальної ескалації. Контрольна верстка A4, DejaVu Sans9.3pt, поля34pt — 2 сторінки; PNG обох сторінок перевірено без обрізання. Markdown не фіксує сторінки в кожному переглядачі. Raw runner answers[:400] не дозволяє вважати всі відповіді повними; модель агента не записано. Залишилося лише підтвердити LLM_PROVIDER/LLM_MODEL зі стенду; історичний ID кожного ходу не відновлюємо з поточної конфігурації.


## Підтвердження моделі — 25.10.2026
Скриншот термінала 00:00:22: команда вибіркового читання конфігурації повернула LLM_PROVIDER=openai та LLM_MODEL=gpt-5-mini. [Структурований запис](model-configuration.json). Це закриває запит на назву провайдера/моделі; точні snapshot ID всіх історичних ходів не записані. Трейс D05 окремо показував gpt-5-mini-2025-08-07. ДЗ_3 підготовлено до перевірки ментором: звіт, незмінений набір, обидва raw, порівняння прогнозу та межі збережені. Додаткові прогони не проводили.


## Повне врахування побажання ментора — mentor-v2
25.10.2026 користувач попросив також замінити чотири кейси і провести нові eval. H04: refund через Verta error; H05: reversal rate at reversal time; H06: 7 calendar days для requested evidence; H07: downgrade at next annual review. Старі SEPA/card/Tier1/unauthorized_debit факти вилучено з цих чотирьох кейсів; вихідні 32 ДЗ_2 і шість інших RET-кейсів не змінено.
Поточний hash **bcf0121ec645**, 42 кейси, retrieval10, daily37, release5. Локальний штатний CLI dry-run exit0, reference/SHA нових чотирьох перевірено. [Зміни](mentor-v2-changes.json). Старі набори/sources/verification з hashbba68640a585 архівовано в datasets/, старі raw лишилися без змін. Старий аналіз не є результатом нової версії.
[Прогноз mentor-v2](forecast-mentor-v2.md) у коміті 7f4d65b5ea9251c7bd3c1fbc2a14d278684d20dc: clean10/10 retrieval, lesson-04 5/10; FAIL H01/H02/H03/H09/H10. Прогноз зроблено до нових model eval. Очікуються Docker dry-run на комп’ютері користувача і нові clean/lesson-04 raw із bcf0121ec645. Зарахування нової версії ще не заявляємо; обов’язкова правка цін і строків розділу6 збережена.
