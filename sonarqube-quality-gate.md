# SonarQube Quality Gate Configuration

## Jenkins URL Parametresi ile Quality Gate

### 1. SonarQube'de Quality Gate Oluşturma

1. SonarQube'e giriş yapın (http://localhost:9000)
2. Quality Gates menüsüne gidin
3. "Create" butonuna tıklayın
4. İsim: "Turkcell Standard Gate"

### 2. Quality Gate Kuralları

Aşağıdaki kuralları ekleyin:
- **Coverage**: %20'nin altına düşerse FAILED
- **Duplicated Lines (%)**: %3'ün üstüne çıkarsa FAILED
- **Maintainability Rating**: C'den kötüyse FAILED
- **Reliability Rating**: C'den kötüyse FAILED
- **Security Rating**: C'den kötüyse FAILED

### 3. Jenkins URL Tracking

Pipeline'da Jenkins URL'i SonarQube'ye şu şekilde gönderilir:

```groovy
-Dsonar.analysis.jenkinsUrl=${JENKINS_URL}
-Dsonar.links.ci=${JENKINS_URL}
```

### 4. SonarQube Web API ile Jenkins URL Alma

```bash
# Proje bilgilerini alma
curl -u admin:admin123 \
  "http://localhost:9000/api/projects/search?projects=java-app"

# Proje analiz bilgilerini alma
curl -u admin:admin123 \
  "http://localhost:9000/api/project_analyses/search?project=java-app"

# Custom parametreleri alma
curl -u admin:admin123 \
  "http://localhost:9000/api/measures/component?component=java-app&metricKeys=analysis.jenkinsUrl"
```

### 5. Python Script Örneği (Gelecek Aşama İçin)

```python
import requests
import json

def get_sonarqube_projects():
    url = "http://localhost:9000/api/projects/search"
    response = requests.get(url, auth=('admin', 'admin123'))
    return response.json()

def get_jenkins_url_from_project(project_key):
    url = f"http://localhost:9000/api/project_analyses/search?project={project_key}"
    response = requests.get(url, auth=('admin', 'admin123'))
    analyses = response.json()
    
    for analysis in analyses.get('analyses', []):
        # Jenkins URL'i analysis event'lerinde bulunabilir
        print(f"Analysis: {analysis}")
    
    return None

# Kullanım
projects = get_sonarqube_projects()
for project in projects.get('components', []):
    jenkins_url = get_jenkins_url_from_project(project['key'])
    print(f"Project: {project['key']}, Jenkins URL: {jenkins_url}")
```

Bu setup ile Jenkins URL'i SonarQube'ye parametre olarak gönderilir ve daha sonra ara katman uygulaması ile bu bilgi alınabilir.