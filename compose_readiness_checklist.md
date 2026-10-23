# Docker Compose Readiness Checklist

## 1. Мета перевірки

Визначити, чи можна підняти локальний інтеграційний стенд однією командою та виконати 
test prediction без ручного запуску окремих сервісів.  
Цей checklist оцінює Compose-стенд окремо від готовності одиничного Docker image.

**Поточний статус:** `TBD`.  
Наявність Compose-файлу й фактична реалізація сервісів мають бути підтверджені.

## 2. Checklist

  | Перевірка | Очікуваний результат | Статус | Evidence / коментар | Owner |
| --- | --- | --- | --- | --- |
| Є `compose.yaml` або `docker-compose.yml` | Compose-файл присутній | TBD | Шлях до файлу | DevOps |
| Описано `model-api` | Визначені build/image, порт і команда запуску | TBD | Service definition | AI/ML Engineer |
| Описано model service/runtime | Модель запускається в погодженому сервісі/runtime | TBD | Service definition | AI/ML Engineer |
| Описано Redis | Є локальна конфігурація, якщо Redis входить до scope | TBD | Service definition | DevOps |
| Описано PostgreSQL | Є локальна БД з окремим локальним volume/config | TBD | Service definition | DevOps |
| Описано monitoring | Є локальний monitoring service або задокументоване рішення про scope | TBD | Compose/scope decision | DevOps + AI PM |
| Залежності стартують автоматично | Налаштовані service dependencies і health checks, де потрібно | TBD | Config і test log | DevOps |
| Порти налаштовані | Немає конфліктів; API доступний локально | TBD | Compose/README | DevOps |
| Є health endpoint | Endpoint документований і перевіряється | TBD | README і запит | AI/ML Engineer |
| Є predict endpoint | Документовані метод, input schema і response schema | TBD | README і test request | AI/ML Engineer |
| Конфігурація винесена в `.env` | Є безпечні приклади в `.env.example` | TBD | Config files | DevOps |
| Production-секретів немає | У репозиторії, Compose, image і логах немає production credentials | TBD | Secret scan/review | DevOps |
| Немає production DB | Локальна конфігурація використовує локальну БД | TBD | Connection settings | DevOps |
| Немає production Redis/queue | Використовуються локальні endpoints або тестові залежності | TBD | Env/config review | DevOps |
| README містить команду запуску | `docker compose up --build` задокументована | TBD | README | DevOps |
| Є інструкція зупинки | README пояснює зупинку; видалення volumes має попередження про втрату даних | TBD | README | DevOps |
| Test prediction проходить end-to-end | Після старту сервісів prediction виконується без ручного втручання | TBD | End-to-end log | QA |

## 3. Мінімальний сценарій перевірки

Після підтвердження наявності Compose-файлу та команд у README:

``` bash
docker compose config
docker compose up --build
```

В іншому терміналі виконати health check і test prediction командами з
README. Endpoint, порт, HTTP-метод і payload мають відповідати фактичній
реалізації. Після перевірки зупинити стенд за інструкцією README; не
видаляти volumes без розуміння, які дані будуть втрачені.

## 4. Критерій готовності

Локальний стенд готовий, якщо всі погоджені сервіси піднімаються однією
командою, проходять health checks, API приймає валідний test input і
повертає очікуваний prediction без ручного запуску залежностей.

Якщо Compose не входить до поточного scope, це має бути явно зазначено в
рішенні щодо scope і плані delivery. Тоді checklist показує невиконані
вимоги, а не підтверджену готовність.

## 5. Рішення

-   **Ready:** усі обов'язкові пункти пройдені, end-to-end prediction підтверджено.
-   **Conditionally ready:** залишилися погоджені некритичні пункти з owner і строком.
-   **Not ready:** відсутній Compose-файл за погодженого scope, сервіси потребують ручного запуску, 
    health/predict не працюють або є ризик доступу до production data.

**Поточне рішення:** `TBD` до отримання evidence.
