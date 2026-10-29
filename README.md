# Customer Churn Prediction — MLflow та Production Readiness

## Короткий опис задачі

Це навчальний кейс для аналізу експериментів і моніторингу ML-моделей з позиції менеджера AI/ML-проєкту.  
Мета проєкту - оцінити експерименти з навчання моделей машинного навчання для прогнозування відтоку клієнтів (*Customer Churn Prediction*) та визначити, чи готова найкраща модель до переходу на наступний етап розгортання.

Для порівняння розглянуто п’ять запусків моделі в MLflow. Основна увага приділяється якості прогнозування, відтворюваності експериментів, наявності артефактів і готовності процесу до production.

## Використані матеріали
- `mlflow_training_runs.png` — таблиця п'яти runs у стилі MLflow.
- `mlflow_metrics_comparison.png` — порівняння accuracy, loss, recall та F1.
- `mlflow_rf_depth8_run_details.png` — деталі run `rf-depth8`.
- `mlflow_rf_depth4_artifacts.png` — artifacts run `rf-depth4`.
- `mlflow_rf_depth8_artifacts.png` — artifacts run `rf-depth8`.
- `prometheus_mlflow_accuracy_graph.png` — графік `mlflow_accuracy` у стилі Prometheus.
- `prometheus_mlflow_loss_graph.png` — графік `mlflow_loss` у стилі Prometheus.
- `prometheus_mlflow_accuracy_table.png` — таблиця `mlflow_accuracy` у стилі Prometheus.
- `prometheus_mlflow_loss_table.png` — таблиця `mlflow_loss` у стилі Prometheus.

Також використано синтетичний набір даних для задачі прогнозування відтоку клієнтів: 600 записів, із яких 450 використано для навчання та 150 — для тестування.

## Обраний run-кандидат

Обраний кандидат - **`rf-depth8`**, Run ID: `cafecb04`.

Основні метрики:
- Accuracy: `0.8733`
- Loss: `0.3602`
- Recall: `0.6471`
- F1: `0.6984`
- ROC-AUC: `0.8953`

Для цього запуску збережено `model.joblib` та супровідні артефакти. Водночас recall `0.6471` означає, що модель пропускає частину клієнтів із відтоком, тому потрібне додаткове оцінювання з урахуванням бізнес-вимог.

## Ухвалене PM-рішення

**Рішення: `Need more experiments`.**

`rf-depth8` є перспективним кандидатом, але наявних доказів недостатньо для переходу до staging. Потрібно порівняти моделі з урахуванням бізнес-метрик, перевірити результати на репрезентативних даних і визначити критерії приймання.

## Виявлені ризики

- Вибір моделі лише за accuracy без урахування бізнес-цілей.
- Недостатній recall для виявлення клієнтів із відтоком.
- Неповна відтворюваність експериментів.
- Відсутність артефакту для запуску `rf-depth4`.
- Не визначено критерії переходу до staging.
- Не описано rollback-процедуру.
- Не визначено відповідального за production-рішення.
- Не описано повний план моніторингу та повторного навчання моделі.

## Що потрібно зробити перед production

- Провести додаткові експерименти та порівняти моделі з урахуванням бізнес-вимог і вартості помилкових прогнозів.
- Перевірити якість моделі на репрезентативних даних.
- Погодити з бізнесом цільові значення recall та інших метрик.
- Зафіксувати версії коду, даних і залежностей для забезпечення відтворюваності експериментів.
- Визначити критерії приймання для переходу до staging і production.
- Призначити відповідального за ухвалення рішення.
- Налаштувати моніторинг якості моделі та data drift після deployment.
- Підготувати й перевірити rollback-процедуру.
- Визначити умови повторного навчання моделі (*retraining*).

## Структура проєкту

```text
lesson-9-pm/
├── README.md
├── pm_report.md
└── screenshots/
    ├── mlflow_training_runs.png
    ├── mlflow_metrics_comparison.png
    ├── mlflow_rf_depth8_run_details.png
    ├── mlflow_rf_depth4_artifacts.png
    ├── mlflow_rf_depth8_artifacts.png
    ├── prometheus_mlflow_accuracy_graph.png
    ├── prometheus_mlflow_loss_graph.png
    ├── prometheus_mlflow_accuracy_table.png
    └── prometheus_mlflow_loss_table.png
```
