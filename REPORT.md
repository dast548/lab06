# Отчёт по лабораторной работе №6

## Тема
Изучение средств пакетирования на примере CPack.

## Цели и задачи
Настроить CPack для создания установочных пакетов
для приложения solver.

## Выполнение

### Подготовка
- Создан репозиторий lab06 на GitHub
- Скопированы файлы из lab03 (solver, formatter)

### Настройка версионирования
В CMakeLists.txt добавлены переменные версии:
- SOLVER_VERSION_MAJOR 1
- SOLVER_VERSION_MINOR 0
- SOLVER_VERSION_PATCH 0
- SOLVER_VERSION_TWEAK 0

### Создание CPackConfig.cmake
Настроены параметры для пакетов:
- TGZ архив
- DEB пакет (libsolver-dev)
- RPM пакет (solver-devel)

### Сборка пакета
Выполнены команды:
mkdir build && cd build
cmake ..
cmake --build .
cpack -G "TGZ"

Результат: solver-1.0.0.0-Linux.tar.gz

### Тег релиза
Создан тег v1.0.0.0 и запушен на GitHub.

## Вывод
Изучена система пакетирования CPack.
Настроено создание пакетов .tar.gz для приложения solver.
Создан релиз v1.0.0.0 на GitHub.
