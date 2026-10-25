# L04 · RAGAS Diagnostic Report — PayPilot

**Перегляд mentor-v2: новий набір підготовлено, нові модельні прогони очікуються.** На прохання користувача виконуємо також побажання ментора: замінено RET-H04/H05/H06/H07 на інші факти корпусу. Умовні ціна й строк виправлень видачі/генерації вже внесені до розділу 6.

Поточний **set_hash=bcf0121ec645**. Разом **42 кейси: 37 daily, 5 release**, із них 10 retrieval/corpus. Вихідні 32 кейси [ДЗ_2](https://github.com/Leed131/paypilot-dz2) і шість незмінених RET-кейсів збережено. Змінені чотири мають case_revision=2 і dataset_revision=mentor-v2. Provider/model за підтвердженою конфігурацією: OpenAI / gpt-5-mini; raw не записують історичний snapshot кожного ходу.

## Файли

- [golden.jsonl](golden.jsonl) — **поточна mentor-v2**, hash bcf0121ec645.
- [ragas-diagnostic-report.md](ragas-diagnostic-report.md) — ручні лабораторні докази й оцінки впровадження; модельні числа v1 підписані як історичні до нових прогонів.
- [Прогноз mentor-v2](evidence/forecast-mentor-v2.md) — окремо закомічений **до** нових eval: [7f4d65b](https://github.com/Leed131/paypilot-dz3/commit/7f4d65b5ea9251c7bd3c1fbc2a14d278684d20dc).
- [Перевірка mentor-v2](evidence/dataset-validation-mentor-v2.json), [точні references і SHA](evidence/retrieval-case-sources.json), [заміна чотирьох кейсів](evidence/mentor-v2-changes.json).
- [Top_k: дев’ять прогонів](evidence/top-k-comparison.json), [broken/k5](evidence/broken-index-top-k-5.json), [ручні докази](evidence/lab-notes.md), [claims audit](evidence/index-pair-claims.json).
- [Метадані моделі](evidence/model-configuration.json).

## Нові факти замість повторів заняття

| Case | Було у v1 | Стало у mentor-v2 |
|---|---|---|
| RET-H04 | Комісія SEPA | Refund лише через помилку Verta |
| RET-H05 | Комісія карткою | Rate in force at reversal time |
| RET-H06 | Ліміти Tier 1 | 7 calendar days на requested evidence |
| RET-H07 | Window unauthorized_debit | Downgrade at next annual review |

Ці чотири правила не повторюють зазначені ментором факти й не дублюють решту кейсів набору. Еталони дослівно взято з fees-detail.md, fx-operations.md, disputes-process.md, onboarding.md, не з відповідей агента. Обмеження contains збережено: перевіряє потрібну фразу, а не факт виклику пошуку чи весь зміст відповіді.

## Поточні retrieval-кейси

Усі 10: layer=retrieval, oracle=corpus, assertion=contains, added_in=hw3, runs=1, gate=daily. Два джерела complaint, вісім corpus.

| id / revision | Перевірка | Corpus source |
|---|---|---|
| RET-H01 | Уся таблиця FX-спредів і allowance | app/rag/corpus/tariffs.md, FX spreads by tier, рядки 14–16 |
| RET-H02 | Усі dispute reason codes та duplicate_charge=60 | app/rag/corpus/disputes.md, Reason codes and chargeback windows, рядки 8–12 |
| RET-H03 | Обидва компоненти SWIFT-комісії | app/rag/corpus/tariffs.md, Transfer fees by type, рядки 23–23 |
| RET-H04 r2 | Причина refund: помилка Verta | app/rag/corpus/fees-detail.md, Fee refunds, рядки 17–18 |
| RET-H05 r2 | Курс при reversal конвертованого платежу | app/rag/corpus/fx-operations.md, Reversals, рядки 25–26 |
| RET-H06 r2 | Строк подання requested evidence | app/rag/corpus/disputes-process.md, Evidence requirements, рядки 26–27 |
| RET-H07 r2 | Момент downgrade тарифного рівня | app/rag/corpus/onboarding.md, Downgrades, рядки 13–14 |
| RET-H08 | Статус settled для dispute | app/rag/corpus/disputes.md, Eligibility conditions, рядки 16–16 |
| RET-H09 | Правило ескалації при restriction | app/rag/corpus/disputes.md, Escalation on blocked disputes, рядки 22–23 |
| RET-H10 | Відмова при перевищенні будь-якого remainder | app/rag/corpus/limits.md, How the remainder is computed, рядки 18–19 |

## Відтворення mentor-v2

Команди виконувати з папки локального кіту l03 / paypilot-l03-eval при запущеному стенді:

```bash
cp ../paypilot-dz3/golden.jsonl sets/golden.jsonl
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --dry-run
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --profile clean
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --profile lesson-04
```

Спершу звірити dry-run: hash **bcf0121ec645**, 42 cases, retrieval10, daily37, release5. Локальний штатний CLI без моделі вже успішний; новий Docker dry-run на комп’ютері користувача ще очікується. До запусків прогноз уже закомічено. Зберегти обидва нові raw JSON, переконатися в однаковому hash. П’ять release-кейсів не запускаються типовим daily gate. .env і ключі не належать до файлів здачі.

## Прогноз mentor-v2 до прогонів

Clean: **10/10** нових retrieval. Lesson-04: **5/10**, очікувані FAIL **RET-H01/H02/H03/H09/H10**; нові H04–H07 і незмінний H08 — PASS. У чотирьох нових фактах потрібні фрази цілі в окремих broken-чанках; це структурна перевірка корпусу, не гарантія ранжування чи поведінки моделі. H06 має ризик плутанини зі строками розгляду спору. Прогноз для 10 retrieval, не для всіх37 daily. [Пояснення по кожному кейсу](evidence/forecast-mentor-v2.md).

## Прогони поточної mentor-v2

| Запуск | Статус | Hash / результат |
|---|---|---|
| Штатний CLI --dry-run | Завершено, exit0; модель не викликалась | bcf0121ec645; 42 cases, retrieval10 |
| Docker --dry-run користувача | Очікується | Звірити bcf0121ec645 |
| clean model eval | Не виконано | Немає raw mentor-v2 |
| lesson-04 model eval | Не виконано | Немає raw mentor-v2 |

## Історична v1 — результати до заміни кейсів

**Ці звіти НЕ описують поточний golden.jsonl.** Їхній hash — bba68640a585. [Точний старий набір](evidence/datasets/golden-bba68640a585.jsonl), [старі sources](evidence/datasets/sources-bba68640a585.json), [перевірка v1](evidence/datasets/verification-bba68640a585.json).

| Профіль v1 | Результат | Raw |
|---|---|---|
| clean/base.v1 | 37/37, retrieval10/10, 494.6с | [golden-clean-20261024T213319.json](reports/golden-clean-20261024T213319.json) |
| lesson-04/base.v1+D05 | 31/37, retrieval4/10, 530.5с | [golden-lesson-04-20261024T214702.json](reports/golden-lesson-04-20261024T214702.json) |

Прогноз v1 [81753a9](https://github.com/Leed131/paypilot-dz3/commit/81753a9decf97c22ff9eff6dbb9ef36353f7725f) був до v1 eval: ожидали clean10/10, lesson-04 8/10; фактично 10/10 і4/10. [Історичний розбір](evidence/model-run-analysis.md) не переносимо на нові H04–H07. Raw обрізають answers на 400 символах, tool fragments повні; judge_model=not-used — не модель агента.

## Наступний крок

Підтвердити новий Docker dry-run, провести clean і lesson-04 з незміненим bcf0121ec645, додати нові raw і зіставити з прогнозом mentor-v2. Після цього оновити модельний підсумок звіту; факти ручних серій та обов’язкові оцінки впровадження зберігаються.
