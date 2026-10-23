# Docker Image Review: PM Checklist

**Проєкт:** image classification inference  
**Образи:** `ml-infer-fat:1.0`, `ml-infer-slim:1.0`  
**Статус:** неперевірені пункти позначати `TBD`, а не вважати
пройденими.

## 1. Dockerfile

| Перевірка                        | Очікуваний результат                                                   | Статус | Evidence / коментар                    | Owner          |
|:---------------------------------|------------------------------------------------------------------------|:-------|:---------------------------------------|:---------------|
| Є `Dockerfile.fat`               | Файл присутній і збирає fat image                                      | TBD    | Файл/build log                         | AI/ML Engineer |
| Є `Dockerfile.slim`              | Файл присутній і збирає slim image                                     | TBD    | Файл/build log                         | AI/ML Engineer |
| Base image fat                   | `python:3.13`                                                          | TBD    | Перевірити `FROM`                      | AI/ML Engineer |
| Base image slim                  | `python:3.13-slim`                                                     | TBD    | Перевірити всі stages                  | AI/ML Engineer |
| Multi-stage build                | Окремі build і runtime stages                                          | TBD    | Перевірити `FROM ... AS` та копіювання | DevOps         |
| Логічний порядок `COPY`          | Залежності копіюються окремо від коду там, де це допомагає build cache | TBD    | Dockerfile і cache behavior            | DevOps         |
| Build tools відокремлені         | Непотрібні build tools не потрапляють у фінальний stage                | TBD    | `docker history`                       | DevOps         |
| Runtime command визначений       | Є документований спосіб запуску                                        | TBD    | `CMD`/`ENTRYPOINT`, README             | AI/ML Engineer |
| `.dockerignore` використовується | Файл присутній у корені build context і виключає непотрібне            | TBD    | Файл і build context                   | DevOps         |

## 2. Залежності

| Перевірка                        | Очікуваний результат                                                            | Статус | Evidence / коментар            | Owner          |
|:---------------------------------|---------------------------------------------------------------------------------|:-------|:-------------------------------|:---------------|
| `requirements.txt` присутній     | Runtime-залежності перелічені                                                   | TBD    | Файл                           | AI/ML Engineer |
| Версії зафіксовані               | Версії PyTorch, torchvision, Pillow та інших пакетів задані явно                | TBD    | Файл                           | AI/ML Engineer |
| Сумісність із Python 3.13        | Встановлення й smoke test успішні для обох образів                              | TBD    | Build/test logs                | AI/ML Engineer |
| Немає зайвих бібліотек           | Кожна runtime-залежність має призначення                                        | TBD    | Dependency review              | AI/ML Engineer |
| Dev-only залежності відокремлені | Лінтери, тестові інструменти й build tools не потрапляють у runtime без потреби | TBD    | Requirements і фінальний image | DevOps         |
| Джерело пакетів зрозуміле        | Використані package indexes і версії задокументовані                            | TBD    | README/build logs              | DevOps         |

## 3. Запуск inference

| Перевірка                      | Очікуваний результат                                 | Статус | Evidence / коментар | Owner          |
|:-------------------------------|:-----------------------------------------------------|:-------|:--------------------|:---------------|
| Fat image збирається           | Build завершується з кодом 0                         | TBD    | Build log           | AI/ML Engineer |
| Slim image збирається          | Build завершується з кодом 0                         | TBD    | Build log           | AI/ML Engineer |
| Fat image запускається         | Inference або сервіс стартує успішно                 | TBD    | Run log             | QA             |
| Slim image запускається        | Inference або сервіс стартує успішно                 | TBD    | Run log             | QA             |
| Можна передати test input      | Спосіб передачі input задокументований               | TBD    | README-команда      | AI/ML Engineer |
| Top-3 predictions повертаються | Є три predictions у погодженому форматі              | TBD    | Output capture      | QA             |
| Результати еквівалентні        | Класи збігаються, scores проходять погоджений допуск | TBD    | Comparison test     | QA             |
| README містить команди         | Build/run/test перевірені на чистому checkout        | TBD    | Повторний запуск    | AI PM + QA     |

## 4. Відтворюваність

| Перевірка                       | Очікуваний результат                                       | Статус | Evidence / коментар     | Owner          |
|:--------------------------------|:-----------------------------------------------------------|:-------|:------------------------|:---------------|
| Один model artifact             | Обидва образи використовують однаковий файл моделі         | TBD    | SHA-256 або artifact ID | AI/ML Engineer |
| Однаковий preprocessing         | Resize, crop, normalization та інші трансформації однакові | TBD    | Код/config review       | AI/ML Engineer |
| Однаковий test input            | Ідентичний файл або стабільний ID input                    | TBD    | Файл/checksum           | QA             |
| Однаковий input/output contract | Формати задокументовані й дотримані                        | TBD    | README/test logs        | AI/ML Engineer |
| Inference збігається            | Результати проходять автоматизоване порівняння             | TBD    | Test report             | QA             |
| Джерело моделі задокументоване  | Є архітектура, джерело/версія ваг і checksum               | TBD    | README/report.md        | AI/ML Engineer |

## 5. Рекомендовані команди

Виконувати з кореня репозиторію після підтвердження назв файлів і build
context:

``` bash
docker build -f Dockerfile.fat -t ml-infer-fat:1.0 .
docker build -f Dockerfile.slim -t ml-infer-slim:1.0 .
docker images | grep ml-infer
docker history ml-infer-fat:1.0
docker history ml-infer-slim:1.0
```

Команди запуску залежать від того, чи inference реалізований як
одноразова CLI-команда, чи як HTTP-сервіс. Команда має надати перевірені
команди для обох образів у README. Endpoint і формат аргументів не слід
припускати до підтвердження реалізації.

## 6. PM review outcome

- **Pass:** усі обов’язкові перевірки пройдені та мають evidence.
- **Conditional pass:** залишилися некритичні відкриті пункти з owner,
  строком і погодженим планом.
- **Fail / Blocked:** збірка або inference не працює, результати
  відрізняються без пояснення, є критичні security findings або бракує
  evidence.

**Рішення щодо staging:** `TBD` до завершення перевірок. Локальний
успішний запуск сам по собі не є достатньою підставою для переходу до
staging.
