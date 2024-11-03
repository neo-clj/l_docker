In this guide, you’ll practice creating and using volumes to persist data created by a Postgres container. When the database runs, it stores files into the `/var/lib/postgresql/data` directory. By attaching the volume here, you will be able to restart the container multiple times while keeping the data.

Start a container using the Postgres image with the following command:

```
docker run --name=db -e POSTGRES_PASSWORD=secret -d -v postgres_data:/var/lib/postgresql/data postgres
```

This will start the database in the background, configure it with a password, and attach a volume to the directory PostgreSQL will persist the database files.

---

Connect to the database by using the following command:

```
docker exec -ti db psql -U postgres
```

---

In the PostgreSQL command line, run the following to create a database table and insert two records:

```postgres
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    description VARCHAR(100)
);
INSERT INTO tasks (description) VALUES ('Finish work'), ('Have fun');
```

---

Verify the data is in the database by running the following in the PostgreSQL command line:

```postgres
SELECT * FROM tasks;
```

You should get output that looks like the following:

```
 id | description
----+-------------
  1 | Finish work
  2 | Have fun
(2 rows)
```

---

Exit out of the PostgreSQL shell by running the following command:

```
\q
```

---

Stop and remove the database container. Remember that, even though the container has been deleted, the data is persisted in the `postgres_data` volume.

```
docker stop db
docker rm db
```

---

Start a new container by running the following command, attaching the same volume with the persisted data:

```
docker run --name=new-db -d -v postgres_data:/var/lib/postgresql/data postgres
```

You might have noticed that the `POSTGRES_PASSWORD` environment variable has been omitted. That’s because that variable is only used when ***bootstrapping a new database***.

---

Verify the database still has the records by running the following command:

```
docker exec -ti new-db psql -U postgres -c "SELECT * FROM tasks"
```

# Remove volumes

Before removing a volume, it must not be attached to any containers. If you haven’t removed the previous container, do so with the following command (the `-f` will ***stop the container first and then remove*** it):

```
docker rm -f new-db
```

There are a few methods to remove volumes, including the following:

Use the `docker volume rm` command:

```
docker volume rm postgres_data
```

Use the `docker volume prune` command to remove all unused volumes:

```
docker volume prune
```
