# Laboratory work VI

Данная лабораторная работа посвещена изучению средств пакетирования на примере CPack.

## Цели и задачи

Настроить CPack для создания установочных пакетов (.tar.gz, .deb, .rpm)
для приложения solver из предыдущего задания.

## Сборка

```bash
mkdir build && cd build
cmake ..
cmake --build .
cpack -G "TGZ"
```
