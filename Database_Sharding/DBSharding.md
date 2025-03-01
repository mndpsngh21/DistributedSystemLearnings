# PostgreSQL Sharding with Citus and Docker

## Step 1: Create `docker-compose.yml`
Create a `docker-compose.yml` file to set up a PostgreSQL Citus cluster with a coordinator and two worker nodes.

```yaml
version: '3.8'

services:
  coordinator:
    image: citusdata/citus:latest
    container_name: coordinator
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    networks:
      - citus_network

  worker1:
    image: citusdata/citus:latest
    container_name: worker1
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    networks:
      - citus_network

  worker2:
    image: citusdata/citus:latest
    container_name: worker2
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    networks:
      - citus_network

  manager:
    image: citusdata/membership-manager:latest
    container_name: manager
    depends_on:
      - coordinator
      - worker1
      - worker2
    networks:
      - citus_network

networks:
  citus_network:
```

## Step 2: Start the Cluster
Run the following command:
```sh
docker-compose up -d
```

## Step 3: Connect to PostgreSQL
```sh
docker exec -it coordinator psql -U postgres
```

## Step 4: Enable Citus and Add Workers
Inside the `psql` prompt:
```sql
CREATE EXTENSION IF NOT EXISTS citus;
SELECT * FROM citus_add_node('worker1', 5432);
SELECT * FROM citus_add_node('worker2', 5432);
SELECT * FROM citus_get_active_worker_nodes();
```

## Step 5: Create and Distribute a Table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);

SELECT create_distributed_table('users', 'id');
```

## Step 6: Insert and Query Data
```sql
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com');

SELECT * FROM users;
```

## How Data Distribution Works
Citus shards data across workers using a **distribution key**. The command below distributes data based on `id`:
```sql
SELECT create_distributed_table('users', 'id');
```
- Queries with `WHERE id = ?` go directly to a single worker.
- `SELECT * FROM users;` runs a parallel query across all workers.

---
This setup ensures PostgreSQL scales efficiently with Citus. 🚀

