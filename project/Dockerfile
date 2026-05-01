# --- ÉTAPE 1 : Build du Frontend ---
FROM node:20-alpine AS build-front
WORKDIR /app/front
# Copie des fichiers de configuration
COPY front/package*.json ./
# Installation des dépendances
RUN npm install
# Copie du code source et build
COPY front/ ./
RUN npm run build

# --- ÉTAPE 2 : Build du Backend ---
FROM maven:3.9-eclipse-temurin-21 AS build-back
WORKDIR /app/back
# Copie du pom.xml et téléchargement des dépendances (optimisation cache)
COPY back/pom.xml ./
RUN mvn dependency:go-offline
# Copie du code source
COPY back/src ./src
# Build du jar en ignorant les tests (pour gagner du temps)
RUN mvn clean package -DskipTests

# --- ÉTAPE 3 : Image Finale (Exécution) ---
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Installation de Node pour pouvoir lancer le serveur de preview du front si nécessaire
# Ou alors on sert le front via Spring Boot (statique)
RUN apk add --no-cache nodejs npm

# Copie du JAR produit à l'étape 2
COPY --from=build-back /app/back/target/puericulture-1.0.0.jar app.jar

# Copie du build frontend produit à l'étape 1
COPY --from=build-front /app/front/dist ./static

# Script de lancement pour démarrer les deux processus
# Note : Pour une production réelle, il est préférable que Spring serve les fichiers statiques
# ou d'utiliser un docker-compose avec Nginx.
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]