## Логічний порядок виконання оновлення:
#### 1. Робимо запит на авторизацію
https://tcomp.com.ua/api/auth?username=:username&password=:password
отримуємо куку для нашого користувача, надалі використовуємо заголовок в кожному запиті: `PHPSESSID=abcdfoobar123`

#### 2. Задля кращого UX забираємо загальний список кодів продуктів (перша частина артикула) з сайту,
Ми зможемо працювати по артикулах субпродуктів та виводити у процесі роботи прогрес синхронізації для користувача аналогічно з логуванням:
https://tcomp.com.ua/api/products?limit=9999

отримуємо відповідь:

```
{
    "results": [
        {
            "productcode": "61-412-0",
            "totalitems": "60",
            "totalinstock": "974"
        },
        {
            "productcode": "61-200-0",
            "totalitems": "55",
            "totalinstock": "1019"
        },
        ...
    ],
    "total": 323
}
```

#### 3. Для кожного продукту у списку виконуємо запит наявності:

https://tcomp.com.ua/api/stockitems?productcode=61-412-0

отримуємо відповідь:

```
{
    "results": [
        {
            "id": 1,
            "productcode": "61-412-0",
            "productcolor": "30",
            "productsize": "s",
            "price1": 2.24,
            "price2": 1.98,
            "price3": 1.89,
            "price4": 1.75,
            "quantity": 77,
            "discount": 0,
            "currentPrice": 2.24
        },
        {
            "id": 2,
            "productcode": "61-412-0",
            "productcolor": "30",
            "productsize": "m",
            "price1": 2.24,
            "price2": 1.98,
            "price3": 1.89,
            "price4": 1.75,
            "quantity": 42,
            "discount": 0,
            "currentPrice": 2.24
        },
        ...
    ],
    "total": 60
}
```

#### 4. Порівнюємо отриманий сток з данними 1С, тільки для артикулів що відрізняються виконуємо запити на редагування:

`PUT` -> https://tcomp.com.ua/api/stockitems?productcode=61-412-0&productcolor=30&productsize=s&price1=2.24&price2=1.98&price3=1.89&price4=1.75&quantity=77&discount=0

отримуємо відповідь `http 200`
```
{
    "success": true,
    "message": "",
    "object": {
        "id": 1,
        "productcode": "61-412-0",
        "productcolor": "30",
        "productsize": "s",
        "price1": 2.24,
        "price2": 1.98,
        "price3": 1.89,
        "price4": 1.75,
        "quantity": 77,
        "discount": 0
    },
    "code": 200
}
```

якщо у 1С є артикули даного продукту (`61-412-0`) яких немає на сайті - виконуємо запит `POST` на створення (абсолютно аналогічно):

`POST` -> https://tcomp.com.ua/api/stockitems?productcode=61-412-0&productcolor=30&productsize=s&price1=2.24&price2=1.98&price3=1.89&price4=1.75&quantity=77&discount=0

 для видалення:

`DELETE` ->https://tcomp.com.ua/api/stockitems?productcode=61-412-0&productcolor=999&productsize=ZZ




> Бенчмарк показує, що при умові необхідності редагування 2000 артикулів весь процес повинен зайняти біля 60с.
забір і редагування всіх артикулів займає біля 6хв
