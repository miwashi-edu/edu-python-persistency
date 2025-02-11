# edu-python-persistency

[psycopg2](https://pypi.org/project/psycopg2/)  
[sql](https://www.w3schools.com/sql/default.asp)  
[sql bolt](https://sqlbolt.com)  

[python redis](https://pypi.org/project/redis/)  
[redis docs](https://redis.io/docs/latest/)  

[pymongo](https://pypi.org/project/pymongo/)  
[mongodb docs](https://www.mongodb.com/docs/)  

[influxdb-client](https://pypi.org/project/influxdb-client/)  
[influx docs](https://docs.influxdata.com)  

## Create PostgreSQL server

```bash
docker run -d --network iotnet --name postgres-server --hostname pg1 -e POSTGRES_USER=[userid] -e POSTGRES_PASSWORD=password -e POSTGRES_DB=iot_db postgres:latest
docker exec -it postgres-server psql -U [userid] -d iot_db
```

## Cretate Redis server

```bash
docker run -d --network iotnet --name redis-server --hostname redis1 redis:latest
```

## Create MongoDB server

```bash
docker run -d --network iotnet --name mongodb-server --hostname mongodb1 mongo:latest
```



## Use PostgreSQL

```bash
cat > connect_db.py << 'EOF'
import psycopg2

def create_table():
    conn = psycopg2.connect(dbname='iot_db', user='[userid]', password='password', host='postgres-server')
    cur = conn.cursor()
    cur.execute('''CREATE TABLE IF NOT EXISTS sensor_data (
                    id SERIAL PRIMARY KEY,
                    sensor_name TEXT NOT NULL,
                    value FLOAT NOT NULL,
                    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
                ''')
    conn.commit()
    cur.close()
    conn.close()

def insert_data(sensor_name, value):
    conn = psycopg2.connect(dbname='iot_db', user='[userid]', password='password', host='postgres-server')
    cur = conn.cursor()
    cur.execute("INSERT INTO sensor_data (sensor_name, value) VALUES (%s, %s);", (sensor_name, value))
    conn.commit()
    cur.close()
    conn.close()

def get_data():
    conn = psycopg2.connect(dbname='iot_db', user='[userid]', password='password', host='postgres-server')
    cur = conn.cursor()
    cur.execute("SELECT * FROM sensor_data;")
    rows = cur.fetchall()
    for row in rows:
        print(row)
    cur.close()
    conn.close()

if __name__ == "__main__":
    create_table()
    insert_data('temperature', 22.5)
    get_data()
EOF
```


## Use Redis 

```bash
cat > connect_redis.py << 'EOF'
import redis

def connect_redis():
    return redis.Redis(host='redis-server', port=6379, decode_responses=True)

def set_value(key, value):
    r = connect_redis()
    r.set(key, value)
    print(f"Set {key}: {value}")

def get_value(key):
    r = connect_redis()
    value = r.get(key)
    print(f"Get {key}: {value}")
    return value

def main():
    set_value('temperature', '22.5')
    get_value('temperature')

if __name__ == "__main__":
    main()
EOF
```

## Use MongoDB

```bash
cat > connect_mongo.py << 'EOF'
from pymongo import MongoClient

def connect_mongo():
    client = MongoClient('mongodb-server', 27017)
    return client['iot_db']

def insert_data(sensor_name, value):
    db = connect_mongo()
    db.sensor_data.insert_one({"sensor_name": sensor_name, "value": value})
    print(f"Inserted {sensor_name}: {value}")

def get_data():
    db = connect_mongo()
    for doc in db.sensor_data.find():
        print(doc)

def main():
    insert_data('temperature', 22.5)
    get_data()

if __name__ == "__main__":
    main()
EOF
```

## Use InfluxDB

```bash
cat > connect_influx.py << 'EOF'
from influxdb_client import InfluxDBClient, Point, WritePrecision

def connect_influx():
    return InfluxDBClient(url="http://influxdb-server:8086", token="password", org="iot_org")

def insert_data(sensor_name, value):
    client = connect_influx()
    write_api = client.write_api(write_options=WritePrecision.NS)
    point = Point("sensor_data").tag("sensor_name", sensor_name).field("value", value)
    write_api.write(bucket="iot_db", record=point)
    print(f"Inserted {sensor_name}: {value}")

def get_data():
    client = connect_influx()
    query_api = client.query_api()
    query = 'from(bucket: "iot_db") |> range(start: -1h)'
    tables = query_api.query(query, org="iot_org")
    for table in tables:
        for record in table.records:
            print(record)

def main():
    insert_data('temperature', 22.5)
    get_data()

if __name__ == "__main__":
    main()
EOF
```

