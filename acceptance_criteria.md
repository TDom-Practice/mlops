# Acceptance Criteria

## 1. Docker images

### AC-01: Base image for fat

`Dockerfile.fat` використовує `python:3.13`.  
Зібраний образ має тег `ml-infer-fat:1.0`.  
**Evidence:** Dockerfile і успішний build log.

### AC-02: Base image for slim

`Dockerfile.slim` використовує `python:3.13-slim`.  
Зібраний образ має тег `ml-infer-slim:1.0`.  
**Evidence:** Dockerfile і успішний build log.

### AC-03: Multi-stage build

`Dockerfile.slim` містить окремий build stage і фінальний runtime stage.  
У фінальний stage копіюються лише потрібні застосунку файли та runtime-залежності.  
**Evidence:** Dockerfile і `docker history ml-infer-slim:1.0`.

### AC-04: Build succeeds

Обидва образи збираються з чистого checkout за командами з README.  
**Evidence:** build logs обох тегів.

### AC-05: Runtime succeeds

Обидва образи запускають inference успішно або надають працюючий сервіс відповідно до задокументованого режиму.  
**Evidence:** команди `docker run` та логи запуску.

## 2. Dependencies and build context

### AC-06: Pinned dependencies

`requirements.txt` містить конкретні версії runtime-залежностей.  
Команда підтверджує сумісність із Python 3.13 та обома base images.  
**Evidence:** requirements і build/smoke-test logs.

### AC-07: Docker ignore

У корені build context є `.dockerignore`.  
Він виключає локальні віртуальні середовища, кеші, тимчасові файли, VCS-метадані й 
локальні файли секретів, не виключаючи потрібні артефакти.  
**Evidence:** файл і перевірка build context.

### AC-08: Minimal runtime content

Slim image не містить очевидно зайвих файлів, наприклад тестових наборів, локальних кешів, 
dev-документації, компіляторів або build tools, якщо вони не потрібні в runtime.  
**Evidence:** `docker history`, аналіз файлової системи фінального image.

## 3. Inference equivalence

### AC-09: Same test input

Fat і slim використовують той самий тестовий input, model artifact і preprocessing.  
**Evidence:** test input ID/checksum, model version і команди запуску.

### AC-10: Top-3 predictions

Inference повертає рівно 3 predictions у погодженому форматі, наприклад клас і score для кожного результату.  
**Evidence:** output обох образів.

### AC-11: Equivalent results

Top-3 класи збігаються. Для floating-point scores допустимі `atol`/`rtol` погоджуються з ML Engineer до запуску тесту.  Відмінності ранжування мають бути досліджені й погоджені до приймання.  
**Evidence:** автоматизований diff або тест із явним правилом допуску.

## 4. Documentation and measurement

### AC-12: README

`README.md` містить перевірені команди `build/run/test`, приклад передачі input, очікуваний output і кроки повторення з чистого checkout.  
**Evidence:** успішне виконання інструкцій іншим членом команди.

### AC-13: Comparison report

`report.md` містить порівняння fat і slim.  
Порівнює розмір, кількість шарів, час збірки, час запуску, inference result, security findings і зайві файли.  
Для кожної метрики наведено метод вимірювання; невідомі значення позначені `TBD` з owner і наступною дією.  
**Evidence:** звіт і логи вимірювань.

### AC-14: Model provenance

Задокументовані архітектура, джерело/версія pretrained weights, checksum або ID model artifact, input/output format.  
**Evidence:** README або report.md.

### AC-15: Security and configuration

Обидва образи й конфігурації перевірені на відомі security findings і випадкове включення секретів.  
Critical/high findings усунуті або мають формально погоджений план до staging.  
**Evidence:** scanner report і результат перевірки секретів.

## 5. Definition of Done

-   [ ] Усі обов'язкові файли присутні й заповнені.
-   [ ] Fat і slim images збираються за README.
-   [ ] Обидва запускають inference на однаковому input.
-   [ ] Обидва повертають top-3 predictions, що збігаються за погодженим правилом.
-   [ ] Slim image використовує multi-stage build на `python:3.13-slim`.
-   [ ] Залежності зафіксовані й перевірені на сумісність.
-   [ ] `.dockerignore` присутній і перевірений.
-   [ ] `report.md` містить вимірювання, методику й висновок.
-   [ ] Відкриті питання та ризики мають owner і наступну дію.
-   [ ] Перед staging є достатньо evidence, а blockers відсутні або формально погоджені.

**Практичний тест приймання:** член команди, який не виконував реалізацію, клонує репозиторій, 
виконує README-команди, збирає обидва образи й отримує узгоджений top-3 результат.
