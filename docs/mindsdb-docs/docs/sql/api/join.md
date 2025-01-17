# `#!sql JOIN` Statement

## Description

The `#!sql JOIN` clause is used to combine rows from the database table and the model table on a related column. This can be very helpful to get bulk predictions. The basic syntax for joining from the data table and model is shown below.

## Syntax

```sql
SELECT t.[column_name], p.[column_name] ...
FROM [integration_name].[table] AS t
JOIN mindsdb.[predictor_name] AS p;
```

On execution, we get:

```sql
+-----------------+-----------------+
| t.[column_name] | p.[column_name] |
+-----------------+-----------------+
| t.[value]       | p.[value]       |
+-----------------+-----------------+
```

Where:

| Name                                | Description                                              |
| ----------------------------------- | -------------------------------------------------------- |
| `[integration_name].[table]`        | Name of the table te be used as input for the prediction |
| `mindsdb.[predictor_name]`          | Name of the model to be used to make predictions         |
| `p.value`                           | Predicted value                                          |

## Example

The following SQL statement joins the `home_rentals` data with the `home_rentals_model` on the predicted price:

```sql
SELECT t.rental_price AS real_price, 
       m.rental_price AS predicted_price,
       t.number_of_rooms,  t.number_of_bathrooms, t.sqft, t.location, t.days_on_market 
FROM example_db.demo_data.home_rentals AS t 
JOIN mindsdb.home_rentals_model AS m 
LIMIT 100;
```

On execution, we get:

```sql
+------------+-----------------+-----------------+---------------------+------+----------+----------------+
| real_price | predicted_price | number_of_rooms | number_of_bathrooms | sqft | location | days_on_market |
+------------+-----------------+-----------------+---------------------+------+----------+----------------+
| 3901       | 3886            | 2               | 1                   | 917  | great    | 13             |
| 2042       | 2007            | 0               | 1                   | 194  | great    | 10             |
| 1871       | 1865            | 1               | 1                   | 543  | poor     | 18             |
| 3026       | 3020            | 2               | 1                   | 503  | good     | 10             |
| 4774       | 4748            | 3               | 2                   | 1066 | good     | 13             |
| 4382       | 4388            | 3               | 2                   | 816  | poor     | 25             |
| 2269       | 2272            | 0               | 1                   | 461  | great    | 6              |
| 2284       | 2272            | 1               | 1                   | 333  | great    | 6              |
| 5420       | 5437            | 3               | 2                   | 1124 | great    | 9              |
| 5016       | 4998            | 3               | 2                   | 1204 | good     | 7              |
| 1421       | 1427            | 0               | 1                   | 538  | poor     | 43             |
| 3476       | 3466            | 2               | 1                   | 890  | good     | 6              |
| 5271       | 5255            | 3               | 2                   | 975  | great    | 6              |
| 3001       | 2993            | 2               | 1                   | 564  | good     | 13             |
| 4682       | 4692            | 3               | 2                   | 953  | good     | 10             |
| 1783       | 1738            | 1               | 1                   | 493  | poor     | 24             |
| 1548       | 1543            | 1               | 1                   | 601  | poor     | 47             |
| 1492       | 1491            | 0               | 1                   | 191  | good     | 12             |
| 2431       | 2419            | 0               | 1                   | 511  | great    | 1              |
| 4237       | 4257            | 3               | 2                   | 916  | poor     | 36             |
+------------+-----------------+-----------------+---------------------+------+----------+----------------+
```

## Example Time Series

Let's create and train a time series predictor using this statement:

```sql
CREATE PREDICTOR mindsdb.house_sales_model
FROM example_db
  (SELECT * FROM demo_data.house_sales)
PREDICT MA
ORDER BY saledate
GROUP BY bedrooms, type
-- as the target is quarterly, we will look back two years to forecast the next one
WINDOW 8
HORIZON 4;  
```

On execution, we get:

```sql
Query OK, 0 row(s) updated - xxxms
```

Now, you can query it to get the predictions like this:

```sql
SELECT m.saledate as date,
    m.ma AS forecast
FROM mindsdb.house_sales_model AS m 
JOIN example_db.demo_data.house_sales AS t
WHERE t.saledate > LATEST
AND t.type = 'house'
LIMIT 4;
```

On execution, we get:

```sql
+----------+------------------+
|date      |forecast          |
+----------+------------------+
|2019-12-31|517506.31349071994|
|2019-12-31|627822.6592658638 |
|2019-12-31|953426.9545788583 |
|2019-12-31|767252.4205039773 |
+----------+------------------+
```
