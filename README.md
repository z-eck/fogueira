# **Criação de Pipeline - Projeto Ignição**
### Objetivo:
O Projeto em si, é focado no funcionamento do CI/CD utilizando de serviços em nuvem e ferramentas que auxiliam nesse ciclo.
### Ferramentas usadas:
* [Google Cloud Plataform](https://console.cloud.google.com/)
	* [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview?hl=pt-br)
	* [Cloud Shell](https://cloud.google.com/shell/docs/using-cloud-shell?hl=pt-br)
	* [Artifact Registry](https://cloud.google.com/artifact-registry/docs/overview?hl=pt-br)
	* [Cloud Build](https://cloud.google.com/build/docs/overview?hl=pt-br)
	* [Secret Manager](https://cloud.google.com/secret-manager/docs?hl=pt-br#docs)
	* [Identity and Access Management (IAM)](https://cloud.google.com/iam/docs/overview?hl=pt-br)
* [Kubernetes](https://kubernetes.io/pt-br/docs/home/)
* [Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/)
* [Sonarqube](https://docs.sonarqube.org/9.8/)
* Git e GitHub ;)
#

## Aviso
Irei **IGNORAR** as partes de criação básica nas ferramentas (como por exemplo, a criação de projeto na GCP).

## Desenvolvimento

### Passo 1 - Repositórios
Inicialmente, é necessário criar um repositório Git, tanto para a integração do site em si, quanto os manifestos Kubernetes que serão utilizados mais tarde.
Deixarei os repositórios utilizados no projeto aqui:

- [Repositório do Site](https://github.com/z-eck/desafio-da-fogueira)
	- Projeto em React JS;
	- Contém os passos criados dentro da esteira da equipe.
- [Respositório dos Manifestos](https://github.com/z-eck/desafio-da-fogueira-manifestos)
	- Arquivos YAML para criação k8s que será utilizado pelo Argo CD mais tarde.

### Passo 2 - GKE

**AVISO** -> Necessário ter um projeto já criado para ser trabalhado!

- Criação do GKE Standart
	-  Acesse a página do [GKE](https://console.cloud.google.com/kubernetes/list/overview)
	- Vá em "CRIAR"
	- Selecione o projeto "STANDART"
	- Altere do cluster para um desejado
	- Selecione tipo "Regional" e insira "us-central1-a"
	- Abra o Dropdown do "default-pool" (localizado na lista à esquerda)
	- Vá em "Nós"
	- Diminua o tamanho do disco de inicialização de 100 GB para 65 GB
	- Clique em "CRIAR"
- Entrar na máquina k8s pelas credenciais
	-  Após aguardar a criação do cluster ser finalizada, clique no nome dele
	- Vá em "Conectar" e copia o comando em "Acesso à linha de comando"
	- Em seguida, ative o Cloud Shell" (fica localizado no canto direito em cima, é um símbolo de prompt)
	- Clique para abrir o Cloud Shell em uma nova aba.
	- Selecione o projeto que estiver trabalhando
	- Cole o comando no terminal e dê "Enter"

### Passo 3 - Argo CD

A utilização do Argo CD nesse cenário, é **ESSENCIAL**. Ele é atribuído em uma importante função de manter o "Continuos Delivery", ou traduzindo, entrega contínua, na qual o site possui atualizações automáticas (Zero interferência humana. Excelente!).

- Criação do Argo CD dentro do GKE
	A própria criação já fica disponível dentro do site do Argo CD, mas para facilitar a vida, irei disponibilizar o código mais recente de criação no momento (março/2023):
	- Inicialmente iremos criar um namespace ao Argo CD
		- "kubectl create namespace argocd"
	- Em seguida, iremos instalar o Argo CD no cluster do GKE recém criado:
		- "kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml"
	- Em seguida, iremos direcionar o Argo CD a um IP Externo (para podermos acessar a sua interface gráfica)
		- "kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}' "
	- E finalizando com a captura da senha da ferramenta.
		-  " kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d "
		- **DETALHE**: A senha é alterada a cada vez que você instancia um novo Argo CD. Ou seja, a senha que você possuirá, será diferente da minha! Trate de guardar ela bem.
	- Agora esperaremos cerca de 2 minutinhos, e iremos inserir o comando:
		- " kubectl get svc -n argocd "
		- Este comando listará todas as services que o Argo CD criou, uma delas terá a linha "Load Balancer" e em seguida um IP. Copie este IP e acesse em seu navegador.
	- Aqui você estará na página inicial do Argo CD então você irá inserir o usuário padrão "admin" e a senha que você pegou anteriormente.
	- Logo após de você acessar a ferramenta, siga essa sequência de criação de um novo app:
		- Vá em "+ NEW APP"
		- Insira o nome da aplicação
		- Insira "default" em "Project Name"
		- Altere a "Sync Policy" de "Manual" para "Automatic"
		- Em "Repository URL", insira o link do repositório git onde se encontra os manifestos do projeto em que você deseja automatizar. Por exemplo "https://github.com/z-eck/desafio-da-fogueira-manifestos.git"
		- Em "Path", insira o caminho onde se encontram os manifestos. No meu exemplo, eles estão na raiz do repositório, então é inserido apenas " . " (Sim, um ponto)
		- Em "Cluster URL", você selecionar "https://kubernetes.default.svc"
		- E em "Namespace", você insira o namespace onde será criado os manifestos dentro do kubernetes. No meu caso, eu criei um a parte denominado "fogueira" (comando é "kubectl create namespace fogueira"), mas caso não se importe de deixar tudo padrão, poderá inserir apenas "default" neste campo.
		- E clique em "CREATE"
		- Aguarde uns minutos e provavelmente seu novo app está criado e pronto para uso!
		
### Passo 4 - Sonarqube

A utilização do Sonarqube é precisa para a validação de código. A ferramenta em si é de excelente qualidade para verificar todas as inconsistências que podem aparecer no decorrer da aplicação desenvolvida (como códigos não seguindo uma boa estrutura, redundância desnecessárias ou até mesmo falhas de seguranças!).
- Criação do Sonarqube no GKE
	- Inicialmente iremos ir ao Cloud Shell para continuar a criação. Assim como o Argo CD, disponibilizarei o código atual de criação a seguir. **AVISO**: Usaremos o Helm para a criação do projeto Sonarqube, é o único momento em que ele é mencionado, por conta deste motivo, não inseri na lista de ferramentas usadas, mas é bom saber para que serve e a finalidade da ferramenta em si, antes de prosseguir com os comandos.
		- Inicialmente iremos adicionar o repo do sonarqube ao helm do GKE
			- helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
		- Em seguida, iremos atualizar o helm
			- helm repo update
		- Criaremos uma namespace para o sonarqube
			- kubectl create namespace sonarqube
		- E finalizaremos com a instalação do Sonarqube
			- helm upgrade --install -n sonarqube sonarqube sonarqube/sonarqube
		- Após instalado, ele pedirá alguns comandos a seguir, iremos ignorar estes para colocar um outro, mas é bom esperar um tempinho para os pods serem criados. 
		- Os comandos listados é para a criação do Sonarqube em uma máquina local, mas como estamos utilizando a nuvem, iremos usufruir novamente do Load Balancer:
			- Começaremos listar mais uma vez os services, só que desta vez, o do Sonarqube:
				- " kubectl get svc -n sonarqube "
			- E iremos inserir o comando para a criação do Load Balancer com o service do Sonarqube (ele não terá [nome-svc] igual ao Argo CD, muito que provavelmente):
				- " kubectl patch svc sonarqube-sonarqube -n sonarqube -p '{"spec": {"type": "LoadBalancer"}}' "
			- E em Seguida, iremos aguardar o novo IP para reutilizarmos o comando para listar as services do Sonarqube.
			- Após ter listado, pego o IP e acessado no navegador, iremos logar no Sonarqube. Seu usuário padrão é "admin" e a senha também é "admin".
			- Ao entrar na plataforma, vá em "Create Project"
			- Escolha as opções de criação manual
			- Insira o nome do projeto em que você deseja criar
			- A chave nova que você deseja também criar
			- E qual é a branch trabalhada (normalmente se mantém como main ou master)
			- Em seguida, selecione a opção "Locally" para a criação local
			- Crie um novo Token para o projeto e insira a sua expiração em que você deseja para o Token
			- Copie o Token inteiro (é desde sqp_***)
			- Continue
			- **AVISO**: No meu caso, eu trabalhei com duas linguagens: Python e JavaScript. Seguirei com a instalação para ambos. Caso você tenha criado utilizando Frameworks como Maven, Gradle ou .NET, siga as informações requerentes. A seguir, eu seguirei com "Other".
			- Selecione a opção "Other (for JS, TS, Go, Python, Php, ...)
			- Selecione o sistema Linux
			- E aqui é apresentado o comando. Neste caso, esse comando é extremamente útil para você que gostaria de instalar o Sonar-Scanner na sua máquina. Mas eu passarei um comando mais tranquilo utilizando Docker para você testar em sua máquina com o seu projeto:
				- É só abrir a pasta do seu projeto dentro da sua máquina, utilizando o terminal do seu sistema e inserindo o comando " docker run --rm sonarsource/sonar-scanner-cli -Dsonar.projectKey={chave_do_projeto} -Dsonar.sources=. -Dsonar.host.url={link_do_seu_Sonarqube}:9000 -Dsonar.login={seu_token} ". Não esqueça de substituir as chaves!
			- E aqui se encerra a criação do seu Sonarqube!

### Passo 5 - Artifact Registry
Ferramenta perfeita para salvar imagens da sua aplicação
- Acesse a página do [Artifact Registry](https://console.cloud.google.com/artifacts)
- Vá em "+ CRIAR REPOSITÓRIO"
- Insira o nome do seu Artifact desejado
- Mantenha como "Docker"
- Insira a região "us-central1 (Iowa)"
- E aperte em "CRIAR"
- Em seguida, vá a pasta recém criada dentro do Artifact
- Abra o repositório recém criado
- Aperte no botão de copiar nos caminhos logo ao lado do nome do repositório. No meu exemplo, está como "us-central1-docker.pkg.dev/site-zecklabs/ignicao-docker-repo/ignicao". Salve o seu caminho em um local em que você lembrará mais tarde (será muito útil na etapa do Cloud Build).
- Pronto, Artifact Registry criado!

### Passo 6 - Secret Manager

Ferramenta extremamente necessária para salvar dados sensíveis como o Token do Sonarqube e o nome de repositórios privados.
- Acesse a página do [Secret Manager](https://console.cloud.google.com/security/secret-manager)
- Vá em "+ CRIAR SECRET"
- Insira no nome como "SONAR_TOKEN"
- E cole o Token anteriormente gerado no Sonarqube"
- Clique em "CRIAR SECRET"
- Recomendo fazer o mesmo com o nome do projeto dentro do Sonar e com o IP do mesmo.
- E está finalizado a parte do Secret Manager, o resto será útil na etapa do Cloud Build!

### Passo 7 - Cloud Build

Ferramenta para capturar uma nova atualização do repositório Git e realizar uma série de criações.
- Inicialmente iremos para a página do [Cloud Build](https://console.cloud.google.com/cloud-build/triggers;region=us-central1) dentro da GCP
- Vá em Gatilhos
- Vá em "CRIAR GATILHO"
- Insira um nome
- Escolha a Região "us-central1"
- Vá Em "Fonte" e Conecte a um repositório desejado
- Em seguida, insira qual a branch onde estará o código e na qual ela ficará "de olho"
- Vá em "Configuração" e selecione "In-line"
- Copie e cole as informações inseridas no arquivo [cloudbuild.yaml](https://github.com/z-eck/fogueira/blob/main/cloudbuild.yaml) neste atual repositório, já alterando para as informações decalaradas no arquivo (Copie o arquivo para o seu computador, e edite em qualquer editor de texto).
- Clique em "Concluído"
- E aperte em "CRIAR"
- Na página "Gatilhos" novamente, vá em "Exibir" no novo gatilho recém criado
- Insira o repositório do código em "Ramificação"
- Aperte em "EXECUTAR GATILHO"
- Irá aparecer uma notificação da execução, e clique em "Mostrar"
- Só aguardar os passos finalizar. Caso fique tudo verde, deu bom! Caso fique vermelho ou cinza, deu ruim e hora de procurar o erro!
- E aqui se encerra a etapa do Cloud Build

### Considerações

A seguir, algumas etapas e explicações menores que ainda são válidas de serem lembradas.

#### Permissões- IAM

As permissões dentro do IAM da GCP, se resume a entregar elas ao Cloud Build. É necessário olhar as documentações, tanto do Artifact, quanto do Secret Manager para inserir as permissões ideais e necessárias para o funcionamento correto das etapas.

#### Site

O site, atualmente, é focado em descrever as 10 etapas da esteira dentro da equipe, não possui nenhum segredo além desse, é apenas para mostrar algum tipo de conteúdo e "bater o martelo" que a aplicação está sendo executada e tendo um bom sucesso.
