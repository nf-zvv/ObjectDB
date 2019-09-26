# Лабораторная работа №2

## Задание

Создать классы на Java для Berkeley DB, описывающие модель базы данных по ЛР №1.

## Примеры

Примеры оформления классов приведены в [официальной документации](https://docs.oracle.com/cd/E17076_03/html/gsg/JAVA/dpl_example.html) - классы `Vendor` и `Inventory`.

## Пояснения

Программа на Java состоит из классов. Каждый класс хранится в отдельном файле. Имя файла должно совпадать с именем класса, расширение файла - `*.java`.

Для того, чтобы класс стал сущностью БД надо выше определения класса добавить аннотацию `@Entity`.

Пример класса сущности:
```
@Entity
public class organization {
  @PrimaryKey(sequence="ID")
  private int id;
  private String name;
  private String addr;
  private String opf;
  private String bank;
  private int inn;
  @SecondaryKey(relate=ONE_TO_ONE, relatedEntity=leaders.class, name="id") 
  private int leader_id;
  private organization() {  }  
  public organization(int id, String name, String addr, String opf, String bank, int inn, int leader_id) {
       this.id = id;
     this.name = name;
     this.addr = addr;
     this.opf = opf;
     this.bank = bank;
     this.inn = inn;
     this.leader_id = leader_id;
    }
```

Аннотация `@PrimaryKey` объявляет первичным ключом поле `id`.

Аннотация `@SecondaryKey` объявляет связь `ONE_TO_ONE` (один к одному) текущего поля `leader_id` класса `organization` с полем `id` класса `leaders`.

Аннотация `@Persistent` определяет постоянный класс. Например, каждый класс сущностей содержит PrimaryIndex по которому производится доступ к экземплярам данного класса. Поэтому можно вынести общее поле классов сущностей в один базовый класс.

Пример:
```
@Persistent
class BaseClass {
    @PrimaryKey(sequence="ID")
    private int id;
}

@Entity
class organization extends BaseClass {
    private String name;
    private String addr;
    ...
    ...
  }
```

