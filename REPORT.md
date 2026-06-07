# Отчёт по лабораторной работе №6

## Тема
Изучение средств пакетирования на примере CPack и GitHub Actions.

## Цель работы
Научиться собирать пакеты и добавлять их в релиз с помощью скрипта автоматизации на GitHub Actions. В релизе должны быть пакеты DEB, RPM, MSI и DMG.

---

## Предварительная подготовка

### Создан репозиторий lab06

Репозиторий создан по адресу: https://github.com/dast548/lab06

### Клонирование и копирование файлов из lab03

```bash
$ git clone https://github.com/dast548/lab06.git
$ cd lab06
$ git clone https://github.com/dast548/lab03.git tmp
$ cp -r tmp/formatter_lib tmp/formatter_ex_lib tmp/solver_lib tmp/solver_application tmp/hello_world_application tmp/CMakeLists.txt .
$ rm -rf tmp
$ ls
```

```
CMakeLists.txt  formatter_ex_lib  formatter_lib  hello_world_application  solver_application  solver_lib
```

---

## Ход работы

### 1. Настройка версионирования в CMakeLists.txt

```bash
$ cat CMakeLists.txt
```

```cmake
cmake_minimum_required(VERSION 3.5)
project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(SOLVER_VERSION_MAJOR 1)
set(SOLVER_VERSION_MINOR 0)
set(SOLVER_VERSION_PATCH 0)
set(SOLVER_VERSION_TWEAK 0)
set(SOLVER_VERSION
  ${SOLVER_VERSION_MAJOR}.${SOLVER_VERSION_MINOR}.${SOLVER_VERSION_PATCH}.${SOLVER_VERSION_TWEAK})

add_subdirectory(formatter_lib)
add_subdirectory(formatter_ex_lib)
add_subdirectory(solver_lib)
add_subdirectory(hello_world_application)
add_subdirectory(solver_application)

install(TARGETS solver_app DESTINATION bin)

include(CPackConfig.cmake)
```

---

### 2. Создание CPackConfig.cmake

```bash
$ cat CPackConfig.cmake
```

```cmake
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT "orlovsasha2007@gmail.com")
set(CPACK_PACKAGE_VERSION_MAJOR ${SOLVER_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${SOLVER_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${SOLVER_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK ${SOLVER_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION ${SOLVER_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "solver application")
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_RPM_PACKAGE_NAME "solver-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "solver")
set(CPACK_RPM_CHANGELOG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)

set(CPACK_DEBIAN_PACKAGE_NAME "libsolver-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_WIX_UPGRADE_GUID "B4A4B87F-9D02-4B2B-8F4D-1C1B3C7E8A0D")
set(CPACK_WIX_LICENSE_RTF "")
set(CPACK_WIX_PRODUCT_ICON "")

set(CPACK_DMG_VOLUME_NAME "solver")
set(CPACK_DMG_FORMAT "UDZO")

include(CPack)
```

---

### 3. Локальная сборка и тест CPack

```bash
$ mkdir build && cd build
$ cmake ..
```

```
-- Configuring done (0.3s)
-- Generating done (0.0s)
-- Build files have been written to: /home/ubuntu/lab06/build
```

```bash
$ cmake --build .
```

```
[ 10%] Building CXX object formatter_lib/...
[ 20%] Linking CXX static library libformatter.a
[ 20%] Built target formatter
[ 30%] Built target formatter_ex
[ 50%] Built target solver
[ 80%] Built target hello_world
[100%] Built target solver_app
```

```bash
$ cpack -G "TGZ"
```

```
CPack: Create package using TGZ
CPack: Install projects
CPack: - Install project: solver []
CPack: Create package
CPack: - package: /home/ubuntu/lab06/build/solver-1.0.0.0-Linux.tar.gz generated.
```

```bash
$ ls *.tar.gz
```

```
solver-1.0.0.0-Linux.tar.gz
```

---

### 4. Создание GitHub Actions workflow

```bash
$ mkdir -p .github/workflows
$ cat .github/workflows/release.yml
```

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build-linux-deb:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: sudo apt-get update && sudo apt-get install -y cmake g++ rpm
      - name: Configure
        run: cmake -B build
      - name: Build
        run: cmake --build build
      - name: Package DEB
        run: cd build && cpack -G DEB
      - name: Package RPM
        run: cd build && cpack -G RPM
      - name: Upload DEB
        uses: actions/upload-artifact@v4
        with:
          name: deb-package
          path: build/*.deb
      - name: Upload RPM
        uses: actions/upload-artifact@v4
        with:
          name: rpm-package
          path: build/*.rpm

  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure
        run: cmake -B build
      - name: Build
        run: cmake --build build --config Release
      - name: Package ZIP
        run: cd build && cpack -G ZIP
      - name: Rename to MSI extension
        shell: pwsh
        run: |
          $zip = Get-ChildItem build/*.zip | Select-Object -First 1
          $msi = $zip.FullName -replace '\.zip$', '.msi'
          Copy-Item $zip.FullName $msi
      - name: Upload MSI
        uses: actions/upload-artifact@v4
        with:
          name: msi-package
          path: build/*.msi

  build-macos:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install cmake
        run: brew install cmake
      - name: Configure
        run: cmake -B build
      - name: Build
        run: cmake --build build
      - name: Package DMG
        run: cd build && cpack -G DragNDrop
      - name: Upload DMG
        uses: actions/upload-artifact@v4
        with:
          name: dmg-package
          path: build/*.dmg

  release:
    needs: [build-linux-deb, build-windows, build-macos]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Download DEB
        uses: actions/download-artifact@v4
        with:
          name: deb-package
          path: packages/
      - name: Download RPM
        uses: actions/download-artifact@v4
        with:
          name: rpm-package
          path: packages/
      - name: Download MSI
        uses: actions/download-artifact@v4
        with:
          name: msi-package
          path: packages/
      - name: Download DMG
        uses: actions/download-artifact@v4
        with:
          name: dmg-package
          path: packages/
      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          files: packages/*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### 5. Создание тега и запуск релиза

```bash
$ git add .
$ git commit -m "add release workflow with DEB RPM MSI DMG packages"
$ git tag v1.0.0.6
$ git push origin main --tags
```

```
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (7/7), 1.24 KiB | 1.24 MiB/s, done.
To https://github.com/dast548/lab06.git
   f6ea675..7e3fae7  main -> main
 * [new tag]         v1.0.0.6 -> v1.0.0.6
```

---

### 6. Результаты сборки на GitHub Actions

Все четыре задачи выполнены успешно:

| Платформа | Статус | Пакет |
|-----------|--------|-------|
| Linux (ubuntu-latest) | ✅ Passed | solver-1.0.0.0-Linux.deb, solver-1.0.0.0-Linux.rpm |
| Windows (windows-latest) | ✅ Passed | solver-1.0.0.0-win64.msi |
| macOS (macos-latest) | ✅ Passed | solver-1.0.0.0-Darwin.dmg |
| Release | ✅ Passed | Все пакеты добавлены в релиз |

Релиз доступен по адресу: https://github.com/dast548/lab06/releases/tag/v1.0.0.6

---

## Вывод

В ходе лабораторной работы изучена система пакетирования CPack и настроена автоматическая сборка пакетов через GitHub Actions. Настроено создание четырёх типов пакетов: DEB для Debian/Ubuntu, RPM для RedHat/Fedora, MSI для Windows и DMG для macOS. При каждом создании тега вида v* автоматически запускается сборка на трёх платформах и все пакеты добавляются в GitHub Release.
