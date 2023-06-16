## Laboratory work VIII

Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере **Docker**

```sh
$ open https://docs.docker.com/get-started/
```

## Tasks

- [ ] 1. Создать публичный репозиторий с названием **lab08** на сервисе **GitHub**
- [ ] 2. Ознакомиться со ссылками учебного материала
- [ ] 3. Выполнить инструкцию учебного материала
- [ ] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_USERNAME=<имя_пользователя>
```

```
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
$ cd lab08
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08
```

```sh
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
```

```sh
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```

```sh
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
```

```sh
$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```

```sh
$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
```

```sh
$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
```

```sh
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```

```sh
$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
```

```sh
$ docker build -t logger .
```

```sh
$ docker images
```

```sh
$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
text2
text3
<C-D>
```

```sh
$ docker inspect logger
```

```sh
$ cat logs/log.txt
```

```sh
$ gsed -i 's/lab07/lab08/g' README.md
```

```sh
$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```

```sh
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
```

```sh
$ travis login --auto
$ travis enable
```

## Report

```sh
$ popd
$ export LAB_NUMBER=08
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Links

- [Book](https://www.dockerbook.com)
- [Instructions](https://docs.docker.com/engine/reference/builder/)

```
Copyright (c) 2015-2021 The ISC Authors
```

## Homework

### include - написанная библотека

Содержимое файла `print.hpp`:

```
#include <fstream>
#include <iostream>
#include <string>

void print(const std::string& text, std::ofstream& out);
void print(const std::string& text, std::ostream& out = std::cout);
```

### source - исходный код

```
#include <print.hpp>

void print(const std::string& text, std::ostream& out)
{
  out << text;
}

void print(const std::string& text, std::ofstream& out)
{
  out << text;
}
```

### logs - папка с текстовым файлом

Содержимое файла `log.txt`:

```
*Пустой файл
```

### CMakeLists

Содержимое файла `CMakeLists.txt`:

```
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#  Создаем опцию BUILD_EXAMPLES, которая отвечает за сборку примеров
option(BUILD_EXAMPLES "Build examples" OFF)

project(print)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/source/print.cpp)

# Создаем исполняемый файл demo
add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print) 
# Устанавливаем исполняемый файл demo в директорию bin
install(TARGETS demo RUNTIME DESTINATION bin)

# Добавляем директорию include из проекта print в список директорий для поиска заголовочных файлов при использовании библиотеки print
target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

# Если опция BUILD_EXAMPLES включена, то
if(BUILD_EXAMPLES)
  # Создается переменная EXAMPLE_SOURCES, которая содержит список всех файлов с расширением .cpp в директории examples
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  # Запускается цикл по всем файлам из списка EXAMPLE_SOURCES
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    # Для каждого файла создается переменная EXAMPLE_NAME, которая содержит название файла без расширения
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    # Создается исполняемый файл с названием EXAMPLE_NAME и исходным файлом EXAMPLE_SOURCE
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    # Исполняемый файл связывается с библиотекой print
    target_link_libraries(${EXAMPLE_NAME} print)
    # Исполняемый файл устанавливается в директорию bin
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

# Устанавливаем библиотеку print в директорию lib и экспортируем ее в файл print-config
install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

# Устанавливаем все заголовочные файлы из директории include в директорию include
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
# Экспортируем файл print-config в директорию cmake
install(EXPORT print-config DESTINATION cmake)
```

### Dockerfile

```
FROM ubuntu:20.04

RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install

ENV LOG_PATH /home/lab08/logs/log.txt
VOLUME /home/lab08/logs

WORKDIR _install/bin
ENTRYPOINT ./demo
```

### Action.yml - файл сценария

Содержимое `Action.yml`:

```
name: docker
on:
  push:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: build docker
        run: docker build -t logger .
```

Затем запускает контейнер `Docker` с образом "hello-world". Этот образ используется для проверки, что Docker установлен и работает корректно.

```
$ sudo apt install docker.io
$ sudo docker run hello-world
```
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Создаём Docker-образ с любым именнем, в нашем случае с именем "logger" на основе `Dockerfile`, который находится в текущей директории (главной)
Смотрим список всех доступных Docker-образов (у нас доступен 1, который мы сделали)

Запускаем контейнер на основе образа "logger"
```
$ sudo docker build -t logger .
$ sudo docker images
$ sudo docker run -it -v "$(pwd)/logs/:/home/lab08/logs/" logger
```
```
Sending build context to Docker daemon  93.18kB
Step 1/12 : FROM ubuntu:20.04
 ---> 88bd68917189
Step 2/12 : RUN apt update
 ---> Using cache
 ---> edc856519bb9
Step 3/12 : RUN apt install -yy gcc g++ cmake
 ---> Using cache
 ---> a0c4f4b1b79a
Step 4/12 : COPY . print/
 ---> d425625e27d8
Step 5/12 : WORKDIR print
 ---> Running in 172ea7276db9
Removing intermediate container 172ea7276db9
 ---> b110a72a2120
Step 6/12 : RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
 ---> Running in da69e0cc801d
-- The C compiler identification is GNU 9.4.0
-- The CXX compiler identification is GNU 9.4.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /print/_build
Removing intermediate container da69e0cc801d
 ---> 10530e033ccf
Step 7/12 : RUN cmake --build _build
 ---> Running in e84fdbf71a33
Scanning dependencies of target print
[ 25%] Building CXX object CMakeFiles/print.dir/source/print.cpp.o
[ 50%] Linking CXX static library libprint.a
[ 50%] Built target print
Scanning dependencies of target demo
[ 75%] Building CXX object CMakeFiles/demo.dir/demo/main.cpp.o
[100%] Linking CXX executable demo
[100%] Built target demo
Removing intermediate container e84fdbf71a33
 ---> dfeb06362aaf
Step 8/12 : RUN cmake --build _build --target install
 ---> Running in c78816a017ad
[ 50%] Built target print
[100%] Built target demo
Install the project...
-- Install configuration: "Release"
-- Installing: /print/_install/bin/demo
-- Installing: /print/_install/lib/libprint.a
-- Installing: /print/_install/include
-- Installing: /print/_install/include/print.hpp
-- Installing: /print/_install/cmake/print-config.cmake
-- Installing: /print/_install/cmake/print-config-release.cmake
Removing intermediate container c78816a017ad
 ---> 236fbd5706da
Step 9/12 : ENV LOG_PATH /home/lab08/logs/log.txt
 ---> Running in 70953b79a6a5
Removing intermediate container 70953b79a6a5
 ---> 34e0ae76c7a0
Step 10/12 : VOLUME /home/lab08/logs
 ---> Running in 096bf6bf0f33
Removing intermediate container 096bf6bf0f33
 ---> 048737c173c4
Step 11/12 : WORKDIR _install/bin
 ---> Running in 7a6946a8f92b
Removing intermediate container 7a6946a8f92b
 ---> 5d6bd88411e1
Step 12/12 : ENTRYPOINT ./demo
 ---> Running in 7c4f96077bb2
Removing intermediate container 7c4f96077bb2
 ---> 7983cec6dd3a
Successfully built 7983cec6dd3a
Successfully tagged logger:latest
```
```
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
logger          latest    7983cec6dd3a   3 minutes ago   388MB
```
Последня команда требует ввести текст, который запишется в наш `log.txt` (она ничего не выводит)

Мы ввели `sokol`

```
$ sudo docker inspect logger
$ cat logs/log.txt
```
```
sokol
```
