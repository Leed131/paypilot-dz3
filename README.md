# L04 · RAGAS Diagnostic Report — PayPilot

Основні артефакти й обидва модельні звіти збережено. Залишилося підтвердити назву моделі з конфігурації стенду: raw JSON її не записують; відоме спостереження D05 — OpenAI / gpt-5-mini-2025-08-07. Це обмеження прямо вказане у звіті.

Набір [ДЗ_2](https://github.com/Leed131/paypilot-dz2): **32 незмінені кейси + 10 нових retrieval/corpus = 42**, із них **37 daily, 5 release**. Поточний golden.jsonl і обидва raw JSON мають **set_hash=bba68640a585**. П’ять release-кейсів не входили до daily-прогонів.

## Файли

- [ragas-diagnostic-report.md](ragas-diagnostic-report.md) — шість розділів і останній рядок; контрольний A4-рендер вклався у дві сторінки (9.3 pt, поля 34 pt; Markdown сам по собі не фіксує пагінацію).
- [golden.jsonl](golden.jsonl) — незмінені 32 + RET-H01…RET-H10.
- [clean raw JSON](reports/golden-clean-20261024T213319.json) і [lesson-04 raw JSON](reports/golden-lesson-04-20261024T214702.json) — збережені без виправлення відповідей/вердиктів.
- [Розбір усіх нових кейсів](evidence/model-run-analysis.md) — причини шести FAIL, literal/semantic межі та PASS без пошуку.
- [Ручні докази](evidence/lab-notes.md), [top_k: дев’ять прогонів](evidence/top-k-comparison.json), [контроль broken/k5](evidence/broken-index-top-k-5.json).
- [Claims audit ізольованої пари](evidence/index-pair-claims.json), [джерела reference](evidence/retrieval-case-sources.json), [валідація набору](evidence/dataset-validation.json), [Docker dry-run](evidence/docker-dry-run.json).

## Відтворення

Стенд PayPilot має бути запущений з налаштованим провайдером. У локальному кіті paypilot-l03-eval (у цьому середовищі папка l03) покласти поточний golden.jsonl у sets/golden.jsonl. Команди запускати з папки кіту. STAND_DIR вказує на сусідній paypilot-stand-shared.

```bash
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --dry-run
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --profile clean
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --profile lesson-04
```

Dry-run: 42 кейси, retrieval 10, daily 37, release 5, hash bba68640a585. Модельні запуски вже виконані; повторювати для здачі не потрібно. Наведені команди — для відтворення. Прогноз нижче був закомічений до обох запусків. Console-повідомлення evals/reports/ відповідає локальній папці reports/ кіту через Docker volume. Стенд, кіт, .env і API-ключі не входять у файли цієї роботи.

## Прогони

| Прогін | Профіль / prompt | Фактичний результат | Джерело |
|---|---|---|---|
| CLI --dry-run | Без моделі | Exit 0; 42 кейси; hash bba68640a585 | [Validation](evidence/dataset-validation.json) |
| Docker --dry-run | Без моделі | 42 кейси; hash bba68640a585 | [Docker evidence](evidence/docker-dry-run.json) |
| Baseline | clean / base.v1 | **37/37**; action 11/11, generation 16/16, retrieval **10/10**; 494.6 с | [Raw](reports/golden-clean-20261024T213319.json) |
| Профіль заняття | lesson-04 / base.v1+D05 | **31/37**; action 11/11, generation 16/16, retrieval **4/10**; 530.5 с | [Raw](reports/golden-lesson-04-20261024T214702.json) |

Кожен новий кейс — один запуск. Це assertion-вердикти, не ймовірність безпомилкової роботи. У raw немає моделі агента та окремих input/output токенів; частину відповідей обрізано ранером на 400 символах. Tool.result.fragments збережено повністю. Загальні токени всіх кейсів: clean 209461, lesson-04 305759; не ототожнюємо їх із вхідними токенами ручного top_k-порівняння.

## Retrieval-кейси
Усі 10 нових кейсів мають layer=retrieval, oracle=corpus, assertion=contains, runs=1, gate=daily, added_in=hw3. Два — реконструкції скарг, вісім — окремі правила/рядки корпусу; кейсів із прикладів вправ 1–2 немає.
| id | Джерело | Перевірка | Файл і рядки корпусу |
|---|---|---|---|
| RET-H01 | complaint C-13 | Уся таблиця FX-спредів і allowance | app/rag/corpus/tariffs.md, FX spreads by tier, рядки 14–16 |
| RET-H02 | complaint C-14 | Усі reason codes + duplicate_charge window | app/rag/corpus/disputes.md, Reason codes and chargeback windows, рядки 8–12 |
| RET-H03 | corpus | Обидва компоненти комісії SWIFT | app/rag/corpus/tariffs.md, Transfer fees by type, рядки 23–23 |
| RET-H04 | corpus | Фіксована комісія SEPA | app/rag/corpus/tariffs.md, Transfer fees by type, рядки 22–22 |
| RET-H05 | corpus | Відсоткова комісія card payment | app/rag/corpus/tariffs.md, Transfer fees by type, рядки 24–24 |
| RET-H06 | corpus | Денний і місячний ліміти Tier 1 | app/rag/corpus/limits.md, Daily and monthly limits by tier, рядки 11–11 |
| RET-H07 | corpus | Строк unauthorized_debit | app/rag/corpus/disputes.md, Reason codes and chargeback windows, рядки 12–12 |
| RET-H08 | corpus | Статус settled для eligibility | app/rag/corpus/disputes.md, Eligibility conditions, рядки 16–16 |
| RET-H09 | corpus | Ескалація при customer-level restriction | app/rag/corpus/disputes.md, Escalation on blocked disputes, рядки 22–23 |
| RET-H10 | corpus | Відмова при перевищенні будь-якого remainder | app/rag/corpus/limits.md, How the remainder is computed, рядки 18–19 |

Contains перевіряє заданий текст/формат, а не всі можливі перефразування. Сам PASS не доводить, що агент викликав пошук чи що весь reference був у retrieval-контексті. Це межа цих перевірок, продемонстрована ручними C-14.


## Прогноз ДО модельних прогонів
**Гіпотеза, не фактичний результат.** Структурна валідація не викликає модель. Після її успіху жоден модельний прогін оновленого набору ще не виконувався.

- На clean очікую PASS усіх 10 нових RET-H01…RET-H10: корпус цілий, еталони перевірено. Формат contains може спричинити FAIL навіть за змістовно правильної відповіді; такий результат потрібно відрізнити від фактичної помилки.
- На lesson-04 прогнозую FAIL **RET-H01**: kb_broken розриває Tier 3 allowance, top_k1 не гарантує доставки хвоста таблиці; можливі повторні пошуки й вичерпання кроків.
- На lesson-04 прогнозую FAIL **RET-H03**: D05 може нав’язати застарілу фіксовану SWIFT-комісію без 0.3%, як у ручному контролі; вимога явного пошуку може змінити це, тому це гіпотеза.
- Для **RET-H02, RET-H04, RET-H05, RET-H06, RET-H07, RET-H08, RET-H09, RET-H10** прогнозую PASS. Невизначеність найбільша для RET-H02: у ручних C-14 обидві частини відповідей були правильними навіть за неповного retrieval; новий кейс додатково вимагає конкретного формату.
- Разом для нових retrieval-кейсів: очікування clean **10/10**, lesson-04 **8/10**. Це не прогноз сукупного результату всіх 37 daily-кейсів.


**Коміт із прогнозом:** [81753a9](https://github.com/Leed131/paypilot-dz3/commit/81753a9decf97c22ff9eff6dbb9ef36353f7725f), **2026-10-24 21:11:32 UTC**; clean ran_at 21:33:19 UTC, lesson-04 ran_at 21:47:02 UTC. Історичний текст прогнозу вище збережено.

## Факт і розбіжність із прогнозом

Clean retrieval **10/10** проти прогнозу **10/10**. Lesson-04 **4/10** проти **8/10**. Очікувані FAIL H01/H03 відбулися; додаткові FAIL **H02, H07, H09, H10**. H02/H07 — step budget після 12 пошуків. H03 — непідтверджені числа, зокрема 0.00% замість 0.3%. H09 — не підтверджена retrieval-фрагментом цитата; H10 — правильне правило either, але інша literal-фраза й обірвана цитата. Первісні вердикти залишено.

H04/H05 отримали PASS без будь-яких інструментів: текст правильний, вимога пошуку не перевірена assertion. H06 знайшов потрібні ліміти за 7 пошуків; H08 підтверджено іншим релевантним документом. Отже сам contains не діагностує всі властивості retrieval. [Повний розбір](evidence/model-run-analysis.md).

## Рішення та межі

- Для цього питання обрано top_k4: повнота 3/3, precision 0.5 проти 0.4 при k5. Фактичний середній input k5 нижчий через інший маршрут, тому не заявляємо, що k4 завжди дешевший.
- Broken/k5 знайшов спред, але повного allowance не відновив. Потрібно виправляти цілісність фрагментів; більший k не гарантує цього.
- D25 ×5 показує генераційний збій на правильному tool result. D05 контроль — відповідь без пошуку.
- C-13/C-14: вихідні симптоми 0/2 у реконструкціях; це не доказ відсутності проблем на інших запитах.
- Оцінка перебудови корпусу у звіті — плановий сценарій з явними припущеннями, не рахунок постачальника.
- Залишковий ризик: правильний опис ескалації без реальної дії. Retrieval-метрики перевіряють текст; автономні дії потребують action-перевірок.
- Не змінюємо golden.jsonl, oracle або прогноз, щоб прибрати фактичні FAIL. Поліпшення assertions — окрема наступна версія.

Потрібне лише підтвердження LLM_PROVIDER/LLM_MODEL із конфігурації без секретів. Воно уточнить метадані, але не відновить відсутній історичний model ID кожного ходу.

