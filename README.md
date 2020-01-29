## Документация по интеграции с сервисами Proil Business

### Общие правила отправки запросов

+ Запросы отправляются на сервер по протоколу https  
+ Успешный ответ сервера на запрос должен иметь статус-код 200
+ Тело успешного ответ сервера содержит валидный JSON-объект с обязательным параметром success: true  
+ Ответ с ошибкой может содержать статус-код, отличный от 200 и 304: 4хх или 5хх.  
+ Тело ответа сервера в случае возникновения обработанной ошибки будет содержать валидный JSON-объект с полем success: false и описанием ошибок в массиве errors.
+ Каждый запрос должен содержать заголовок для авторизации: Authorization: Token \<uuid\>
+ Сервисный токен выдается единожды администратором сервиса и не меняется.
+ В случае утечки следует получить новый сервисный токен.
+ Доменное имя боевого контура api.corp.proil.moscow
+ Доменное имя тестового контура alpha.corp.proil.moscow
+ Для тестового контура и боевого сервисные токены отличаются
+ Формат отправляемых данных может быть как json, так и с помощью формы


### Список существующих статусов заказа:

+ processing (заправщик выехал, авто забронировано)
+ closed_by_driver (закрыт заправщиком)
+ accepted (заказ создан)
+ not_available (машина неудачно припаркована/за шлагбаумом и тд)

### Методы для обращения к API Proil Business


#### Создание заявки на заправку

Метод: POST  
Путь: {domain name}/carshaing_api/orders/  
Пример тела запроса:
```json
{
    "car_id": "А788РЕ799",
    "lat": 55.124325873,
    "lon": 37.23847292,
    "fuel_type": "92"
}
```
Поддерживаемые типы топлива: 92, 95, 100, Diesel

Пример ответа сервера:
```json
{
    "order_id": "700bfca0-bbcc-4a3f-b58c-23f5c6299ab0",
    "success": true
}
```

Параметр order_id - уникальный uuid (идентификатор) заявки, генерируется при формировании заявки  


#### Удаление заявки на заправку
Метод: DELETE  
Путь: {domain name}/carshaing_api/orders/<order_id>/ 
Тело запроса пустое  
    
Пример ответа сервера:
```json
{
    "success": true
}
```  
Внимание! Заявки в статусе processing не удаляются.

#### Получение информации о текущих заявках
Метод: GET  
Путь: {domain name}/carshaing_api/orders/

Пример ответа сервера:
```json
{
    "orders": [
        {
            "order_id": "700bfca0-bbcc-4a3f-b58c-23f5c6299ab0",
            "car": {
                "car_id": "А788РЕ799",
                "fuel_type": "АИ-92"
            },
            "lat": 55.12342342,
            "lon": 37.65657657,
            "status": {
                "name": "processing",
                "display_name": "Заправщик выехал"
            }
        },
        {
            "order_id": "91059843-7940-4630-8046-45ca49360084",
            "car": {
                "car_id": "А999ОО799",
                "fuel_type": "АИ-92"
            },
            "lat": 55.58912342,
            "lon": 37.82657657,
            "status": {
                "name": "accepted",
                "display_name": "Новая заявка"
            }
        }
    ],
    "success": true
}

```
Данный метод стоит применять для периодической дополнительной синхронизации с сервисом.
Например после сбоев или других ошибок, как со стороны сервиса Proil Business, так и со стороны партнеров.
Также наш IT отдел готов поддержать тот формат интеграции, который удобен партнеру.  


### Методы для обращения к API партнеров
Следующие методы нужно реализовать на стороне сервиса партнеров для бронирования авто, открытия/закрытия центрального замка, отказа от заявки водителем после брони (сброса брони) и закрытия заявки со всей необходимой информацией и ссылками на фотографии.  
#### Заправщик выехал на заправку по заявке (начало брони заправляемого автомобиля)

Метод: POST  
Путь: на выбор партнера
    
Пример ответа сервера партнера:
```json
{
    "success": true
}
```

#### Заправщик отказался от заявки

Метод: POST  
Путь: на выбор партнера
    
Тело запроса:
```json
{
    "order_id": "e5059874-73ff-4fe0-b082-b333b69d967e",
    "message": "Причина отказа"
}
```
Пример ответа сервера:
```json
{
    "success": true
}
```
При успешном выполнении запроса, заявка удаляется из базы сервиса Proil Business и списка заявок при GET запросе от партнера соответсвенно. При 
этом партнер должен завершить бронь автомобиля.  
Если  автомобиль находится в зоне, где заправка физически невозможна, взымается штраф, а заказ остается в базе в статусе not_available.


#### Завершение заявки

Метод: POST  
Путь: на выбор партнера
    
Тело запроса:
```json
{
    "order_id": "e5059874-73ff-4fe0-b082-b333b69d967e",
    "fuel_price": "42.00",
    "fuel_volume": "31.00",
    "cost": "1302.00",
    "photos": "http://89.223.26.179:1111/corp-proil-point-image/637533_2020-01-29_19-14-27_order_picture_637533_98266265.jpg?AWSAccessKeyId=proil_moscow_2111&Expires=1580318243&Signature=BdllDUHvsdCg8M52y%2Btsgy2A5VU%3D,http://89.223.26.179:1111/corp-proil-point-image/637533_2020-01-29_19-14-27_order_picture_637533_98266265.jpg?AWSAccessKeyId=proil_moscow_2111&Expires=1580318243&Signature=BdllDUHvsdCg8M52y%2Btsgy2A5VU%3D"
}

```
Пример ответа сервера:
```json
{
    "success": true
}
```

#### Открытие центрального замка автомобиля

Метод: POST  
Путь: на выбор партнера
    
Тело запроса:
```json
{
    "order_id": "e5059874-73ff-4fe0-b082-b333b69d967e",
    "cmd": "open"
}

```

#### Закрытие центрального замка автомобиля

Метод: POST  
Путь: на выбор партнера
    
Тело запроса:
```json
{
    "order_id": "e5059874-73ff-4fe0-b082-b333b69d967e",
    "cmd": "close"
}

```