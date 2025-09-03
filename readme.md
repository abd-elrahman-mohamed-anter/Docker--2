تمام 👌 هنزود للـ **README** رسمة بسيطة (ASCII diagram) توضح العلاقات بين **Spring apps** و **Databases** مع الـ networks. ده يخلي أي حد يقرأ يفهم الصورة بسرعة.

---

# 🐳 Spring PetClinic with Multiple Databases and Networks

This demo shows how to run multiple instances of **Spring PetClinic** connected to different MySQL databases across **two Docker networks**.
The goal is to demonstrate **data isolation** and **container networking**.

---

## 📌 Architecture Diagram

```text
               ┌─────────────────────┐
               │   Docker Network    │
               │    petclinic-net    │
               └─────────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
 ┌─────────────┐               ┌─────────────┐
 │  spring1    │               │  spring2    │
 │  (8081)     │               │  (8082)     │
 └─────────────┘               └─────────────┘
        │                             │
        └───────────┬─────────────────┘
                    │
              ┌─────────────┐
              │   db1       │
              │ clinic1     │
              └─────────────┘


               ┌─────────────────────┐
               │   Docker Network    │
               │   petclinic-net2    │
               └─────────────────────┘
                       │
                ┌─────────────┐
                │   spring3   │
                │   (8083)    │
                └─────────────┘
                       │
                ┌─────────────┐
                │    db3      │
                │  clinic3    │
                └─────────────┘
```

---

## 1️⃣ Create Docker Networks

```bash
docker network create petclinic-net
docker network create petclinic-net2
```

---

## 2️⃣ Run MySQL Databases

* **db1** and **db2** are on the same network (`petclinic-net`).
* **db3** is on a separate network (`petclinic-net2`).

```bash
# DB1
docker run -d --name db1 --network petclinic-net \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=clinic1 \
  -v mysql-data1:/var/lib/mysql \
  mysql:8.0

# DB2
docker run -d --name db2 --network petclinic-net \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=clinic2 \
  -v mysql-data2:/var/lib/mysql \
  mysql:8.0

# DB3
docker run -d --name db3 --network petclinic-net2 \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=clinic3 \
  -v mysql-data3:/var/lib/mysql \
  mysql:8.0
```

---

## 3️⃣ Run Spring PetClinic Apps

```bash
# Spring 1 → uses db1
docker run -d --name spring1 --network petclinic-net \
  -p 8081:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://db1:3306/clinic1 \
  -e SPRING_DATASOURCE_USERNAME=root \
  -e SPRING_DATASOURCE_PASSWORD=root \
  spring-petclinic:latest

# Spring 2 → also uses db1
docker run -d --name spring2 --network petclinic-net \
  -p 8082:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://db1:3306/clinic1 \
  -e SPRING_DATASOURCE_USERNAME=root \
  -e SPRING_DATASOURCE_PASSWORD=root \
  spring-petclinic:latest

# Spring 3 → uses db3 (different network & DB)
docker run -d --name spring3 --network petclinic-net2 \
  -p 8083:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://db3:3306/clinic3 \
  -e SPRING_DATASOURCE_USERNAME=root \
  -e SPRING_DATASOURCE_PASSWORD=root \
  -e SPRING_JPA_PROPERTIES_HIBERNATE_DIALECT=org.hibernate.dialect.MySQL8Dialect \
  spring-petclinic:latest
```

---

## 4️⃣ Verify Networking

```bash
# Get spring1 IP
docker exec -it spring1 sh
ifconfig

# From spring2 → ping spring1
docker exec -it spring2 sh
ping <spring1_IP>
```

---

## 5️⃣ Query Databases

### From db1 (shared by spring1 & spring2):

```bash
docker exec -it db1 mysql -uroot -proot -e "SELECT * FROM clinic1.owners;"
```

Example Output:

```
+----+--------------+-----------+---------+------+------------+
| id | first_name   | last_name | address | city | telephone  |
+----+--------------+-----------+---------+------+------------+
|  5 | abdelrahman  | anter     | 1       | 1    | 1275356521 |
|  6 | ahmed        | ibra      | 2       | 2    | 1111111111 |
|  7 | omar         | hamdy     | *       | *    | 1234567891 |
|  8 | gamal        | ezzat     | -       | -    | 1234567891 |
+----+--------------+-----------+---------+------+------------+
```

---

### From db3 (used only by spring3):

```bash
docker exec -it db3 mysql -uroot -proot -e "SELECT * FROM clinic3.owners;"
```

Example Output:

```
+----+------------+-----------+---------+------+------------+
| id | first_name | last_name | address | city | telephone  |
+----+------------+-----------+---------+------+------------+
|  1 | 112        | 2         | 33      | 1    | 1234567891 |
|  2 | jemy       | nour      | -       | -    | 1111111111 |
+----+------------+-----------+---------+------+------------+
```

---



تحب أضيف **لقطات شاشة (screenshots)** من الـ **UI بتاع Spring PetClinic** بحيث توري الاختلاف في الداتا بين 8081/8082 و 8083؟
