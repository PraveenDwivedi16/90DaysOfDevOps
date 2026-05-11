# Day 32 – Docker Volumes & Networking
## Goal
Today's goal was to solve two real Docker problems:
1. **Data persistence**
2. **Container communication**

Containers are temporary by default. If a container is removed, data written inside the container can be lost. 
Also, containers need a proper network to communicate reliably with each other.

# Topics Covered

- Why containers are ephemeral
- Why database data is lost without volumes
- Named Docker volumes
- Bind mounts
- Difference between named volumes and bind mounts
- Docker default networks
- Default bridge network behavior
- Custom bridge networks
- Container communication by IP
- Container communication by name
- MySQL container with persistent volume
- App container connecting to MySQL by container name
- Basic, scenario-based, and advanced interview questions

# 1. Why Containers Lose Data
Docker containers are ephemeral.
This means:
> A container is temporary. If we remove the container, the writable layer of that container is deleted.
Docker image layers are read-only. When a container starts, Docker adds a writable layer on top.

Container Writable Layer   ← temporary data
------------------------
Image Layer 3
Image Layer 2
Image Layer 1

If the container is removed:
Container writable layer is deleted
So if a database stores data only inside the container writable layer, that data will be lost when the container is removed.

# 2. Why Volumes Are Needed

Databases like MySQL and Postgres store data inside specific folders.

For MySQL:
/var/lib/mysql

For Postgres:
/var/lib/postgresql/data

If we do not mount a volume to these folders, the data stays inside the container writable layer.

When the container is removed, data is lost.

To persist data, we use Docker volumes.

# 3. Docker Volumes

A Docker volume is persistent storage managed by Docker.

Create volume:
docker volume create mysql-data

Use volume with MySQL:
docker run -d --name mysql-with-volume \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

Meaning:
Docker named volume mysql-data
        ↓
Mounted inside container at /var/lib/mysql

If the container is removed:

Container removed ❌
Volume still exists ✅
Data still exists ✅

# 4. Bind Mounts
A bind mount maps a specific host machine folder into a container folder.

Example:
docker run -d --name nginx-bind \
  -p 8084:80 \
  -v /root/day32-bind:/usr/share/nginx/html \
  nginx:alpine

Meaning:
Host folder:      /root/day32-bind
Container folder: /usr/share/nginx/html

If we edit `index.html` on the host, the container immediately sees the change.

Bind mounts are useful for local development and live file editing.

# 5. Named Volume vs Bind Mount

| Point | Named Volume | Bind Mount |
|---|---|---|
| Managed by | Docker | User / Host OS |
| Host path | Docker decides | User decides |
| Best for | Database persistence | Local development |
| Portability | More portable | Depends on host path |
| Example | `mysql-data:/var/lib/mysql` | `/root/site:/usr/share/nginx/html` |
| Production use | Common for persistent data | Used carefully |
| Security | Safer because Docker manages path | Host path exposure risk |

## Simple Explanation
Named volume:
Docker manages storage location.
Best for database data.

Bind mount:
User gives exact host folder.
Best for live editing and development.

# 6. Docker Networking
Docker networking allows containers to communicate.
List networks:
docker network ls
Common default networks:
bridge
host
none

## bridge network

Default network used when no network is specified.

Containers on the default bridge can usually communicate by IP, but name-based communication does not work automatically.


## host network
Container shares host network directly.

Used in special cases.

## none network
Container gets no network access.

Useful for isolated workloads.

# 7. Custom Bridge Network
A custom bridge network allows containers to communicate by container name.

Create custom network:

docker network create my-app-net

Run containers on it:

docker run -d --name app1 --network my-app-net alpine sleep 1000
docker run -d --name app2 --network my-app-net alpine sleep 1000

Ping by name:
docker exec app1 ping -c 3 app2

This works because Docker provides embedded DNS for containers on custom bridge networks.


# 8. Why Custom Network is Important
In real applications, we should not use container IP addresses because they can change.

Bad:
env
DB_HOST=172.18.0.3

Good:
env
DB_HOST=mysql-db
When using custom Docker networks, Docker resolves container names automatically.

Example:
text
app container → mysql-db

# Task 1 – The Problem: Data Loss Without Volume

## Goal

Run a database container without volume, create data, remove the container, and verify data is lost.

We used MySQL for easier understanding.

## Step 1: Run MySQL Without Volume
docker rm -f mysql-no-volume

docker run -d --name mysql-no-volume \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  mysql:8.0

Check container:
docker ps
Check logs:
docker logs mysql-no-volume
Wait until MySQL shows:
ready for connections

## Step 2: Enter MySQL
docker exec -it mysql-no-volume mysql -uroot -prootpass demo

Important:

There is no space between `-p` and the password:
-prootpass

## Step 3: Create Table and Insert Data

Inside MySQL:
sql
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);

INSERT INTO students (name) VALUES ('Hardik'), ('DevOps');

SELECT * FROM students;

Expected result:
+----+--------+
| id | name   |
+----+--------+
|  1 | Hardik |
|  2 | DevOps |
+----+--------+
Exit:
sql
exit;

## Step 4: Stop and Remove Container
bash
docker stop mysql-no-volume
docker rm mysql-no-volume

## Step 5: Run New MySQL Container Without Volume
docker run -d --name mysql-no-volume \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  mysql:8.0

Enter MySQL:
docker exec -it mysql-no-volume mysql -uroot -prootpass demo

Run:
sql
SELECT * FROM students;

Observed output:
ERROR 1146 (42S02): Table 'demo.students' doesn't exist

## Task 1 Result

The `students` table was lost after removing the container.

This happened because MySQL data was stored inside the container writable layer.

When the container was removed, that writable layer was deleted.

Conclusion:

> Without volumes, database data is not persistent.


## Extra Learning: No Database Selected Error

Observed error:
ERROR 1046 (3D000): No database selected

Reason:
MySQL shell was opened without selecting the database.

Fix:
USE demo;
SELECT * FROM students;

or enter directly with database name:
docker exec -it mysql-no-volume mysql -uroot -prootpass demo

# Task 2 – Named Volumes

## Goal
Use a named volume with MySQL and verify that data survives container deletion.


## Step 1: Create Named Volume
docker volume create mysql-data

Verify:
docker volume ls

## Step 2: Run MySQL With Named Volume
docker run -d --name mysql-with-volume \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0


Explanation:
mysql-data:/var/lib/mysql

means:
Docker volume mysql-data → MySQL data directory

## Step 3: Create Table and Insert Data
docker exec -it mysql-with-volume mysql -uroot -prootpass demo

Inside MySQL:
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);

INSERT INTO students (name) VALUES ('Hardik'), ('DevOps'), ('Docker Volume');

SELECT * FROM students;

Expected:
+----+---------------+
| id | name          |
+----+---------------+
|  1 | Hardik        |
|  2 | DevOps        |
|  3 | Docker Volume |
+----+---------------+
Exit:
exit;

## Step 4: Stop and Remove Container
docker stop mysql-with-volume
docker rm mysql-with-volume

The container is removed, but the volume still exists.

Verify:
docker volume ls

## Step 5: Create New Container With Same Volume
docker run -d --name mysql-with-volume-new \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

Enter MySQL:
docker exec -it mysql-with-volume-new mysql -uroot -prootpass demo
Run:
SELECT * FROM students;

Observed:
+----+---------------+
| id | name          |
+----+---------------+
|  1 | Hardik        |
|  2 | DevOps        |
|  3 | Docker Volume |
+----+---------------+
3 rows in set (0.00 sec)

## Step 6: Inspect Volume
docker volume inspect mysql-data
```
Observed:
json
[
    {
        "CreatedAt": "2026-05-11T07:42:24Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql-data/_data",
        "Name": "mysql-data",
        "Options": null,
        "Scope": "local"
    }
]

## Task 2 Result

The data was still available after removing and recreating the container.

This proves:

> Named volumes persist data even after containers are deleted.

# Task 3 – Bind Mounts

## Goal

Use a host folder as the source of files for an Nginx container.

## Step 1: Create Host Folder
docker rm -f nginx-bind

mkdir -p /root/day32-bind
cd /root/day32-bind

Create `index.html`:

cat > index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Docker Bind Mount Demo</title>
</head>
<body>
    <h1>Version 1: Hello from Bind Mount</h1>
    <p>This file is stored on the host machine.</p>
    <p>Nginx container is reading it through a bind mount.</p>
</body>
</html>
EOF

## Step 2: Run Nginx With Bind Mount
docker run -d --name nginx-bind \
  -p 8084:80 \
  -v /root/day32-bind:/usr/share/nginx/html \
  nginx:alpine

Explanation:
/root/day32-bind                  host folder
/usr/share/nginx/html             container folder

Nginx serves files from:

/usr/share/nginx/html

## Step 3: Access Page

From server:
curl http://localhost:8084

If using browser from outside EC2:
http://YOUR_EC2_PUBLIC_IP:8084

Security group must allow inbound TCP port `8084`.

## Step 4: Edit File on Host
cat > /root/day32-bind/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Docker Bind Mount Demo</title>
</head>
<body>
    <h1>Version 2: Updated from Host Machine</h1>
    <p>I edited index.html on the EC2 host.</p>
    <p>The Nginx container shows the updated file immediately.</p>
</body>
</html>
EOF

Run:
curl http://localhost:8084

Observed:
<h1>Version 2: Updated from Host Machine</h1>


## Step 5: Inspect Bind Mount

docker inspect -f '{{json .Mounts}}' nginx-bind

Observed:

```json
[
  {
    "Type": "bind",
    "Source": "/root/day32-bind",
    "Destination": "/usr/share/nginx/html",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  }
]


## Task 3 Result
The page changed immediately after editing `index.html` on the host.
This proves:
> Bind mounts connect a specific host folder directly to a container folder.
No image rebuild was needed.


# Task 4 – Docker Networking Basics
## Goal
Understand default Docker bridge networking.

## Step 1: List Docker Networks

docker network ls
Expected networks:
bridge
host
none

## Step 2: Inspect Default Bridge
docker network inspect bridge
Filtered:
docker network inspect bridge --format '{{json .IPAM.Config}}'
docker network inspect bridge --format '{{json .Containers}}'

## Step 3: Run Two Containers on Default Bridge
docker rm -f net-default-1 net-default-2

docker run -d --name net-default-1 alpine sleep 1000
docker run -d --name net-default-2 alpine sleep 1000

## Step 4: Find IP Addresses
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' net-default-1
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' net-default-2

Observed:
net-default-1: 172.17.0.4
net-default-2: 172.17.0.5


## Step 5: Ping by Name
docker exec net-default-1 ping -c 3 net-default-2
Observed:
ping: bad address 'net-default-2'

This means default bridge did not resolve container name.

## Step 6: Ping by IP
docker exec net-default-1 ping -c 3 172.17.0.5
Observed:
3 packets transmitted, 3 packets received, 0% packet loss

## Task 4 Result
On default bridge:
Ping by name: failed
Ping by IP: worked
Conclusion:

> Containers on the default bridge can communicate by IP, but name-based communication does not work automatically.

# Task 5 – Custom Networks

## Goal
Create a custom bridge network and verify name-based communication.

## Step 1: Create Custom Network
docker rm -f app1 app2
docker network rm my-app-net

docker network create my-app-net


## Step 2: Run Two Containers on Custom Network
docker run -d --name app1 --network my-app-net alpine sleep 1000
docker run -d --name app2 --network my-app-net alpine sleep 1000

## Step 3: Ping by Name
docker exec app1 ping -c 3 app2

Observed:
PING app2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.193 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.067 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.076 ms

3 packets transmitted, 3 packets received, 0% packet loss


## Step 4: Ping by IP
docker exec app1 ping -c 3 172.18.0.3

Observed:
3 packets transmitted, 3 packets received, 0% packet loss

## Step 5: Inspect Custom Network
docker network inspect my-app-net --format '{{json .Containers}}'

Observed containers:
app1 → 172.18.0.2/16
app2 → 172.18.0.3/16

## Task 5 Result
On custom bridge network:
Ping by name: worked
Ping by IP: worked
Conclusion:
> Custom bridge networks provide Docker DNS-based name resolution.

# Why Custom Network Allows Name-Based Communication

Docker provides an embedded DNS server for user-defined bridge networks.

So when `app1` tries to reach:
app2
Docker resolves:
app2 → 172.18.0.3
Default bridge does not provide the same automatic name-based discovery.

# Task 6 – Put It Together

## Goal
Combine:

Custom Network + Named Volume
We created:

- Custom network: `day32-net`
- Named volume: `mysql-task6-data`
- MySQL container: `mysql-db`
- App/test container: `app-test`

The app container connects to MySQL using container name:
mysql-db

## Step 1: Cleanup
docker rm -f mysql-db app-test
docker network rm day32-net

## Step 2: Create Network and Volume

docker network create day32-net
docker volume create mysql-task6-data

## Step 3: Run MySQL on Custom Network With Volume

docker run -d --name mysql-db \
  --network day32-net \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  -v mysql-task6-data:/var/lib/mysql \
  mysql:8.0

## Step 4: Create Table and Data
docker exec -it mysql-db mysql -uroot -prootpass demo

Inside MySQL:
CREATE TABLE services (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);

INSERT INTO services (name) VALUES ('app-container'), ('mysql-db'), ('docker-network');

SELECT * FROM services;

exit;

Expected:
+----+----------------+
| id | name           |
+----+----------------+
|  1 | app-container  |
|  2 | mysql-db       |
|  3 | docker-network |
+----+----------------+


## Step 5: App Container Connects to DB by Container Name

Use MySQL image as a temporary client container:
docker run -it --rm --name app-test \
  --network day32-net \
  mysql:8.0 \
  mysql -hmysql-db -uroot -prootpass \
  -e "SELECT 'Connected to mysql-db by container name' AS result;"

Important:
-hmysql-db
means connect to host named `mysql-db`.
Because both containers are on `day32-net`, Docker DNS resolves `mysql-db`.

## Step 6: Query Actual Table from App Container
docker run -it --rm --name app-test \
  --network day32-net \
  mysql:8.0 \
  mysql -hmysql-db -uroot -prootpass demo \
  -e "SELECT * FROM services;"

Expected:
+----+----------------+
| id | name           |
+----+----------------+
|  1 | app-container  |
|  2 | mysql-db       |
|  3 | docker-network |
+----+----------------+

## Step 7: Inspect Volume
docker volume inspect mysql-task6-data
Observed:

```json
[
    {
        "CreatedAt": "2026-05-11T10:02:45Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql-task6-data/_data",
        "Name": "mysql-task6-data",
        "Options": null,
        "Scope": "local"
    }
]

## Step 8: Inspect Network

docker network inspect day32-net --format '{{json .Containers}}'
Observed:

```json
{
  "d206e3ec35c85caab9081fe311424d3cfdf27b89b9657730ab59a67d98105367": {
    "Name": "mysql-db",
    "EndpointID": "b7c0e36eca26dcf70280add327d88b323175af4ca415dc1ce3f903f1e7b7ff6a",
    "MacAddress": "86:3a:7f:62:be:e5",
    "IPv4Address": "172.19.0.2/16",
    "IPv6Address": ""
  }
}

Note:
`app-test` may not appear because it was run with `--rm`, so it is removed after execution.

## Task 6 Result
Task 6 combines the real production-style pattern:
App container → connects to DB container using name
DB container → stores data in named volume
Both containers → use same custom Docker network

This is how many local development and Docker Compose environments work.


# Important Commands Summary

## Volume Commands

docker volume create mysql-data
docker volume ls
docker volume inspect mysql-data
docker volume rm mysql-data

## Run MySQL Without Volume
docker run -d --name mysql-no-volume \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  mysql:8.0

## Run MySQL With Named Volume
docker run -d --name mysql-with-volume \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=demo \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

## Bind Mount Command
docker run -d --name nginx-bind \
  -p 8084:80 \
  -v /root/day32-bind:/usr/share/nginx/html \
  nginx:alpine

## Network Commands
docker network ls
docker network inspect bridge
docker network create my-app-net
docker network rm my-app-net

## Run Container on Custom Network
docker run -d --name app1 --network my-app-net alpine sleep 1000

## Ping Container
docker exec app1 ping -c 3 app2
docker exec app1 ping -c 3 172.18.0.3
```

## MySQL Client Container on Same Network

```bash
docker run -it --rm --network day32-net mysql:8.0 \
  mysql -hmysql-db -uroot -prootpass demo \
  -e "SELECT * FROM services;"


## Basic Questions

1. What is a Docker volume?
2. Why do containers lose data when removed?
3. What is the difference between container writable layer and volume?
4. What command creates a named volume?
5. What command lists Docker volumes?
6. What command inspects a Docker volume?
7. What is a bind mount?
8. What is Docker networking?
9. What command lists Docker networks?
10. What is the default Docker network?

## Scenario-Based Questions

1. You created data inside a MySQL container. After removing the container, the data disappeared. Why?
2. You want MySQL data to survive container deletion. What should you use?
3. You want to edit HTML on the host and instantly see changes in Nginx. Should you use a named volume or bind mount?
4. Your app container cannot reach DB by IP after DB container restart. What is the better approach?
5. Containers can ping each other by IP but not by name. What network are they probably using?
6. You want containers to talk by name. What should you create?
7. Your EC2 Nginx container works with curl localhost but not from browser. What should you check?
8. You run a temporary app container with `--rm`. Why does it not appear in network inspect later?


# Interview Questions and Answers

## Q1. What is a Docker volume?

A Docker volume is persistent storage managed by Docker. It stores data outside the container writable layer so data survives even if the container is removed.

Example:

docker volume create mysql-data
docker run -v mysql-data:/var/lib/mysql mysql:8.0

## Q2. Why do containers lose data when removed?

Containers have a writable layer on top of image layers. Data written inside the container is stored in this writable layer. When the container is removed, the writable layer is deleted. That is why data is lost unless we use volumes or bind mounts.


## Q3. What is the difference between named volume and bind mount?

A named volume is managed by Docker. Docker decides where it is stored on the host.

A bind mount maps a specific host path into the container.

Named volumes are commonly used for database persistence. Bind mounts are commonly used for local development and live file editing.

---

## Q4. Why is a named volume good for databases?

Databases need persistent storage. A named volume stores data outside the container writable layer, so even if the database container is stopped or removed, the data remains available. A new container can reuse the same volume and access the old data.

---

## Q5. What is the MySQL data directory inside a container?

For MySQL, the default data directory is:

```text
/var/lib/mysql
```

So we mount a volume there:

```bash
-v mysql-data:/var/lib/mysql
```

---

## Q6. What is a bind mount?

A bind mount connects a specific host machine folder to a folder inside the container.

Example:

```bash
-v /root/day32-bind:/usr/share/nginx/html
```

If the host file changes, the container sees the update immediately.

---

## Q7. What is Docker bridge network?

Docker bridge network is the default network used by containers when no custom network is specified. Containers on the default bridge can communicate by IP address, but automatic name-based communication is limited.

---

## Q8. What is a custom bridge network?

A custom bridge network is a user-defined Docker network. Containers attached to the same custom bridge network can communicate using container names because Docker provides built-in DNS resolution.

Example:

```bash
docker network create my-app-net
```

---

## Q9. Why does ping by name fail on default bridge but work on custom bridge?

The default bridge network does not provide the same automatic DNS-based name resolution for containers. Custom bridge networks include Docker embedded DNS, so container names are resolved to container IP addresses automatically.

---

## Q10. Why should apps connect to database by container name instead of IP?

Container IP addresses can change when containers are recreated. Container names on a custom network are more stable and Docker DNS resolves them automatically. Therefore, using container names like `mysql-db` is better than using IP addresses.

---

## Q11. How do you create a custom Docker network?

Use:

```bash
docker network create my-app-net
```

Then run containers on it:

```bash
docker run -d --name app1 --network my-app-net alpine sleep 1000
```

---

## Q12. How do you connect a container to a named volume?

Use the `-v` option:

```bash
docker run -v volume-name:/container/path image-name
```

Example:

```bash
docker run -v mysql-data:/var/lib/mysql mysql:8.0
```

---

## Q13. How can one container connect to MySQL running in another container?

Put both containers on the same custom network. Then use the MySQL container name as the host.

Example:

```bash
docker network create app-net
docker run -d --name mysql-db --network app-net mysql:8.0
docker run --rm --network app-net mysql:8.0 mysql -hmysql-db -uroot -prootpass
```

---

## Q14. What does `--rm` do in `docker run`?

`--rm` automatically removes the container when it exits. It is useful for temporary test or client containers.

Example:

```bash
docker run --rm alpine echo hello
```

---

## Q15. What is the difference between `docker volume inspect` and `docker inspect`?

`docker volume inspect` shows details about a volume, such as name, driver, and mountpoint.

`docker inspect` shows detailed information about Docker objects like containers, images, networks, and volumes.

Examples:

```bash
docker volume inspect mysql-data
docker inspect mysql-db
```

---

# Advanced Interview Questions and Answers

## Q16. What happens if two containers use the same named volume?

Both containers can access the same data stored in the volume. This can be useful for shared files, but for databases it must be done carefully because running two database containers writing to the same data directory can corrupt data.

---

## Q17. Can a volume exist without any container?

Yes. Docker volumes exist independently of containers. A container can be removed, but the volume remains until it is explicitly removed.

Check volumes:

```bash
docker volume ls
```

Remove volume:

```bash
docker volume rm volume-name
```

---

## Q18. Does `docker rm` remove volumes automatically?

No, `docker rm container-name` removes the container only. Named volumes are not removed automatically.

To remove anonymous volumes with container:

```bash
docker rm -v container-name
```

Named volumes usually need to be removed manually:

```bash
docker volume rm volume-name
```

---

## Q19. Why are bind mounts risky in production?

Bind mounts depend on specific host paths. If the path does not exist or permissions are wrong, the container may fail. Bind mounts can also expose sensitive host files to containers. Named volumes are usually safer and more portable for persistent data.

---

## Q20. What is Docker embedded DNS?

Docker embedded DNS is the internal DNS service Docker provides for containers on user-defined networks. It resolves container names to container IP addresses.

Example:

```text
mysql-db → 172.19.0.2
```

This allows app containers to connect using service names instead of IP addresses.

---

# Real-World DevOps Use Cases

## 1. MySQL With Persistent Data

```bash
docker network create app-net
docker volume create mysql-data

docker run -d --name mysql-db \
  --network app-net \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=appdb \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

App connects using:

```env
DB_HOST=mysql-db
DB_PORT=3306
DB_DATABASE=appdb
DB_USERNAME=root
DB_PASSWORD=rootpass
```

---

## 2. Nginx Bind Mount for Local Development

```bash
docker run -d --name web \
  -p 8080:80 \
  -v /path/to/site:/usr/share/nginx/html \
  nginx:alpine
```

Edit files on host and refresh browser.

---

## 3. App and Database on Same Network

```bash
docker network create app-net

docker run -d --name mysql-db --network app-net mysql:8.0

docker run -d --name backend --network app-net my-backend:v1
```

Backend uses:

```env
DB_HOST=mysql-db
```

---

# Final Revision Notes

## Key Points

- Containers are ephemeral.
- Data inside container writable layer is lost when container is removed.
- Volumes persist data outside the container writable layer.
- Named volumes are managed by Docker.
- Bind mounts use specific host paths.
- MySQL data path is `/var/lib/mysql`.
- Nginx serves files from `/usr/share/nginx/html`.
- Default bridge allows communication by IP but not reliable name resolution.
- Custom bridge network supports container name DNS.
- Apps should connect to databases by container name, not IP.
- Use volumes for database persistence.
- Use bind mounts for live local development.
