# Парковка ТЦ
 
## Требования
!  В реальных условиях я бы просил уточнения требований, но так как  
   во-первых, это тестовое задание  
   во-вторых, писал на выходных и получение уточнений могло затянуться  
я позволил себе пофантазировать, прошу отнестись с пониманием, если ожидалось получить от меня какие-то вопросы.

```text
Реализовать REST АПИ и интерфейс оператора для системы парковки в торговом центре.

АПИ нужно для «открытия шлагбаумов» на въезд и выезд.

На входе номер автомобиля, на выходе можно или нет открывать шлагбаум.

В интерфейсе оператора – увидеть сколько мест на парковке сейчас занято.

Также необходимо иметь историю «посещений» для дальнейшей аналитики.
```

## API
Реализовал два API
+ простое
+ rest

### Simple Api

Зачем: потому что предполагаю, что шлагбаум не должен быть слишком "умным". И исходя из требования `АПИ нужно для «открытия шлагбаумов» на въезд и выезд.`, делаю вывод что должно быть возможно  
1. получить разрешение на открытие
2. зафиксировать заезд/выезд (полагаю что шлагбаум может открыться, но автомобиль все-таки не заехал - развернулся, сломался и т.д.) 

__методы__

GET `permission/:type/:vehicleNumber` (ожидаемые статусы: `200`)  
  получить разрешение на открытие, где  
  `:type` - тип действия (`in`- заезд, `out` - выезд)  
  `:vehicleNumber` - гос номер автомобиля

POST `action/:type/:vehicleNumber` (ожидаемые статусы: `200`, `403` если ТС не должно выполнить действие)  
  получить разрешение на открытие, где  
  `:type` - тип действия (`in`- заезд, `out` - выезд)  
  `:vehicleNumber` - гос номер автомобиля

### REST Api
Зачем: слово REST в требованиях все-таки может быть не просто так, поэтому создал несколько методов для доступа к ресурсам.

GET `/vehicles` 200, 204  
  Получить список авторизованных ТС.  
  Примерный ответ:
```json
{
  "payload": [
    {
      "id": 102,
      "number": "y000yy777",
      "isActive": true,
      "visits": []
    },
    {
      "id": 103,
      "number": "y001yy777",
      "isActive": false,
      "visits": []
    },
    {
      "id": 104,
      "number": "y002yy777",
      "isActive": true,
      "visits": [
        {
          "id": 98,
          "vehicle": 104,
          "createdAt": "2020-10-04T00:19:50+00:00",
          "closedAt": null
        }
      ]
    }
  ]
}
```
GET `/vehicle/:vehicleNumber` 200, 404  
    Получить информацию по определенному ТС по его гос. номеру, где  
  `:vehicleNumber` - гос номер автомобиля
  Примерный ответ:
  ```json
{
  "payload": {
    "id": 105,
    "number": "y003yy777",
    "isActive": true,
    "visits": [
      {
        "id": 99,
        "vehicle": 105,
        "createdAt": "2020-10-04T00:19:50+00:00",
        "closedAt": "2020-10-04T00:19:50+00:00"
      }
    ]
  }
}
```

GET `/vehicle/:vehicleNumber/visits` 200, 204  
  Получение списка "посещения" конкретного ТС, где  
  `:vehicleNumber` - гос номер автомобиля

```json
{
  "payload": [
    {
      "id": 99,
      "vehicle": {
        "id": 105,
        "number": "y003yy777",
        "isActive": true,
        "visits": [
          99
        ]
      },
      "createdAt": "2020-10-04T00:19:50+00:00",
      "closedAt": "2020-10-04T00:19:50+00:00"
    }
  ]
}
```
POST `/vehicle/:vehicleNumber/visits` 200, 404  
Сохранение "посещения", тело запроса может содержать только один параметр `closed_at` в формате `Y-m-d H:i:s`: 

```json
{
  "payload": {
    "id": 99,
    "vehicle": {
      "id": 105,
      "number": "y003yy777",
      "isActive": true,
      "visits": [
        99
      ]
    },
    "createdAt": "2020-10-04T00:19:50+00:00",
    "closedAt": "2020-10-04T00:19:50+00:00"
  }
}
```

POST `/vehicle/:vehicleNumber/visit/:visitId` 200, 404  
Изменение "посещения", тело запроса может содержать только один параметр `closed_at` в формате `Y-m-d H:i:s`. (Можно было сделать запрос PUT, PATCH, думаю не критично)

```json
{
  "payload": {
    "id": 99,
    "vehicle": {
      "id": 105,
      "number": "y003yy777",
      "isActive": true,
      "visits": [
        99
      ]
    },
    "createdAt": "2020-10-04T00:19:50+00:00",
    "closedAt": "2020-10-04T00:19:50+00:00"
  }
}
```

POST `/vehicle/:vehicleNumber/last-visit` 200, 404  
Получение последнего посещения ТС.

```json
{
  "payload": {
    "id": 99,
    "vehicle": {
      "id": 105,
      "number": "y003yy777",
      "isActive": true,
      "visits": [
        99
      ]
    },
    "createdAt": "2020-10-04T00:19:50+00:00",
    "closedAt": "2020-10-04T00:19:50+00:00"
  }
}
```

## UI оператора
Реализован полноценный CRUD для ТС и их "посещений". Например:

### Число занятых мест.
Число занятых мест вывел под заголовком списка всех посещений `/visit/` 
![alt text](https://github.com/smamykin/project_j3__proto/blob/main/readme_assets/vehicle_list.png?raw=true)

### История посещения
Добавил список посещений конкретного ТС на страницу детальной информации этого ТС.
![alt text](https://github.com/smamykin/project_j3__proto/blob/main/readme_assets/concreate_vehicle.png?raw=true)
