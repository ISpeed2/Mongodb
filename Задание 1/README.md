# Моя работа по MongoDB

Практическая работа 01 выполнена на C++.

## Что нужно установить

- запущенный MongoDB на `mongodb://localhost:27017`;
- MongoDB C++ Driver;
- `pkg-config`;
- `g++` или `clang++`.

## Сборка и запуск

Команды выполняются из этой папки:

```bash
g++ -std=c++17 practice01/solution.cpp -o practice01/solution \
  $(pkg-config --cflags --libs libmongocxx1)
./practice01/solution
```

Программа читает `shop.products` и изменяет только `sandbox.products`.

## Проверка

Проверка находится рядом с решением:

```bash
python3 -m pip install -r requirements.txt
python3 practice01/check.py
```

Ожидаемый результат: `Пройдено 12 из 12`.

Перед первым запуском в MongoDB должны быть загружены учебные базы `shop` и
`sandbox`. После выполнения решения проверку можно запускать из `mongodb-work`.