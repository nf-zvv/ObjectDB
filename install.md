# Подготовка окружения для работы с Berkeley DB Java Edition

Java Development Kit (JDK) 13
```
https://www.oracle.com/technetwork/java/javase/downloads/jdk13-downloads-5672538.html
```

IntelliJ IDEA 2019.2.2
```
https://www.jetbrains.com/idea/download/#section=windows
```

Berkeley DB Java Edition 7.5.11
```
https://www.oracle.com/database/technologies/related/berkeleydb-downloads.html
```
Распаковать архив
```
unzip -U je-7.5.11.zip
```
Для использования JE необходим только файл `je-7.5.11/lib/je-7.5.11.jar`

Документация размещена в папке `je-7.5.11/docs/`

**Добавление библиотеки (`je-7.5.11.jar`) в IntelliJ IDEA**

В открытом проекте, в который нужно добавить библиотеку.

File -> Project Structure

Пункт Modules, вкладка Dependencies

Находим там знак '+', нажимаем на него и в появившемся блоке выбираем JARs or directories...

В появившемся окне выбираем нужный jar файл и нажимаем OK, и еще раз OK.

Найти только что добавленный jar файл можно в открывающемся списке External Libraries вашего проекта Intellij IDEA.




