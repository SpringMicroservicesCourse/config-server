# config-server

> Centralized configuration management with Spring Cloud Config Server and Git backend

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.5-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/)
[![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2024.0.2-blue.svg)](https://spring.io/projects/spring-cloud)
[![Consul](https://img.shields.io/badge/Consul-1.4.5-purple.svg)](https://www.consul.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A comprehensive demonstration of **Spring Cloud Config Server** for centralized configuration management in microservices architecture, featuring Git-based configuration storage, multi-environment support, HTTP API access, and Consul integration for service registration.

## Features

- Centralized configuration management via HTTP API
- Git repository as configuration backend
- Multi-environment support (dev, staging, prod)
- Configuration file versioning with Git
- Consul service registration and discovery
- Multiple configuration file formats (YAML, Properties, JSON)
- Configuration merging mechanism
- Spring Boot Actuator monitoring endpoints
- ${user.home} variable support for cross-platform paths

## Tech Stack

- Spring Boot 3.4.5
- Spring Cloud Config Server
- Spring Cloud Consul Discovery
- Java 21
- Git (configuration version control)
- Consul 1.4.5 (service registry)
- Spring Boot Actuator
- Maven 3.8+

## Getting Started

### Prerequisites

- JDK 21 or higher
- Maven 3.8+ (or use included Maven Wrapper)
- Git (for configuration repository)
- Consul 1.4.5 or higher
- Docker (for Consul)

### Quick Start

**Step 1: Create Git Configuration Repository**

```bash
# Create configuration repository directory
mkdir -p ~/workspace/SynologyDrive/java/fengqing-spring-cloud-config-git-repo
cd ~/workspace/SynologyDrive/java/fengqing-spring-cloud-config-git-repo

# Initialize Git repository
git init

# Create default configuration file (waiter-service.yml)
cat > waiter-service.yml <<'YAML'
order:
  discount: 80
  waiterPrefix: "fengqing-"
YAML

git add waiter-service.yml
git commit -m "Add waiter-service default configuration"

# Create dev environment configuration (waiter-service-dev.yml)
cat > waiter-service-dev.yml <<'YAML'
order:
  discount: 60
YAML

git add waiter-service-dev.yml
git commit -m "Add waiter-service dev configuration"
```

**Step 2: Start Consul**

```bash
# Using Docker (recommended)
docker run -d --name consul \
  -p 8500:8500 \
  -p 8600:8600/udp \
  consul:1.4.5

# Verify Consul is running
curl http://localhost:8500/v1/status/leader
# Expected: "127.0.0.1:8300"
```

**Step 3: Run Config Server**

```bash
./mvnw spring-boot:run
```

**Step 4: Test Configuration API**

```bash
# Health check
curl http://localhost:8888/actuator/health

# Get default configuration (YAML format)
curl http://localhost:8888/waiter-service.yml

# Get dev environment configuration (YAML format)
curl http://localhost:8888/waiter-service-dev.yml

# Get configuration in JSON format
curl http://localhost:8888/waiter-service/dev
```

## Configuration

### Application Properties

```properties
# Server configuration
server.port=8888  # Config Server default port

# Error response configuration (Development only)
server.error.include-message=always
server.error.include-binding-errors=always

# Actuator configuration (Development only)
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
management.endpoint.configprops.show-values=always
management.endpoint.env.show-values=always
management.info.env.enabled=true  # Required for Spring Boot 3.x

# Consul service registration
spring.cloud.consul.host=localhost
spring.cloud.consul.port=8500
spring.cloud.consul.discovery.prefer-ip-address=true

# Git repository configuration
spring.cloud.config.server.git.uri=file:///${user.home}/workspace/SynologyDrive/java/fengqing-spring-cloud-config-git-repo
```

**Git URI Formats:**

| Format | Example | Use Case |
|--------|---------|----------|
| **Absolute path with ${user.home}** | `file:///${user.home}/workspace/config-repo` | ✅ Recommended (cross-platform) |
| **Absolute path** | `file:///Users/aaron/workspace/config-repo` | ✅ Works (platform-specific) |
| **Relative path** | `file://../config-repo` | ⚠️ Relative to config-server directory |
| **Relative path (no protocol)** | `../config-repo` | ⚠️ Spring auto-adds `file://` |
| **Remote Git (HTTPS)** | `https://github.com/your-org/config-repo.git` | ✅ Recommended for production |
| **Remote Git (SSH)** | `git@github.com:your-org/config-repo.git` | ✅ Recommended for production |

### Bootstrap Properties

```properties
# Application name (used for Consul registration)
spring.application.name=configserver
```

## HTTP API Endpoints

### Configuration Retrieval

| Endpoint Format | Description | Example |
|----------------|-------------|---------|
| `/{application}/{profile}` | JSON format | `/waiter-service/default` |
| `/{application}/{profile}/{label}` | JSON with Git branch | `/waiter-service/dev/main` |
| `/{application}.yml` | YAML format | `/waiter-service.yml` |
| `/{application}-{profile}.yml` | YAML with profile | `/waiter-service-dev.yml` |
| `/{label}/{application}-{profile}.yml` | YAML with Git branch | `/main/waiter-service-dev.yml` |

### API Examples

**Get YAML Configuration (Default Environment):**

```bash
curl http://localhost:8888/waiter-service.yml
```

**Response:**
```yaml
order:
  discount: 80
  waiterPrefix: fengqing-
```

**Get YAML Configuration (Dev Environment):**

```bash
curl http://localhost:8888/waiter-service-dev.yml
```

**Response:**
```yaml
order:
  discount: 60
  waiterPrefix: fengqing-
```

> **Configuration Merging**: `order.discount` uses dev value (60), `order.waiterPrefix` uses default value ("fengqing-")

**Get JSON Configuration (Dev Environment):**

```bash
curl http://localhost:8888/waiter-service/dev | jq
```

**Response:**
```json
{
  "name": "waiter-service",
  "profiles": ["dev"],
  "label": null,
  "version": "2ef2004affd8dc365827bc31c45ab86c9211f5d0",
  "state": "",
  "propertySources": [
    {
      "name": "file:///path/to/fengqing-spring-cloud-config-git-repo/waiter-service-dev.yml",
      "source": {
        "order.discount": 60
      }
    },
    {
      "name": "file:///path/to/fengqing-spring-cloud-config-git-repo/waiter-service.yml",
      "source": {
        "order.discount": 80,
        "order.waiterPrefix": "fengqing-"
      }
    }
  ]
}
```

**JSON Response Fields:**

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Application name | `"waiter-service"` |
| `profiles` | Active profile list | `["dev"]` |
| `label` | Git branch/tag | `null` (default: main) |
| `version` | Git commit hash | `"2ef2004a..."` |
| `propertySources` | Configuration sources (priority order) | Array of sources |

## Configuration File Naming

### Naming Components

| Component | Description | Mapping | Example |
|-----------|-------------|---------|---------|
| **Application** | Application name | `spring.application.name` | `waiter-service` |
| **Profile** | Environment identifier | `spring.profiles.active` | `dev`, `prod` |
| **Label** | Git branch/tag | Git branch/tag | `main`, `develop` |

### File Format Examples

| Format | Description | Example |
|--------|-------------|---------|
| `{application}.yml` | Application default config | `waiter-service.yml` |
| `{application}-{profile}.yml` | Environment-specific config | `waiter-service-dev.yml` |
| `application.yml` | Global shared config | `application.yml` |
| `application-{profile}.yml` | Global environment config | `application-dev.yml` |

## Configuration Repository Structure

### Recommended Structure

```
fengqing-spring-cloud-config-git-repo/
├── application.yml                 # Global default configuration
├── application-dev.yml             # Global dev environment configuration
├── application-prod.yml            # Global prod environment configuration
├── waiter-service.yml              # waiter-service default configuration
├── waiter-service-dev.yml          # waiter-service dev configuration
├── waiter-service-prod.yml         # waiter-service prod configuration
├── customer-service.yml            # customer-service default configuration
├── customer-service-dev.yml        # customer-service dev configuration
└── README.md                       # Configuration documentation
```

### Configuration Priority

```
High Priority
  ↓
{application}-{profile}.yml     ← Environment-specific (e.g., waiter-service-dev.yml)
  ↓
{application}.yml               ← Application default (e.g., waiter-service.yml)
  ↓
application-{profile}.yml       ← Global environment (e.g., application-dev.yml)
  ↓
application.yml                 ← Global default
  ↓
Low Priority
```

**Example: waiter-service with dev profile**

```
1. Load: waiter-service-dev.yml   → order.discount=60
2. Load: waiter-service.yml       → order.discount=80, order.waiterPrefix="fengqing-"
3. Merge result:
   - order.discount=60            ← From waiter-service-dev.yml (higher priority)
   - order.waiterPrefix="fengqing-" ← From waiter-service.yml
```

## Key Components

### ConfigServerApplication

```java
@SpringBootApplication
@EnableConfigServer  // Enable Config Server functionality
public class ConfigServerApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

**Key Annotations:**
- `@SpringBootApplication`: Spring Boot application marker
- `@EnableConfigServer`: **The only required annotation** to enable Config Server
- No additional configuration needed - Spring Boot auto-configures all necessary beans

### Auto-Configuration

**What @EnableConfigServer Does:**

1. **Enables HTTP API endpoints** for configuration retrieval
2. **Configures Git repository client** for fetching configuration files
3. **Sets up Environment Repository** for configuration storage
4. **Registers HTTP message converters** for JSON/YAML/Properties formats
5. **Exposes Actuator endpoints** for monitoring

## Consul Integration

### Service Registration

**Registration Flow:**

```
1. Spring Boot starts
   ↓
2. Read bootstrap.properties → spring.application.name=configserver
   ↓
3. Read application.properties → Consul host/port
   ↓
4. Auto-register to Consul
   - Service Name: configserver
   - Port: 8888
   ↓
5. Start health check heartbeat
```

**Verify Registration:**

```bash
# Query configserver in Consul
curl -s http://localhost:8500/v1/catalog/service/configserver | jq '.[] | {ID: .ServiceID, IP: .ServiceAddress, Port: .ServicePort}'

# Expected output:
# {
#   "ID": "configserver-8888",
#   "IP": "192.168.100.194",
#   "Port": 8888
# }
```

**DNS Discovery:**

```bash
# Query via DNS
dig @127.0.0.1 -p 8600 configserver.service.consul

# Expected output:
# configserver.service.consul. 0 IN A 192.168.100.194
```

## Monitoring

### Actuator Endpoints

| Endpoint | Description |
|----------|-------------|
| `/actuator/health` | Health check status |
| `/actuator/env` | Environment variables and properties |
| `/actuator/configprops` | Configuration properties |
| `/actuator/health/consul` | Consul connection status |
| `/actuator/info` | Application information |

**Examples:**

```bash
# Health check
curl http://localhost:8888/actuator/health | jq

# Environment variables
curl http://localhost:8888/actuator/env | jq

# Configuration properties
curl http://localhost:8888/actuator/configprops | jq

# Consul health
curl http://localhost:8888/actuator/health/consul | jq
```

### Logging

**Enable Detailed Logging:**

```properties
# Config Server logs
logging.level.org.springframework.cloud.config=DEBUG

# Consul discovery logs
logging.level.org.springframework.cloud.consul=DEBUG

# Git repository logs
logging.level.org.springframework.cloud.config.server.environment=DEBUG
```

**Important Log Messages:**

```
DEBUG o.s.c.c.s.e.NativeEnvironmentRepository : Loading property source from: file:///path/to/repo/waiter-service-dev.yml
DEBUG o.s.c.c.s.e.NativeEnvironmentRepository : Loading property source from: file:///path/to/repo/waiter-service.yml
INFO  o.s.c.c.s.ConsulServiceRegistry         : Registering service with consul: configserver
```

### Git Repository Monitoring

```bash
# View Git commit history
cd ~/workspace/SynologyDrive/java/fengqing-spring-cloud-config-git-repo
git log --oneline

# Expected output:
# bf638d6 (HEAD -> main) Add waiter-service dev configuration
# 584e3db Add waiter-service default configuration

# View Git status
git status

# View latest commit details
git show HEAD
```

## Configuration Strategies

### 1. Single Repository (Simple)

```
fengqing-spring-cloud-config-git-repo/
├── waiter-service.yml
├── waiter-service-dev.yml
├── waiter-service-prod.yml
├── customer-service.yml
├── customer-service-dev.yml
└── customer-service-prod.yml
```

**Configuration:**
```properties
spring.cloud.config.server.git.uri=file:///${user.home}/workspace/config-repo
```

### 2. Multi-Repository (Advanced)

**Configuration:**
```properties
# Default repository (common configuration)
spring.cloud.config.server.git.uri=https://github.com/your-org/common-config.git

# waiter-service specific repository
spring.cloud.config.server.git.repos.waiter-service.uri=https://github.com/your-org/waiter-config.git
spring.cloud.config.server.git.repos.waiter-service.pattern=waiter-service*

# customer-service specific repository
spring.cloud.config.server.git.repos.customer-service.uri=https://github.com/your-org/customer-config.git
spring.cloud.config.server.git.repos.customer-service.pattern=customer-service*
```

**Benefits:**
- ✅ Different teams manage their own repositories
- ✅ Reduced permission management complexity
- ✅ Smaller change impact scope

### 3. Branch-Based Environments

```bash
# Create environment branches
git checkout -b develop      # Development environment
git checkout -b staging      # Staging environment
git checkout -b production   # Production environment
```

**Access Configuration by Branch:**

```bash
# Get dev configuration from develop branch
curl http://localhost:8888/waiter-service/dev/develop

# Get prod configuration from production branch
curl http://localhost:8888/waiter-service/prod/production
```

**Benefits:**
- ✅ Complete environment isolation
- ✅ Test configuration changes in dev branch first
- ✅ Use Git merge to promote changes across environments

## Security

### Configuration Encryption

**Step 1: Generate Encryption Key**

```bash
# Generate JKS keystore
keytool -genkeypair -alias config-server -keyalg RSA \
  -keystore config-server.jks -storepass changeit \
  -keypass changeit -dname "CN=Config Server"
```

**Step 2: Configure Config Server**

```properties
# Enable encryption
spring.cloud.config.server.encrypt.enabled=true
encrypt.key-store.location=classpath:config-server.jks
encrypt.key-store.password=changeit
encrypt.key-store.alias=config-server
encrypt.key-store.secret=changeit
```

**Step 3: Encrypt Sensitive Data**

```bash
# Encrypt password
curl -X POST http://localhost:8888/encrypt -d "mySecretPassword"

# Output (encrypted value):
# AQA12B34C56D78E90F...

# Decrypt (for testing)
curl -X POST http://localhost:8888/decrypt -d "AQA12B34C56D78E90F..."
```

**Step 4: Use Encrypted Values in Configuration**

```yaml
# waiter-service-prod.yml
spring:
  datasource:
    password: '{cipher}AQA12B34C56D78E90F...'
```

### Basic Authentication

**Add Spring Security Dependency:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Configure Authentication:**

```properties
# Config Server (application.properties)
spring.security.user.name=admin
spring.security.user.password=${CONFIG_SERVER_PASSWORD}
```

**Client Configuration:**

```properties
# Config Client (bootstrap.properties)
spring.cloud.config.uri=http://localhost:8888
spring.cloud.config.username=admin
spring.cloud.config.password=${CONFIG_SERVER_PASSWORD}
```

## Architecture

### System Architecture

```
┌─────────────────────────────────────────────┐
│          Microservice Instances              │
│  ┌──────────────┐  ┌──────────────┐        │
│  │waiter-service│  │customer-service│       │
│  └──────┬───────┘  └──────┬────────┘        │
└─────────┼──────────────────┼─────────────────┘
          │ HTTP GET         │
          │ /waiter-service/dev
          │                  │
┌─────────▼──────────────────▼─────────────────┐
│         Config Server (Port: 8888)           │
│  ┌───────────────────────────────────────┐  │
│  │  HTTP API Endpoints                    │  │
│  │  - /{application}/{profile}            │  │
│  │  - /{application}.yml                  │  │
│  │  - /{application}-{profile}.yml        │  │
│  └───────────────────────────────────────┘  │
│  ┌───────────────────────────────────────┐  │
│  │  Environment Repository                │  │
│  │  - Git backend client                  │  │
│  │  - Configuration caching               │  │
│  │  - File format conversion              │  │
│  └───────────────────────────────────────┘  │
└──────────────────┬───────────────────────────┘
                   │ Clone/Pull
┌──────────────────▼───────────────────────────┐
│     Git Configuration Repository            │
│  ┌───────────────────────────────────────┐  │
│  │  waiter-service.yml                    │  │
│  │  waiter-service-dev.yml                │  │
│  │  customer-service.yml                  │  │
│  │  application.yml                       │  │
│  └───────────────────────────────────────┘  │
│  Version Control: Git commits, branches     │
└─────────────────────────────────────────────┘
```

### Configuration Merging Flow

```
Client Request: http://localhost:8888/waiter-service/dev
  ↓
1. Parse request parameters
   - Application: waiter-service
   - Profile: dev
   - Label: null (default: main branch)
  ↓
2. Search configuration files in Git repository:
   - application.yml               (if exists)
   - application-dev.yml           (if exists)
   - waiter-service.yml            ← Found ✅
   - waiter-service-dev.yml        ← Found ✅
  ↓
3. Load files in priority order:
   - waiter-service-dev.yml (High) → order.discount=60
   - waiter-service.yml (Low)      → order.discount=80, order.waiterPrefix="fengqing-"
  ↓
4. Merge configurations:
   - order.discount=60             ← From dev (higher priority)
   - order.waiterPrefix="fengqing-" ← From default
  ↓
5. Return merged JSON response
```

## Testing

### Manual Testing

**Test Configuration Retrieval:**

```bash
# Test 1: Get YAML configuration
curl http://localhost:8888/waiter-service.yml

# Test 2: Get dev environment YAML
curl http://localhost:8888/waiter-service-dev.yml

# Test 3: Get JSON configuration
curl http://localhost:8888/waiter-service/default | jq

# Test 4: Get dev environment JSON
curl http://localhost:8888/waiter-service/dev | jq

# Test 5: Get configuration from specific branch
curl http://localhost:8888/waiter-service/dev/main | jq
```

**Test Consul Registration:**

```bash
# Query configserver in Consul
curl http://localhost:8500/v1/catalog/service/configserver | jq

# Check health status
curl http://localhost:8500/v1/health/service/configserver | jq
```

### Automated Testing

```bash
# Run unit tests
./mvnw test

# Run with coverage
./mvnw clean test jacoco:report
```

## Best Practices

### 1. Git Repository Management

**Commit Message Convention:**

```bash
# ✅ Good: Semantic commit messages
git commit -m "feat(waiter-service): Add discount configuration for dev environment"
git commit -m "fix(customer-service): Correct database connection timeout"
git commit -m "chore(config): Update logging level for production"

# ❌ Bad: Meaningless commit messages
git commit -m "update config"
```

**Branch Strategy:**

| Branch | Purpose | Configuration Files |
|--------|---------|---------------------|
| `main` | Production | `*-prod.yml` |
| `staging` | Staging | `*-staging.yml` |
| `develop` | Development | `*-dev.yml` |

### 2. Configuration File Management

**File Naming:**

```bash
# ✅ Good: Clear naming convention
waiter-service.yml           # Default
waiter-service-dev.yml       # Development
waiter-service-staging.yml   # Staging
waiter-service-prod.yml      # Production

# ❌ Bad: Unclear naming
waiter-service-test.yml      # Ambiguous
waiter-service-1.yml         # Not descriptive
```

**Documentation:**

```yaml
# ✅ Good: Add comments in configuration files
order:
  discount: 60  # Development discount: 40% off
  waiterPrefix: "fengqing-"  # Waiter name prefix

# ❌ Bad: No comments
order:
  discount: 60
  waiterPrefix: "fengqing-"
```

### 3. Security Best Practices

**Development Environment:**

```properties
# No authentication required
management.endpoints.web.exposure.include=*
```

**Production Environment:**

```properties
# ✅ Recommended: Restrict endpoint exposure
management.endpoints.web.exposure.include=health,info,prometheus

# ✅ Recommended: Require authentication for details
management.endpoint.health.show-details=when-authorized

# ✅ Recommended: Enable Basic Auth
spring.security.user.name=admin
spring.security.user.password=${CONFIG_SERVER_PASSWORD}

# ✅ Recommended: Encrypt sensitive data
spring.datasource.password={cipher}AQA12B34C56D...
```

### 4. Performance Optimization

**Git Clone Configuration:**

```properties
# Clone on startup (default)
spring.cloud.config.server.git.clone-on-start=true

# Refresh interval
spring.cloud.config.server.git.refresh-rate=60  # Seconds

# Force pull before serving
spring.cloud.config.server.git.force-pull=true
```

**Caching Strategy:**

```properties
# Enable server-side caching
spring.cloud.config.server.git.cache-size=100

# Cache TTL
spring.cloud.config.server.git.cache-ttl=3600  # Seconds
```

## Common Issues

### Issue 1: Git Repository Not Found

**Error:**
```
Cannot clone or checkout repository: file:///${user.home}/workspace/config-repo
```

**Solutions:**

```bash
# 1. Verify Git repository exists
ls -la ~/workspace/SynologyDrive/java/fengqing-spring-cloud-config-git-repo

# 2. Verify it's a valid Git repository
cd ~/workspace/SynologyDrive/java/fengqing-spring-cloud-config-git-repo
git status

# 3. Check Config Server logs for exact error
tail -f logs/spring.log | grep "config.server"
```

### Issue 2: Configuration File Not Found

**Error:**
```
{
  "name": "waiter-service",
  "profiles": ["dev"],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": []  ← Empty!
}
```

**Solutions:**

```bash
# 1. Verify configuration file exists in Git repository
cd ~/workspace/SynologyDrive/java/fengqing-spring-cloud-config-git-repo
ls -la waiter-service*.yml

# 2. Verify files are committed to Git
git log --all --full-history -- waiter-service*.yml

# 3. Check file naming matches application name
# Application name: spring.application.name=waiter-service
# File name: waiter-service.yml ✅
```

### Issue 3: Consul Connection Failed

**Error:**
```
com.ecwid.consul.transport.TransportException: 
Connect to localhost:8500 failed
```

**Solutions:**

```bash
# 1. Check Consul is running
docker ps | grep consul

# 2. Verify Consul port
curl http://localhost:8500/v1/status/leader

# 3. Restart Consul if needed
docker restart consul
```

### Issue 4: Port Already in Use

**Error:**
```
Web server failed to start. Port 8888 was already in use.
```

**Solutions:**

```bash
# Solution 1: Kill process using port 8888
lsof -ti:8888 | xargs kill -9

# Solution 2: Use different port
server.port=8889

# Solution 3: Check what's using the port
lsof -i:8888
```

## Production Deployment

### Consul Cluster

```bash
# Production: Use Consul cluster (3 nodes)
consul agent -server -bootstrap-expect=3 \
  -data-dir=/tmp/consul \
  -node=consul-1 \
  -bind=192.168.1.100 \
  -client=0.0.0.0 \
  -ui
```

### Remote Git Repository

**Configuration:**

```properties
# HTTPS authentication
spring.cloud.config.server.git.uri=https://github.com/your-org/config-repo.git
spring.cloud.config.server.git.username=${GIT_USERNAME}
spring.cloud.config.server.git.password=${GIT_PASSWORD}

# SSH authentication (recommended)
spring.cloud.config.server.git.uri=git@github.com:your-org/config-repo.git
spring.cloud.config.server.git.ignore-local-ssh-settings=false
spring.cloud.config.server.git.private-key=classpath:config-server-key
spring.cloud.config.server.git.host-key=AAAA...  # SSH host key
```

### High Availability

**Deploy Multiple Config Server Instances:**

```bash
# Instance 1
java -jar config-server.jar --server.port=8888

# Instance 2
java -jar config-server.jar --server.port=8889

# Instance 3
java -jar config-server.jar --server.port=8890
```

**Load Balancer Configuration (Nginx):**

```nginx
upstream config_server {
    server localhost:8888;
    server localhost:8889;
    server localhost:8890;
}

server {
    listen 80;
    location / {
        proxy_pass http://config_server;
    }
}
```

## Config Server vs Other Solutions

| Feature | Config Server | Consul KV | Kubernetes ConfigMap | Vault |
|---------|--------------|-----------|----------------------|-------|
| **Version Control** | ✅ Git | ❌ No | ❌ No | ⚠️ Limited |
| **Multiple Formats** | ✅ YAML, Properties, JSON | ⚠️ KV only | ✅ Multiple | ⚠️ KV only |
| **Encryption** | ✅ Built-in | ❌ No | ❌ No | ✅ Built-in |
| **Spring Integration** | ✅ Native | ✅ Good | ⚠️ External | ✅ Good |
| **Multi-Environment** | ✅ Profile-based | ⚠️ Manual | ⚠️ Manual | ⚠️ Manual |
| **Audit Trail** | ✅ Git history | ⚠️ Limited | ❌ No | ✅ Built-in |
| **Learning Curve** | Medium | Easy | Easy | Hard |

**Selection Guide:**
- **Config Server**: Best for Spring Boot microservices with Git workflow
- **Consul KV**: Simple key-value storage, good for non-Spring apps
- **Kubernetes ConfigMap**: Best for Kubernetes-native applications
- **Vault**: Best for secret management and high-security requirements

## Best Practices Demonstrated

1. **Git-Based Configuration**: Version control for all configuration changes
2. **Multi-Environment Support**: Profile-based configuration separation
3. **Configuration Merging**: Automatic merging of default and environment-specific configs
4. **Consul Integration**: Service registration and discovery
5. **Security**: Support for configuration encryption
6. **Monitoring**: Actuator endpoints for health and metrics
7. **${user.home} Variable**: Cross-platform path configuration
8. **HTTP API**: RESTful API for configuration retrieval

## Advanced Topics

### 1. Configuration Refresh (@RefreshScope)

```java
@Component
@RefreshScope  // Auto-refresh when configuration changes
public class DiscountService {
    
    @Value("${order.discount}")
    private int discount;
    
    public int getDiscount() {
        return discount;  // Updated automatically after refresh
    }
}
```

**Trigger Refresh:**

```bash
# POST to /actuator/refresh endpoint (requires spring-boot-starter-actuator)
curl -X POST http://localhost:8080/actuator/refresh
```

### 2. Spring Cloud Bus (Auto Refresh)

**Add Dependency:**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

**Configuration:**

```properties
# RabbitMQ configuration
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

**Trigger Broadcast Refresh:**

```bash
# Refresh all clients
curl -X POST http://localhost:8888/actuator/bus-refresh
```

### 3. Vault Backend

```properties
# Use Vault as configuration backend
spring.cloud.config.server.vault.host=localhost
spring.cloud.config.server.vault.port=8200
spring.cloud.config.server.vault.scheme=http
spring.cloud.config.server.vault.backend=secret
spring.cloud.config.server.vault.default-key=application
```

## References

- [Spring Cloud Config Documentation](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/)
- [Spring Cloud Config Server Properties](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_environment_repository)
- [Consul Service Discovery](https://www.consul.io/docs/discovery)
- [Git Documentation](https://git-scm.com/doc)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)

## License

MIT License - see [LICENSE](LICENSE) file for details.

## About Us

我們主要專注在敏捷專案管理、物聯網（IoT）應用開發和領域驅動設計（DDD）。喜歡把先進技術和實務經驗結合，打造好用又靈活的軟體解決方案。近來也積極結合 AI 技術，推動自動化工作流，讓開發與運維更有效率、更智慧。持續學習與分享，希望能一起推動軟體開發的創新和進步。

## Contact

**風清雲談** - 專注於敏捷專案管理、物聯網（IoT）應用開發和領域驅動設計（DDD）。

- 🌐 官方網站：[風清雲談部落格](https://blog.fengqing.tw/)
- 📘 Facebook：[風清雲談粉絲頁](https://www.facebook.com/profile.php?id=61576838896062)
- 💼 LinkedIn：[Chu Kuo-Lung](https://www.linkedin.com/in/chu-kuo-lung)
- 📺 YouTube：[雲談風清頻道](https://www.youtube.com/channel/UCXDqLTdCMiCJ1j8xGRfwEig)
- 📧 Email：[fengqing.tw@gmail.com](mailto:fengqing.tw@gmail.com)

---

**⭐ If this project helps you, please give it a Star!**
