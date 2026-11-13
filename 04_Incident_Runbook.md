# Incident Runbook: ContractPilot AI

## Мета документа

Runbook визначає порядок реагування на критичні інциденти ContractPilot AI. Він призначений для MLOps, ML Platform, Security, AI Safety, Product Management та Compliance і використовується для локалізації проблем, відновлення сервісу й координації комунікації.

### Ролі та відповідальність

| Role | Responsibility |
|---|---|
| Incident Commander | Координація реагування та фіксація рішень |
| MLOps Lead | Deployment, rollback і відновлення сервісу |
| ML Lead | Якість моделей, evaluation та data drift |
| Security Lead | Розслідування інцидентів безпеки й збереження доказів |
| AI Safety Lead | Prompt injection, hallucinations і safety policies |
| Legal Product Lead | Вплив помилкових юридичних результатів |
| Compliance Lead | GDPR та EU AI Act |
| Customer Support Lead | Повідомлення клієнтів |
| Customer Success Lead | Комунікація з корпоративними клієнтами |
| CPO | Рішення щодо суттєвих бізнес-ризиків |

Incident Commander призначається з числа відповідальних керівників. Для інцидентів безпеки розслідування координує Security Lead.

---

# Scenario 1: Model quality degradation

**Severity:** P1 / P2  
**Expected frequency:** Medium, орієнтовно 1–2 випадки на рік.  
**MTTR target:** до 60 хвилин для відновлення стабільної версії.

### 1. Detection

Основні сигнали:

- Recall <0.90.
- Hallucination rate ≥2.0%.
- PSI свідчить про суттєвий data drift.
- Citation coverage <98%.
- Зростання некоректних юридичних пояснень або скарг клієнтів.

Джерела: Prometheus/Grafana alerts, MLflow evaluation metrics, Loki logs і Customer Support tickets.

**P1:** порушення критичних quality thresholds або підтверджена помилка з потенційними юридичними чи фінансовими наслідками.  
**P2:** погіршення окремих метрик без підтвердженого критичного впливу.

### 2. Initial response (first 15 minutes)

- [ ] **0–3 хв:** ML Lead перевіряє Grafana та підтверджує відхилення Recall, PSI або LLM quality metrics.
- [ ] **3–5 хв:** MLOps Lead визначає Severity, відкриває incident ticket і призначає Incident Commander.
- [ ] **5–10 хв:** MLOps Lead перевіряє активну model version, останній deployment і зупиняє подальший Canary rollout.
- [ ] **10–15 хв:** ML Lead та AI Safety Lead перевіряють evaluation results і RAG quality. Incident Commander створює Slack incident channel та повідомляє відповідальних.

### 3. Diagnosis

ML Lead і MLOps Lead перевіряють:

1. Recall, PSI та інші production metrics у Grafana.
2. Evaluation results і champion baseline у MLflow.
3. Hallucination rate, citation coverage та RAG retrieval results.
4. Останню model version, prompt template і Git commit.
5. Deployment history в Argo CD та помилки в Loki.

Мета: визначити, чи спричинена деградація зміною моделі, даних, prompt або retrieval pipeline.

### 4. Mitigation

| Action | Owner |
|---|---|
| Зупинити Canary rollout | MLOps Lead |
| Повернути traffic на стабільну champion version | MLOps Lead |
| Перевірити model artifacts та evaluation results | ML Lead |
| Відкотити проблемні prompt або RAG parameters | ML Lead / AI Safety Lead |
| Обмежити high-risk сценарії, якщо якість не підтверджена | Legal Product Lead |

Rollback виконується через Argo CD. За порушення hard SLO застосовується автоматичний rollback; ручний rollback погоджує MLOps Lead. Повторний deployment дозволяється після проходження Release Checklist.

### 5. Communication plan

- **Internal:** Incident Commander повідомляє ML Lead, MLOps Lead, AI Safety Lead, Legal Product Lead і CPO про P1 та суттєві зміни статусу.
- **Customer:** Customer Support Lead і Customer Success Lead повідомляють affected customers про проблему, її вплив і відновлення. За наявності юридично значущих помилок повідомлення погоджує Legal Product Lead.
- **Compliance:** Compliance Lead оцінює вплив на GDPR та EU AI Act. Якщо інцидент пов'язаний із витоком персональних даних, застосовується процедура Scenario 2.

### 6. Recovery

Сервіс вважається відновленим, коли:

- Recall ≥0.90;
- Hallucination rate <2.0%;
- Citation coverage ≥98%;
- відсутні критичні помилки в evaluation dataset;
- API availability відповідає SLO 99.9%, p95 latency <4 seconds.

Результати підтверджують ML Lead та AI Safety Lead. Incident Commander погоджує закриття інциденту.

### 7. Post-mortem checklist

- [ ] Blameless post-mortem заплановано протягом 5 днів.
- [ ] Root cause identified and documented.
- [ ] Update Threat Model.
- [ ] Update Release Checklist.
- [ ] Update this Runbook.

### 8. Preventive measures

- Offline evaluation і baseline comparison перед кожним релізом.
- Моніторинг Recall, PSI, hallucination rate та citation coverage.
- Canary deployment і автоматичні quality gates.
- Human review юридично значущих результатів.
- Регулярні adversarial та multilingual tests.

**Related Threat Model:** T-004, T-005, T-011, T-014.

---

# Scenario 2: PII leakage through RAG/LLM

**Severity:** P1  
**Expected frequency:** Low, менше одного підтвердженого випадку на рік.  
**MTTR target:** припинення витоку до 15 хвилин; відновлення affected functionality після перевірки безпеки.

### 1. Detection

Ознаки інциденту:

- Користувач отримує документи або фрагменти іншого tenant.
- OpenSearch повертає результати з неправильним tenant ID.
- DLP виявляє PII у відповіді LLM.
- Виявлено помилку API-side filter або document-level ACL.
- Security alerts фіксують несанкціонований доступ.

Джерела: OpenSearch audit events, Loki, CloudTrail, DLP alerts і повідомлення клієнтів.

Будь-яке підтверджене міжtenant розкриття даних класифікується як P1.

### 2. Initial response (first 15 minutes)

- [ ] **0–3 хв:** Security Lead перевіряє alert або повідомлення та підтверджує ознаки витоку.
- [ ] **3–5 хв:** Security Lead оголошує P1, відкриває incident ticket і призначає Incident Commander.
- [ ] **5–8 хв:** MLOps Lead блокує affected RAG route через API gateway або застосовує default-deny.
- [ ] **8–12 хв:** Security Lead зберігає audit events, request IDs і визначає потенційно affected tenants.
- [ ] **12–15 хв:** Incident Commander створює Slack incident channel та залучає Compliance Lead, Legal Product Lead і Customer Support Lead.

Пріоритет: припинити подальший доступ до чужих даних і зберегти докази.

### 3. Diagnosis

Security Lead, ML Lead і MLOps Lead перевіряють:

1. Чи встановлюється tenant ID на серверній стороні.
2. Чи застосовується API-side filter до кожного OpenSearch request.
3. Чи правильно працюють document-level ACL та default-deny policy.
4. Які документи, embeddings і tenants могли бути affected.
5. Чи потрапили чужі дані до LLM context або response.
6. Чи є ознаки порушення IAM або KMS permissions.

Джерела: OpenSearch audit events, Loki, CloudTrail, DLP та regional S3 audit logs.

### 4. Mitigation

| Action | Owner |
|---|---|
| Заблокувати affected RAG route | MLOps Lead |
| Відкликати скомпрометовані credentials за потреби | Security Lead |
| Виправити API-side filter та ACL | MLOps Lead / ML Lead |
| Визначити scope of exposure | Security Lead |
| Перевірити IAM і KMS permissions | Security Lead |
| Виконати cross-tenant retrieval tests | ML Lead |

RAG functionality залишається обмеженою до підтвердження ізоляції. Перед відновленням необхідно провести 50 тестових спроб міжtenant retrieval: усі 50 мають бути відхилені та зафіксовані в audit logs.

### 5. Communication plan

- **Internal:** Security Lead негайно повідомляє Incident Commander, MLOps Lead, CPO, Legal Product Lead і Compliance Lead. Customer Support Lead отримує підтверджену інформацію для комунікації.
- **Customer:** Customer Support Lead і Customer Success Lead координують повідомлення affected customers після погодження із Security та Compliance. Публічна заява потрібна лише за рішенням CPO та Legal.
- **Compliance:** Compliance Lead оцінює, чи є інцидент personal data breach. Якщо ContractPilot AI виступає Data Controller і breach підлягає повідомленню, supervisory authority повідомляється без невиправданої затримки, за можливості протягом 72 годин після виявлення. Якщо компанія є Data Processor, вона без невиправданої затримки повідомляє відповідного Data Controller. Також оцінюється необхідність повідомлення affected individuals за високого ризику для їхніх прав і свобод.

### 6. Recovery

Відновлення дозволяється після виконання таких умов:

- API-side filter та ACL виправлено;
- tenant ID і document ownership перевірено;
- усі 50 cross-tenant tests успішні;
- потенційно скомпрометовані credentials відкликано;
- Security Lead підтвердив усунення причини витоку;
- Compliance Lead задокументував регуляторні рішення.

Incident Commander погоджує відновлення RAG functionality.

### 7. Post-mortem checklist

- [ ] Blameless post-mortem заплановано протягом 5 днів.
- [ ] Root cause та scope of exposure documented.
- [ ] Update Threat Model.
- [ ] Update Release Checklist.
- [ ] Update this Runbook.

### 8. Preventive measures

- Tenant isolation на рівнях application, retrieval і storage.
- Обов'язкові API-side filters та document-level ACL.
- Automated cross-tenant tests перед кожним релізом.
- IAM least privilege та KMS encryption.
- DLP для виявлення PII.
- Audit events для доступу до документів і retrieval results.
- Зберігання security logs у regional S3 протягом 13 місяців.

**Related Threat Model:** T-006, T-007, T-009, T-017.

---

# Scenario 3: Emergency Canary rollback

**Severity:** P1  
**Expected frequency:** Medium, орієнтовно 1–2 випадки на рік.  
**MTTR target:** до 15 хвилин для повернення до champion version.

### 1. Detection

Інцидент виникає, коли нова model version або application release порушує quality, safety чи availability thresholds під час Canary.

Основні сигнали:

- HTTP 5xx error rate >2%.
- p95 latency >4 seconds.
- Hallucination rate ≥2.0%.
- Recall <0.90.
- Помилки Kubernetes pods або downstream services.
- Порушення tenant isolation чи інших security controls.

Джерела: Prometheus/Grafana, Argo CD, Kubernetes events, Loki та OpenSearch monitoring.

Canary виконується послідовно: **5% → 20% → 50% → 100% eligible low-risk traffic**. Кожен етап триває щонайменше 60 хвилин. High-risk contracts і tenants, які відмовилися від Canary, залишаються на champion version.

### 2. Initial response (first 15 minutes)

- [ ] **0–2 хв:** MLOps Lead перевіряє alert, affected release і поточний Canary stage.
- [ ] **2–5 хв:** MLOps Lead оголошує P1, відкриває incident ticket і перевіряє automatic rollback.
- [ ] **5–8 хв:** Якщо automatic rollback не спрацював, MLOps Lead погоджує та запускає ручний rollback.
- [ ] **8–12 хв:** ML Lead перевіряє champion version і quality metrics; MLOps Lead перевіряє Kubernetes health.
- [ ] **12–15 хв:** Incident Commander створює Slack incident channel та повідомляє CPO і відповідальні команди.

Першочергове завдання: припинити проблемний rollout і відновити стабільну версію.

### 3. Diagnosis

MLOps Lead та ML Lead перевіряють:

1. Canary stage і deployment status в Argo CD.
2. HTTP errors, p95 latency та resource utilization у Grafana.
3. Kubernetes pod status, restarts і events.
4. Model version та evaluation results у MLflow.
5. Останні Git commits, prompt changes і deployment manifests.
6. Доступність OpenSearch та LLM provider API.

Мета: визначити, чи спричинена проблема моделлю, конфігурацією, application release або інфраструктурою.

### 4. Mitigation

| Action | Owner |
|---|---|
| Зупинити Canary rollout | MLOps Lead |
| Перевірити automatic rollback | MLOps Lead |
| Виконати ручний rollback за потреби | MLOps Lead |
| Підтвердити champion model та evaluation results | ML Lead |
| Перевірити Kubernetes pods і service health | MLOps Lead |
| Заблокувати повторний deployment проблемної версії | MLOps Lead |

Rollback виконується через Argo CD у межах погодженого GitOps process. Ручний rollback потребує погодження MLOps Lead. Повторний rollout дозволяється лише після виправлення причини та проходження Release Checklist.

### 5. Communication plan

- **Internal:** Incident Commander повідомляє MLOps Lead, ML Lead, Security Lead та CPO про P1 і результат rollback.
- **Customer:** Customer Support Lead повідомляє affected customers, якщо інцидент спричинив недоступність або обмеження функцій. Публічна заява погоджується CPO та Legal Product Lead.
- **Compliance:** Security Lead і Compliance Lead перевіряють, чи спричинив deployment інцидент безпеки або personal data breach. У разі витоку застосовується процедура Scenario 2, включно з GDPR notification requirements.

### 6. Recovery

Сервіс вважається відновленим, коли:

- активна версія відповідає champion version;
- проблемний Canary зупинено;
- Kubernetes pods перебувають у стані Ready;
- HTTP 5xx error rate ≤2%;
- p95 latency <4 seconds;
- Recall ≥0.90;
- Hallucination rate <2.0%.

MLOps Lead і ML Lead підтверджують стабільність. Incident Commander погоджує закриття інциденту. Повторний Canary потребує нового погодження.

### 7. Post-mortem checklist

- [ ] Blameless post-mortem заплановано протягом 5 днів.
- [ ] Root cause identified and documented.
- [ ] Update Threat Model.
- [ ] Update Release Checklist.
- [ ] Update this Runbook.

### 8. Preventive measures

- Canary rollout із контрольованими етапами та мінімум 60 хвилинами на кожному.
- Автоматичний rollback при порушенні hard SLO.
- Перевірка quality, safety, cost і latency gates.
- Перевірка deployment changes через pull/merge requests.
- Тестування rollback у staging.
- Збереження model version, Git commit і evaluation report.

**Related Threat Model:** T-013, T-014, T-016, T-018.

---

# General incident management rules

## Severity

| Severity | Description |
|---|---|
| P1 | Критичний вплив на безпеку, конфіденційність, доступність або юридично значущі результати |
| P2 | Значна деградація окремих функцій без підтвердженого критичного впливу |
| P3 | Обмежена проблема з незначним впливом на користувачів |

P1 alerts мають бути підтверджені відповідальним інженером менш ніж за 5 хвилин.

## Incident closure

Інцидент закривається після усунення критичного впливу, перевірки стабільності сервісу, повідомлення необхідних stakeholders та збереження доказів. Усі corrective actions повинні мати призначених owners і due dates.

Incident Commander підтверджує закриття. Compliance Lead окремо фіксує виконання регуляторних зобов'язань, якщо вони застосовуються.

## Evidence

Для кожного інциденту зберігаються Incident ID, Severity, час подій, affected services і tenants, відповідальні особи, Grafana dashboards, Loki traces, model version, Git commit, mitigation actions і post-mortem.

Security logs зберігаються в regional S3 протягом 13 місяців відповідно до ADR-004. Доступ до incident records обмежується IAM policies.
