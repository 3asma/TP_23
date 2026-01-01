# Guide de Migration - Eureka vers Consul

## 📝 Checklist de Migration

### Phase 1: Préparation

- [ ] Cloner le repository avec Eureka
- [ ] Vérifier que tous les services démarrent avec Eureka
- [ ] Installer Consul
- [ ] Tester Consul en mode dev

### Phase 2: Migration Code

Pour **CHAQUE** service (Client, Gateway, Voiture):

#### A. Dépendances (pom.xml)

- [ ] Commenter/supprimer `spring-cloud-starter-netflix-eureka-client`
- [ ] Ajouter `spring-cloud-starter-consul-discovery`
- [ ] Exécuter `mvn clean package -DskipTests`

#### B. Configuration (application.yml)

- [ ] Supprimer toutes les propriétés `eureka.*`
- [ ] Ajouter les propriétés `spring.cloud.consul.*`
- [ ] Configurer le nom du service (`spring.application.name`)
- [ ] Ajouter les health checks

#### C. Code Java

- [ ] Vérifier que `@EnableDiscoveryClient` est présent
- [ ] Supprimer toute annotation spécifique Eureka (`@EnableEurekaClient`)
- [ ] Vérifier les appels de services (doivent utiliser le nom logique)

### Phase 3: Test

- [ ] Démarrer Consul (`consul agent -dev`)
- [ ] Démarrer chaque service un par un
- [ ] Vérifier l'enregistrement dans Consul UI
- [ ] Vérifier le health check (doit être "passing")
- [ ] Tester la communication inter-services
- [ ] Tester via Gateway

### Phase 4: Conteneurisation

- [ ] Créer Dockerfile pour chaque service
- [ ] Créer docker-compose.yml
- [ ] Tester le build: `docker-compose build`
- [ ] Démarrer: `docker-compose up -d`
- [ ] Vérifier les logs: `docker-compose logs -f`
- [ ] Vérifier Consul UI: http://localhost:8500

---

## 🔧 Commandes Utiles

### Consul

```bash
# Démarrer en mode dev
consul agent -dev

# Vérifier les services
consul catalog services

# Vérifier le health d'un service
consul catalog nodes -service=SERVICE-CLIENT

# Arrêter Consul
Ctrl+C
```

### Docker

```bash
# Build tous les services
docker-compose build

# Démarrer en détaché
docker-compose up -d

# Voir les logs
docker-compose logs -f [service-name]

# Arrêter tous les services
docker-compose down

# Nettoyer complètement
docker-compose down -v
```

### Maven

```bash
# Build sans tests
mvn clean package -DskipTests

# Lancer un service
mvn spring-boot:run

# Voir les dépendances
mvn dependency:tree
```

---

## 🐛 Dépannage

### Services ne s'enregistrent pas dans Consul

**Symptômes:** Service démarre mais n'apparaît pas dans Consul UI

**Solutions:**
1. Vérifier que Consul est démarré
2. Vérifier `spring.cloud.consul.host` et `port`
3. Vérifier les logs du service pour les erreurs de connexion
4. Vérifier que la dépendance `spring-cloud-starter-consul-discovery` est présente

### Health Check échoue

**Symptômes:** Service apparaît en rouge dans Consul UI

**Solutions:**
1. Vérifier que les endpoints actuator sont exposés
2. Vérifier la configuration du health check
3. Augmenter l'intervalle: `health-check-interval: 30s`
4. Vérifier les logs: `docker-compose logs service-name`

### Erreur "Connection refused" entre services

**Symptômes:** Gateway ne peut pas contacter les services

**Solutions:**
1. Vérifier que tous les services sont dans le même network Docker
2. Vérifier les noms des services dans les routes Gateway
3. Utiliser le nom du service, pas localhost
4. Vérifier que `prefer-ip-address: true` est configuré

---

## 📊 Tableau de Comparaison

| Aspect | Eureka | Consul |
|--------|--------|--------|
| **Dépendance** | `spring-cloud-starter-netflix-eureka-client` | `spring-cloud-starter-consul-discovery` |
| **Config Host** | `eureka.client.service-url.defaultZone` | `spring.cloud.consul.host` + `port` |
| **Nom Service** | `spring.application.name` | `spring.application.name` + `spring.cloud.consul.discovery.service-name` |
| **Health Check** | HTTP par défaut | Configurable (HTTP, TCP, Script, TTL) |
| **UI Port** | 8761 | 8500 |
| **Annotation** | `@EnableEurekaClient` ou `@EnableDiscoveryClient` | `@EnableDiscoveryClient` |

---

## 🎯 Points d'Attention

1. **Compatibilité Versions**
   - Spring Boot 2.7+ recommandé
   - Spring Cloud 2021.0.x minimum

2. **Ports**
   - Consul: 8500 (HTTP), 8600 (DNS)
   - Vérifier qu'aucun service n'écoute déjà sur 8500

3. **Base de Données**
   - Adapter le port MySQL si nécessaire (3309 dans le TP)
   - Vérifier `createDatabaseIfNotExist=true`

4. **Docker Network**
   - Tous les services doivent être dans le même réseau
   - Utiliser les noms de services comme hostnames

5. **Health Checks**
   - Toujours exposer l'endpoint `/actuator/health`
   - Configurer `show-details: always` pour debug

---

**Bonne migration!** 🚀
