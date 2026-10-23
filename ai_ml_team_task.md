# Task: Containerize Image Classification Inference and Compare Fat vs Slim Images

## Background

Потрібно підготувати відтворюваний Docker delivery-пакет для inference
pretrained-моделі класифікації зображень на основі `torchvision.models`.  
Команда надає базовий `ml-infer-fat:1.0` і оптимізований `ml-infer-slim:1.0`.  
Мета оптимізації - зменшити розмір і зайвий вміст образу, зберігши еквівалентну поведінку inference.

## Scope

1.  Підготувати inference на основі погодженого pretrained model artifact.
2.  Створити `Dockerfile.fat` на базі `python:3.13` і `Dockerfile.slim` на базі `python:3.13-slim`.
3.  Для slim image використати multi-stage build; build-залежності, якщо вони потрібні, 
    не мають потрапляти у фінальний runtime stage.
4.  Додати `.dockerignore`.
5.  Зафіксувати версії runtime-залежностей у `requirements.txt` і перевірити їх сумісність із Python 3.13.
6.  Забезпечити передачу тестового зображення або іншого погодженого input.
7.  Повертати top-3 predictions у задокументованому форматі.
8.  Перевірити обидва образи на однаковому model artifact, preprocessing і test input.
9.  Порівняти результати inference та задокументувати метрики.
10. Надати `README.md` з командами build/run/test і `report.md` із порівнянням.

## Out of Scope

-   Model retraining.
-   Покращення accuracy або зміна цільової якості моделі.
-   Production Kubernetes deployment.
-   Full monitoring implementation.
-   Зміна архітектури моделі чи її бізнес-призначення.
-   Production rollout без окремого погодження.

## Expected Deliverables

-   `model/model.pt` або інший погоджений і версіонований model artifact.
-   `app/inference.py`.
-   `requirements.txt` із зафіксованими версіями runtime-залежностей.
-   `Dockerfile.fat`.
-   `Dockerfile.slim` із multi-stage build.
-   `.dockerignore`.
-   `README.md` із командами збірки, запуску та тестування.
-   `report.md` із виміряними результатами порівняння.
-   Логи збірки й запуску обох образів.
-   Результати порівняння top-3 predictions на одному input.
-   Перелік обмежень і відкритих ризиків.

## Technical Constraints

-   Fat image: base image `python:3.13`, тег `ml-infer-fat:1.0`.
-   Slim image: base image `python:3.13-slim`, тег `ml-infer-slim:1.0`.
-   Slim image обов'язково використовує multi-stage build.
-   Обидва образи використовують один model artifact, однаковий preprocessing і test input.
-   Inference повертає top-3 predictions у документованому форматі.
-   Версії залежностей зафіксовані; сумісність Python, PyTorch, torchvision і Pillow підтверджена build/test.
-   Фінальний slim stage містить лише потрібні runtime-файли й бібліотеки.
-   Секрети, локальні кеші, віртуальні середовища та непотрібні файли не потрапляють у build context або image.
-   Команди працюють із чистого checkout відповідно до README.

## Acceptance Criteria Summary

-   [ ] `Dockerfile.fat` використовує `python:3.13`.
-   [ ] `Dockerfile.slim` використовує `python:3.13-slim` і multi-stage build.
-   [ ] Однакові inference для fat і slim.
-   [ ] `requirements.txt` містить зафіксовані сумісні версії.
-   [ ] `.dockerignore` присутній і перевірений.
-   [ ] Обидва образи збираються та запускаються.
-   [ ] Обидва використовують однакові model artifact, preprocessing і test input.
-   [ ] Обидва повертають top-3 predictions у погодженому форматі.
-   [ ] Результати збігаються за заздалегідь погодженим правилом.
-   [ ] README містить перевірені команди build/run/test.
-   [ ] `report.md` містить вимірювання й методику.
-   [ ] Security findings, зайві файли та обмеження задокументовані.

## Evidence to Attach

Додати логи успішної збірки кожного образу, вивід inference для однакового input, 
top-3 comparison, розміри образів, `docker history`, результати security scan та посилання на README/report.
