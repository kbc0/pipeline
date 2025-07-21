# Jenkins CI/CD Pipeline with Security Governance

## ✅ Current Status (Working PoC)
1. **Local CI Stack Running**
   - Jenkins LTS, SonarQube Community, PostgreSQL via docker-compose
   - Persistent data volumes (jenkins_home, sonar_*)

2. **Java Reference Pipeline (java-app)**
   - Stages: Checkout → Maven build → Unit Tests → JaCoCo coverage → SonarQube scan
   - Quality Gate enforcement with `waitForQualityGate()`
   - Build fails if quality gate fails

3. **Jenkins ⇄ SonarQube Integration**
   - Jenkins passes build URL to SonarQube: `-Dsonar.links.ci=${BUILD_URL}`
   - SonarQube webhook sends gate results back to Jenkins

## 🚧 Next Phase: Security Governance Layer

### A. Shared Pipeline Library (Priority: HIGH)
- Create reusable security scanning functions
- Enforce mandatory SonarQube stage (cannot be deleted)
- Functions: `sonarStage()`, `fortifyScan()`, `blackduckScan()`, `twistlockScan()`

### B. Security Scanners Integration
- Fortify, BlackDuck, Twistlock containers/SaaS
- Fail builds on critical findings before deploy

### C. Python SonarQube Watcher Service
- Poll SonarQube API for projects
- Clone repos, parse Jenkinsfiles
- Alert on missing/disabled security stages
- Slack/Teams notifications + Prometheus metrics

### D. Artifactory Deploy & Governance
- Deploy only on `currentBuild.result == 'SUCCESS'`
- Publish artifacts with build metadata

---

## 1. Container Status Check
```bash
docker compose ps
```

## 2. Jenkins Setup (http://localhost:8080)

### İlk Kurulum:
1. Jenkins'e erişin: http://localhost:8080
2. İlk şifreyi alın:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
3. "Install suggested plugins" seçin
4. Admin kullanıcısı oluşturun

### Gerekli Plugin'ler:
- SonarQube Scanner
- Pipeline
- Git
- JUnit
- HTML Publisher

### Plugin Kurulumu:
1. Dashboard > Manage Jenkins > Plugins
2. Available plugins sekmesinde "SonarQube Scanner" ara ve kur
3. Restart Jenkins if needed

## 3. SonarQube Setup (http://localhost:9000)

### İlk Kurulum:
1. SonarQube'e erişin: http://localhost:9000
2. Default giriş: admin/admin
3. Şifre değiştirin (örn: admin123)

### Token Oluşturma:
1. Administration > Security > Users
2. Admin kullanıcısı için token oluştur
3. Token'ı kaydet

## 4. Jenkins'te SonarQube Configuration

### SonarQube Server Yapılandırması:
1. Dashboard > Manage Jenkins > Configure System
2. SonarQube servers bölümünde:
   - Name: SonarQube
   - Server URL: http://host.docker.internal:9000
   - Server authentication token: (SonarQube'den aldığınız token)

### SonarQube Scanner Yapılandırması:
1. Dashboard > Manage Jenkins > Global Tool Configuration
2. SonarQube Scanner bölümünde:
   - Name: SonarQube Scanner
   - Install automatically seçin

## 5. Pipeline Job Oluşturma

### Yeni Pipeline Job:
1. Dashboard > New Item
2. Name: java-app-pipeline
3. Pipeline seçin
4. Pipeline script from SCM yerine "Pipeline script" seçin
5. Jenkinsfile içeriğini kopyalayın

## 6. Test Etme

### Build Çalıştırma:
1. Pipeline job'ı çalıştırın
2. SonarQube'de proje raporlarını kontrol edin
3. Jenkins URL'nin SonarQube'de görünüp görünmediğini kontrol edin

## 7. Manuel Test Komutları

Java uygulamasını test etmek için:
```bash
cd java-app
mvn clean test
mvn clean verify sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=admin -Dsonar.password=admin123
```

## Önemli Notlar:
- Jenkins URL'i pipeline'da otomatik olarak ${BUILD_URL} ile alınır
- SonarQube'de Jenkins URL'i parametre olarak gönderilir
- Quality Gate %20 test coverage ile kurulabilir