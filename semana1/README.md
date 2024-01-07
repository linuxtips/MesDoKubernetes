# MesDoKubernetes

### O que é o Kubernetes?

**Versão resumida:**

O projeto Kubernetes foi desenvolvido pela Google, em meados de 2014, para atuar como um orquestrador de contêineres para a empresa. O Kubernetes (k8s), cujo termo em Grego significa "timoneiro", é um projeto *open source* que conta com *design* e desenvolvimento baseados no projeto Borg, que também é da Google [1](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes/). Alguns outros produtos disponíveis no mercado, tais como o Apache Mesos e o Cloud Foundry, também surgiram a partir do projeto Borg.

Como Kubernetes é uma palavra difícil de se pronunciar - e de se escrever - a comunidade simplesmente o apelidou de **k8s**, seguindo o padrão [i18n](http://www.i18nguy.com/origini18n.html) (a letra "k" seguida por oito letras e o "s" no final), pronunciando-se simplesmente "kates".

**Versão longa:**

Praticamente todo software desenvolvido na Google é executado em contêiner [2](https://www.enterpriseai.news/2014/05/28/google-runs-software-containers/). A Google já gerencia contêineres em larga escala há mais de uma década, quando não se falava tanto sobre isso. Para atender a demanda interna, alguns desenvolvedores do Google construíram três sistemas diferentes de gerenciamento de contêineres: **Borg**, **Omega** e **Kubernetes**. Cada sistema teve o desenvolvimento bastante influenciado pelo antecessor, embora fosse desenvolvido por diferentes razões.

O primeiro sistema de gerenciamento de contêineres desenvolvido no Google foi o Borg, construído para gerenciar serviços de longa duração e jobs em lote, que anteriormente eram tratados por dois sistemas:  **Babysitter** e **Global Work Queue**. O último influenciou fortemente a arquitetura do Borg, mas estava focado em execução de jobs em lote. O Borg continua sendo o principal sistema de gerenciamento de contêineres dentro do Google por causa de sua escala, variedade de recursos e robustez extrema.

O segundo sistema foi o Omega, descendente do Borg. Ele foi impulsionado pelo desejo de melhorar a engenharia de software do ecossistema Borg. Esse sistema aplicou muitos dos padrões que tiveram sucesso no Borg, mas foi construído do zero para ter a arquitetura mais consistente. Muitas das inovações do Omega foram posteriormente incorporadas ao Borg.

O terceiro sistema foi o Kubernetes. Concebido e desenvolvido em um mundo onde desenvolvedores externos estavam se interessando em contêineres e o Google desenvolveu um negócio em amplo crescimento atualmente, que é a venda de infraestrutura de nuvem pública.

O Kubernetes é de código aberto - em contraste com o Borg e o Omega que foram desenvolvidos como sistemas puramente internos do Google. O Kubernetes foi desenvolvido com um foco mais forte na experiência de desenvolvedores que escrevem aplicativos que são executados em um cluster: seu principal objetivo é facilitar a implantação e o gerenciamento de sistemas distribuídos, enquanto se beneficia do melhor uso de recursos de memória e processamento que os contêineres possibilitam.

Estas informações foram extraídas e adaptadas deste [artigo](https://static.googleusercontent.com/media/research.google.com/pt-BR//pubs/archive/44843.pdf), que descreve as lições aprendidas com o desenvolvimento e operação desses sistemas.
&nbsp;
### Arquitetura do k8s

Assim como os demais orquestradores disponíveis, o k8s também segue um modelo *control plane/workers*, constituindo assim um *cluster*, onde para seu funcionamento é recomendado no mínimo três nós: o nó *control-plane*, responsável (por padrão) pelo gerenciamento do *cluster*, e os demais como *workers*, executores das aplicações que queremos executar sobre esse *cluster*.

É possível criar um cluster Kubernetes rodando em apenas um nó, porém é recomendado somente para fins de estudos e nunca executado em ambiente produtivo.

Caso você queira utilizar o Kubernetes em sua máquina local, em seu desktop, existem diversas soluções que irão criar um cluster Kubernetes, utilizando máquinas virtuais ou o Docker, por exemplo.

Com isso você poderá ter um cluster Kubernetes com diversos nós, porém todos eles rodando em sua máquina local, em seu desktop.

Alguns exemplos são:

* [Kind](https://kind.sigs.k8s.io/docs/user/quick-start): Uma ferramenta para execução de contêineres Docker que simulam o funcionamento de um cluster Kubernetes. É utilizado para fins didáticos, de desenvolvimento e testes. O **Kind não deve ser utilizado para produção**;

* [Minikube](https://github.com/kubernetes/minikube): ferramenta para implementar um *cluster* Kubernetes localmente com apenas um nó. Muito utilizado para fins didáticos, de desenvolvimento e testes. O **Minikube não deve ser utilizado para produção**;

* [MicroK8S](https://microk8s.io): Desenvolvido pela [Canonical](https://canonical.com), mesma empresa que desenvolve o [Ubuntu](https://ubuntu.com). Pode ser utilizado em diversas distribuições e **pode ser utilizado em ambientes de produção**, em especial para *Edge Computing* e IoT (*Internet of things*);

* [k3s](https://k3s.io): Desenvolvido pela [Rancher Labs](https://rancher.com), é um concorrente direto do MicroK8s, podendo ser executado inclusive em Raspberry Pi;

* [k0s](https://k0sproject.io): Desenvolvido pela [Mirantis](https://www.mirantis.com), mesma empresa que adquiriu a parte enterprise do [Docker](https://www.docker.com). É uma distribuição do Kubernetes com todos os recursos necessários para funcionar em um único binário, que proporciona uma simplicidade na instalação e manutenção do cluster. A pronúncia é correta é kay-zero-ess e tem por objetivo reduzir o esforço técnico e desgaste na instalação de um cluster Kubernetes, por isso o seu nome faz alusão a *Zero Friction*. **O k0s pode ser utilizado em ambientes de produção**;

* **API Server**: É um dos principais componentes do k8s. Este componente fornece uma API que utiliza JSON sobre HTTP para comunicação, onde para isto é utilizado principalmente o utilitário ``kubectl``, por parte dos administradores, para a comunicação com os demais nós, como mostrado no gráfico. Estas comunicações entre componentes são estabelecidas através de requisições [REST](https://restfulapi.net);

* **etcd**: O etcd é um *datastore* chave-valor distribuído que o k8s utiliza para armazenar as especificações, status e configurações do *cluster*. Todos os dados armazenados dentro do etcd são manipulados apenas através da API. Por questões de segurança, o etcd é por padrão executado apenas em nós classificados como *control plane* no *cluster* k8s, mas também podem ser executados em *clusters* externos, específicos para o etcd, por exemplo;

* **Scheduler**: O *scheduler* é responsável por selecionar o nó que irá hospedar um determinado *pod* (a menor unidade de um *cluster* k8s - não se preocupe sobre isso por enquanto, nós falaremos mais sobre isso mais tarde) para ser executado. Esta seleção é feita baseando-se na quantidade de recursos disponíveis em cada nó, como também no estado de cada um dos nós do *cluster*, garantindo assim que os recursos sejam bem distribuídos. Além disso, a seleção dos nós, na qual um ou mais pods serão executados, também pode levar em consideração políticas definidas pelo usuário, tais como afinidade, localização dos dados a serem lidos pelas aplicações, etc;

* **Controller Manager**: É o *controller manager* quem garante que o *cluster* esteja no último estado definido no etcd. Por exemplo: se no etcd um *deploy* está configurado para possuir dez réplicas de um *pod*, é o *controller manager* quem irá verificar se o estado atual do *cluster* corresponde a este estado e, em caso negativo, procurará conciliar ambos;

* **Kubelet**: O *kubelet* pode ser visto como o agente do k8s que é executado nos nós workers. Em cada nó worker deverá existir um agente Kubelet em execução. O Kubelet é responsável por de fato gerenciar os *pods* que foram direcionados pelo *controller* do *cluster*, dentro dos nós, de forma que para isto o Kubelet pode iniciar, parar e manter os contêineres e os pods em funcionamento de acordo com o instruído pelo controlador do cluster;

* **Kube-proxy**: Age como um *proxy* e um *load balancer*. Este componente é responsável por efetuar roteamento de requisições para os *pods* corretos, como também por cuidar da parte de rede do nó;

&nbsp;
### Portas que devemos nos preocupar

**CONTROL PLANE**

Protocol|Direction|Port Range|Purpose|Used By
--------|---------|----------|-------|-------
TCP|Inbound|6443*|Kubernetes API server|All
TCP|Inbound|2379-2380|etcd server client API|kube-apiserver, etcd
TCP|Inbound|10250|Kubelet API|Self, Control plane
TCP|Inbound|10251|kube-scheduler|Self
TCP|Inbound|10252|kube-controller-manager|Self

* Toda porta marcada por * é customizável, você precisa se certificar que a porta alterada também esteja aberta.


&nbsp;
**WORKERS**

Protocol|Direction|Port Range|Purpose|Used By
--------|---------|----------|-------|-------
TCP|Inbound|10250|Kubelet API|Self, Control plane
TCP|Inbound|30000-32767|NodePort|Services All

&nbsp;
### Conceitos-chave do k8s

É importante saber que a forma como o k8s gerencia os contêineres é ligeiramente diferente de outros orquestradores, como o Docker Swarm, sobretudo devido ao fato de que ele não trata os contêineres diretamente, mas sim através de *pods*. Vamos conhecer alguns dos principais conceitos que envolvem o k8s a seguir:

- **Pod**: É o menor objeto do k8s. Como dito anteriormente, o k8s não trabalha com os contêineres diretamente, mas organiza-os dentro de *pods*, que são abstrações que dividem os mesmos recursos, como endereços, volumes, ciclos de CPU e memória. Um pod pode possuir vários contêineres;

- **Deployment**: É um dos principais *controllers* utilizados. O *Deployment*, em conjunto com o *ReplicaSet*, garante que determinado número de réplicas de um pod esteja em execução nos nós workers do cluster. Além disso, o Deployment também é responsável por gerenciar o ciclo de vida das aplicações, onde características associadas a aplicação, tais como imagem, porta, volumes e variáveis de ambiente, podem ser especificados em arquivos do tipo *yaml* ou *json* para posteriormente serem passados como parâmetro para o ``kubectl`` executar o deployment. Esta ação pode ser executada tanto para criação quanto para atualização e remoção do deployment;

- **ReplicaSets**: É um objeto responsável por garantir a quantidade de pods em execução no nó;

- **Services**: É uma forma de você expor a comunicação através de um *ClusterIP*, *NodePort* ou *LoadBalancer* para distribuir as requisições entre os diversos Pods daquele Deployment. Funciona como um balanceador de carga.


### Instalando e customizando o Kubectl

#### Instalação do Kubectl no GNU/Linux

Vamos instalar o ``kubectl`` com os seguintes comandos.

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client
```
&nbsp;
#### Instalação do Kubectl no MacOS

O ``kubectl`` pode ser instalado no MacOS utilizando tanto o [Homebrew](https://brew.sh), quanto o método tradicional. Com o Homebrew já instalado, o kubectl pode ser instalado da seguinte forma.

```
sudo brew install kubectl

kubectl version --client
```
&nbsp;
Ou:

```
sudo brew install kubectl-cli

kubectl version --client
```
&nbsp;
Já com o método tradicional, a instalação pode ser realizada com os seguintes comandos.

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client
```
&nbsp;
#### Instalação do Kubectl no Windows

A instalação do ``kubectl`` pode ser realizada efetuando o download [neste link](https://dl.k8s.io/release/v1.24.3/bin/windows/amd64/kubectl.exe). 

Outras informações sobre como instalar o kubectl no Windows podem ser encontradas [nesta página](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/).


### Customizando o kubectl

#### Auto-complete

Execute o seguinte comando para configurar o alias e autocomplete para o ``kubectl``.

No Bash:

```bash
source <(kubectl completion bash) # configura o autocomplete na sua sessão atual (antes, certifique-se de ter instalado o pacote bash-completion).

echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanentemente ao seu shell.
```
&nbsp;
No ZSH:

```bash 
source <(kubectl completion zsh)

echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)"
```
&nbsp;
#### Criando um alias para o kubectl

Crie o alias ``k`` para ``kubectl``:

```
alias k=kubectl

complete -F __start_kubectl k
```
&nbsp;
### Criando um cluster Kubernetes

### Criando o cluster em sua máquina local

Vamos mostrar algumas opções, caso você queira começar a brincar com o Kubernetes utilizando somente a sua máquina local, o seu desktop.

Lembre-se, você não é obrigado a testar/utilizar todas as opções abaixo, mas seria muito bom caso você testasse. :D

#### Minikube

##### Requisitos básicos

É importante frisar que o Minikube deve ser instalado localmente, e não em um *cloud provider*. Por isso, as especificações de *hardware* a seguir são referentes à máquina local.

* Processamento: 1 core;
* Memória: 2 GB;
* HD: 20 GB.

##### Instalação do Minikube no GNU/Linux

Antes de mais nada, verifique se a sua máquina suporta virtualização. No GNU/Linux, isto pode ser realizado com o seguinte comando:

```
grep -E --color 'vmx|svm' /proc/cpuinfo
```
&nbsp;
Caso a saída do comando não seja vazia, o resultado é positivo.

Há a possibilidade de não utilizar um *hypervisor* para a instalação do Minikube, executando-o ao invés disso sobre o próprio host. Iremos utilizar o Oracle VirtualBox como *hypervisor*, que pode ser encontrado [aqui](https://www.virtualbox.org).

Efetue o download e a instalação do ``Minikube`` utilizando os seguintes comandos.

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

chmod +x ./minikube

sudo mv ./minikube /usr/local/bin/minikube

minikube version
```
&nbsp;
##### Instalação do Minikube no MacOS

No MacOS, o comando para verificar se o processador suporta virtualização é:

```
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```
&nbsp;
Se você visualizar `VMX` na saída, o resultado é positivo.

Efetue a instalação do Minikube com um dos dois métodos a seguir, podendo optar-se pelo Homebrew ou pelo método tradicional.

```
sudo brew install minikube

minikube version
```
&nbsp;
Ou:

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64

chmod +x ./minikube

sudo mv ./minikube /usr/local/bin/minikube

minikube version
```
&nbsp;
##### Instalação do Minikube no Microsoft Windows

No Microsoft Windows, você deve executar o comando `systeminfo` no prompt de comando ou no terminal. Caso o retorno deste comando seja semelhante com o descrito a seguir, então a virtualização é suportada.

```
Hyper-V Requirements:     VM Monitor Mode Extensions: Yes
                          Virtualization Enabled In Firmware: Yes
                          Second Level Address Translation: Yes
                          Data Execution Prevention Available: Yes
```
&nbsp;
Caso a linha a seguir também esteja presente, não é necessária a instalação de um *hypervisor* como o Oracle VirtualBox:

```
Hyper-V Requirements:     A hypervisor has been detected. Features required for Hyper-V will not be displayed.:     A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```
&nbsp;
Faça o download e a instalação de um *hypervisor* (preferencialmente o [Oracle VirtualBox](https://www.virtualbox.org)), caso no passo anterior não tenha sido acusada a presença de um. Finalmente, efetue o download do instalador do Minikube [aqui](https://github.com/kubernetes/minikube/releases/latest) e execute-o.


##### Iniciando, parando e excluindo o Minikube

Quando operando em conjunto com um *hypervisor*, o Minikube cria uma máquina virtual, onde dentro dela estarão todos os componentes do k8s para execução.

É possível selecionar qual *hypervisor* iremos utilizar por padrão, através no comando abaixo:

```
minikube config set driver <SEU_HYPERVISOR> 
```
&nbsp;
Você deve substituir <SEU_HYPERVISOR> pelo seu hypervisor, por exemplo o KVM2, QEMU, Virtualbox ou o Hyperkit.


Caso não queria configurar um hypervisor padrão, você pode digitar o comando ``minikube start --driver=hyperkit`` toda vez que criar um novo ambiente. 


##### Certo, e como eu sei que está tudo funcionando como deveria?

Uma vez iniciado, você deve ter uma saída na tela similar à seguinte:

```
minikube start

😄  minikube v1.26.0 on Debian bookworm/sid
✨  Using the qemu2 (experimental) driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🔥  Creating qemu2 VM (CPUs=2, Memory=6000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.24.1 on Docker 20.10.16 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

```

Você pode então listar os nós que fazem parte do seu *cluster* k8s com o seguinte comando:

```
kubectl get nodes
```
&nbsp;
A saída será similar ao conteúdo a seguir:

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   20s   v1.25.3
```
&nbsp;
Para criar um cluster com mais de um nó, você pode utilizar o comando abaixo, apenas modificando os valores para o desejado:

```
minikube start --nodes 2 -p multinode-cluster

😄  minikube v1.26.0 on Debian bookworm/sid
✨  Automatically selected the docker driver. Other choices: kvm2, virtualbox, ssh, none, qemu2 (experimental)
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.24.1 preload ...
    > preloaded-images-k8s-v18-v1...: 405.83 MiB / 405.83 MiB  100.00% 66.78 Mi
    > gcr.io/k8s-minikube/kicbase: 385.99 MiB / 386.00 MiB  100.00% 23.63 MiB p
    > gcr.io/k8s-minikube/kicbase: 0 B [_________________________] ?% ? p/s 11s
🔥  Creating docker container (CPUs=2, Memory=8000MB) ...
🐳  Preparing Kubernetes v1.24.1 on Docker 20.10.17 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node minikube-m02 in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=8000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.11.11
🐳  Preparing Kubernetes v1.24.1 on Docker 20.10.17 ...
    ▪ env NO_PROXY=192.168.11.11
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

```
&nbsp;
Para visualizar os nós do seu novo cluster Kubernetes, digite:

```
kubectl get nodes
```
&nbsp;
Inicialmente, a intenção do Minikube é executar o k8s em apenas um nó, porém a partir da versão 1.10.1 é possível usar a função de multi-node.

Caso os comandos anteriores tenham sido executados sem erro, a instalação do Minikube terá sido realizada com sucesso.

##### Ver detalhes sobre o cluster 

```
minikube status
```
&nbsp;
##### Descobrindo o endereço do Minikube

Como dito anteriormente, o Minikube irá criar uma máquina virtual, assim como o ambiente para a execução do k8s localmente. Ele também irá configurar o ``kubectl`` para comunicar-se com o Minikube. Para saber qual é o endereço IP dessa máquina virtual, pode-se executar:

```
minikube ip
```
&nbsp;
O endereço apresentado é que deve ser utilizado para comunicação com o k8s.

##### Acessando a máquina do Minikube via SSH

Para acessar a máquina virtual criada pelo Minikube, pode-se executar:

```
minikube ssh
```
&nbsp;
##### Dashboard do Minikube

O Minikube vem com um *dashboard* *web* interessante para que o usuário iniciante observe como funcionam os *workloads* sobre o k8s. Para habilitá-lo, o usuário pode digitar:

```
minikube dashboard
```
&nbsp;
##### Logs do Minikube

Os *logs* do Minikube podem ser acessados através do seguinte comando.

```
minikube logs
```
&nbsp;
##### Remover o cluster 

```
minikube delete
```
&nbsp;
Caso queira remover o cluster e todos os arquivos referente a ele, utilize o parametro *--purge*, conforme abaixo:

```
minikube delete --purge
```
&nbsp;
#### Kind

O Kind (*Kubernetes in Docker*) é outra alternativa para executar o Kubernetes num ambiente local para testes e aprendizado, mas não é recomendado para uso em produção.

##### Instalação no GNU/Linux

Para fazer a instalação no GNU/Linux, execute os seguintes comandos.

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind
```
&nbsp;
##### Instalação no MacOS

Para fazer a instalação no MacOS, execute o seguinte comando.

```
sudo brew install kind
```
&nbsp;
ou

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-darwin-amd64
chmod +x ./kind
mv ./kind /usr/bin/kind
```
&nbsp;
##### Instalação no Windows

Para fazer a instalação no Windows, execute os seguintes comandos.

```
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.14.0/kind-windows-amd64

Move-Item .\kind-windows-amd64.exe c:\kind.exe
```
&nbsp;
###### Instalação no Windows via Chocolatey

Execute o seguinte comando para instalar o Kind no Windows usando o Chocolatey.

```
choco install kind
```
&nbsp;
##### Criando um cluster com o Kind

Após realizar a instalação do Kind, vamos iniciar o nosso cluster.

```
kind create cluster

Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

```
&nbsp;
É possível criar mais de um cluster e personalizar o seu nome.

```
kind create cluster --name giropops

Creating cluster "giropops" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-giropops"
You can now use your cluster with:

kubectl cluster-info --context kind-giropops

Thanks for using kind! 😊
```
&nbsp;
Para visualizar os seus clusters utilizando o kind, execute o comando a seguir.

```
kind get clusters
```
&nbsp;
Liste os nodes do cluster.

```
kubectl get nodes
```
&nbsp;
##### Criando um cluster com múltiplos nós locais com o Kind

É possível para essa aula incluir múltiplos nós na estrutura do Kind, que foi mencionado anteriormente.

Execute o comando a seguir para selecionar e remover todos os clusters locais criados no Kind.

```
kind delete clusters $(kind get clusters)

Deleted clusters: ["giropops" "kind"]
```
&nbsp;
Crie um arquivo de configuração para definir quantos e o tipo de nós no cluster que você deseja. No exemplo a seguir, será criado o arquivo de configuração ``kind-3nodes.yaml`` para especificar um cluster com 1 nó control-plane (que executará o control plane) e 2 workers.

```
cat << EOF > $HOME/kind-3nodes.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
```
&nbsp;
Agora vamos criar um cluster chamado ``kind-multinodes`` utilizando as especificações definidas no arquivo ``kind-3nodes.yaml``.

```
kind create cluster --name kind-multinodes --config $HOME/kind-3nodes.yaml

Creating cluster "kind-multinodes" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind-multinodes"
You can now use your cluster with:

kubectl cluster-info --context kind-kind-multinodes

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```
&nbsp;
Valide a criação do cluster com o comando a seguir.

```
kubectl get nodes
```
&nbsp;
Mais informações sobre o Kind estão disponíveis em: https://kind.sigs.k8s.io

&nbsp;
### Primeiros passos no k8s
&nbsp;

### Instalação de um cluster Kubernetes


#### O que é um cluster Kubernetes?

Um cluster Kubernetes é um conjunto de nodes que trabalham juntos para executar todos os nossos `pods`. Um cluster Kubernetes é composto por nodes que podem ser tanto `control plane` quanto `workers`. O `control plane` é responsável por gerenciar o cluster, enquanto os `workers` são responsáveis por executar os `pods` que são criados no cluster pelos usuários.

Quando estamos pensando em um cluster Kubernetes, precisamos lembrar que a principal função do Kubernetes é orquestrar containers. O Kubernetes é um orquestrador de containers, sendo assim quando estamos falando de um cluster Kubernetes, estamos falando de um cluster de orquestradores de containers. Eu sempre gosto de pensar em um cluster Kubernetes como se fosse uma orquestra, onde temos uma pessoa regendo a orquestra, que é o `control plane`, e temos as pessoas musicistas, que estão executando os instrumentos, que são os `workers`.

Sendo assim, o `control plane` é responsável por gerenciar o cluster, como por exemplo:

* Criar e gerenciar os recursos do cluster, como por exemplo, `namespaces`, `deployments`, `services`, `configmaps`, `secrets`, etc.
* Gerenciar os `workers` do cluster.
* Gerenciar a rede do cluster. 
* O `etcd` desempenha um papel crucial na manutenção da estabilidade e confiabilidade do cluster. Ele armazena as informações de configuração de todos os componentes do `control plane`, incluindo os detalhes dos serviços, `pods` e outros recursos do cluster. Graças ao seu design distribuído, o `etcd` é capaz de tolerar falhas e garantir a continuidade dos dados, mesmo em caso de falha de um ou mais nós. Além disso, ele suporta a comunicação segura entre os componentes do cluster, usando criptografia `TLS` para proteger os dados.

* O `scheduler` é o componente responsável por decidir em qual nó os `pods` serão executados, levando em consideração os requisitos e os recursos disponíveis. O `scheduler` também monitora constantemente a situação do cluster e, se necessário, ajusta a distribuição dos pods para garantir a melhor utilização dos recursos e manter a harmonia entre os componentes. 

* O `controller-manager` é responsável por gerenciar os diferentes controladores que regulam o estado do cluster e mantêm tudo funcionando. Ele monitora constantemente o estado atual dos recursos e compara-os com o estado desejado, fazendo ajustes conforme necessário.

* Onde está o `api-server` é o componente central que expõe a API do Kubernetes, permitindo que outros componentes do `control plane`, como o `controller-manager` e o `scheduler`, bem como ferramentas externas, se comuniquem e interajam com o cluster. O `api-server` é a principal interface de comunicação do Kubernetes, autenticando e autorizando solicitações, processando-as e fornecendo as respostas apropriadas. Ele garante que as informações sejam compartilhadas e acessadas de forma segura e eficiente, possibilitando uma colaboração harmoniosa entre todos os componentes do cluster.


Já no lado dos `workers`, as coisa são bem mais simples, pois a principal função deles é executar os `pods` que são criados no cluster pelos usuários. Nos `workers` nós temos, por padrão, os seguintes componentes do Kubernetes:

* O `kubelet` é o agente que funciona em cada nó do cluster, garantindo que os containers estejam funcionando conforme o esperado dentro dos pods. Ele assume o controle de cada nó, garantindo que os containers estejam sendo executados conforme as instruções recebidas do `control plane`. Ele monitora constantemente o estado atual dos `pods` e compara-os com o estado desejado. Caso haja alguma divergência, o `kubelet` faz os ajustes necessários para que os containers sigam funcionando perfeitamente.

* O `kube-proxy`, que é o componente responsável fazer ser possível que os `pods` e os `services` se comuniquem entre si e com o mundo externo. Ele observa o `control plane` para identificar mudanças na configuração dos serviços e, em seguida, atualiza as regras de encaminhamento de tráfego para garantir que tudo continue fluindo conforme o esperado.


* Todos os `pods` de nossas aplicações.



#### Formas de instalar o Kubernetes

Hoje nós iremos focar a instalação do Kubernetes utilizando o `kubeadm`, que é uma das formas mais antigas para a criação de um cluster Kubernetes. Mas existem outras formas de instalar o Kubernetes, vou detalhar algumas delas aqui:

* **`kubeadm`**: É uma ferramenta para criar e gerenciar um cluster Kubernetes em vários nós. Ele automatiza muitas das tarefas de configuração do cluster, incluindo a instalação do control plane e dos nodes. É altamente configurável e pode ser usado para criar clusters personalizados.

* **`Kubespray`**: É uma ferramenta que usa o Ansible para implantar e gerenciar um cluster Kubernetes em vários nós. Ele oferece muitas opções para personalizar a instalação do cluster, incluindo a escolha do provedor de rede, o número de réplicas do control plane, o tipo de armazenamento e muito mais. É uma boa opção para implantar um cluster em vários ambientes, incluindo nuvens públicas e privadas.

* **`Cloud Providers`**: Muitos provedores de nuvem, como AWS, Google Cloud Platform e Microsoft Azure, oferecem opções para implantar um cluster Kubernetes em sua infraestrutura. Eles geralmente fornecem modelos predefinidos que podem ser usados para implantar um cluster com apenas alguns cliques. Alguns provedores de nuvem também oferecem serviços gerenciados de Kubernetes que lidam com toda a configuração e gerenciamento do cluster.

* **`Kubernetes Gerenciados`**: São serviços gerenciados oferecidos por alguns provedores de nuvem, como Amazon EKS, o GKE do Google Cloud e o AKS, da Azure. Eles oferecem um cluster Kubernetes gerenciado onde você só precisa se preocupar em implantar e gerenciar seus aplicativos. Esses serviços lidam com a configuração, atualização e manutenção do cluster para você. Nesse caso, você não tem que gerenciar o `control plane` do cluster, pois ele é gerenciado pelo provedor de nuvem.

* **`Kops`**: É uma ferramenta para implantar e gerenciar clusters Kubernetes na nuvem. Ele foi projetado especificamente para implantação em nuvens públicas como AWS, GCP e Azure. Kops permite criar, atualizar e gerenciar clusters Kubernetes na nuvem. Algumas das principais vantagens do uso do Kops são a personalização, escalabilidade e segurança. No entanto, o uso do Kops pode ser mais complexo do que outras opções de instalação do Kubernetes, especialmente se você não estiver familiarizado com a nuvem em que está implantando.

* **`Minikube` e `kind`**: São ferramentas que permitem criar um cluster Kubernetes localmente, em um único nó. São úteis para testar e aprender sobre o Kubernetes, pois você pode criar um cluster em poucos minutos e começar a implantar aplicativos imediatamente. Elas também são úteis para pessoas desenvolvedoras que precisam testar suas aplicações em um ambiente Kubernetes sem precisar configurar um cluster em um ambiente de produção.

Ainda existem outras formas de instalar o Kubernetes, mas essas são as mais comuns. Para mais detalhes sobre as outras formas de instalar o Kubernetes, você pode consultar a documentação oficial do Kubernetes.


#### Criando um cluster Kubernetes com o kubeadm

Agora que você já sabe o que é o Kubernetes e quais são as suas principais funcionalidades, vamos começar a instalar o Kubernetes em nosso cluster. Nesse momento iremos ver como realizar a criação de um cluster Kubernetes utilizando o `kubeadm`, porém no decorrer da nossa jornada iremos ver outras formas de instalar o Kubernetes.

Como falado, o `kubeadm` é uma ferramenta para criar e gerenciar um cluster Kubernetes em vários nós. Ele automatiza muitas das tarefas de configuração do cluster, incluindo a instalação do `control plane` e dos `nodes`.

Primeira coisa, para que possamos seguir em frente, temos que entender quais são os pré-requisitos para a instalação do Kubernetes. Para isso, você pode consultar a documentação oficial do Kubernetes, mas vou listar aqui os principais pré-requisitos:

* Linux

* 2 GB ou mais de RAM por máquina (menos de 2 GB não é recomendado)

* 2 CPUs ou mais

* Conexão de rede entre todas os nodes no cluster (pode ser via rede pública ou privada)

* Algumas portas precisam estar abertas para que o cluster funcione corretamente, as principais:

    * Porta 6443: É a porta padrão usada pelo Kubernetes API Server para se comunicar com os componentes do cluster. É a porta principal usada para gerenciar o cluster e deve estar sempre aberta.

    * Portas 10250-10255: Essas portas são usadas pelo kubelet para se comunicar com o control plane do Kubernetes. A porta 10250 é usada para comunicação de leitura/gravação e a porta 10255 é usada apenas para comunicação de leitura.

    * Porta 30000-32767: Essas portas são usadas para serviços NodePort que precisam ser acessíveis fora do cluster. O Kubernetes aloca uma porta aleatória dentro desse intervalo para cada serviço NodePort e redireciona o tráfego para o pod correspondente.

    * Porta 2379-2380: Essas portas são usadas pelo etcd, o banco de dados de chave-valor distribuído usado pelo control plane do Kubernetes. A porta 2379 é usada para comunicação de leitura/gravação e a porta 2380 é usada apenas para comunicação de eleição.


&nbsp;

##### Instalando o kubeadm
Estamos aqui para configurar nosso ambiente Kubernetes e, olha só, é simples como voar! Vamos lá!

##### Desativando o uso do swap no sistema

Primeiro, vamos desativar a utilização de swap no sistema. Isso é necessário porque o Kubernetes não trabalha bem com swap ativado:

```
sudo swapoff -a
```

##### Carregando os módulos do kernel

Agora, vamos carregar os módulos do kernel necessários para o funcionamento do Kubernetes:

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

##### Configurando parâmetros do sistema

Em seguida, vamos configurar alguns parâmetros do sistema. Isso garantirá que nosso cluster funcione corretamente:

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

##### Instalando os pacotes do Kubernetes

Hora de instalar os pacotes do Kubernetes! Coisa linda de ai meu Deus! Aqui vamos nós:

```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl
```

##### Instalando o containerd

Em seguida, vamos instalar o containerd, que são essenciais para nosso ambiente Kubernetes:

```
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update && sudo apt-get install -y containerd.io
```

##### Configurando o containerd

Agora, vamos configurar o containerd para que ele funcione adequadamente com o nosso cluster:

```
sudo containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl status containerd
```

##### Habilitando o serviço do kubelet

Por fim, vamos habilitar o serviço do kubelet para que ele inicie automaticamente com o sistema:

```
sudo systemctl enable --now kubelet
```

##### Configurando as portas

Antes de iniciar o cluster, lembre-se das portas que precisam estar abertas para que o cluster funcione corretamente. Precisamos ter as portas TCP 6443, 10250-10255, 30000-32767 e 2379-2380 abertas entre os nodes do cluster. No nosso exemplo, onde estaremos usando somente um node `control plane`, não precisamos nos preocupar com algumas quando temos mais de um node `control plane`, pois eles precisam se comunicar entre si para manter o estado do cluster, ou ainda as portas 30000-32767, que são usadas para serviços NodePort que precisam ser acessíveis fora do cluster. Elas nós podemos ir abrindo conforme a necessidade, conforme vamos criando os nossos serviços.

Por agora o que precisamos garantir são as portas TCP 6443 somente no `control plane` e as 10250-10255 abertas em todos nodes do cluster.

Em nosso exemplo vamos utilizar como CNI o Weave Net, que é um CNI que utiliza o protocolo de roteamento de pacotes do Kubernetes para criar uma rede entre os pods. Falarei mais sobre ele mais pra frente, mas como estamos falando das portas que são importantes para o cluster funcionar, precisamos abrir a porta TCP 6783 e as portas UDP 6783 e 6784, para que o Weave Net funcione corretamente.

Então já sabe, não esqueça de abrir as portas TCP 6443, 10250-10255 e 6783 no seu firewall.

##### Inicializando o cluster

Agora que temos tudo configurado, vamos iniciar o nosso cluster:

```
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=<O IP QUE VAI FALAR COM OS NODES>
```

&nbsp;


Substitua `<O IP QUE VAI FALAR COM OS NODES>` pelo endereço IP da máquina que está atuando como `control plane`.

Após a execução bem-sucedida do comando acima, você verá uma mensagem informando que o cluster foi inicializado com sucesso e todos os detalhes de sua inicialização, conforme é possível ver na saída do comando:

```bash
[init] Using Kubernetes version: v1.26.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.31.57.89]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-01 localhost] and IPs [172.31.57.89 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-01 localhost] and IPs [172.31.57.89 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 7.504091 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-01 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-01 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: if9hn9.xhxo6s89byj9rsmd
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.57.89:6443 --token if9hn9.xhxo6s89byj9rsmd \
	--discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477 

```

&nbsp;

Inclusive, você verá uma lista de comandos para configurar o acesso ao cluster com o kubectl. Copie e cole esse comando em seu terminal:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

&nbsp;

Essa configuração é necessária para que o kubectl possa se comunicar com o cluster, pois quando estamos copiando o arquivo `admin.conf` para o diretório `.kube` do usuário, estamos copiando o arquivo com as permissões de root, esse é o motivo de executarmos o comando `sudo chown $(id -u):$(id -g) $HOME/.kube/config` para alterar as permissões do arquivo para o usuário que está executando o comando.


##### Entendendo o arquivo admin.conf
Agora precisamos entender o que temos dentro do arquivo `admin.conf`. Antes de mais nada precisamos conhecer alguns pontos importantes sobre o a estrutura do arquivo `admin.conf`:

- É um arquivo de configuração do kubectl, que é o cliente de linha de comando do Kubernetes. Ele é usado para se comunicar com o cluster Kubernetes.

- Contém as informações de acesso ao cluster, como o endereço do servidor API, o certificado de cliente e o token de autenticação.

- Eu posso ter mais de um contexto dentro do arquivo `admin.conf`, onde cada contexto é um cluster Kubernetes. Por exemplo, eu posso ter um contexto para o cluster de produção e outro para o cluster de desenvolvimento, simples como voar.

- Ele contém os dados de acesso ao cluster, portanto, se alguém tiver acesso a esse arquivo, ele terá acesso ao cluster. (Desde que tenha acesso ao cluster, claro).

- O arquivo `admin.conf` é criado quando o cluster é inicializado.


Vou copiar aqui o conteúdo de um exemplo de arquivo `admin.conf`:

```yaml
apiVersion: v1

clusters:
- cluster:
    certificate-authority-data: SEU_CERTIFICADO_AQUI
    server: https://172.31.57.89:6443
  name: kubernetes

contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes

current-context: kubernetes-admin@kubernetes

kind: Config

preferences: {}

users:
- name: kubernetes-admin
  user:
    client-certificate-data: SUA_CHAVE_PUBLICA_AQUI
    client-key-data: SUA_CHAVE_PRIVADA_AQUI
```

&nbsp;

Simplificando, temos a seguinte estrutura:

```yaml
apiVersion: v1
clusters:
#...
contexts:
#...
current-context: kind-kind-multinodes
kind: Config
preferences: {}
users:
#...
```

&nbsp;

Vamos ver o que temos dentro de cada seção:

**Clusters**

A seção clusters contém informações sobre os clusters Kubernetes que você deseja acessar, como o endereço do servidor API e o certificado de autoridade. Neste arquivo, há somente um cluster chamado kubernetes, que é o cluster que acabamos de criar.

```yaml
- cluster:
    certificate-authority-data: SEU_CERTIFICADO_AQUI
    server: https://172.31.57.89:6443
  name: kubernetes
```

&nbsp;

**Contextos**

A seção contexts define configurações específicas para cada combinação de cluster, usuário e namespace. Nós somente temos um contexto configurado. Ele é chamado kubernetes-admin@kubernetes e combina o cluster kubernetes com o usuário kubernetes-admin.

```yaml
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
```

&nbsp;


**Contexto atual**

A propriedade current-context indica o contexto atualmente ativo, ou seja, qual combinação de cluster, usuário e namespace será usada ao executar comandos kubectl. Neste arquivo, o contexto atual é o kubernetes-admin@kubernetes.

```yaml
current-context: kubernetes-admin@kubernetes
```

&nbsp;

**Preferências**

A seção preferences contém configurações globais que afetam o comportamento do kubectl. Aqui podemos definir o editor de texto padrão, por exemplo.

```yaml
preferences: {}
```

&nbsp;

**Usuários**

A seção users contém informações sobre os usuários e suas credenciais para acessar os clusters. Neste arquivo, há somente um usuário chamado kubernetes-admin. Ele contém os dados do certificado de cliente e da chave do cliente.

```yaml
- name: kubernetes-admin
  user:
    client-certificate-data: SUA_CHAVE_PUBLICA_AQUI
    client-key-data: SUA_CHAVE_PRIVADA_AQUI
```

&nbsp;


Outra informação super importante que está contida nesse arquivo é referente as credenciais de acesso ao cluster. Essas credenciais são usadas para autenticar o usuário que está executando o comando kubectl. Essas credenciais são:

- **Token de autenticação**: É um token de acesso que é usado para autenticar o usuário que está executando o comando kubectl. Esse token é gerado automaticamente quando o cluster é inicializado. Esse token é usado para autenticar o usuário que está executando o comando kubectl. Esse token é gerado automaticamente quando o cluster é inicializado.

- **certificate-authority-data**: Este campo contém a representação em base64 do certificado da autoridade de certificação (CA) do cluster. A CA é responsável por assinar e emitir certificados para o cluster. O certificado da CA é usado para verificar a autenticidade dos certificados apresentados pelo servidor de API e pelos clientes, garantindo que a comunicação entre eles seja segura e confiável.

- **client-certificate-data**: Este campo contém a representação em base64 do certificado do cliente. O certificado do cliente é usado para autenticar o usuário ao se comunicar com o servidor de API do Kubernetes. O certificado é assinado pela autoridade de certificação (CA) do cluster e inclui informações sobre o usuário e sua chave pública.

- **client-key-data**: Este campo contém a representação em base64 da chave privada do cliente. A chave privada é usada para assinar as solicitações enviadas ao servidor de API do Kubernetes, permitindo que o servidor verifique a autenticidade da solicitação. A chave privada deve ser mantida em sigilo e não compartilhada com outras pessoas ou sistemas.

Esses campos são importantes para estabelecer uma comunicação segura e autenticada entre o cliente (geralmente o kubectl ou outras ferramentas de gerenciamento) e o servidor de API do Kubernetes. Eles permitem que o servidor de API verifique a identidade do cliente e vice-versa, garantindo que apenas usuários e sistemas autorizados possam acessar e gerenciar os recursos do cluster.

&nbsp;

Você pode encontrar os arquivos que são utilizados para adicionar essas credentiais ao seu cluster em `/etc/kubernetes/pki/`.
Lá temos os seguintes arquivos que são utilizados para adicionar essas credenciais ao seu cluster:

- **client-certificate-data**: O arquivo de certificado do cliente geralmente é encontrado em /etc/kubernetes/pki/apiserver-kubelet-client.crt.

- **client-key-data**: O arquivo da chave privada do cliente geralmente é encontrado em /etc/kubernetes/pki/apiserver-kubelet-client.key.

- **certificate-authority-data**: O arquivo do certificado da autoridade de certificação (CA) geralmente é encontrado em /etc/kubernetes/pki/ca.crt.

Vale lembrar que esse arquivo é gerado automaticamente quando o cluster é inicializado, e são adicionados ao arquivo `admin.conf` que é utilizado para acessar o cluster. Essas credenciais são copiadas para o arquivo `admin.conf` já convertidas para base64.

&nbsp;

Pronto, agora você já sabe o porquê copiamos o arquivo `admin.conf` para o diretório `~/.kube/` e como ele funciona.


Caso você queira, você pode acessar o conteúdo do arquivo `admin.conf` com o seguinte comando:


```bash
kubectl config view
```

&nbsp;

Ele somente irá omitir os dados de certificados e chaves privadas, que são muito grandes para serem exibidos no terminal.

&nbsp;

##### Adicionando os demais nodes ao cluster

Agora que já temos o nosso cluster inicializado e já estamos entendendo muito bem o que é o arquivo `admin.conf`, chegou o momento de adicionar os demais nodes ao nosso cluster.

Para isso, vamos novamente utilizar o comando `kubeadm`, porém ao invés de executar o comando no node do control plane, nesse momento precisamos rodar o comando diretamente no node que queremos adicionar ao cluster.

Quando inicializamos o nosso cluster, o `kubeadm` nos mostrou o comando que precisamos executar no novos nodes, para que eles possam ser adicinados ao cluster como `workers`.

```bash
sudo kubeadm join 172.31.57.89:6443 --token if9hn9.xhxo6s89byj9rsmd \
	--discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477 
```

&nbsp;

O comando kubeadm join é usado para adicionar um novo node ao cluster Kubernetes existente. Ele é executado nos worker nodes para que eles possam se juntar ao cluster e receber instruções do control plane. Vamos analisar as partes do comando fornecido:

- **kubeadm join**: O comando base para adicionar um novo nó ao cluster.

- **172.31.57.89:6443**: Endereço IP e porta do servidor de API do nó mestre (control plane). Neste exemplo, o nó mestre está localizado no IP 172.31.57.89 e a porta é 6443.

- **--token if9hn9.xhxo6s89byj9rsmd**: O token é usado para autenticar o nó trabalhador no nó mestre durante o processo de adesão. Os tokens são gerados pelo nó mestre e têm uma validade limitada (por padrão, 24 horas). Neste exemplo, o token é if9hn9.xhxo6s89byj9rsmd.

- **--discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477**: Este é um hash criptográfico do certificado da autoridade de certificação (CA) do control plane. Ele é usado para garantir que o nó worker esteja se comunicando com o nó do control plane correto e autêntico. O valor após sha256: é o hash do certificado CA.

Ao executar este comando no worker, ele iniciará o processo de adesão ao cluster. Se o token for válido e o hash do certificado CA corresponder ao certificado CA do nó do control plane, o nó worker será autenticado e adicionado ao cluster. Após a adesão bem-sucedida, o novo node começará a executar os Pods e a receber instruções do control plane, conforme necessário.

Após executar o `join` em cada worker node, vá até o node que criamos para ser o control plane, e execute:

```bash
kubectl get nodes
```

&nbsp;

```bash
NAME     STATUS   ROLES           AGE   VERSION
k8s-01   NotReady    control-plane   4m   v1.26.3
k8s-02   NotReady    <none>          3m   v1.26.3
k8s-03   NotReady    <none>          3m   v1.26.3
```

&nbsp;

Agora você já consegue ver que os dois novos nodes foram adicionados ao cluster, porém ainda estão com o status `Not Ready`, pois ainda não instalamos o nosso plugin de rede para que seja possível a comunicação entre os pods. Vamos resolver isso agora. :)

##### Instalando o Weave Net

Agora que o cluster está inicializado, vamos instalar o Weave Net:

```
$ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

&nbsp;

Aguarde alguns minutos até que todos os componentes do cluster estejam em funcionamento. Você pode verificar o status dos componentes do cluster com o seguinte comando:

```
kubectl get pods -n kube-system
```
&nbsp;

```
kubectl get nodes
```

&nbsp;

```bash
NAME     STATUS   ROLES           AGE   VERSION
k8s-01   Ready    control-plane   7m   v1.26.3
k8s-02   Ready    <none>          6m   v1.26.3
k8s-03   Ready    <none>          6m   v1.26.3
```

&nbsp;

O Weave Net é um plugin de rede que permite que os pods se comuniquem entre si. Ele também permite que os pods se comuniquem com o mundo externo, como outros clusters ou a Internet. 
Quando o Kubernetes é instalado, ele resolve vários problemas por si só, porém quando o assunto é a comunicação entre os pods, ele não resolve. Por isso, precisamos instalar um plugin de rede para resolver esse problema.

##### O que é o CNI?

CNI é uma especificação e conjunto de bibliotecas para a configuração de interfaces de rede em containers. A CNI permite que diferentes soluções de rede sejam integradas ao Kubernetes, facilitando a comunicação entre os Pods (grupos de containers) e serviços.

Com isso, temos diferentes plugins de redes, que seguem a especificação CNI, e que podem ser utilizados no Kubernetes. O Weave Net é um desses plugins de rede.

Entre os plugins de rede mais utilizados no Kubernetes, temos:

- **Calico** é um dos plugins de rede mais populares e amplamente utilizados no Kubernetes. Ele fornece segurança de rede e permite a implementação de políticas de rede. O Calico utiliza o BGP (Border Gateway Protocol) para rotear tráfego entre os nós do cluster, proporcionando um desempenho eficiente e escalável.

- **Flannel** é um plugin de rede simples e fácil de configurar, projetado para o Kubernetes. Ele cria uma rede overlay que permite que os Pods se comuniquem entre si, mesmo em diferentes nós do cluster. O Flannel atribui um intervalo de IPs a cada nó e utiliza um protocolo simples para rotear o tráfego entre os nós.

- **Weave** é outra solução popular de rede para Kubernetes. Ele fornece uma rede overlay que permite a comunicação entre os Pods em diferentes nós. Além disso, o Weave suporta criptografia de rede e gerenciamento de políticas de rede. Ele também pode ser integrado com outras soluções, como o Calico, para fornecer recursos adicionais de segurança e políticas de rede.

- **Cilium** é um plugin de rede focado em segurança e desempenho. Ele utiliza o BPF (Berkeley Packet Filter) para fornecer políticas de rede e segurança de alto desempenho. O Cilium também oferece recursos avançados, como balanceamento de carga, monitoramento e solução de problemas de rede.

- **Kube-router** é uma solução de rede leve para Kubernetes. Ele utiliza o BGP e IPVS (IP Virtual Server) para rotear o tráfego entre os nós do cluster, proporcionando um desempenho eficiente e escalável. Kube-router também suporta políticas de rede e permite a implementação de firewalls entre os Pods.

Esses são apenas alguns dos plugins de rede mais populares e amplamente utilizados no Kubernetes. Você pode encontrar uma lista completa de plugins de rede no site do Kubernetes.

Agora, qual você deverá escolher? A resposta é simples: o que melhor se adequar às suas necessidades. Cada plugin de rede tem suas vantagens e desvantagens, e você deve escolher aquele que melhor se adequar ao seu ambiente.

Minha dica, procure não ficar inventando muita moda, tenta utilizar os que são já validados e bem aceitos pela comunidade, como o Weave Net, Calico, Flannel, etc. 

O meu preferido é o `Weave Net` pela simplicidade de instalação e os recursos oferecidos.

Um cara que eu tenho gostado bastante é o `Cilium`, ele é bem completo e tem uma comunidade bem ativa, além de utilizar o BPF, que é um assunto super quente no mundo Kubernetes!

&nbsp;

Pronto, já temos o nosso cluster inicializado e o Weave Net instalado. Agora, vamos criar um Deployment para testar a comunicação entre os Pods.

```bash
kubectl create deployment nginx --image=nginx --replicas 3
```

&nbsp;

```bash
kubectl get pods -o wide
```

&nbsp;

```bash
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE     NOMINATED NODE   READINESS GATES
nginx-748c667d99-8brrj   1/1     Running   0          12s   10.32.0.4   k8s-02   <none>           <none>
nginx-748c667d99-8knx2   1/1     Running   0          12s   10.40.0.2   k8s-03   <none>           <none>
nginx-748c667d99-l6w7r   1/1     Running   0          12s   10.40.0.1   k8s-03   <none>           <none>
```

&nbsp;

Pronto, nosso cluster está funcionando e os Pods estão em execução em diferentes nós.

Agora você já pode se divertir e utilizar o seu mais novo cluster Kubernetes!

&nbsp;
