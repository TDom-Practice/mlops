# Чекліст релізу — ContractPilot AI

**Призначення:** чекліст використовується перед кожним релізом нової ML-моделі або зміною конфігурації prompt, LLM-провайдера, логіки retrieval чи vector index у production. Кожен пункт перевіряється перед початком Canary або переходом до наступного етапу розгортання.

Для кожного пункту фіксується результат перевірки: `Pass`, `Fail` або `N/A`.  
Статус `N/A` має супроводжуватися поясненням і погодженням відповідального.  
Для пунктів із `Blocker if failed: Yes` невиконання перевірки блокує реліз або його подальше просування.  
Evidence додається до відповідного change request.

## 1. Model quality gates

- [ ] **Q1.** Recall моделі для пріоритизації ризиків на зафіксованому holdout dataset становить ≥0,90 і не нижчий за показник поточної champion model більш ніж на 1 процентний пункт.  
  Owner: ML Lead  
  Evidence: MLflow comparison report, підписаний evaluation report  
  Blocker if failed: Yes

- [ ] **Q2.** Precision для класу P1-ризиків становить ≥0,82 за результатами оцінювання на наборі щонайменше з 10 000 розмічених прикладів.  
  Owner: ML Lead  
  Evidence: MLflow run ID, версія evaluation dataset  
  Blocker if failed: Yes

- [ ] **Q3.** PSI для 10 ключових features становить <0,20 порівняно з production baseline.  
  Owner: Data Scientist  
  Evidence: експорт Grafana dashboard із метриками data drift  
  Blocker if failed: Yes

- [ ] **Q4.** Різниця між error rate для сегментів EN/DE/FR і груп постачальників різного розміру та загальним показником не перевищує 5 процентних пунктів. p95 latency API response — <4 секунди.  
  Owner: ML Lead  
  Evidence: fairness evaluation report із результатами за сегментами  
  Blocker if failed: Yes

- [ ] **Q5.** Hallucination rate на evaluation dataset із 1 000 прикладів, для яких обов'язкове посилання на джерело, становить <2,0% (поточний baseline: 3,1%).  
  Owner: AI Safety Lead  
  Evidence: evaluation scorecard, вибірка перевірених результатів reviewer-а  
  Blocker if failed: Yes

- [ ] **Q6.** Citation coverage для юридично значущих тверджень становить ≥98%, а частка таких тверджень без підтвердження джерелами — <1%.  
  Owner: AI Safety Lead  
  Evidence: RAG evaluation report  
  Blocker if failed: Yes

- [ ] **Q7.** На наборі тестів prompt injection модель блокує ≥95% атак, а 100% атак із високим рівнем критичності переводяться в режим abstain або передаються на додаткову перевірку (escalation).  
  Owner: AI Safety Lead  
  Evidence: red-team testing report  
  Blocker if failed: Yes

- [ ] **Q8.** Refusal accuracy становить ≥95% на evaluation dataset із 1 000 перевірених прикладів. Модель коректно відмовляє на заборонені запити та відповідає на дозволені без необґрунтованих відмов.  
  Owner: AI Safety Lead  
  Evidence: MLflow evaluation report із розподілом результатів за типами запитів  
  Blocker if failed: Yes

## 2. Data readiness

- [ ] **D1.** Для навчального dataset зафіксовано версію, походження даних (data lineage) та кількість записів. Метадані зареєстровано в MLflow.  
  Owner: Data Engineer  
  Evidence: посилання на dataset і його версію в MLflow  
  Blocker if failed: Yes

- [ ] **D2.** Усі прямі PII-ідентифікатори в навчальному dataset видалено або замасковано. DLP-сканування не виявляє незатверджених персональних даних.  
  Owner: Data Engineer  
  Evidence: DLP scan report  
  Blocker if failed: Yes

- [ ] **D3.** Schema validation успішно проходить для 100% обов'язкових полів вхідних даних. Breaking changes мають окрему версію та погоджені до використання.  
  Owner: Backend Lead  
  Evidence: CI schema validation report  
  Blocker if failed: Yes

- [ ] **D4.** Порівняно розподіли навчальних і production-даних. Відхилення понад 20% для ключових ознак мають документоване пояснення та погодження відповідального.  
  Owner: Data Scientist  
  Evidence: звіт порівняння розподілів даних  
  Blocker if failed: Yes

- [ ] **D5.** Усі проіндексовані RAG-документи мають tenant ID, ACL і retention class. Перевірка охоплює 100% записів у vector index.  
  Owner: Data Engineer  
  Evidence: результат автоматизованої перевірки OpenSearch index  
  Blocker if failed: Yes

- [ ] **D6.** Тест видалення документа підтверджує, що після deletion request документ зникає з результатів retrieval протягом 24 годин.  
  Owner: Privacy Engineer  
  Evidence: deletion test report і пов'язаний ticket  
  Blocker if failed: Yes

## 3. Infrastructure readiness

- [ ] **I1.** Для EKS визначено CPU/memory requests і limits. Load testing підтверджує, що за прогнозованого пікового навантаження використання ресурсів залишається нижчим за 70%.  
  Owner: MLOps Lead  
  Evidence: load testing report із метриками використання ресурсів  
  Blocker if failed: Yes

- [ ] **I2.** HPA успішно масштабує ML serving від 3 до 12 replicas, а p95 latency ML serving під час тестування залишається <250 ms.  
  Owner: MLOps Engineer  
  Evidence: HPA test report і Grafana dashboard  
  Blocker if failed: Yes

- [ ] **I3.** У Grafana доступні dashboards для моніторингу latency, error rate, якості моделі, data drift, безпеки та вартості одного запиту.  
  Owner: MLOps Engineer  
  Evidence: посилання на Grafana dashboards  
  Blocker if failed: Yes

- [ ] **I4.** P1 alerts надходять черговому інженеру, якщо error rate перевищує 2%, p95 latency — 4 секунди або hallucination rate досягає або перевищує 2%.  
  Owner: MLOps Engineer  
  Evidence: результати тестування alerts і запис у системі моніторингу  
  Blocker if failed: Yes

- [ ] **I5.** Nightly backup для OpenSearch успішно створюється. Тестове відновлення vector index після збою завершується не довше ніж за 4 години.  
  Owner: MLOps Lead  
  Evidence: backup job status і restore testing report  
  Blocker if failed: Yes

- [ ] **I6.** Canary routing виключає контракти з високим рівнем ризику та tenants, які відмовилися від участі в Canary. Тестова cohort становить рівно 5% eligible traffic.  
  Owner: Backend Lead  
  Evidence: результати тестування policy-as-code  
  Blocker if failed: Yes

- [ ] **I7.** Fallback model і механізм повернення кешованої policy-відповіді успішно проходять simulated failure LLM-провайдера.  
  Owner: MLOps Lead  
  Evidence: chaos testing report  
  Blocker if failed: No

## 4. Security & compliance

- [ ] **S1.** Threat Model переглянуто з урахуванням змін у релізі. Для цієї зміни немає невирішених P1-ризиків без погоджених заходів зниження ризику.  
  Owner: Security Lead  
  Evidence: підписаний результат Threat Model review у change request  
  Blocker if failed: Yes

- [ ] **S2.** Tenant isolation test виконує 50 спроб міжтенантного retrieval. Усі 50 спроб доступу до даних іншого tenant відхиляються та реєструються в audit logs.  
  Owner: Security Lead  
  Evidence: результати автоматизованих тестів tenant isolation  
  Blocker if failed: Yes

- [ ] **S3.** IAM access review підтверджує дотримання least privilege. Жоден користувач не має одночасно прав на підвищення статусу моделі в MLflow Model Registry та деплой у production.  
  Owner: Security Lead  
  Evidence: звіт IAM access review  
  Blocker if failed: Yes

- [ ] **S4.** TLS encryption in transit і KMS encryption at rest перевірено для всіх нових або змінених сховищ і черг повідомлень.  
  Owner: Cloud Security Engineer  
  Evidence: IaC security scan report  
  Blocker if failed: Yes

- [ ] **S5.** SBOM і vulnerability scanning не виявляють незатверджених Critical або High CVE у release images.  
  Owner: MLOps Engineer  
  Evidence: CI security scan report і SBOM  
  Blocker if failed: Yes

- [ ] **S6.** Audit events містять request ID, pseudonymized tenant ID, ідентифікатори prompt і моделі, версію моделі, результати retrieval та security outcome.  
  Owner: Backend Lead  
  Evidence: вибіркова перевірка Loki trace та audit events  
  Blocker if failed: Yes

- [ ] **S7.** Оновлено технічну документацію відповідно до застосовних вимог EU AI Act, model card та опис механізмів human oversight.  
  Owner: Compliance Lead  
  Evidence: посилання на оновлену документацію та підтвердження погодження  
  Blocker if failed: Yes

## 5. Operational readiness

- [ ] **O1.** `04_Incident_Runbook.md` переглянуто та оновлено з урахуванням змін у релізі. Для кожного критичного сценарію визначено відповідального, а on-call roster підтверджено.  
  Owner: MLOps Lead  
  Evidence: оновлений runbook і підтверджений on-call roster  
  Blocker if failed: Yes

- [ ] **O2.** Rollback до попередньої версії через Argo CD успішно протестовано у staging-середовищі. Час від початку rollback до відновлення попередньої версії становить <15 хвилин.  
  Owner: MLOps Engineer  
  Evidence: rollback test report із часовими мітками  
  Blocker if failed: Yes

- [ ] **O3.** Canary gate протестовано на 5% трафіку. Automatic rollback спрацьовує під час перевищення налаштованих порогів якості або помилок.  
  Owner: MLOps Engineer  
  Evidence: synthetic gate test report  
  Blocker if failed: Yes

- [ ] **O4.** Опубліковано SLO: доступність API — 99,9%, p95 latency відповіді — <4 секунд, Recall моделі для пріоритизації ризиків — ≥0,90.  
  Owner: Senior AI PM  
  Evidence: затверджений release brief із SLO  
  Blocker if failed: Yes

- [ ] **O5.** Підготовлено support macros і текст для customer status page на випадок інцидентів, пов'язаних із якістю моделі, даними або доступністю LLM-провайдера.  
  Owner: Customer Support Lead  
  Evidence: погоджені шаблони комунікації  
  Blocker if failed: No

- [ ] **O6.** Cost dashboard підтверджує, що прогнозована пряма змінна вартість одного запиту за очікуваного навантаження не перевищує $0,0125.  
  Owner: FinOps Analyst  
  Evidence: знімок cost dashboard із прогнозом витрат  
  Blocker if failed: No

## 6. Business readiness

- [ ] **B1.** Senior AI PM погодив success metrics, склад eligible customer cohort і критерії завершення Canary (Canary exit criteria).  
  Owner: Senior AI PM  
  Evidence: погоджений change request  
  Blocker if failed: Yes

- [ ] **B2.** CPO або уповноважений Product Director погодив оцінку впливу релізу на клієнтів і визначив допустимий рівень ризику для запуску.  
  Owner: CPO  
  Evidence: approval comment у change request  
  Blocker if failed: Yes

- [ ] **B3.** Compliance Lead перевірив необхідність оновлення Data Protection Impact Assessment (DPIA), якщо зміни впливають на обробку персональних даних.  
  Owner: Compliance Lead  
  Evidence: задокументоване рішення щодо DPIA  
  Blocker if failed: Yes

- [ ] **B4.** Зафіксовано baseline бізнес-метрик: середній час перевірки договору — 42 хвилини, conversion rate розширення співпраці — 18%, кількість звернень до підтримки на один запит — 0,08.  
  Owner: Product Analyst  
  Evidence: знімок dashboard бізнес-метрик із датою фіксації baseline  
  Blocker if failed: No

- [ ] **B5.** Визначених design partners повідомлено про вікно релізу щонайменше за 5 робочих днів до переходу Canary на етап 20%.  
  Owner: Customer Success Lead  
  Evidence: журнал клієнтських повідомлень  
  Blocker if failed: No

- [ ] **B6.** Навчальні матеріали пояснюють, що результати ContractPilot AI мають рекомендаційний характер, потребують перевірки юристом і повинні містити посилання на джерела.  
  Owner: Product Enablement Lead  
  Evidence: опублікована сторінка Help Center або навчальний матеріал  
  Blocker if failed: No

## Sign-off

| Role | Ім’я / дата | Рішення |
|---|---|---|
| ML Lead |  |  |
| MLOps Lead |  |  |
| Security Lead |  |  |
| Compliance Lead |  |  |
| Senior AI PM |  |  |
