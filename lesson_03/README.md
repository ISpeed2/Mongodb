# Занятие 3 · Практика «Каталог товаров»

Практическая работа из справочника MongoDB:
https://maximbytecamp.github.io/mongodb_theory_makarov/praktiki/01-katalog-tovarov/index.html

## Что делаем

Выполняем практику полностью на своей копии репозитория: запускаем программу,
подключаем её к MongoDB, создаём или загружаем каталог товаров, выполняем
предусмотренные запросы и проверяем результаты в базе.

Важно показать рабочий результат, а не только переписать код из справочника.
Команду запуска, используемую базу и коллекцию, нужно записать в отчёт.

## Что сдавать

- [homework_03.md](homework_03.md) — заполненный отчёт;
- обязательно шесть скриншотов в папке [screens/](screens/README.md);
- ответы на три вопроса в конце отчёта.

Скриншоты — обязательная часть сдачи: без них работа считается незавершённой.
Сдача выполняется в ветке `hw-03` через Pull Request в свою `main`.

## Как запустить

Из этой папки:

```bash
python3 -m pip install -r requirements.txt
python3 practice01/check.py
```

Для C++-решения:

```bash
g++ -std=c++17 practice01/solution.cpp -o practice01/solution \
  $(pkg-config --cflags --libs libmongocxx1)
./practice01/solution
```

## Чек-лист перед сдачей

- программа запускается повторно без ошибки подключения;
- товары действительно записаны в MongoDB;
- выполнены все операции из практической работы;
- результат проверен в Compass или через запрос в терминале;
- в отчёте нет паролей и секретных строк подключения;
- все ссылки на скриншоты открываются.

## Требуемая структура сдачи

```text
lesson_03/
├── README.md
├── homework_03.md
├── requirements.txt
├── practice01/
│   ├── check.py
│   └── solution.cpp
└── screens/
    ├── README.md
    ├── 01-program-start.png
    ├── 02-products-created.png
    ├── 03-catalog-result.png
    ├── 04-search-result.png
    ├── 05-update-or-delete-result.png
    └── 06-database-check.png
```
