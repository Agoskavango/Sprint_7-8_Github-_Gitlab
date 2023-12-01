# Sprint_7-8_Github-_Gitlab
DESAFIO 3: Criar um repositorio no gitlab e uma pipeline que execute um "echo Hello World"
1. Crie um Repositório no GitLab:
Acesse o GitLab (https://gitlab.com/) e faça login na sua conta.
No menu lateral, clique em "New Project".
Escolha a opção para criar um projeto em branco ou importar um projeto do GitHub, Bitbucket, etc.
Preencha os detalhes do projeto, como nome, descrição, visibilidade, etc.
Clique em "Create project" para criar o repositório.
2. Configure a Pipeline:
No seu projeto recém-criado, clique na seção "CI/CD" no menu lateral.
No submenu, clique em "Pipelines".
Você será redirecionado para a página de pipelines vazia, pois ainda não configuramos nada.
3. Crie um Arquivo de Configuração da Pipeline:
No diretório raiz do seu projeto, crie um arquivo chamado .gitlab-ci.yml. Este arquivo contém as configurações da pipeline.
yaml
Copy code
# .gitlab-ci.yml

stages:
  - hello_world

hello_world:
  stage: hello_world
  script:
    - echo "Hello World"
Adicione e faça commit do arquivo .gitlab-ci.yml ao seu repositório.
4. Execute a Pipeline:
Agora, sempre que você fizer push de alterações para o repositório, a GitLab CI/CD executará automaticamente a pipeline.

Faça uma alteração em seu repositório (por exemplo, adicione um novo commit).
Faça push dessas alterações para o GitLab.
A GitLab CI/CD detectará automaticamente o arquivo .gitlab-ci.yml e iniciará a execução da pipeline. A seção "CI/CD > Pipelines" mostrará o progresso da execução da pipeline.

O job hello_world deve exibir "Hello World" no log de execução.

DESAFIO 4: Criar uma pipeline que faça o deploy do NGINX e que rode automaticamente após cada alteração

1. Dockerizar o NGINX:
Crie um Dockerfile para o NGINX no diretório do seu projeto. Isso permitirá que você construa uma imagem Docker para o NGINX.

Dockerfile
Copy code
# Dockerfile
FROM nginx:latest
COPY ./html /usr/share/nginx/html
Certifique-se de ter um diretório chamado html no mesmo diretório do seu Dockerfile, e adicione o conteúdo que deseja servir com o NGINX.

2. Configurar o Arquivo .gitlab-ci.yml:
Agora, você precisa adicionar a configuração da pipeline no arquivo .gitlab-ci.yml. Este exemplo assume que você já possui um registro de container configurado no GitLab.

yaml
Copy code
# .gitlab-ci.yml

stages:
  - deploy

deploy:
  stage: deploy
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
    - echo "Deploying to production..."
    - kubectl apply -f ./k8s/nginx-deployment.yaml  # Substitua pelo seu arquivo de configuração do Kubernetes
  only:
    - master  # Execute esta pipeline apenas para a branch master
3. Configurar o Kubernetes:
Você precisa ter um arquivo de configuração do Kubernetes (nginx-deployment.yaml) que define como o NGINX será implantado. Aqui está um exemplo básico:

yaml
Copy code
# nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry.gitlab.com/your-username/your-repo-name:$CI_COMMIT_REF_SLUG
        ports:
        - containerPort: 80
Ajuste este arquivo conforme necessário para atender às suas necessidades específicas.

4. Configurar Variáveis de Ambiente no GitLab:
Configure as variáveis de ambiente no GitLab para armazenar as credenciais necessárias para fazer push das imagens Docker e para interagir com o Kubernetes. As variáveis importantes podem incluir:

CI_REGISTRY_USER e CI_REGISTRY_PASSWORD para as credenciais do registro de container.
Credenciais para acessar o cluster Kubernetes.
5. Testar e Executar:
Faça um push de suas alterações para a branch master, e a pipeline será acionada automaticamente. Certifique-se de monitorar o progresso no GitLab CI/CD > Pipelines.
