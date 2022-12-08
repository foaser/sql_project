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
FROM invoice i
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

Написать запрос, в котором будет указан магазин, поставщик, количество товара, поставленного от этого поставщика за предыдущие 30 дней (начиная с 8 ноября 2022 года). 

```sql
Select s.name, invoice.date, p2.name, s2.quantity
from invoice
join store s on invoice.id_store = s.id
join shipment s2 on s2.id = invoice.id_shipment
join product p on s2.id_product = p.id
join provider p2 on p.id_provider = p2.id
where date >= (date '2022-11-08')
```

Проверить, есть ли у нас этаж стеллажа с полностью пустыми ячейками. 

```sql
select s.id as stack_id, c.stack_level as level
from stack s
join cell c on s.id = c.id_stack
group by s.id, c.stack_level
having count(c.id) = sum(case cell_status when 'EMPTY' then 1 else 0 end)
order by s.id;
```

Составить отчёт по дням, сколько партий товара в день убывало с 7 по 17 сентября. 

```sql
Select invoice.date, count(s.quantity)
from invoice
join shipment s on invoice.id = s.id_invoice
where invoice.date >= (date '2022-09-07') and invoice.date <= (date '2022-09-17')
group by invoice.date
```

Найти поставщика, который больше всего нам поставляет товара в магазины.

```sql
select shipment.quantity, p2.name
from shipment
join product p on shipment.id_product = p.id
join provider p2 on p.id_provider = p2.id
order by shipment.quantity desc
limit 1
```
Найти магазин, в который прибыло меньше всего товара с ценой от 50 до 500 рублей с 1 сентября 2022 года по 30 ноября 2022 года.

```sql
select s.name, sum(quantity) as sum
from shipment
join invoice i on shipment.id = i.id_shipment
join store s on i.id_store = s.id
join product p on shipment.id_product = p.id
where i.date >= (date '2022-09-01')
  and i.date <= (date '2022-11-30')
  and p.price >= 50
  and p.price <= 500
group by s.name
order by sum
limit 1
```

Найти месяц, в который убывала самая большая партия пачек печенья в магазины.

```sql
select s.name, p.name, date, s2.quantity from invoice
join store s on invoice.id_store = s.id
join shipment s2 on invoice.id = s2.id_invoice
join product p on s2.id_product = p.id
where p.name = 'cookies'
order by quantity
limit 1
```
### Выполнили:
- Фомина Арина
- Глезин Илья
