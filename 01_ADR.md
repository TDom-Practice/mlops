# Architectural Decision Records — ContractPilot AI

## ADR-001: Гібридна модель розміщення: Managed LLM та ML у EKS

Status: Accepted  
Date: 2026-11-03  
Decision maker: Senior AI Product Manager  
Stakeholders consulted: CTO, MLOps Lead, ML Lead, CFO, Head of Compliance

## Context

Для аналізу контрактів потрібні якісні юридичні пояснення англійською, німецькою та французькою мовами, а для визначення пріоритетності ризиків (risk prioritization) — швидке оцінювання, незалежне від tenant. За обсягу 5,4M запитів на місяць повністю керована LLM створює ризик непередбачуваних витрат. Водночас self-hosted велика модель потребувала б цілодобової роботи GPU, яку поточна команда з двох MLOps-спеціалістів не може належним чином підтримувати. Клієнтам у ЄС та США потрібні регіональне зберігання даних (data residency), гарантія невикористання prompts для навчання моделей і можливість відстежити походження кожної відповіді (traceability).

## Decision

Використовуємо гібридний підхід (hybrid): commercial LLM API із zero-retention через AWS regional gateway для генерації пояснень; distilled risk-prioritization model розміщуємо на CPU nodes в EKS. Prompts, retrieved passages і outputs зберігаються в регіональному AWS storage, а provider отримує лише контекст, необхідний для обробки запиту. У разі збою provider, а також для обробки low-risk запитів, використовується менша approved fallback model.

## Alternatives considered

1. **Self-host 70B LLM в EKS.** Орієнтовні витрати на GPU compute у Year 1 та додаткові 0,8 MLOps FTE становлять **$1,06M**. Підхід забезпечує більше контролю, але перевищує загальний бюджет програми $1,68M і збільшує ризики, пов'язані з підтримкою необхідної обчислювальної потужності (capacity risk).

2. **Commercial API для LLM і ML.** Орієнтовна вартість сервісу у Year 1 становить **$0,93M**, але спеціалізований ML hosting від vendor додає $0,11 за еквівалентний запит і обмежує можливості відстеження походження ML-артефактів у MLflow.

3. **Hybrid (обрано).** LLM API та ML у EKS: **$0,71M** прямих щорічних витрат на платформу, без урахування розподілених витрат на персонал (allocated personnel). Такий підхід дає змогу відокремити змінні витрати на генерацію від більш передбачуваних витрат на ML serving.

## Consequences

**Positive:**
- Швидший time-to-value завдяки можливості одразу використовувати якісну LLM.
- ML-модель, features та артефакти зберігаються в MLflow і залишаються доступними для аудиту.
- Запланована середня вартість inference становить **$0,0118 за запит**, що відповідає встановленому діапазону $0,001–$0,05.
- Гібридна архітектура дає змогу масштабувати LLM та ML незалежно одне від одного.

**Negative:**
- Зміни цін або доступності API впливають на margin.
- Система має два окремі операційні контури, що збільшує складність підтримки.
- Умови обробки даних у provider потребують регулярної юридичної перевірки.

**Risks:**
- Збій provider може погіршити якість відповідей або вплинути на доступність сервісу.
- Надмірне скорочення запиту може призвести до втрати важливого контексту.
- Заходи з мінімізації ризиків (Mitigation): regional fallback, кешування фрагментів політик (cached policy snippets) і щоквартальний перегляд умов співпраці з vendor.

## Related ADRs

ADR-002, ADR-003.  
Threat Model: T-005, T-010, T-016.

---

## ADR-002: Canary-реліз для змін моделей, промптів і пошуку

Status: Accepted  
Date: 2026-11-03  
Decision maker: Senior AI Product Manager  
Stakeholders consulted: CTO, MLOps Lead, ML Lead, Head of Support, Head of Compliance

## Context

Вересневий incident із renewal clause показав, що навіть технічно успішний deployment може завдати шкоди клієнту. Зміни можуть стосуватися classic ML weights, prompt templates, параметрів ранжування RAG (RAG ranking parameters) і налаштувань LLM provider. Повне перемикання на нову версію (full cutover) не дає змоги обмежити вплив змін до отримання підтверджень їхньої якості в production.

## Decision

Виконуємо реліз через Argo CD із послідовними етапами Canary: **5% → 20% → 50% → 100%** дозволеного low-risk traffic. Кожен етап триває щонайменше 60 хвилин. Перехід до наступного етапу можливий лише після проходження перевірок якості, безпеки, вартості та затримки відповіді (quality, safety, cost і latency gates).

Контракти з високим рівнем ризику та клієнти, які відмовилися від участі в Canary, продовжують працювати на стабільній версії моделі (champion version). Rollback виконується автоматично у разі порушення критичних SLO (hard SLO breaches). Для ручного відкату потрібне погодження MLOps Lead.

## Alternatives considered

1. **Blue-Green.** Орієнтовні додаткові витрати на compute під час релізів становлять +$14K/рік. Повернення до попередньої версії відбувається швидко, але всі клієнти отримують зміни одночасно, а якість юридичних відповідей неможливо безпечно оцінити до повного переходу.

2. **A/B experiment.** Дає змогу якісно порівняти продукт, але передбачає показ клієнтам різних юридичних рекомендацій і може затримати рішення про emergency rollback.

3. **Canary (обрано).** Додає близько $9K/рік витрат на паралельні обчислювальні ресурси (parallel capacity) та 1–2 години до тривалості релізу. Натомість на першому етапі вплив змін обмежується 5% дозволеного traffic.

## Consequences

**Positive:**
- Обмежений blast radius завдяки поетапному збільшенню частки traffic.
- Можливість оцінити якість змін у production до повного релізу.
- Git commit, evaluation report і rollout events формують доказову базу для аудиту.
- Rollback до champion виконується менш ніж за 15 хвилин.

**Negative:**
- Довший цикл релізу через обов'язкові перевірки на кожному етапі.
- Маршрутизація traffic та виключення окремих груп клієнтів (cohort exclusions) додають операційної складності.
- Паралельна робота версій потребує додаткових обчислювальних ресурсів.

**Risks:**
- Навіть у cohort 5% може трапитися рідкісний випадок із негативними наслідками для клієнта.
- Затримка надходження метрик може відтермінувати виявлення проблеми.
- Заходи з мінімізації ризиків (Mitigation): adversarial eval до релізу, перевірка вибірки відповідей людиною (human review of sampled answers) і kill switch.
- Жоден Canary не переходить до наступного етапу, якщо hallucination rate становить ≥2,0%.

## Related ADRs

ADR-001, ADR-004.  
Threat Model: T-004, T-008, T-014.

---

## ADR-003: MLflow Registry із підписаними артефактами та етапами погодження

Status: Accepted  
Date: 2026-11-03  
Decision maker: Senior AI Product Manager  
Stakeholders consulted: MLOps Lead, ML Lead, Security Lead, Head of Compliance

## Context

Оцінки risk-prioritization визначають, які контракти першими передаються на юридичну перевірку. Компанія має забезпечити відтворюваність результатів, прозоре походження навчальних даних і контроль доступу до EKS, щоб туди не потрапляли непогоджені артефакти. Наявних журналів експериментів недостатньо для повного відтворення результатів і проведення аудиту.

## Decision

Використовуємо self-hosted **MLflow Model Registry** в AWS із версіонованими артефактами в S3, ролями IAM за принципом найменших привілеїв (least privilege), підписаними model images і обов'язковим погодженням двох відповідальних осіб (ML Lead + MLOps Lead) перед переходом моделі на наступний етап.

Argo CD розгортає лише версію з Registry зі статусом `Production`, до якої додано model card, evaluation report і SBOM.

## Alternatives considered

1. **SageMaker Model Registry.** Орієнтовні додаткові витрати на managed service становлять $38K/рік. Рішення потребує менше операційної роботи, але дублює вже наявний робочий процес у MLflow.

2. **Лише Git tags і S3.** Приблизно на $8K/рік дешевше, але не забезпечує статусів погодження, метаданих моделі та зручного порівняння історії версій.

3. **MLflow Registry (обрано).** $22K/рік на інфраструктуру та обслуговування. Рішення сумісне з поточним процесом відстеження експериментів і забезпечує контрольоване просування моделей та необхідні дані для аудиту.

## Consequences

**Positive:**
- Відтворюваність моделей і можливість відстежити походження кожного артефакту.
- Чітко визначена версія моделі для rollback.
- Наявність підтверджувальних матеріалів для пунктів Release Checklist Q1–Q6.
- Для всіх моделей, розгорнутих у production, зберігаються відомості про походження, підписані артефакти та затверджені model cards.

**Negative:**
- Self-hosted service потребує регулярного встановлення оновлень і тестування резервних копій.
- Promotion вимагає двох погоджень і може затримати термінові зміни.
- Підтримка Registry потребує окремих операційних ресурсів.

**Risks:**
- Неправомірне використання привілеїв Registry або втручання в артефакти.
- Заходи з мінімізації ризиків (Mitigation): розмежування прав IAM, CloudTrail logs, S3 Object Lock на 90 днів і щоквартальна перевірка доступів (access review).
- Щоквартальна перевірка відновлення має завершуватися протягом 4 годин.

## Related ADRs

ADR-002, ADR-004.  
Threat Model: T-013, T-015.

---

## ADR-004: Prometheus, Grafana та Loki як система моніторингу

Status: Accepted  
Date: 2026-11-03  
Decision maker: Senior AI Product Manager  
Stakeholders consulted: MLOps Lead, VP Engineering, Head of Support, CFO

## Context

ContractPilot уже використовує Prometheus/Grafana/Loki в EKS. Для управління платформою потрібен єдиний операційний огляд стану сервісу, якості моделі, показників безпеки (safety signals), стану релізів і витрат. Використання окремого monitoring product призвело б до дублювання instrumentation.

## Decision

Залишаємо наявний стек Prometheus/Grafana/Loki, доповнивши його надходженням evaluation data з MLflow і даними про витрати AWS.

Створюємо alerts для p95 latency, error rate, drift, quality lag, hallucination rate, prompt-injection blocks, retrieval tenant-deny events і Canary health. Логи, важливі для безпеки, зберігаємо протягом 13 місяців у regional S3.

## Alternatives considered

1. **Datadog-only.** Прискорює налаштування завдяки managed service, але за поточного обсягу logs коштуватиме орієнтовно **$96K/рік**. Додатково знадобиться перенесення наявних dashboards.

2. **Mixed Datadog + current stack.** Орієнтовно $54K/рік додаткових витрат, але дублює alerts і ускладнює розподіл відповідальності під час incident.

3. **Existing stack extended (обрано).** **$31K/рік** на інструменти та зберігання даних (tools and storage allocation). Для підтримки observability потрібно 0,2 FTE.

## Consequences

**Positive:**
- Єдине достовірне джерело інформації про incident і нижчий TCO.
- Dashboards пов'язують технічний стан платформи з business gates.
- P1 alerts мають підтверджуватися менш ніж за 5 хвилин.
- 95% alerts мають потребувати конкретних дій.
- Усі інциденти в production мають пов'язаний Loki trace і посилання на відповідний Grafana dashboard.

**Negative:**
- Роботи з instrumentation мають бути завершені до релізу.
- Quality labels надходять із затримкою.
- Для підтримки observability потрібні ресурси команди MLOps.

**Risks:**
- Втома команди від надмірної кількості alerts може призвести до ігнорування важливих сповіщень.
- Semantic safety signals можуть залишитися непоміченими за недостатньої якості evaluation data.
- Заходи з мінімізації ризиків (Mitigation): severity policy, щомісячний перегляд alerts і human evaluation на вибірці.

## Related ADRs

ADR-002, ADR-003.  
Threat Model: T-004, T-009, T-017.

---

## ADR-005: OpenSearch для зберігання векторів із ізоляцією tenant

Status: Accepted  
Date: 2026-11-03  
Decision maker: Senior AI Product Manager  
Stakeholders consulted: CTO, MLOps Lead, Security Lead, Head of Compliance

## Context

RAG використовує customer playbooks, виконані контракти та clause libraries. Розкриття даних одного клієнта іншому (cross-tenant disclosure) є неприпустимим з огляду на договірні зобов'язання та GDPR. Очікуваний обсяг у 4,2M vectors має забезпечувати пошук у регіонах ЄС та США, а також підтримувати процедури видалення даних.

## Decision

Використовуємо Amazon OpenSearch Service з окремими regional domains, tenant namespace, обов'язковим API-side filter, метаданими контролю доступу на рівні документів (document-level ACL metadata), KMS encryption і щоденними тестами відновлення (nightly restore tests).

Retrieval повертає citation IDs, а рішення щодо доступу фіксуються в логах. LLM не має прямого доступу до OpenSearch: усі запити проходять через API.

## Alternatives considered

1. **Pinecone.** Орієнтовна вартість становить $142K/рік за запланованого обсягу. Рішення має розвинені можливості роботи з векторами, але додає окремого постачальника та додатковий шлях передавання даних.

2. **pgvector на RDS.** Орієнтовна вартість становить $88K/рік. Рішення простіше в експлуатації, але за обсягу понад 4,2M vectors створює ризики щодо повноти результатів пошуку (recall) і затримки відповіді (latency).

3. **OpenSearch (обрано).** Орієнтовно $118K/рік разом із backup. Рішення інтегрується з AWS IAM, підтримує регіональне зберігання даних і відповідає наявним компетенціям команди з експлуатації AWS.

## Consequences

**Positive:**
- Ізоляцію tenant можна перевірити ще до генерації відповіді.
- Процедури видалення даних і аудиту залишаються в AWS.
- Цільовий p95 retrieval — <350 ms.
- Доступність retrieval має становити 99,9%.

**Negative:**
- Налаштування індексів (index tuning) і планування shard потребують часу спеціалістів.
- API-side filter є критично важливим механізмом контролю доступу.
- Підтримка regional domains і backup потребує додаткових операційних ресурсів.

**Risks:**
- Помилка у filter або застарілі ACL metadata можуть відкрити доступ до чужих даних. Такий інцидент класифікується як P1.
- Заходи з мінімізації ризиків (Mitigation): тестування deny cases під час кожного релізу та використання default-deny gateway.
- Видалення документа має бути підтверджене для index і відповідно до backup lifecycle протягом 30 днів.

## Related ADRs

ADR-001, ADR-004.  
Threat Model: T-006, T-007, T-018.
