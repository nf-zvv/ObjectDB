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



### Создание класса доступа к данным

Создать класс доступа к данным (data accessor class, DA class)

Пример из документации:

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

Здесь важно правильно указать индексы доступа (Index Accessors) `PrimaryIndex` и `SecondaryIndex`. Их количество должно совпадать с количеством ключей в сущности (первичный и один или несколько вторичных).

Чтобы извлечь первичный индекс необходимо использовать метод `EntityStore.getPrimaryIndex()`. Для этого необходимо указать тип данных первичного ключа (если id - цифровое значение, тогда указывается тип Integer) и класс сущности.

Например, чтобы извлечь первичный индекс класса `organization` (см. описание этого класса в лаб.2), где первичный ключ имеет тип Integer:

```
PrimaryIndex<Integer, organization> pIdx;
```

Чтобы извлечь вторичный индекс необходимо использовать метод `EntityStore.getSecondaryIndex()`. Для этого необходимо указать тип данных первичного ключа, тип данных вторичного ключа и класс сущности. Например, чтобы извлечь вторичный индекс класса `organization` (см. описание этого класса в [лаб.2](ObjDB_Lab2.md)), где первичный и вторичный ключи имеют тип Integer:

```
SecondaryIndex<Integer, Integer, organization> sIdx1;
```


### Подключение и закрытие окружения и хранилища

```java
private static File envHome = new File("./JEDB");

//...

public void setup() {
	EnvironmentConfig envConfig = new EnvironmentConfig();
	StoreConfig storeConfig = new StoreConfig();

	envConfig.setAllowCreate(true);
	storeConfig.setAllowCreate(true);

	// Открыть окружение и хранилище
	envmnt = new Environment(envHome, envConfig);
	store = new EntityStore(envmnt, "EntityStore", storeConfig);
}


// Закрыть окружение и хранилище
public void shutdown()
		throws DatabaseException {

	store.close();
	envmnt.close();
}

```


### Добавление объектов в хранилище (Entity Store)

См. [Placing Objects in an Entity Store](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/simpleput.html)

Для того, чтобы поместить объект в хранилище необходимо:

1. Открыть окружение и хранилище - см. пример `setup()`.
2. Создать экземпляр класса доступа к данным - см. пример `SimpleDA`.
3. Создать объект (наполнить данными), который будет помещен в хранилище.
4. Поместить созданный объект в хранилище используя метод `put()`.
5. Закрыть окружение и хранилище.

Пример из документации:

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

Пример для сущности `organization` из [лаб.2](ObjDB_Lab2.md):

```java
// Добавить в БД демо-записи
public void fillDemo()
		throws DatabaseException {

	// 1) Открыть окружение и хранилище
	setup();

	// 2) Создать экземпляр класса доступа к данным (DA, Data accessor)
	orgda = new organizationDA(store);

	// 3) Создать объекты и заполнить их данными
	organization org1 = new organization(1, "УдГУ", "Университетская, 1", "0000000000", "Сбербанк", 1812345678, 1);
	organization org2 = new organization(2, "ИжГТУ", "Студенческая, 7", "0000000000", "Сбербанк", 1809876543, 2);
	organization org3 = new organization(3, "ИжГСХА", "Студенческая, 11", "0000000000", "Сбербанк", 1865432178, 3);

	// 4) Поместить созданные объекты в хранилище
	orgda.pIdx.put(org1);
	orgda.pIdx.put(org2);
	orgda.pIdx.put(org3);

	// 5) Закрыть окружение и хранилище
	shutdown();
}
```

Аналогично реализуются другие методы задания.

**Замечание:**

Чтобы добавить новую запись, ей нужно присвоить новый `id`.
Получаем последний `id` в базе и увеличиваем его.
Это делается при помощи следующей строки:

```java
int new_id = orgda.pIdx.sortedMap().lastKey() + 1;
```


### Считывание объекта из хранилища (Entity Store)

Получение объекта из хранилища происходит аналогично, только используется метод `get()`. См. [Retrieving Objects from an Entity Store](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/simpleget.html)

Для того, чтобы получить объект из хранилище необходимо:

1. Открыть окружение и хранилище - см. пример `setup()`.
2. Создать экземпляр класса доступа к данным - см. пример `SimpleDA`.
3. Извлечь объект по первичному ключу: `organization org = orgda.pIdx.get(Id);`
4. Выполнить необходимые действия с полученным объектом: `String name = org.getName()`
5. Закрыть окружение и хранилище.

### Получение списка всех записей

Для получения списка всех записей рекомендуется использовать следующую конструкцию.
Пример для сущности `organization` из [лаб.2](ObjDB_Lab2.md):

```java
	// Перебрать все записи класса organization по первичному ключу
	for( Map.Entry<Integer, organization> entry : orgda.pIdx.map().entrySet() ){
		// entry - итератор
		// getValue() - получить объект класса organization
		// getId(),getName() - методы класса organization
		int id = entry.getValue().getId();
		String name = entry.getValue().getName();
		// ...
		// выполнить необходимые действия с полученными значениями
	}
```

### Удаление объекта из хранилища (Entity Store)

Для удаление объекта из хранилища используется метод `delete()`. См. [Deleting Entity Objects](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/dpl_delete.html)

Для того, чтобы удалить объект из хранилища необходимо:

1. Открыть окружение и хранилище - см. пример `setup()`.
2. Создать экземпляр класса доступа к данным - см. пример `SimpleDA`.
3. Удалить объект по первичному ключу: `orgda.pIdx.delete(Id);`
4. Закрыть окружение и хранилище.

### Обновление записи

1. Открыть окружение и хранилище - см. пример `setup()`.
2. Создать экземпляр класса доступа к данным - см. пример `SimpleDA`.
3. Извлечь объект по первичному ключу: `organization org = orgda.pIdx.get(Id);`.
4. Внести изменения.
5. Поместить отредактированный объект в хранилище: `orgda.pIdx.put(Id);`.
6. Закрыть окружение и хранилище.

### Получение списка всех зависимых записей (один-ко-многим)

Получение списка всех зависимых записей с использованием связи "один-ко-многим" осуществляется при помощи механизма курсоров. См. [Retrieving Objects from an Entity Store](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/dpl_entityjoin.html)


