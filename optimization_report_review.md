# Optimization Report Review: Fat vs Slim

## 1. Мета

Перевірити, чи зменшився розмір Docker image після оптимізації та чи збереглася поведінка inference.  
Фактичні вимірювання надають AI/ML-команда або DevOps; PM перевіряє методику, докази та висновок.

## 2. Таблиця порівняння

| Метрика                 | Fat: `ml-infer-fat:1.0` | Slim: `ml-infer-slim:1.0` | Коментар AI PM / evidence                                            |
|:------------------------|:------------------------|:--------------------------|:---------------------------------------------------------------------|
| Розмір image            | TBD                     | TBD                       | DevOps: `docker images`; зафіксувати дату, Docker environment і теги |
| Кількість шарів         | TBD                     | TBD                       | DevOps: `docker history <image>`; уточнити метод підрахунку          |
| Час збірки              | TBD                     | TBD                       | DevOps: виміряти в зіставних умовах; зазначити cache on/off          |
| Час запуску             | TBD                     | TBD                       | QA/DevOps: однакове середовище; визначити cold start і готовність    |
| Час inference           | TBD                     | TBD                       | QA: однаковий input; відокремити час старту від prediction           |
| Inference result        | TBD                     | TBD                       | QA: top-3 класи збігаються, scores проходять погоджений допуск       |
| Security findings       | TBD                     | TBD                       | DevOps/Security: той самий scanner і policy, severity зафіксована    |
| Зайві файли             | TBD                     | TBD                       | DevOps: фінальна файлова система, `docker history`, build context    |
| Model artifact checksum | TBD                     | TBD                       | AI/ML Engineer: підтвердити однаковий artifact                       |
| Python/library versions | TBD                     | TBD                       | AI/ML Engineer: порівняти runtime-залежності й пояснити відмінності  |

## 3. Команди збору даних

``` bash
docker images | grep ml-infer
docker history ml-infer-fat:1.0
docker history ml-infer-slim:1.0
```

Додатково команда надає build logs, команди й логи запуску, top-3 output
для одного input, автоматизоване порівняння prediction, security scan та
аналіз фінального slim image. Команди запуску й сканування залежать від
реалізації та обраних інструментів, тому їх потрібно зафіксувати в
README після підтвердження командою.

## 4. Правила коректного порівняння

1.  Порівнювати точні версії образів за тегами й, за можливості, image
    digest.
2.  Використовувати однаковий model artifact, підтверджений checksum.
3.  Використовувати один test input і однаковий preprocessing.
4.  Проводити вимірювання на тому самому хості або в еквівалентному
    середовищі.
5.  Для build time фіксувати, чи використовувався build cache.
6.  Для startup time визначити конкретну подію завершення: запуск
    процесу, готовність health endpoint або завершення одноразового
    inference.
7.  Для prediction scores до тесту погодити `atol`/`rtol`. Top-3 класи
    мають збігатися, якщо інша поведінка окремо не погоджена.
8.  Security findings порівнювати одним інструментом, його версією та
    однаковою policy.

## 5. Висновок AI PM

На момент підготовки шаблону фактичні вимірювання відсутні, тому
висновок попередній.

- **Чи є оптимізація вимірюваною?** Так, якщо команда заповнить таблицю
  числовими значеннями, вкаже методику й додасть логи. Поточний статус:
  `TBD`.
- **Чи зменшився розмір image?** `TBD`, потрібні результати
  `docker images` для обох тегів.
- **Чи зберігся inference?** `TBD`, потрібні однаковий input, top-3
  output і результат автоматизованого порівняння.
- **Чи є ризик, що оптимізація зламала модель?** Так, до завершення
  regression test ризик залишається відкритим. Його знижують однаковий
  artifact, preprocessing і тест.
- **Чи достатньо даних для staging?** Наразі ні, оскільки метрики й
  результати перевірок не надані. Рішення переглядається після
  заповнення таблиці, security review і закриття staging blockers.

## 6. Дії перед закриттям звіту

| Дія                         | Відповідальний      | Evidence                | Статус |
|:----------------------------|:--------------------|:------------------------|:-------|
| Надати розмір обох образів  | DevOps              | `docker images`         | TBD    |
| Надати build/run timing     | DevOps              | Логи й методика         | TBD    |
| Надати top-3 comparison     | QA + AI/ML Engineer | Output і тест           | TBD    |
| Підтвердити checksum моделі | AI/ML Engineer      | SHA-256 або artifact ID | TBD    |
| Надати security scan        | DevOps / Security   | Звіт scanner            | TBD    |
| Перевірити зайві файли slim | DevOps              | Аналіз фінального image | TBD    |
