## База данных для складского управления
### Схема базы данных
  ![Картинка базы данных](https://drive.google.com/uc?id=1x59jzmMItcwBY0PtwJDJdLzg7llnawt6)
  

  В базе данных содержится информация о складских процессах. Существуют склады `warehouse` со стеллажами `stack` для разных категорий товаров: еда, одежда, электроника, авто запчасти. На стеллажах расположены ячейки `cell`. В таблице `cell` указан этаж стелажа, а также статус ячейки: `FULL` или `EMPTY`.
  
  Также в базе данных хранится информация о товарах `product` на складе. Товар прибывает от поставщика `provider` и убывает в магазин `store`. Товар убывает определенным количеством – партиями `shipment`. В накладной `invoice` указывается основная информация об уходе партии. Таблица `transaction` содержит информацию о переводе денежным средств.   
### Запросы для примера:

1.  Посчитать стоимость партии для каждого магазина, создать столбец `sum` с результатом в таблице `transaction`.
```sql
SELECT sh.quantity, p.price, sh.quantity*p.price AS sum
FROM transaction
JOIN invoice i on i.id = transaction.id_invoice
JOIN store s on s.id = i.id_store
JOIN shipment sh on i.id = sh.id_invoice
JOIN product p on sh.id_product = p.id
```
Результат:

2. Узнать о наличии свободного места на каждом из складов, значения сохранить в столбце `available_space`.
```sql
SELECT w.id AS warehouse_is, w.capacity - sum(p.in_stock) AS avaliable_space
FROM product p
JOIN cell c ON c.id = p.id_cell
JOIN stack s ON c.id_stack = s.id
JOIN warehouse w ON s.id_warehouse = w.id
GROUP BY w.id
```
Результат:


3. Вывести все пустые ячейки и стеллажи, на которых они расположены. Сортировать по этажам, начиная с первого. 
```sql
SELECT id_stack, stack_level, cell_status
FROM cell
WHERE cell_status = 'EMPTY'
ORDER BY stack_level
```
Результат:

4. Вывести названия компании поставщика и магазина, между которыми проводится транзакция.
```sql
SELECT pv.name, s.name
FROM invoice
JOIN shipment sh on invoice.id = sh.id_invoice
JOIN product pd on pd.id = sh.id_product
JOIN store s on invoice.id_store = s.id
JOIN provider pv on pd.id_provider = pv.id
```
Результат:

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
