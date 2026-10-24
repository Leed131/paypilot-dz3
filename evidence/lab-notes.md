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

## Невиконані вимоги ДЗ_3
- C-13 і C-14 перевірено по дві спроби: вихідні симптоми не відтворено; перевірку скарг завершено, обмеження реконструкції записано.
- По 3 прогони top_k 4/5/3 (є тільки по одному з токенами для 4/5).
- kb_broken/top_k5, перевірити саме tariffs.md#b4.
- ≥10 нових кейсів, corpus reference, dry-run, прогноз окремим комітом до eval, clean/lesson-04 eval і новий set_hash.
- Заповнити ціни/ризики/рішення та остаточний звіт ≤2 сторінки.
