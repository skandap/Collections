# Docker Deployment Guide for Booking_MS_Project

---

## **1️⃣ Install Docker**

1. Go to: [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
2. Open CMD and run:
   ```bash
   wsl --install
   ```
3. Open Docker Desktop.

---

## **2️⃣ Run Ubuntu Terminal inside Docker**

```bash
docker run -it ubuntu bash
```
- Docker will download the Ubuntu image (first time only).
- You will get a Linux terminal inside the container: `root@<container-id>:/#`
- You can run Linux commands:
  ```bash
  ls
  pwd
  apt update
  exit
  ```

To keep the Ubuntu container running in the background:
```bash
docker run -dit ubuntu
```

---

## **3️⃣ Build your Spring Boot service**

Navigate to your service folder:
```
C:\Users\SKANDA PRASAD\OneDrive\Desktop\Booking_MS_Project\user_service
```

Build the jar:
```bash
mvn clean package -DskipTests
```
Check `target` folder to confirm jar name.

---

## **4️⃣ Create Dockerfile**

Create a file named `Dockerfile` (no extensions) in your service folder:
```dockerfile
FROM eclipse-temurin:21-jdk

WORKDIR /app

COPY target/User-Service-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## **5️⃣ Run service manually (local Postgres)**

Update `application.properties` for local Postgres:
```properties
spring.datasource.url=jdbc:postgresql://host.docker.internal:5432/postgres
```

Build and run:
```bash
docker build -t user_service .
docker run -p 8081:8081 user_service
```

---

## **6️⃣ Run all services automatically using Docker Compose**

Folder structure:
```
Booking_MS_Project/
  docker-compose.yml
  user_service/
      Dockerfile
      target/user_service.jar
  train_service/
      Dockerfile
  referencecode_service/
      Dockerfile
  booking_service/
      Dockerfile
```

Example `docker-compose.yml`:
```yaml
services:
  postgres:
    image: postgres:13
    container_name: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: bookingdb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  user_service:
    build:
      context: ./user_service
      dockerfile: Dockerfile
    container_name: user_service
    ports:
      - "8081:8081"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/userdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
    depends_on:
      - postgres

  train_service:
    build:
      context: ./train_service
      dockerfile: Dockerfile
    container_name: train_service
    ports:
      - "8082:8082"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/traindb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
    depends_on:
      - postgres

  reference_codes:
    build:
      context: ./reference_codes
      dockerfile: Dockerfile
    container_name: reference_codes
    ports:
      - "8083:8083"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/refdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
    depends_on:
      - postgres

  booking_service:
    build:
      context: ./booking_service
      dockerfile: Dockerfile
    container_name: booking_service
    ports:
      - "8084:8084"
    environment:
      USER_SERVICE_URL: http://user_service:8081
      TRAIN_SERVICE_URL: http://train_service:8082
      REFERENCE_SERVICE_URL: http://reference_codes:8083
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/bookingdb
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
    depends_on:
      - user_service
      - train_service
      - reference_codes
      - postgres

volumes:
  postgres_data:
```

Run all services:
```bash
docker-compose down
docker-compose up --build
```

---

✅ **All services and Postgres will start automatically**.

