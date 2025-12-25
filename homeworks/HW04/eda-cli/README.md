# S03 – eda_cli: мини-EDA для CSV

Небольшое CLI-приложение для базового анализа CSV-файлов.
Используется в рамках Семинара 03 курса «Инженерия ИИ».

## Требования

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) установлен в систему

## Инициализация проекта

В корне проекта (S03):

```bash
uv sync
```

Эта команда:

- создаст виртуальное окружение `.venv`;
- установит зависимости из `pyproject.toml`;
- установит сам проект `eda-cli` в окружение.

## Запуск CLI

### Краткий обзор

```bash
uv run eda-cli overview data/example.csv
```

Параметры:

- `--sep` – разделитель (по умолчанию `,`);
- `--encoding` – кодировка (по умолчанию `utf-8`).

### Полный EDA-отчёт

```bash
uv run eda-cli report data/example.csv --out-dir reports
```

В результате в каталоге `reports/` появятся:

- `report.md` – основной отчёт в Markdown;
- `summary.csv` – таблица по колонкам;
- `missing.csv` – пропуски по колонкам;
- `correlation.csv` – корреляционная матрица (если есть числовые признаки);
- `top_categories/*.csv` – top-k категорий по строковым признакам;
- `hist_*.png` – гистограммы числовых колонок;
- `missing_matrix.png` – визуализация пропусков;
- `correlation_heatmap.png` – тепловая карта корреляций.

## Тесты

```bash
uv run pytest -q
```
## HTTP сервер

/health — проверка работоспособности сервиса

Пример запроса:
```bash
GET /health
```

Пример ответа:
```bash
{
  "status": "ok",
  "service": "dataset-quality",
  "version": "0.2.0"
}
```

POST/quality — оценка качества по агрегированным признакам
Принимает агрегированные признаки датасета и возвращает эвристическую оценку качества.

Параметры запроса:
```bash
{
  "n_rows": 1500,
  "n_cols": 10,
  "max_missing_share": 0.2,
  "numeric_cols": 7,
  "categorical_cols": 3
}
```

Пример ответа:
```bash
{
  "ok_for_model": true,
  "quality_score": 0.85,
  "message": "Данных достаточно, модель можно обучать (по текущим эвристикам).",
  "latency_ms": 5.2,
  "flags": 
  {
    "too_few_rows": false,
    "too_many_columns": false,
    "too_many_missing": false,
    "no_numeric_columns": false,
    "no_categorical_columns": false
  },
  "dataset_shape":
   {
    "n_rows": 1500,
    "n_cols": 10
    }
}
```

POST/quality-from-csv — оценка качества по CSV
Загружает CSV-файл, считает статистики через EDA и возвращает оценку качества.

Параметры запроса:  
Multipart/form-data
file: CSV-файл


Пример ответа:
```bash
{
  "ok_for_model": false,
  "quality_score": 0.65,
  "message": "CSV требует доработки перед обучением модели (по текущим эвристикам).",
  "latency_ms": 12.4,
  "flags": 
  {
    "too_few_rows": true,
    "too_many_columns": false,
    "too_many_missing": true,
    "no_numeric_columns": false,
    "no_categorical_columns": false
  },
  "dataset_shape": 
  {
    "n_rows": 50,
    "n_cols": 12
  }
}
```

POST/quality-flags-from-csv — полный набор флагов качества
Загружает CSV файл и возвращает все эвристические флаги качества.

Параметры запроса: 
Multipart/form-data
file: CSV-файл

Пример ответа:
```bash
{
  "flags": 
  {
    "too_few_rows": false,
    "too_many_columns": false,
    "too_many_missing": true,
    "has_constant_columns": false,
    "has_suspicious_id_duplicates": true,
    "has_many_zero_values": true
  }
}
```

POST/head — первые n строк CSV
Описание: Возвращает первые n строк загруженного CSV.

Параметры запроса: Multipart/form-data + query
file: CSV-файл
n (query, default=5): количество строк

Пример ответа:
```bash
{
  "rows": [
    {
        "col1": 1, "col2": "A"
    },
    {
        "col1": 2, "col2": "B"
    },
    ...
  ]
}
```