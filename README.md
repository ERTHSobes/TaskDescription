# Тестовое задание
В данном тестовом задании вам потребуется реализовать API состоящее из 3х сервисов взаимодействующих с базой данных и другим API(имитация сторонней системы).  
В качестве тестового окружения можно использовать представленный docker-compose

Доп. требования к результату выполнения:
- результат выполнения должен быть представлен на github
- приложение должно билдится в docker image 
- возможность задания endpoint внешней системы({host}:{port}) через переменные окружения
- возможность задания строки подключеиня к БД через переменную окружения
- в случае реализации на c# - использовать .net core 3.1 и для взаимодействия с БД EFCore
- в случае реализации на java - использовать jdk 8/11, Spring Boot, для взаимодействия с БД Spring Data JPA

## Описание требований к сервисам

### Сервис создания новой записи **СreateObject**

Сервис должен работать как через Get так и через Post  
На входе принимает 
> Поле | Тип | Описание
> --- | --- |---
> Name | строка | имя запрашивающей стороны
> dataType | строка | тип объекта ( phone, email, other)

По dataType получает новый обект из [API сторонней системы](#api) и сохраняет его в БД

Сервис возвращает Guid полученного объетка

Пример вызова:

``` Get http://{host}:{port}/Api/СreateObject?Name=test&dataType=phone ```
Или
``` Post http://{host}:{port}/Api/СreateObject```
``` json
{
"name":"test",
"dataType":"phone"
}
```
В ответ:

``` d483776a-c8a8-456a-ac68-742cd03fcbbf ```
### Сервис получения файла - **GetFileById**

Сервсиc должен работать только через **Get**  
По Id полученному в первом сервисе выгружает файл, если он есть,
в противном случае возвращает 404 статус + описание 

Входные данные:
> Поле | Тип | Описание
> --- | --- |---
> id | Guid | id объекта для которого запрашивается файл


В ответ поток с файлом(имя и расширене файла должно быть сохранено, относительно возвращаемого сервисом GetFile API сторонней системы)

Либо если не найден объект:

 - 404 + *Запрашиваемый объект не найден*

Если у объекта нет файла: 

 - 404 + *У запрашиваемого объекта отсутсвует файл*

Пример:

``` Get http://{host}:{port}/Api/GetFileById?id=d483776a-c8a8-456a-ac68-742cd03fcbbf ```


### Сервис получения статистики - **GetStats**

Сервсиc должен работать только через Get
Возвращает общую статистику по сохранненным в базу объектам в виде json
Описание структуры

> Поле | Тип | Описание
> --- | --- |---
> PhoneCount | число | количество объектов типа phone
> PhoneCountWhitFile | число | количество объектов типа phone к которым прикреплен файл
> TopPhones | строка | 10 последних телефонных номеров через ;
> EmailСount | число | количество объектов типа email
> EmailCountWhitFile | число | количество объектов типа email к которым прикреплен файл
> TopEmails | строка | 10 последних email через ;
> OhterCount | число | количество объектов типа other
> OhterCountWhitFile | число | количество объектов типа other к которым прикреплен файл

Пример:

Запрос:

```Get http://{host}:{port}/Api/GetStats```
Ответ:

```json
{
    "PhoneCount":"1",
    "PhoneCountWhitFile":"1",
    "TopPhones":"88008008000;88008008001",
    "EmailСount":"1",
    "EmailCountWhitFile":"1",
    "TopEmails":"test@test.com;test1@.test.com",
    "OhterCount":"10",
    "OhterCountWhitFile":"15",
   }
```

## Описание API сторонней системы<a name="api"></a>

Веб приложение доступно в виде docker image *erthsobes/erthsobesservis*.

Внутри контейнера слушает на 6500 порту.

У приложения доступен swagger */swagger/index.html*.

Api состоит из 2х сервисов:

### Сервис получениея нового объекта по его типу - **GetObjectInfo**:

Get сервис, на входе принимает тип запрашиваемого объекта:
phone, email или 0, 1.  
 Все что не подходит под эти типы будет считаться как тип прочее.
В ответ возвращает структуру следующего вида:

> Поле | Тип | Описание
> --- | --- |---
> dataType | строка | тип объекта ( phone, email, other)
> data | строка | объект в виде json 
> Error | строка | в случае ошибки - тут будет ее описание

Возможные объекты в поле data:

1. Телефонный номер

    > Поле | Тип | Описание
    > --- | --- |---
    > Id | Guid | уникальный идентификатор объекта
    > Cost | decimal | Цена
    > PhoneNumber | строка | номер телефона
    > File | object | описание файла вложения

2. Email
    > Поле | Тип | Описание
    > --- | --- |---
    > Id | Guid | уникальный идентификатор объекта
    > Cost | decimal | Цена
    > Email | строка | email адрес
    > File | object | описание файла вложения

3. Прочее
    > Поле | Тип | Описание
    > --- | --- |---
    > Id | Guid | уникальный идентификатор объекта
    > Cost | decimal | Цена
    > Value | строка | описание объекта
    > File | object | описание файла вложения

 Структура объекта **File**:<a name="fileStructure"></a>

> Поле | Тип | Описание
> --- | --- |---
> Id | число | может быть очень большим
> Hash | строка | хеш использующийся при запросе файла

Коды возврата:

    - 200 - если все ок
    - 500 - в случае ошибки


Примеры вызова:

Запрос:

``` Get http://{host}:{port}/api/GetObjectInfo?type=phone```

Ответ :

```json
{"dataType":"phone","data":"{\"PhoneNumber\":\"89392975637\",\"Id\":\"34b57891-71ce-43e7-b49f-b243d7f851e6\",\"Cost\":79.58}"}
```
```json
{"dataType":"phone","data":"{\"PhoneNumber\":\"89897797065\",\"Id\":\"d483776a-c8a8-456a-ac68-742cd03fcbbf\",\"Cost\":0.17,\"File\":{\"Id\":323456789123456789,\"Hash\":\"f5e95e995d1ed66f727d2b8136ee7db679889e0b7fb7ca49d0bd407840a3a8e1\"}}"}
```

### Сервис выгрузки файла - **GetFile**:

Post сервис, по Id файла и hash - возвращает файл

На входе json объект типа [File](#fileStructure)

На выходе поток с содержимым файла

Коды возврата:

    - 200 - если все ок
    - 403 - в случае не корректного hash
    - 404 - в случае отсутсвия файла
    - 500 - в случае ошибки

Пример:

Запрос:
``` POST http://{host}:{port}/Api/GetFile ```
```json
{
	"id":1,
	"hash":"c8293c1122925a4a56ef572d280ffe587f5f003917b621b3166ab0bce438a793"
}
```

## Описание БД

В качестве СУБД используется PostgreSQL 12.4   
Название базы - **orders**  
### Структура
- Таблица **order_info**

> Поле | Тип | Описание
> --- | --- |---
> id | bigint | PK
> product_id | uuid | уникальный идентификатор объекта
> type | varchar | тип объекта ( phone, email, other)
> cost | numeric | Цена
> phoneNumber | varchar | номер телефона
> email | varchar | email адрес
> value | varchar | описание объекта
> attachment_id | bigint | FK на attachment
- Таблица **attachment**
> Поле | Тип | Описание
> --- | --- |---
> id | uuid | уникальный идентификатор объекта
> hash | varchar | хеш использующийся при запросе файла

