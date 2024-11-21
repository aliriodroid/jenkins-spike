# Configuración de Jenkins para Proyecto .NET Core con Docker

## Requisitos Previos
- Docker
- Docker Compose
- Git
- .NET SDK

## Estructura de Proyecto

```
mi-proyecto/
├── docker-compose.yml
├── Dockerfile.jenkins
├── Jenkinsfile
└── project/
    ├── MiProyectoSolucion.sln
    ├── MiProyecto/
    │   └── Controllers/
    └── MiProyecto.Tests/
```

## Paso 1: Crear Estructura del Proyecto

### 1.1 Crear Directorios
```bash
mkdir -p mi-proyecto/project/MiProyecto/Controllers
mkdir -p mi-proyecto/project/MiProyecto.Tests
cd mi-proyecto
```

### 1.2 Inicializar Proyecto .NET
```bash
cd project
dotnet new sln -n MiProyectoSolucion
dotnet new webapi -n MiProyecto
dotnet new xunit -n MiProyecto.Tests
dotnet sln add MiProyecto/MiProyecto.csproj
dotnet sln add MiProyecto.Tests/MiProyecto.Tests.csproj
dotnet add MiProyecto.Tests/MiProyecto.Tests.csproj reference MiProyecto/MiProyecto.csproj
dotnet restore
```

## Paso 2: Archivos de Configuración

### 2.1 docker-compose.yml
```yaml
version: '3.8'
services:
  jenkins:
    build: 
      context: .
      dockerfile: Dockerfile.jenkins
    privileged: true
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    container_name: jenkins
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - ./project:/var/jenkins_home/workspace/project
    networks:
      - devops-network
    environment:
      - DOTNET_CLI_TELEMETRY_OPTOUT=1
      - DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1

networks:
  devops-network:
    driver: bridge
```

### 2.2 Dockerfile.jenkins
```dockerfile
FROM jenkins/jenkins:lts

# Cambiar a root para instalaciones
USER root

# Instalar dependencias
RUN apt-get update && apt-get install -y \
    wget \
    curl \
    git \
    software-properties-common

# Instalar Microsoft GPG key y repositorio
RUN wget https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && \
    dpkg -i packages-microsoft-prod.deb && \
    rm packages-microsoft-prod.deb

# Instalar .NET SDK
RUN apt-get update && \
    apt-get install -y dotnet-sdk-6.0

# Configurar variables de entorno
ENV DOTNET_CLI_TELEMETRY_OPTOUT=1
ENV DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1

# Volver a usuario jenkins
USER jenkins
```

### 2.3 Jenkinsfile
```groovy
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:6.0'
            args '-u root'
        }
    }
    
    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        DOTNET_SKIP_FIRST_TIME_EXPERIENCE = '1'
        PROJECT_PATH = './project/MiProyecto'
        TEST_PATH = './project/MiProyecto.Tests'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Restore') {
            steps {
                sh "dotnet restore ${PROJECT_PATH}"
            }
        }
        
        stage('Build') {
            steps {
                sh "dotnet build ${PROJECT_PATH} -c Release"
            }
        }
        
        stage('Test') {
            steps {
                sh "dotnet test ${TEST_PATH}"
            }
        }
    }
}
```

## Paso 3: Preparar Código de Ejemplo

### 3.1 WeatherForecastController.cs
```csharp
using Microsoft.AspNetCore.Mvc;

namespace MiProyecto.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        [HttpGet(Name = "GetWeatherForecast")]
        public IEnumerable<WeatherForecast> Get()
        {
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = Summaries[Random.Shared.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }

    public class WeatherForecast
    {
        public DateTime Date { get; set; }
        public int TemperatureC { get; set; }
        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
        public string? Summary { get; set; }
    }
}
```

### 3.2 WeatherForecastControllerTests.cs
```csharp
using MiProyecto.Controllers;
using System;
using Xunit;

namespace MiProyecto.Tests
{
    public class WeatherForecastControllerTests
    {
        [Fact]
        public void Get_ReturnsExpectedNumberOfForecasts()
        {
            var controller = new WeatherForecastController();
            var result = controller.Get();
            Assert.Equal(5, result.Count());
        }

        [Fact]
        public void WeatherForecast_TemperatureConversionIsCorrect()
        {
            var forecast = new WeatherForecast { TemperatureC = 0 };
            Assert.Equal(32, forecast.TemperatureF);
        }
    }
}
```

## Paso 4: Configuración de Jenkins

1. Levantar Jenkins
```bash
docker-compose up -d jenkins
```

2. Acceder a Jenkins
- URL: http://localhost:8080
- Obtener contraseña inicial:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

3. Configuración Inicial
- Instalar plugins recomendados
- Crear usuario administrador

## Paso 5: Crear Pipeline en Jenkins

1. Nuevo Item
- Nombre: MiProyectoDotNet
- Tipo: Pipeline
- Configurar SCM (Git)
- Rama: main/master
- Script Path: Jenkinsfile

## Paso 6: Inicializar Repositorio Git

```bash
git init
git add .
git commit -m "Configuración inicial proyecto .NET con Jenkins"
```




## Recursos Adicionales

- [Documentación de Jenkins](https://www.jenkins.io/doc/)
- [Guía .NET Core](https://docs.microsoft.com/dotnet/core/)
- [Docker Documentation](https://docs.docker.com/)
