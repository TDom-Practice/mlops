# Lesson 3 PM: Docker delivery для ML-моделі

## Мета завдання

Цей репозиторій містить управлінський пакет AI Project Manager для перевірки контейнеризації pretrained-моделі класифікації зображень.  
Описано вимоги до двох Docker images, критерії приймання, review-інструменти, вимірювання оптимізації, готовність локального
Compose-стенду та delivery-ризики.

Це PM-пакет, а не реалізація моделі. Dockerfile, inference-код, модель і фактичні результати тестування має надати й підтвердити AI/ML-команда.  
Невідомі результати позначені `TBD`.

## Короткий контекст

Команда має підготувати:  
- `ml-infer-fat:1.0` на базі `python:3.13`;  
- `ml-infer-slim:1.0` на базі `python:3.13-slim` із multi-stage build.

Обидва образи використовують той самий model artifact, preprocessing і test input та повертають еквівалентні top-3 predictions.  
Slim image має бути оптимізованим за розміром і вмістом без зміни очікуваної поведінки inference.

## Структура PM-артефактів

| Файл | Призначення | Як використовувати |
| --- | --- | --- |
| [`project_brief.md`](./project_brief.md) | Бізнесовий і технічний контекст | Узгодити мету, scope, середовища й очікуваний результат |
| [`ai_ml_team_task.md`](./ai_ml_team_task.md) | Задача для AI/ML-команди | Перенести в Jira, Linear або GitLab Issues і відстежувати deliverables |
| [`acceptance_criteria.md`](./acceptance_criteria.md) | Критерії приймання та Definition of Done | Використовувати під час QA та приймання |
| [`docker_image_review.md`](./docker_image_review.md) | Технічний PM-review Docker-рішення | Перевіряти Dockerfile, залежності, inference і відтворюваність |
| [`optimization_report_review.md`](./optimization_report_review.md) | Таблиця метрик fat vs slim | Заповнити фактичними даними та evidence |
| [`compose_readiness_checklist.md`](./compose_readiness_checklist.md) | Перевірка локального інтеграційного стенду | Перевірити запуск сервісів і end-to-end prediction |
| [`risk_register.md`](./risk_register.md) | Ризики, mitigation та owners | Переглядати на статусних зустрічах і перед staging gate |
| [`stakeholder_summary.md`](./stakeholder_summary.md) | Короткий звіт для нетехнічного стейкхолдера | Комунікувати статус і умови успіху |
| [`README.md`](./README.md) | Навігація по пакету | Початкова точка для reviewer і команди |

## Рекомендований порядок роботи

1.  Прочитати `project_brief.md` і погодити контекст.
2.  Передати `ai_ml_team_task.md` команді та підтвердити scope.
3.  Зафіксувати критерії в `acceptance_criteria.md`.
4.  Проводити перевірку за `docker_image_review.md`.
5.  Заповнити `optimization_report_review.md` фактичними даними.
6.  Перевірити локальну інтеграційну готовність за `compose_readiness_checklist.md`.
7.  Оновити `risk_register.md` на основі evidence.
8.  Підготувати статус для стейкхолдера в `stakeholder_summary.md`.

## Questions to AI/ML Team

### Model and inference

1.  Який фінальний model artifact використовується і де він зберігається?
2.  Які архітектура, версія, джерело pretrained weights та checksum/ID моделі?
3.  Який input format очікується: тип файлу, колірний формат, розмір і preprocessing?
4.  Який output format повертається та як представляються top-3 predictions?
5.  Який test input є еталонним і як перевіряється його незмінність?
6.  Яке правило порівняння scores погоджено для fat і slim?

### Docker and dependencies

7.  Чи використовує `Dockerfile.fat` базовий образ `python:3.13`?
8.  Чи використовує `Dockerfile.slim` базовий образ `python:3.13-slim`?
9.  Чи реалізовано multi-stage build для slim image?
10. Чи сумісні зафіксовані версії Python, torch, torchvision і Pillow з обома образами?
11. Чи є `.dockerignore` і чи виключає він кеші, локальні середовища та секрети, не виключаючи потрібні файли?
12. Чи немає build tools або dev-only залежностей у фінальному slim runtime image?

### Validation and delivery

13. Чи успішно збираються й запускаються обидва образи?
14. Чи збігається inference для однакового model artifact, preprocessing і test input?
15. Чи є `report.md` з розміром образів, кількістю шарів, часом збірки/запуску та security findings?
16. Чи містить README перевірені команди build/run/test?
17. Чи проскановані образи на відомі вразливості й секрети?
18. Чи потрібен Docker Compose для локального стенду в межах поточного scope?
19. Якщо потрібен Compose, чи описані `model-api`, model runtime, Redis, PostgreSQL і monitoring?
20. Чи гарантовано, що локальний стенд не підключається до production DB, Redis або queue?
21. Які блокери залишаються перед staging і хто є owner кожного з них?

### Security and Docker Compose

22. Чи перевірено compose.yaml або docker-compose.yml на наявність hardcoded secrets, токенів, паролів, API keys і production credentials?
23. Чи використовує Compose змінні середовища та .env замість зберігання секретів безпосередньо у файлі конфігурації? Чи виключено .env із Git?
24. Чи перевірено, що docker compose config не розкриває секрети у виводі, який зберігається в логах або CI artifacts?
25. Чи використовують локальні сервіси окремі тестові credentials і чи виключено доступ до production DB, Redis, queue та інших production-ресурсів?
26. Чи проскановано Docker images і конфігураційні файли на секрети та відомі вразливості? Де зберігаються результати перевірки?
27. Чи визначено owner і порядок усунення виявлених секретів або вразливостей до переходу на staging?

## Команди перевірки Docker

Виконувати з кореня репозиторію після того, як технічна команда додасть
відповідні файли:

``` bash
docker build -f Dockerfile.fat -t ml-infer-fat:1.0 .
docker build -f Dockerfile.slim -t ml-infer-slim:1.0 .
docker images | grep ml-infer
docker history ml-infer-fat:1.0
docker history ml-infer-slim:1.0
```

Команди запуску inference залежать від способу передачі input і режиму контейнера.  
Їх потрібно отримати від команди й внести в README після перевірки.

Якщо Compose входить до scope і файл наявний:

``` bash
docker compose config
docker compose up --build
```

Після старту потрібно виконати документований health check і test prediction.  
Endpoint і payload мають відповідати реалізації сервісу.

## Evidence для приймання

Перед закриттям задачі зберегти або додати посилання на:  
- build logs обох образів;  
- команди запуску та output inference;  
- порівняння top-3 predictions;  
- model artifact ID/checksum;  
- розмір образів і `docker history`;  
- security scan findings;  
- заповнений `report.md`;  
- результати Compose end-to-end test, якщо Compose входить до scope.

## Definition of Done на рівні PM-пакета

-   [ ] Усі дев'ять Markdown-файлів створені й заповнені.
-   [ ] Вимоги узгоджені між brief, task та acceptance criteria.
-   [ ] Невиміряні значення позначені `TBD`, для них визначено owner/наступну дію.
-   [ ] У risk register є щонайменше вісім ризиків із likelihood, impact, mitigation та owner.
-   [ ] Stakeholder summary містить 8--12 речень.
-   [ ] Команди й evidence не вигадані: непідтверджені деталі залишаються відкритими питаннями.
