# L04 · RAGAS Diagnostic Report — PayPilot
**Статус: ДЗ_3 у роботі. Вимірювання стенду та підготовку набору завершено; clean eval пройдено; lesson-04, сирі JSON і фіналізація звіту ще попереду.**

Набір із [ДЗ_2](https://github.com/Leed131/paypilot-dz2): 32 кейси збережено без змін, додано 10 нових retrieval/corpus-кейсів. Разом **42 кейси: 37 daily і 5 release**. П’ять human/release-кейсів успадковано з ДЗ_2; типовий daily-прогін їх не запускає.

`set_hash: bba68640a585` — обчислено штатним CLI кіту для поточних байтів golden.jsonl. Після копіювання у локальний кіт Docker dry-run підтвердив такий самий hash; див. evidence/docker-dry-run.json.

## Файли
- [ragas-diagnostic-report.md](ragas-diagnostic-report.md) — чернетка шести розділів; TODO ще не закрито.
- [golden.jsonl](golden.jsonl) — 32 попередні + RET-H01…RET-H10.
- [evidence/lab-notes.md](evidence/lab-notes.md) — ручні вимірювання, трейси, токени.
- [evidence/top-k-comparison.json](evidence/top-k-comparison.json) — усі дев’ять прогонів top_k3/4/5.
- [evidence/broken-index-top-k-5.json](evidence/broken-index-top-k-5.json) — bigger top_k не відновив повний Tier 3 allowance у цьому прогоні.
- [evidence/retrieval-case-sources.json](evidence/retrieval-case-sources.json) — дослівні reference, рядки, секції та corpus blob SHA.
- [evidence/dataset-validation.json](evidence/dataset-validation.json) — успішна локальна перевірка оригінальним CLI та перевірка всіх reference. Це не Docker- і не модельний eval-прогін.
- `reports/*.json` — TODO після реальних запусків ранера.

## Відтворення
У локальному кіті `paypilot-l03-eval` замінити `sets/golden.jsonl` копією поточного файлу цього репозиторію. Команди виконувати з теки кіту. У ДЗ_2 використовувався шлях `STAND_DIR=../paypilot-stand-shared`; звірити з фактичним розташуванням стенду.

Спершу перевірити набір у своєму Docker:
```bash
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --dry-run
```

Очікувані структурні дані: 42 cases, retrieval 10, daily 37, release 5, hash bba68640a585. Локальна перевірка тут уже пройшла без Docker; результат у evidence/dataset-validation.json.

Модельні запуски виконувати лише після збереженого нижче прогнозу:
```bash
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --profile clean
STAND_DIR=../paypilot-stand-shared docker compose run --rm eval --set golden --profile lesson-04
```

Записати фактичні вердикти, помилки та зберегти сирі JSON із `reports/` кіту. Якщо lesson-04 не підтримується, зберегти точне повідомлення замість вигаданого результату. Звіти порівнювати тільки з однаковим set_hash.
`.env` і ключі не належать до файлів здачі.

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

**Коміт із прогнозом:** [`81753a9`](https://github.com/Leed131/paypilot-dz3/commit/81753a9decf97c22ff9eff6dbb9ef36353f7725f) — окремий коміт до будь-яких модельних eval оновленого набору.
**Спостереження після прогнозу:** clean — 37/37 daily PASS; нові retrieval — 10/10 PASS. Console: [evidence/clean-eval-console-20261024.txt](evidence/clean-eval-console-20261024.txt). lesson-04 ще не виконано.
**Розбіжність із прогнозом:** clean 10/10 збігається з прогнозом 10/10; lesson-04 ще не виміряно. Первинний прогноз вище збережено.

## Прогони
| Прогін | Профіль | Фактичний результат | Сирий звіт |
|---|---|---|---|
| Локальний штатний CLI --dry-run | Без стенду | Exit 0; 42 кейси; hash bba68640a585 | evidence/dataset-validation.json |
| Docker --dry-run користувача | Без модельних запитів | Завершено; 42 кейси, retrieval 10, hash bba68640a585 | evidence/docker-dry-run.json |
| Baseline | clean | 37/37 daily PASS: action 11/11, generation 16/16, retrieval 10/10; 494.6 с | Console збережено; raw golden-clean-20261024T213319.json очікується |
| Профіль заняття | lesson-04 | Не виконано для оновленого набору | TODO |

## Підтверджені ручні результати
- C-13/C-14: по дві нові сесії; вихідні симптоми не відтворено у реконструйованих сценаріях. Побічні збої та обмеження збережено окремо.
- Top_k3/4/5: повнота 2/3, 3/3, 3/3; середній вхід 5403.67, 4688.67, 4308.67. Менший вхід при 5 має застереження про інший маршрут викликів.
- Broken/top_k5: tariffs.md#b4 не прийшов; спред знайдено, повного allowance немає. Fx-guide.md#b4 — інший документ.
- D25 ×5 і правило промпту збережено; D05 контроль без пошуку збережено.

## Залишилося до здачі
1. Docker dry-run і модельний clean eval підтверджено. Далі провести lesson-04 eval.
2. Додати raw reports із тим самим hash; зіставити фактичні retrieval FAIL із закоміченим прогнозом.
3. Закрити TODO звіту: метадані/faithfulness пари індексів, оцінка вартості переіндексації, залишковий ризик і останній рядок рішення.
4. Скоротити остаточний звіт до двох сторінок і звірити чекліст ДЗ.
