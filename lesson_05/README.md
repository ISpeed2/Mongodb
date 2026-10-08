# Занятие 5 · Скрипт подбора кандидатов

Домашнее задание по главе 2.9 справочника MongoDB:
<https://maximbytecamp.github.io/mongodb_theory_makarov/temy/18-skript-podbor/index.html>

Решение написано на C++17. Скрипт читает базы `hh.resumes`, `hh.vacancies` и
`hh.interviews`, строит восемь разделов отчёта и не изменяет данные.

## Как запустить

Из корня репозитория:

```bash
g++ -std=c++17 lesson_05/hh_search.cpp -o lesson_05/hh_search \
  $(pkg-config --cflags --libs libmongocxx1)
./lesson_05/hh_search
```

По умолчанию используется `mongodb://localhost:27017`. Другой адрес можно
передать через переменную окружения `MONGO_URI`.

## Состав сдачи

- [homework_05.md](homework_05.md) — заполненная таблица, разбор операторов и вывод;
- [hh_search.cpp](hh_search.cpp) — C++-скрипт;
- [screens/](screens/) — три требуемых снимка запусков.

Сдача подготовлена в ветке `hw-05` для Pull Request в `main`.
