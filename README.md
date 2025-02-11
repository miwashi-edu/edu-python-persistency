# edu-python-persistency

[psycopg2](https://pypi.org/project/psycopg2/)  
[sql](https://www.w3schools.com/sql/default.asp)  
[sql bolt](https://sqlbolt.com)  

## Create PostgreSQL Machine

```bash
docker run -d --network iotnet --name postgres-server --hostname pg1 -e POSTGRES_USER=[userid] -e POSTGRES_PASSWORD=password -e POSTGRES_DB=iot_db postgres:latest
docker exec -it postgres-server psql -U [userid] -d iot_db
```

## Use database

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
