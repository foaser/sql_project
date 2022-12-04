# sql_project
## База данных для складского управления
### Схема базы данных
  ![Картинка базы данных](https://drive.google.com/uc?id=1x59jzmMItcwBY0PtwJDJdLzg7llnawt6)
  

  В базе данных содержится информация о складских процессах. Существуют склады `warehouse` со стеллажами `stack` для разных категорий товаров: еда, одежда, электроника, авто запчасти. На стеллажах расположены ячейки `cell`. В таблице `cell` указан этаж стелажа, а также статус ячейки: `FULL` или `EMPTY`.
  
  Также в базе данных хранится информация о товарах `product` на складе. Товар прибывает от поставщика `provider` и убывает в магазин `store`. Товар убывает определенным количеством – партиями `shipment`. В накладной `invoice` указывается основная информация об уходе партии. Таблица `transaction` содержит информацию о переводе денежным средств.   
### Запросы для примера:

1.  Посчитать стоимость партии для каждого магазина, создать столбец `sum` с результатом. 
```sql
SELECT s.name, sh.quantity, p.price, sh.quantity*p.price AS sum
FROM transaction
JOIN invoice i on i.id = transaction.id_invoice
JOIN store s on s.id = i.id_store
JOIN shipment sh on i.id = sh.id_invoice
JOIN product p on sh.id_product = p.id
```
Результат:
| name         | quantity | price  | sum    |
|--------------|----------|--------|--------|
| Pyatyorochka | 100      | 70     | 7000   |
| Pyatyorochka | 10       | 30     | 300    |
| Perecryostok | 7        | 150    | 1050   |
| Lenta        | 12       | 90     | 1080   |
| Eldorado     | 11       | 70000  | 770000 |
| Eldorado     | 5        | 130000 | 650000 |
| Euro auto    | 2        | 5000   | 10000  |
| Euro auto    | 1        | 2000   | 2000   |
| Bi-bi        | 5        | 500    | 2500   |


2. Узнать о наличии свободного места на каждом из складов, значения сохранить в столбце `available_space`.
```sql
SELECT w.id AS warehouse_id, w.capacity - sum(p.in_stock) AS avaliable_space
FROM product p
JOIN cell c ON c.id = p.id_cell
JOIN stack s ON c.id_stack = s.id
JOIN warehouse w ON s.id_warehouse = w.id
GROUP BY w.id
```
Результат:
| warehouse_id | avaliable_space |
|--------------|-----------------|
| 1            | 9531            |
| 3            | 990             |
| 4            | 1460            |
| 2            | 490             |


3. Вывести 10 пустых ячеек и стеллажи, на которых они расположены. Сортировать по этажам, начиная с первого. 
```sql
SELECT id_stack, stack_level, cell_status
FROM cell
WHERE cell_status = 'EMPTY'
ORDER BY stack_level
LIMIT 10
```
Результат:
| id_stack | stack_level | cell_status |
|----------|-------------|-------------|
| 25       | 1           | EMPTY       |
| 21       | 1           | EMPTY       |
| 25       | 1           | EMPTY       |
| 21       | 1           | EMPTY       |
| 17       | 2           | EMPTY       |
| 24       | 2           | EMPTY       |
| 24       | 2           | EMPTY       |
| 17       | 2           | EMPTY       |
| 44       | 2           | EMPTY       |

4. Вывести названия компании поставщика и магазина, между которыми проводилась транзакция.
```sql
SELECT pv.name AS provider, s.name AS store
FROM invoice
JOIN shipment sh on invoice.id = sh.id_invoice
JOIN product pd on pd.id = sh.id_product
JOIN store s on invoice.id_store = s.id
JOIN provider pv on pd.id_provider = pv.id
```
Результат:
| provider       | store        |
|----------------|--------------|
| Prostocvashino | Pyatyorochka |
| Karavay        | Pyatyorochka |
| Viborzhec      | Perecryostok |
| Karavay        | Lenta        |
| Electronika    | Eldorado     |
| Electronika    | Eldorado     |
| Shell          | Euro auto    |
| Marshall       | Euro auto    |
| Marshall       | Bi-bi        |


5. Найти телефоны поставщиков авто запчастей `auto parts`.
```sql
SELECT DISTINCT pv.name, pv.owner_name, pv.phone
FROM stack s
JOIN cell c on s.id = c.id_stack
JOIN product p on c.id = p.id_cell
JOIN provider pv on pv.id = p.id_provider
WHERE s.category = 'auto parts'
```
Результат:
| name     | owner_name  | phone   |
|----------|-------------|---------|
| Marshall | D. Nikulina | 6551342 |
| Michelin | A. Davis    | 4756543 |
| Shell    | J. Henrics  | 3552555 |
