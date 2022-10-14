# Python vs. PostgresML

## PostgresML

```
cargo pgx run --release
```

### Schema

```postgresql
CREATE TABLE public.flights_delay_mat (
    depdelayminutes real,
    year real NOT NULL,
    quarter real NOT NULL,
    month real NOT NULL,
    distance real NOT NULL,
    dayofweek real NOT NULL,
    dayofmonth real NOT NULL,
    flight_number_operating_airline real NOT NULL,
    originairportid real NOT NULL,
    destairportid real NOT NULL,
    tail_number real NOT NULL
);
```

### Data

```bash
curl -L -o ~/Desktop/flights.csv https://static.postgresml.org/benchmarks/flights.csv
```

```postgresql
\copy flights_delay_mat FROM '~/Desktop/flights.csv' CSV HEADER;

CREATE INDEX ON flights_delay_mat USING btree(originairportid, year, month, dayofmonth);
```

### Train

```postgresql
SELECT * FROM pgml.train(
	'r2',
	'regression',
	'flights_delay_mat',
	'depdelayminutes',
	algorithm => 'xgboost',
	hyperparams => '{ "n_estimators": 25 }'
);
```

### Test

```bash
pgbench -f pgbench.sql -p 28813 -h 127.0.0.1 pgml -t 10000 -c 1
```

## Python + Redis

### Setup

Inside virtualenv,

```bash
pip install -r requirements.txt
```

### Train

```bash
python train.py
```

### Feature Store

Install and start Redis if you don't have it already.

```bash
curl -L -o ~/Desktop/flights_sub.csv https://static.postgresml.org/benchmarks/flights_sub.csv
python load_redis.py
```

### Test

```bash
gunicorn predict:app -w 1
```

In a separate tab (install Apache Bench first):

```bash
ab -n 10000 -c 10 -T application/json -k -p ab.txt http://localhost:8000/
```


