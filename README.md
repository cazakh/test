Замечание. Локально, эта ЛР выполнялась из папки lab08, то есть ЛР8 была просто дополнена, а репозиторий переинициализирован, в следствие чего часть кода имеет несоответствие с номер дабораторной работы

В данной ЛР не очень много изменений по сравнению с ЛР8. Вот все из них:

### Action.yml

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
      - name: 'Upload Artifact'
        uses: actions/upload-artifact@v3
        with:
          name: my-artifact
          path: logs/log.txt
          retention-days: 7

      - name: build docker
        run: docker build -t logger .
```

Содержимое `CPackConfig.cmake`:
```
include(InstallRequiredSystemLibraries)
set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR 0)
set(CPACK_PACKAGE_VERSION_MINOR 1)
set(CPACK_PACKAGE_VERSION_PATCH 0)
set(CPACK_PACKAGE_VERSION_TWEAK 0)
set(CPACK_PACKAGE_VERSION 0.1.0.0)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "logger")

include(CPack)
```

Далее создаем релиз и ищем артефакт в разделе `Actionsd` (да, будет работать и без релиза, однако так более корректно делать)

Артефакт - чаще всего это промежуточный результат выполнения кода, который необходим для отладки работы. В следствие того, что данное приложение не совсем подходит для демонстрации использования артефактов, в качестве такового был загружен `log.txt`
