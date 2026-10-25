# Модельні прогони mentor-v2 — поточна версія

Набір **bcf0121ec645**, 42 кейси (37 daily, 5 release). Новий lesson-04 raw отримано; новий clean raw ще очікується. Старі clean37/37 і lesson-04 31/37 належать hash bba68640a585, не переносяться на v2.

## Отриманий lesson-04

[Raw JSON](../reports/golden-lesson-04-20261025T082038.json): profile lesson-04, prompt base.v1+D05, ran_at 2026-10-25T08:20:38.025791+00:00, hash bcf0121ec645, gate daily, judge_model not-used. **34/37 PASS**: action11/11, generation16/16, retrieval**7/10**. Час559.1с, input+output загалом245705 токенів. П'ять release не запускалися; JSON не записує окремих input/output та моделі агента, відповіді обмежено 400 символами.

[Прогноз](forecast-mentor-v2.md) закомічено як [7f4d65b](https://github.com/Leed131/paypilot-dz3/commit/7f4d65b5ea9251c7bd3c1fbc2a14d278684d20dc), created_at **2026-10-25T07:54:20Z**, до ran_at нового звіту. Очікували lesson-04 **5/10** retrieval, отримали **7/10**. Несподівані PASS — **H02 і H09**; інші вісім прогнозів lesson-04 збіглися. Clean прогноз10/10 поки не звірений із raw.

## Кейси та доказ

| Case | Прогноз / факт lesson-04 | Пошуків | Розбір |
|---|---|---:|---|
| RET-H01 | FAIL / FAIL | 3 | fx-guide#b4, потім двічі #b2; тарифних рядків немає. Агент повідомляє про неповноту. Query також безпідставно вводить Bronze/Silver/Gold; це внутрішній query агента, не доказ таких tier у корпусі. |
| RET-H02 | FAIL / PASS | 5 | disputes#b1 тричі, #b2 двічі. #b2 містить fraud_card_not_present, goods_not_received, duplicate_charge та60. Відповідь містить усі5кодів+60; service_not_rendered/unauthorized_debit відсутні в tool.results. Recall4/6=0.667, precision2/5=0.4, faithfulness цих6фактів4/6=0.667. Повний текст правильний за корпусом, але retrieval неповний. Самостійно складений query містить останні два коди і не є їхнім незалежним джерелом. |
| RET-H03 | FAIL / FAIL | 1 | fees-detail#b1 каже flat plus percentage без значень. Відповідь EUR15.00+0.00%, корпус15.00+0.3%. Обидва числа не підтверджені retrieval; неповний вхід і генерація без опори співіснують. D05 не ізольовано від D16/D17. |
| RET-H04 r2 | PASS / PASS | 1 | fees-detail#b4 прямо містить because of a Verta error; нова умова refund доставлена й названа. |
| RET-H05 r2 | PASS / PASS | 3 | fx-operations#b1 без потрібного факту, потім двічі #b7 із rate in force at reversal time. Правило підтверджено, але знадобилися повтори. |
| RET-H06 r2 | PASS / PASS | 1 | disputes-process#b8 містить Failure to supply requested evidence within 7 calendar days results in the case being declined. Строк і наслідок у відповіді правильні. |
| RET-H07 r2 | PASS / PASS | 1 | onboarding#b3 містить downgraded at the next annual review; потрібний момент downgrade підтверджено. |
| RET-H08 | PASS / PASS | 1 | disputes-process#b1 містить single settled transaction; правильне підтвердження з альтернативного релевантного документа. |
| RET-H09 | FAIL / PASS | 2 | Обидва disputes#b6 містять blocked by a customer-level restriction, the case must be escalated to a human agent. Основне правило в контексті, на відміну від v1. Відповідь додає If a dispute… і називає це цитатою, тоді як повне джерело починається When a dispute…: policy підтримана, дослівність цитати неповна. |
| RET-H10 | FAIL / FAIL | 4 | limits#b4 тричі без правила, далі limits-operations#b4, що починається з either the daily or the monthly remainder…. Пояснення правильно каже either, не both, але literal exceeds either remainder відсутній, цитата обірвана. Це не доказ хибної бізнес-логіки. |

Усі чотири змінені кейси r2 викликали пошук. Для одного цільового факту кожного: H04/H06/H07 recall1/1=1, precision1/1=1; H05 recall1/1=1, precision2/3=0.667 (повтор враховано). Faithfulness саме цільових відповідей цих чотирьох — по1/1=1; не оцінка всього тексту чи довільних наступних пропозицій.

Не приписуємо зміні чотирьох кейсів усе покращення загального результату v1→v2: H02/H09 не змінювалися, але їхні вердикти змінилися; в H09 прийшов інший fragment. Це стохастична поведінка/маршрутизація пошуку, не окремий ізольований експеримент причинності.

## Що залишається

Зберегти новий clean raw golden-clean-20261025T081118.json (ім'я видно на скриншоті, вміст ще не отримано), перевірити його hash і вердикти, звірити clean прогноз10/10. Після цього завершити поточний підсумок розділу1/6 звіту. Повторювати вже завершені прогони не потрібно. Прогноз, набір та отриманий raw залишаємо незмінними.
