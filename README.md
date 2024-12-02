# Criando-um-Pipeline-de-Deploy-de-uma-Aplica-o-Utilizando-Gitlab-Docker-e-Kubernetes
Criando um Pipeline de Deploy de uma Aplicação Utilizando Gitlab, Docker e Kubernetes

Criar um pipeline de deploy de uma aplicação utilizando GitLab, Docker e Kubernetes envolve vários passos. O objetivo desse projeto é automatizar a construção da imagem Docker, fazer o deploy no Kubernetes e integrar tudo com o GitLab CI/CD. A seguir, apresento um passo a passo completo para esse processo.

### Passo 1: Preparar o Repositório GitLab

1. **Crie um Repositório no GitLab**:
   - Crie um repositório no GitLab para a aplicação que você deseja automatizar o deploy.
   - Adicione o código-fonte da aplicação, incluindo os arquivos necessários, como `Dockerfile`, `k8s` (para as configurações do Kubernetes) e o código da aplicação.

### Passo 2: Criar o Dockerfile

O `Dockerfile` é usado para criar a imagem Docker que será usada para executar sua aplicação.

```Dockerfile
# Usando a imagem base oficial do Node.js (caso seja uma aplicação Node.js, por exemplo)
FROM node:16

# Defina o diretório de trabalho
WORKDIR /app

# Copie o package.json e o package-lock.json
COPY package*.json ./

# Instale as dependências
RUN npm install

# Copie o restante do código da aplicação
COPY . .

# Exponha a porta que a aplicação irá rodar
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["npm", "start"]
```

Esse exemplo é para uma aplicação Node.js. Modifique conforme a necessidade de sua aplicação.

### Passo 3: Configurar o Pipeline no GitLab (`.gitlab-ci.yml`)

O arquivo `.gitlab-ci.yml` é responsável por configurar o pipeline de CI/CD no GitLab. Vamos configurar etapas para construir a imagem Docker, fazer push da imagem para um repositório (como o Docker Hub ou GitLab Container Registry), e aplicar as configurações no Kubernetes.

```yaml
stages:
  - build
  - deploy

# Variáveis para configuração
variables:
  DOCKER_REGISTRY: registry.gitlab.com
  DOCKER_IMAGE: $CI_PROJECT_PATH
  KUBERNETES_CLUSTER: my-cluster
  KUBE_CONFIG: /path/to/your/kubeconfig

# Etapa de build
build:
  stage: build
  script:
    - echo "Construindo a imagem Docker..."
    - docker build -t $DOCKER_REGISTRY/$DOCKER_IMAGE:$CI_COMMIT_SHA .
    - docker push $DOCKER_REGISTRY/$DOCKER_IMAGE:$CI_COMMIT_SHA
  only:
    - master  # Ou a branch que você utiliza para produção

# Etapa de deploy para Kubernetes
deploy:
  stage: deploy
  script:
    - echo "Iniciando deploy no Kubernetes..."
    - kubectl --kubeconfig=$KUBE_CONFIG set image deployment/my-app my-app=$DOCKER_REGISTRY/$DOCKER_IMAGE:$CI_COMMIT_SHA
    - kubectl --kubeconfig=$KUBE_CONFIG rollout restart deployment/my-app
  only:
    - master
```

#### Explicação:
- **Estágio de Build**:
  - Construa a imagem Docker e a envie para o repositório de containers (no caso, GitLab Container Registry, mas poderia ser Docker Hub ou outro).
  
- **Estágio de Deploy**:
  - Usando o comando `kubectl`, o pipeline atualiza a imagem do deploy do Kubernetes para a nova versão da aplicação.
  - `kubectl set image` atualiza a imagem do deployment no Kubernetes.
  - `kubectl rollout restart` reinicia o deployment para garantir que a nova versão da aplicação seja executada.

### Passo 4: Configuração do Kubernetes

1. **Configuração do Cluster Kubernetes**:
   - Crie ou use um cluster Kubernetes existente. Pode ser em qualquer provedor (GKE, EKS, AKS ou até mesmo local).
   
2. **Criar os Manifests do Kubernetes**:
   Crie os arquivos de configuração para o deployment e o serviço no Kubernetes. Exemplo para um serviço simples de aplicação web:

**deployment.yml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: registry.gitlab.com/your-username/your-project:$CI_COMMIT_SHA
          ports:
            - containerPort: 3000
```

**service.yml**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

Esses arquivos descrevem o deployment da aplicação com duas réplicas e um serviço para expor a aplicação.

### Passo 5: Configuração do GitLab Runner

Certifique-se de que o GitLab Runner esteja configurado para usar Docker e Kubernetes.

1. **Instalar o GitLab Runner**:
   - Para que o GitLab possa executar o pipeline, você precisa configurar o GitLab Runner na sua máquina ou na infraestrutura de nuvem.

2. **Configurar Acesso ao Kubernetes**:
   - Crie um arquivo `kubeconfig` para o Kubernetes e defina a variável de ambiente `KUBE_CONFIG` no GitLab para apontar para esse arquivo.

### Passo 6: Testar o Pipeline

Agora que todos os arquivos estão configurados, faça um commit e push para o repositório. O GitLab CI/CD deve executar automaticamente o pipeline.

- O pipeline irá:
  - Construir a imagem Docker da aplicação.
  - Enviar a imagem para o registro de containers.
  - Atualizar o deployment no Kubernetes para usar a nova imagem.
  - Reiniciar a aplicação no Kubernetes.

### Conclusão

Agora você tem um pipeline de CI/CD automatizado usando GitLab, Docker e Kubernetes. Esse pipeline constrói e faz o deploy da sua aplicação sempre que houver um novo commit na branch de produção (geralmente `master`), garantindo que a aplicação esteja sempre atualizada com a versão mais recente do código.
