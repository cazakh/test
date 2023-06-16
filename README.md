## Laboratory work V

[![Coverage Status](https://coveralls.io/repos/github/DimaSokkk/lab05/badge.svg?branch=master)](https://coveralls.io/github/DimaSokkk/lab05?branch=master)

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**

```sh
$ open https://github.com/google/googletest
```

## Tasks

- [ ] 1. Создать публичный репозиторий с названием **lab05** на сервисе **GitHub**
- [ ] 2. Выполнить инструкцию учебного материала
- [ ] 3. Ознакомиться со ссылками учебного материала
- [ ] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_USERNAME=<имя_пользователя>
$ alias gsed=sed # for *-nix system
```

```sh
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab04 projects/lab05
$ cd projects/lab05
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab05
```

```sh
$ mkdir third-party
$ git submodule add https://github.com/google/googletest third-party/gtest
$ cd third-party/gtest && git checkout release-1.8.1 && cd ../..
$ git add third-party/gtest
$ git commit -m"added gtest framework"
```

```sh
$ gsed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a\
option(BUILD_TESTS "Build tests" OFF)
' CMakeLists.txt
$ cat >> CMakeLists.txt <<EOF

if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check \${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```

```sh
$ mkdir tests
$ cat > tests/test1.cpp <<EOF
#include <print.hpp>

#include <gtest/gtest.h>

TEST(Print, InFileStream)
{
  std::string filepath = "file.txt";
  std::string text = "hello";
  std::ofstream out{filepath};

  print(text, out);
  out.close();

  std::string result;
  std::ifstream in{filepath};
  in >> result;

  EXPECT_EQ(result, text);
}
EOF
```

```sh
$ cmake -H. -B_build -DBUILD_TESTS=ON
$ cmake --build _build
$ cmake --build _build --target test
```

```sh
$ _build/check
$ cmake --build _build --target test -- ARGS=--verbose
```

```sh
$ gsed -i 's/lab04/lab05/g' README.md
$ gsed -i 's/\(DCMAKE_INSTALL_PREFIX=_install\)/\1 -DBUILD_TESTS=ON/' .travis.yml
$ gsed -i '/cmake --build _build --target install/a\
- cmake --build _build --target test -- ARGS=--verbose
' .travis.yml
```

```sh
$ travis lint
```

```sh
$ git add .travis.yml
$ git add tests
$ git add -p
$ git commit -m"added tests"
$ git push origin master
```

```sh
$ travis login --auto
$ travis enable
```

```sh
$ mkdir artifacts
$ sleep 20s && gnome-screenshot --file artifacts/screenshot.png
# for macOS: $ screencapture -T 20 artifacts/screenshot.png
# open https://github.com/${GITHUB_USERNAME}/lab05
```

## Report

```sh
$ popd
$ export LAB_NUMBER=05
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Homework

### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.
2. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
3. Настройте сборочную процедуру на **TravisCI**.
4. Настройте [Coveralls.io](https://coveralls.io/).

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2021 The ISC Authors
```

## Homework

## Part I

Для начала скопируем папку banking из `tp labs`:

```
$ wget https://raw.githubusercontent.com/tp-labs/lab05/master/banking/Account.cpp
$ wget https://raw.githubusercontent.com/tp-labs/lab05/master/banking/Account.h
$ wget https://raw.githubusercontent.com/tp-labs/lab05/master/banking/Transaction.cpp
$ wget https://raw.githubusercontent.com/tp-labs/lab05/master/banking/Transaction.h
```

В файле Transaction.cpp допущена ошибка - нужно изменить 34 строку: 

Меняем `Debit(to, sum + fee_)` на `Debit(from, sum + fee_)`

Теперь подключим библиотеку `gtest`:

```
$ mkdir third-party
$ git submodule add https://github.com/google/googletest third-party/gtest
$ cd third-party/gtest && git checkout release-1.8.1 && cd ../..
$ git add third-party/gtest
$ git commit -m "added gtest"
$ git push origin master
```

Создадим CMakeLists.txt внутри banking:
```
project(banking_lib)

if (NOT TARGET libbanking)
    add_library(libbanking STATIC
        /Account.cpp
        /Transaction.cpp
    )

    install(TARGETS libbanking
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
    )
endif(NOT TARGET libbanking)

include_directories()
```

Создадим "общий" CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TEST "Build tests" OFF)

if(BUILD_TESTS)
  add_compile_options(--coverage)
endif()

option(COVERAGE "Check coverage" ON)

project(banking)

add_library(banking STATIC banking/Account.cpp banking/Transaction.cpp)
target_include_directories(banking PUBLIC banking/)

target_link_libraries(banking gcov)

if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB BANKING_TEST_SOURCES tests/test1.cpp)
  add_executable(check tests/test1.cpp)
  target_link_libraries(check banking gtest_main gmock_main)
  add_test(NAME check COMMAND check)
endif()

if (COVERAGE)
	target_compile_options(check PRIVATE --coverage)
	target_link_libraries(check --coverage)
endif()
```

## Part II

Создаём тест внутри папки tests:

Содержимое файла test1:

```
#include "Account.h"
#include "Transaction.h"

#include <gtest/gtest.h>
#include <gmock/gmock.h>

class AccountMock : public Account {
public:
    AccountMock(int id, int balance) : Account(id, balance) {}
    MOCK_CONST_METHOD0(GetBalance, int());
    MOCK_METHOD1(ChangeBalance, void(int diff)); 
    MOCK_METHOD0(Lock, void());
    MOCK_METHOD0(Unlock, void());
};

class TransactionMock : public Transaction {
public:
    MOCK_METHOD3(Make, bool(Account& from, Account& to, int sum));
};

TEST(Account, Mock) {
    AccountMock acc(1, 666);
    EXPECT_CALL(acc, GetBalance()).Times(1);
    EXPECT_CALL(acc, ChangeBalance(testing::_)).Times(2);
    EXPECT_CALL(acc, Lock()).Times(2);
    EXPECT_CALL(acc, Unlock()).Times(1);
    acc.GetBalance();
    acc.ChangeBalance(100); 
    acc.Lock();
    acc.ChangeBalance(100);
    acc.Lock(); 
    acc.Unlock();
}

TEST(Account, SimpleTest) {
    Account acc(1, 666);
    EXPECT_EQ(acc.id(), 1);
    EXPECT_EQ(acc.GetBalance(), 666);
    EXPECT_THROW(acc.ChangeBalance(200), std::runtime_error);
    EXPECT_NO_THROW(acc.Lock());
    acc.ChangeBalance(200);
    EXPECT_EQ(acc.GetBalance(), 866);
    EXPECT_THROW(acc.Lock(), std::runtime_error);
    EXPECT_NO_THROW(acc.Unlock());
}

TEST(Transaction, Mock) {
    TransactionMock tr;
    Account ac1(1, 100);
    Account ac2(2, 300);
    EXPECT_CALL(tr, Make(testing::_, testing::_, testing::_))
    .Times(5);
    tr.set_fee(200);
    tr.Make(ac1, ac2, 200);
    tr.Make(ac2, ac1, 300);
    tr.Make(ac1, ac1, 0); 
    tr.Make(ac1, ac2, -5); 
    tr.Make(ac2, ac1, 50); 
}

TEST(Transaction, SimpleTest) {
    Transaction tr;
    Account ac1(1, 100);
    Account ac2(2, 300);
    tr.set_fee(10);
    EXPECT_EQ(tr.fee(), 10);
    EXPECT_THROW(tr.Make(ac1, ac2, 40), std::logic_error);
    EXPECT_THROW(tr.Make(ac1, ac2, -5), std::invalid_argument);
    EXPECT_THROW(tr.Make(ac1, ac1, 100), std::logic_error);
    EXPECT_FALSE(tr.Make(ac1, ac2, 400));
    EXPECT_FALSE(tr.Make(ac2, ac1, 300));
    EXPECT_FALSE(tr.Make(ac2, ac1, 290));
    EXPECT_TRUE(tr.Make(ac2, ac1, 150));
}
```

## Part III

Создаём lcov.info в папке coverage
```
$ mkdir coverage
$ cd coverage
$ cat >> lcov.info << EOF
```

`lcov` — графический интерфейс для gcov. Он собирает файлы gcov для нескольких файлов с исходниками и создает комплект HTML-страниц с кодом и сведениями о покрытии. 

Создадим файл Action.yml:

```
name: CMake

on:
 push:
  branches: [master]
 pull_request:
  branches: [master]

jobs:
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Adding gtest
    run: git clone https://github.com/google/googletest.git third-party/gtest -b release-1.11.0

  - name: Install lcov
    run: sudo apt-get install -y lcov

  - name: Config banking with tests
    run: cmake -H. -B ${{github.workspace}}/build -DBUILD_TESTS=ON

  - name: Build banking
    run: cmake --build ${{github.workspace}}/build

  - name: Run tests
    run: |
      build/check
      cmake --build ${{github.workspace}}/build --target test -- ARGS=--verbose

  - name: Do lcov stuff
    run: lcov -c -d build/CMakeFiles/banking.dir/banking/ --include *.cpp --output-file ./coverage/lcov.info

  - name: Publish to coveralls.io
    uses: coverallsapp/github-action@v1.1.2
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

# Некоторые сведения об использованных командах

`target_include_directories` Указывает включаемые каталоги, которые будут использоваться при компиляции заданной цели banking

`target_link_libraries` связывание banking и gcov

`GLOB` сгенерирует список всех файлов, соответствующих выражениям подстановки, и сохранит его в переменной

`add_executable` Добавляет в проект исполняемый файл, используя указанные исходные файлы

`add_test` Добавляет тест под названием

`add_library` Добавляет целевой объект библиотеки с именем libbanking для построения из исходных файлов, перечисленных в вызове команды

Библиотеки `STATIC` - это архивы объектных файлов для использования при компоновке других целей.

`install(TARGETS` указаны правила установки целей из проекта

`ARCHIVE DESTINATION lib` статическая, `LIBRARY DESTINATION lib` общие библиотеки

`MOCK_METHOD1(ChangeBalance, void(int diff))` Первым аргументом идет имя того самого метода, который мы ожидаем что будет выполнен в нашем будущем тесте. 
Далее идет сигнатура этого метода. Цифра 1 в названии макроса означает число аргументов у метода ChangeBalance - один.

`EXPECT_CALL` позволяет описать, что должно произойти с методом за время теста

`EXPECT_EQ` EQ - Equal, проверяет равны ли

`EXPECT_THROW(tr.Make(ac1, ac1, 100), std::logic_error);` Проверяет, что tr.Make(ac1, ac1, 100) выдает исключение типа logic_error

`EXPECT_NO_THROW(acc.Lock());` Проверяет, что (acc.Lock() не генерирует никаких исключений

`EXPECT_FALSE` проверяет неверно ли это 

`sudo apt-get install` используется для загрузки последней версии нужного вам приложения из онлайн-хранилища программного обеспечения, на которое указывают ваши источники

`cmake -H. -B ${{github.workspace}}/build -DBUILD_TESTS=ON` Подготовка процесса сборки, запись результата в файл

`cmake --build ${{github.workspace}}/build` Старт сборки

`cmake --build ${{github.workspace}}/build --target test -- ARGS=--verbose` Начинаем сборку test, которая будет транслировать результаты теста
