# Risk Register

Оцінки нижче є початковою PM-оцінкою для планування.  
Їх потрібно переглядати після отримання результатів build, inference, security scan і перевірки середовища.

| ID | Ризик | Ймовірність | Вплив | Mitigation / план реагування | Owner |
| :--- | :--- | :--- | :--- | :--- | :--- |
| R01 | Image працює локально, але не готовий до staging через відмінності середовища чи конфігурації | Medium | High | Перевірити запуск у CI та staging-like середовищі; зафіксувати runtime config; погодити staging gate | DevOps |
| R02 | Slim image повертає інший prediction, ніж fat image | Medium | High | Однакові model artifact, preprocessing і test input; автоматизований regression comparison; погоджений tolerance | QA + AI/ML Engineer |
| R03 | Несумісність Python 3.13, PyTorch, torchvision або Pillow | Medium | High | Зафіксувати версії, перевірити сумісність пакетів і наявність wheels, виконати build/smoke test для обох образів | AI/ML Engineer |
| R04 | Зайві файли потрапляють у build context або image | Medium | Medium | Додати й перевірити `.dockerignore`; переглянути `docker history`; виключити кеші, dev-файли та непотрібні datasets | DevOps |
| R05 | README відсутній або команди в ньому не працюють | Medium | Medium | Інший член команди повторює інструкції на чистому checkout; README є acceptance criterion | AI PM + QA |
| R06 | Немає порівняння fat і slim, тому оптимізація не підтверджена даними | Medium | High | Зробити `report.md` обов'язковим deliverable; визначити метрики, методику й owner | AI PM |
| R07 | Production-секрети потрапляють у Compose, build context, image або логи | Low | High | Локальні тестові значення, `.env.example`, виключення `.env` з Git; secret scan і review | DevOps |
| R08 | Локальний стенд підключається до production DB, Redis або queue | Low | High | Локальні сервіси й тестові endpoints; перевірити connection strings; обмежити мережевий доступ за потреби | DevOps |
| R09 | Немає версії або походження model artifact, тому результат неможливо відтворити | Medium | High | Задокументувати джерело, версію, checksum/ID моделі та кероване місце зберігання | AI/ML Engineer |
| R10 | Image завеликий і уповільнює передачу та розгортання | Medium | Medium | Виміряти розмір і шари; прибрати build tools і непотрібні файли; порівняти за однаковою методикою | DevOps |
| R11 | Multi-stage build реалізовано формально, але build-залежності залишилися у фінальному stage | Medium | Medium | Перевірити фінальний stage, копійовані артефакти й `docker history`; пояснити потрібні системні бібліотеки | DevOps |
| R12 | Top-3 classes збігаються, але scores відрізняються через runtime або числову похибку | Medium | Medium | Заздалегідь визначити допуск scores; фіксувати runtime та hardware; дослідити зміну ранжування біля межі top-3 | QA + AI/ML Engineer |
| R13 | Input/output contract не визначений, що робить тестування неоднозначним | Medium | Medium | Зафіксувати формат зображення, preprocessing, response schema та приклади валідного/невалідного input | AI/ML Engineer |
| R14 | Security findings невідомі або не мають відповідального | Medium | High | Сканувати обидва образи одним інструментом і policy; класифікувати findings; призначити remediation owner | DevOps |
| R15 | Compose-стенд вимагає ручного запуску залежностей або не має health checks | Medium | Medium | Перевірити `docker compose up --build` із чистого середовища; додати health checks і end-to-end test | DevOps + QA |

## Пріоритетні дії

1.  Підтвердити model artifact, версію та checksum.
2.  Перевірити сумісність залежностей із Python 3.13 у двох образах.
3.  Автоматизувати порівняння top-3 inference.
4.  Виміряти розмір, час збірки/запуску та security findings.
5.  Переконатися, що локальна конфігурація не має доступу до production data.
6.  Визначити staging gate та відповідальних за відкриті блокери.

## Правило ескалації

Ризик із високим впливом, який може змінити prediction, розкрити production-секрети, 
надати доступ до production data або заблокувати staging, ескалується AI PM відповідальному 
технічному owner і стейкхолдеру до приймання delivery-артефакту.
