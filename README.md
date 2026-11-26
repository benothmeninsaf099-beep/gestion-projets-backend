# spring-boot-devops

Application Spring Boot avec pipeline CI/CD Jenkins :

Git -> Jenkins -> Build & Test Maven -> Docker Build -> Docker Push -> Déploiement MySQL -> Déploiement backend-app -> Vérification

- Image Docker : backend-app:latest
- Registre privé : localhost:5000
- Conteneurs : mysql, backend-app
- API : http://localhost:8089/entreprise/all
