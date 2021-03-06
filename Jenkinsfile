@Library('sharedlib')
import static com.orcilatam.devops.Const.*
import static com.orcilatam.devops.Stage.*

def sonarqube = 'sonarqube:9000'
def registry = 'artifactory:8082/docker-local'
def registryId = 'registry-push-user'
def containerPort = '9090'
def kubeConfig = 'kubeconfig-multivac'
def kubeNamespace = 'default'
def ingressName = 'cervecero'

pipeline {
  agent any

  stages {
    stage('Compilación') {
      steps {
        script {
          sendSlackStageMessage(this)
          buildPython(this, 'cervecero', '1.0')
        }
      }
    }

    stage('Tests unitarios') {
      steps {
        script {
          sendSlackStageMessage(this)
          testPython(this)
        }
      }
    }

    stage('OWASP Dependency Check') {
      steps {
        script {
          sendSlackStageMessage(this)
          runOWASPDependencyChecks(this, 'cervecero')
        }
      }
    }

    stage('Calidad de código') {
      steps {
        script {
          sendSlackStageMessage(this)
          runSonarQubePython(this, sonarqube)
        }
      }
    }

    stage('Construcción de imagen Docker') {
      steps {
        script {
          sendSlackStageMessage(this)
          buildDockerImage(this, registry)
        }
      }
    }

    stage('Subida a Artifactory') {
      steps {
        script {
          sendSlackStageMessage(this)
          pushImageToArtifactory(this, registry, registryId)
        }
      }
    }

    stage('Despliegue a Kubernetes') {
      steps {
        script {
          sendSlackStageMessage(this)

          replacePlaceholder(this, 'deployment.yaml', 'registry', registry)
          replacePlaceholder(this, 'deployment.yaml', 'project.port', containerPort)
          replacePlaceholder(this, 'service.yaml', 'project.port', containerPort)

          deployToKubernetes(this, kubeConfig, kubeNamespace, 'deployment.yaml')
          deployToKubernetes(this, kubeConfig, kubeNamespace, 'service.yaml')
        }
      }
    }

    stage('Instalación de LoadBalancer e Ingress Controller') {
      steps {
        script {
          sendSlackStageMessage(this)
          updateHelmRepositories(this)
          installNginxIngressController(this, kubeConfig, kubeNamespace, ingressName)
        }
      }
    }

    stage('Configuración de Ingress') {
      steps {
        script {
          sendSlackStageMessage(this)
          deployToKubernetes(this, kubeConfig, kubeNamespace, 'ingress.yaml')
        }
      }
    }

  }

  post {
    success {
      script {
        sendSlackSuccess(this, "Pipeline terminó exitosamente")
      }
    }
    failure {
      script {
        sendSlackAlert(this, "Pipeline **falló**")
      }
    }
  }

}
