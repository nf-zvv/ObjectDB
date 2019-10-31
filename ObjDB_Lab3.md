# Лабораторная работа №3

## Задание

Реализовать методы (для одного класса сущности):

1. Получение списка всех записей;
2. Получение списка всех зависимых записей (один-ко-многим);
3. Получение одной записи по первичному ключу;
4. Сохранение одной записи;
5. Обновление одной записи;
6. Удаление  одной записи.

В результате должны быть созданы два класса (для выбранного класса сущности):
1. Класс доступа к данным (data accessor class, DA class).
2. Класс с методами, перечисленными выше.

Вызов методов будет осуществляться из класса `Main()`.

## Примеры

Примеры приведены в официальной документации [persist_access](http://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/persist_access.html) и  [dpl_example](http://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/dpl_example.html)

## Пояснения

### Окружение (Environment)

Окружение требуется для приложений, созданных с использованием DPL.

Окружение - это инкапсуляция одной или нескольких баз данных. 
Вы открываете окружение, а затем базы данных в этой среде.

Окружение предлагает множество функций, которые не может предложить 
обычная база данных:

1. Файлы с несколькими базами данных.

    В одном физическом файле можно хранить несколько БД. 
    Это необходимо, когда приложение работает с несколькими БД.

2. Многопоточная и многопроцессорная поддержка

    Несколько потоков и/или процессов могут обращаться к БД.
    Кэширование.

3. Транзакционная обработка

    Идентификаторы транзакции

4. Репликация (высокая доступность)

    БД с одним мастером  и несколькими реплицированными read-only копиями.

5. Логгирование

    Ведение журнала с опережением записи. 
    Высокая степень восстановления в случае сбоя приложения или системы.


### Хранение данных

Существует два способа работы с данными (API):

1. Direct Persistent Layer (DPL)
2. Base database API (java)

### Работа с Berkeley DB

1. Задать параметры подключени к окружению
2. Получить из окружения нужную базы (по имени)
   * создать хранилище для объектов (Entity Store)
   * DML (Data Manipulation Language) операции (SELECT, INSERT, UPDATE, DELETE, MERGE)
   * CRUDL (Create, Read, Update, Delete, List)
   * ...
3. Закрыть соединение (окружение)


### Подключение окружения и хранилища

```java
private static File envHome = new File("./JEDB");

...

public void setup() {
	EnvironmentConfig envConfig = new EnvironmentConfig();
	StoreConfig storeConfig = new StoreConfig();

	envConfig.setAllowCreate(true);
	storeConfig.setAllowCreate(true);

	// Open the environment and entity store
	envmnt = new Environment(envHome, envConfig);
	store = new EntityStore(envmnt, "EntityStore", storeConfig);
}
```
### Создание класса доступа к данным

Создать класс доступа к данным (data accessor class, DA class)

```java
public class SimpleDA {
    // Open the indices
    public SimpleDA(EntityStore store)
        throws DatabaseException {

        // Primary key for SimpleEntityClass classes
        pIdx = store.getPrimaryIndex(
            String.class, SimpleEntityClass.class);

        // Secondary key for SimpleEntityClass classes
        // Last field in the getSecondaryIndex() method must be
        // the name of a class member; in this case, an 
        // SimpleEntityClass.class data member.
        sIdx = store.getSecondaryIndex(
            pIdx, String.class, "sKey");
    }

    // Index Accessors
    PrimaryIndex<String,SimpleEntityClass> pIdx;
    SecondaryIndex<String,String,SimpleEntityClass> sIdx;
}
```

### Помещение объектов в хранилище (Entity Store)

См. [Placing Objects in an Entity Store](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/simpleput.html)

Для того, чтобы поместить объект в хранилище необходимо:

1. Открыть окружение и хранилище - см. пример `setup()`.
2. Создать экземпляр класса доступа к данным - см. пример `SimpleDA`.
3. Создать объект (наполнить данными), который будет помещен в хранилище.
4. Поместить созданный объект в хранилище используя метод `put()`.
5. Закрыть окружение и хранилище.

```java
private void run()
throws DatabaseException {
    setup();

    // Open the data accessor. This is used to store
    // persistent objects.
    sda = new SimpleDA(store);

    // Instantiate and store some entity classes
    SimpleEntityClass sec1 = new SimpleEntityClass();

    sec1.setPKey("keyone");
    sec1.setSKey("skeyone");

    sda.pIdx.put(sec1);

    shutdown();
}
```

### Считывание объектов из хранилища (Entity Store)

Получение объектов из хранилища происходит аналогично, только используется метод `get()`. См. [Retrieving Objects from an Entity Store](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/simpleget.html)

### Получение списка всех зависимых записей (один-ко-многим)

Получение списка всех зависимых записей с использованием связи "один-ко-многим" осуществляется при помощи механизма курсоров. См. [Retrieving Objects from an Entity Store](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/simpleget.html)


