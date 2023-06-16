

```sql

CREATE TABLE sales (
  customer_id VARCHAR(1) NOT NULL,
  order_date DATE NOT NULL,
  product_id INTEGER NOT NULL,

  PRIMARY KEY (customer_id, order_date),
  FOREIGN KEY (product_id) REFERENCES menu (product_id)
);

INSERT INTO sales (customer_id, order_date, product_id) VALUES
('A', '2021-01-01', 1),
('A', '2021-01-01', 2),
('A', '2021-01-07', 2),
('A', '2021-01-10', 3),
('A', '2021-01-11', 3),
('A', '2021-01-11', 3),
('B', '2021-01-01', 2),
('B', '2021-01-02', 2),
('B', '2021-01-04', 1),
('B', '2021-01-11', 1),
('B', '2021-01-16', 3),
('B', '2021-02-01', 3),
('C', '2021-01-01', 3),
('C', '2021-01-01', 3),
('C', '2021-01-07', 3);

CREATE TABLE menu (
  product_id INTEGER NOT NULL,
  product_name VARCHAR(5) NOT NULL,
  price INTEGER NOT NULL,

  PRIMARY KEY (product_id)
);

INSERT INTO menu (product_id, product_name, price) VALUES
(1, 'sushi', 10),
(2, 'curry', 15),
(3, 'ramen', 12);

CREATE TABLE members (
  customer_id VARCHAR(1) NOT NULL,
  join_date DATE NOT NULL,

  PRIMARY KEY (customer_id)
);

INSERT INTO members (customer_id, join_date) VALUES
('A', '2021-01-07'),
('B', '2021-01-09');




```