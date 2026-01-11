# Système de Gestion de Bibliothèque - Architecture Microservices

**Auteur :** REDA IKARNADE  
**Formation :** M2SI - INSEA Rabat  
**Date :** Janvier 2026  

## Vue d'ensemble

Ce projet implémente une architecture microservices pour un système de bibliothèque avec Spring Boot, MySQL, Kafka et Docker.

## Architecture

```
Gateway:9999 ─── Eureka:8761 ─── Kafka:9092
     │              │              │
     └──────────────┼──────────────┘
                    │
    ┌───────────────┼───────────────┐
    │               │               │
User:8082      Book:8081     Emprunt:8083 ── Notification:8084
    │               │               │
MySQL_user    MySQL_book    MySQL_emprunter
```

## Technologies

- **Spring Boot** 3.4.1 - Framework principal
- **MySQL** 8.0 - Base de données (3 instances séparées)
- **Apache Kafka** 7.4.0 - Communication événementielle
- **Netflix Eureka** - Service Discovery
- **Spring Cloud Gateway** - API Gateway
- **Docker Compose** - Orchestration

## Services

### User Service (8082)
```bash
GET    /users              # Liste des utilisateurs
POST   /users              # Créer utilisateur
```

### Book Service (8081)
```bash
GET    /books              # Liste des livres
POST   /books              # Créer livre
```

### Emprunt Service (8083)
```bash
GET    /emprunts                    # Liste des emprunts
POST   /emprunts/{userId}/{bookId} # Créer emprunt
```

### Notification Service (8084)
```java
@KafkaListener(topics = "emprunt-created")
public void handleEmprunt(EmpruntEvent event) {
    // Traite les notifications
}
```

## Base de Données

### db_user
```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

### db_book
```sql
CREATE TABLE books (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    titre VARCHAR(255) NOT NULL,
    auteur VARCHAR(255)
);
```

### db_emprunter
```sql
CREATE TABLE emprunter (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    book_id BIGINT NOT NULL,
    emprunt_date DATE NOT NULL
);
```

## Installation

### Prérequis
```bash
java -version     # Java 17+
mvn -version      # Maven 3.6+
docker --version  # Docker 20.10+
```

### Build
```bash
git clone https://github.com/redax255/microservicesapp.git
cd microservicesapp

# Compilation
./user/mvnw -f ./user/pom.xml clean package -DskipTests
./book/mvnw -f ./book/pom.xml clean package -DskipTests
./emprunter/mvnw -f ./emprunter/pom.xml clean package -DskipTests
./eurika/mvnw -f ./eurika/pom.xml clean package -DskipTests
./gateway/mvnw -f ./gateway/pom.xml clean package -DskipTests
./notification-service/mvnw -f ./notification-service/pom.xml clean package -DskipTests
```

### Démarrage
```bash
docker-compose up -d --build
docker-compose ps
```

## Tests

### Test 1 : Créer utilisateur
```powershell
$user = @{
    name  = "REDA IKARNADE"
    email = "reda@insea.ac.ma"
}

Invoke-RestMethod -Uri "http://localhost:8082/users" -Method POST -ContentType "application/json" -Body ($user | ConvertTo-Json)
```

### Test 2 : Créer livre
```powershell
$book = @{
    titre  = "Microservices Guide"
    auteur = "Expert"
}

Invoke-RestMethod -Uri "http://localhost:8081/books" -Method POST -ContentType "application/json" -Body ($book | ConvertTo-Json)
```

### Test 3 : Créer emprunt
```powershell
$emprunt = Invoke-RestMethod -Uri "http://localhost:8083/emprunts/1/1" -Method POST
Write-Output "Emprunt créé: $emprunt"
```

### Test 4 : Vérifier Kafka
```bash
# Voir les événements
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic emprunt-created --from-beginning

# Logs notification
docker logs notification-service --tail 10
```

## Vérification

```bash
# Services
curl http://localhost:8082/users    # User Service
curl http://localhost:8081/books    # Book Service  
curl http://localhost:8083/emprunts # Emprunt Service

# Infrastructure
curl http://localhost:8761          # Eureka
curl http://localhost:9999          # Gateway
```

## Monitoring

```bash
# Logs en temps réel
docker logs -f user-service
docker logs -f book-service
docker logs -f emprunt-service
docker logs -f notification-service

# État des services
docker-compose ps
```

## Patterns Microservices

- ✅ Database per Service (3 MySQL séparées)
- ✅ Service Discovery (Eureka)
- ✅ API Gateway (Spring Cloud Gateway)
- ✅ Event-Driven Architecture (Kafka)
- ✅ Circuit Breaker (Spring Cloud)
- ✅ Containerisation (Docker)

## Configuration

```yaml
# Variables d'environnement importantes
SPRING_DATASOURCE_URL=jdbc:mysql://db-user:3306/db_user
EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-service:8761/eureka
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:29092
```

## Résultats

L'application démontre :
- Microservices autonomes
- Communication synchrone (REST) et asynchrone (Kafka)
- Isolation des données
- Scalabilité horizontale
- Résilience des services

## Améliorations futures

- [ ] Spring Security
- [ ] Monitoring Prometheus
- [ ] Distributed Tracing
- [ ] CI/CD Pipeline
- [ ] Tests automatisés

## Contact

**REDA IKARNADE**  
Email: reda@insea.ac.ma  
INSEA - Rabat, Maroc