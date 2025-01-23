# Тестирование интернет магазина мужской одежды в Postman
С помощью Postman необходимо произвести нагрузочное тестировани критического пути для сайта [20line](https://20line.ru).

## Результаты тестирования
Экспортированная коллекция запросов Postman находится в [json-файле](https://github.com/QA-Abu-Dhabi/20line.ru-in-Postman/blob/main/20line.ru.postman_collection.json).  
Отчёт о нагрузочном тестировании представлен в [pdf-файле](https://github.com/QA-Abu-Dhabi/20line.ru-in-Postman/blob/main/20line.ru-performance-report-1.pdf).  
В ходе нагрузочного тестирования возникли ошибки **500 Internal Server Error**.

## 1. Открытие главной страницы
Отправляется GET-запрос на URL главной страницы сайта https://20line.ru.  
Возвращается HTML-код главной страницы, статус код 200.

## 2. Открытие страницы товара
Отправляется GET-запрос на получение страницы с товаром https://20line.ru/catalog/futbolka-iz-khlopka-20line-1820362.html.  
В ответ приходит HTML-код страницы с товаром, статус код 200.  
Для получения CSRF-токена, хранимого в **bitrix_sessid** добавим во вкладку **Scripts/Post-response** следующий код:
```javascript
var responseText = pm.response.text();
var match = responseText.match(/"bitrix_sessid":"(.*?)"/);
if (match) {
    var bitrix_sessid = match[1];
    pm.collectionVariables.set("BX_SessId", bitrix_sessid)
    console.log("BX_SessId найден:", bitrix_sessid);
} else {
    console.log("Не удалось найти BX.message в ответе");
}
```
Переменная **BX_SessId** сохраняется в качестве переменной коллекции.

## 3. Добавление товара в корзину
Направляется post-запрос на URL: https://20line.ru/bitrix/services/main/ajax.php.  
В заголовке указывается обязательно **x-bitrix-csrf-token**.  
В теле запроса передается id товара для добавления в корзину.  
Возвращается ответ с успешным добавлением товара в корзину и статус код 200.  

## 4. Переход в корзину
Направляется GET-запрос на URL: https://20line.ru/basket.  
В заголовке указывается обязательно **x-bitrix-csrf-token**.  
Возвращается HTML-код страницы, после чего с помощью **post-response** выполняется проверка наличия товара в корзине и получение id элемента корзины, необходимый для последующего удаления:
```javascript
var responseText = pm.response.text();
var itemInBasket = responseText.includes('data-offer-id="1820541"');
if (itemInBasket) {
    pm.test("Товар присутствует в корзине", function() {
        console.log("Товар с data-offer-id=1820541 найден в корзине.");
    });
} else {
    pm.test("Товар отсутствует в корзине", function() {
        console.log("Товар с data-offer-id=1820541 не найден в корзине.");
    });
}
var match = responseText.match(/removeItemFromCart\((\d+)\)/);
console.log(match);
if (match) {
    var remove_id = match[1];
    pm.collectionVariables.set("removeItemFromCart", remove_id)
    console.log("removeItemFromCart найден:", remove_id);
} else {
    console.log("Не удалось найти removeItemFromCart в ответе");
}
```

## 5. Удаление товара из корзины
Направляется post-запрос на URL: https://20line.ru/bitrix/services/main/ajax.php?mode=class&c=ega%3Abasket&action=deleteItem.  
В заголовке указывается обязательно **x-bitrix-csrf-token**.  
В теле запроса передается id элемента корзины.  
Возвращается ответ с успешным удалением товара с соответствующим id и статус код 200.  

## 6. Проверка корзины на отсутствие товаров
Направляется GET-запрос на URL: https://20line.ru/basket.  
Возвращается HTML-код страницы, скрипт в **post-response** выполняет проверку отсутствия товара в корзине.




