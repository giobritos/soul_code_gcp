# Passo a Passo - GCP

Date: November 1, 2022
Notes: Passo a passo de todos os serviços vistos da GCP até a presente data. Feito por Giovana de Brito Silva.

### **Criação de VPC**

- Criar 2 VPCs:
    - GCP Dashboard
    - Navigation menu
    - VPC netork
    - VPC networks
    - Create VPC networks
    - Name: vpc-a
    - Description: Rede para o setor A.
    - VPC network ULA IPv6 renge: Disabled
    - Subnets: Custom
        - Name: rede-a
        - Description: Sub-rede da região Brasil.
        - Região: southamerica-east1
        - IP stack type: IPv4
        - IP range: 192.168.5.0/24 (exemplo)
        - Private Google Access: Off
        - Flow logs: Off
        - Done
    - Firewall Rulles (não criar nenhuma agora)
    - Dynamic routing mode: Global
    - Create
    
    Repetir o passo a passo com VPC B.
    

### **Criação de Instance Template**

- Criar um template:
    - GCP Dashboard
    - Navigation menu
    - Compute Engine
    - Instance template
    - Create instance template
    - Name: instance-template-vpc-a
    - Tipo de máquina: E2 - e2-micro
    - Boot disk: escolher a S.O., nesse caso usaremos debian 11 (padrão)
    - Firewall:
        - Habilitar HTTP
        - Habilitar HTTPS
    - Opções avançadas
        - Networking
            - Interfaces
            - Clicar em cima do default e mudar para a vpc criada (vpc-a) e colocar a rede-a embaixo.
        - Management
            - Para copiar um código de automação, vai em [Learn more](https://cloud.google.com/compute/docs/startupscript?authuser=2&_ga=2.147490736.66293142.1667138081-1603924746.1666696967&_gac=1.95345134.1666958283.Cj0KCQjw--2aBhD5ARIsALiRlwDBrvKoCh3C5FyIG6R0UUvE68YKIFvZxhMBlrLxEgL2XPdEDhYhWzgaAlaoEALw_wcB), [VMs do Linux](https://cloud.google.com/compute/docs/instances/startup-scripts/linux?authuser=2#passing-directly), e copia o código que está em cinza:
            
            ```bash
            #! /bin/bash
             apt update
             apt -y install apache2
             cat <<EOF > /var/www/html/index.html
             <html><body><p>Linux startup script added directly.</p></body></html>
            ```
            
            Ou copia o código do professor como abaixo:
            
            - Criar automação:
            
            ```bash
            #! /bin/bash
            apt update
            apt -y install apache2
            cat <<EOF > /var/www/html/index.html
            <html><body><h1>Hello World</h1>
            <p>CRIEI ESSA PAGINA DE TESTE</p>
            </body></html>
            EOF
            ```
            
    - Create
    
    Repetir o passo a passo com o template B.
    

### **Criação de Grupos de Instâncias**

- Criar um instance group:
    - GCP Dashboard
    - Navigation menu
    - Compute Engine
    - Instance group
    - Create instance group
    - Name: vms-vpc-a
    - Description: Grupo de instâncias na VPC A.
    - Location: região especificada pela VPC criada
    - Instance template: instance-template-vpc-a
    - Autoscaling
        - On: add and remove (min: 2 max: 2, nesse caso apenas para manter 2 máquinas, o ideal é que tenha mais máquinas no máximo)
        - Metrics: clicar em cima da CPU utilization - 80% - off
        - Schedules: cool down period 15
    - Autohealing: não precisa ativar para esse caso, mas se precisar é só escolher o health check criado
    - Create
    
    Repetir o passo a passo com o grupo B.
    
- Mudando o texto do index via SSH:
    - GCP Dashboard
    - Navigation menu
    - Compute Engine
    - Instance group
    - Clica no nome do grupo
    - No lado direito da tela, clica no SSH da primeira máquina
    - Escrever o comando para ir para a pasta do index.html:
    
    ```bash
    cd /var/www/html 
    ```
    
    - Escrever comando para abrir o editor de texto como super usuário:
    
    ```bash
    sudo vim index.html
    ```
    
    - Escrever comando para começar a edição de texto:
    
    ```bash
    i 
    ```
    
    - Altera o texto entre <p> </p>, se quiser adicionar imagem adicionar o código: <img src="caminho_da_imagem">, onde caminho_da_imagem será a URL da imagem
    pública na bucket que foi criada
    - Clicar ESC quando terminar a edição
    - Escreve o comando para salvar e sair:
    
    ```bash
    :wq
    ```
    
    - Escreve o comando para sair do SSH (ou feche a janela dele):
    
    ```bash
    exit 
    ```
    
    - Para testar se seu certo, procurar o IP externo e colar o endereço na barra de navegação do navegador
    
    Repetir o passo a passo com a segunda máquina.
    
- Testando o Auto Scaling:
    - Abrir 2 SSH da mesma máquina
    - Escrever o comando no primeiro SSH, para instalar o HTOP:
    
    ```bash
    sudo apt install htop
    ```
    
    - Após instalação, abra o app da verificação. Escrever o comando no primeiro SSH:
    
    ```bash
    htop 
    ```
    
    - No segundo SSH, escrever o comando para causar congestionamento na máquina. Assim conseguimos testar nosso auto scaling:
    
    ```bash
    cat /dev/zero > /dev/null
    ```
    
    - **CTRL+C** para o loop
    - Voltar no primeiro SSH e clicar F10 para finalizar o htop.

### **Criação de Firewall rules**

- Adicionar um firewall a uma VPC existente:
    - GCP Dashboard
    - Navigation menu
    - VPC netork
    - VPC networks
    - Clica na vpc escolhida
    - Ir na aba Firewall
    - Add firewall rule
    - Name: libera-ssh (colocar sempre o nome com relação ao que será feito - ex. libera-web, libera-ping)
    - Logs: off
    - Network: escolha a vpc
    - Priority: 1000
    - Direction of traffic: ingress
    - Action on match: allow
    - Targets: all instances
    - Source filter: IPv4 ranges
    - Source IPv4 ranges: 0.0.0.0/0 (libera para qualquer rede, qualquer lugar, qualquer IP) *não aplicável em empresas*
    - Second source filter: none
    - Protocols and ports:
        
        Selecionar apenas uma opção.
        
        - TPC: quando tiver nº de porta (SSH - 22, HTTP - 80, HTTPS - 443)
        - Other: quando tiver apenas protocolo (PING - icmp)
    - Create
    
    Repetir o passo a passo com VPC B.
    
    Criar uma regra para caso de “incêndio”:
    
    - Name: incendio
    - Priority: 65535 - em caso de ataque mudar para 0
    - Action on match: Deny all
    - Protocol: Deny all

### **Criação de VPC Peering**

- Criar VPC peering:
    - GCP Dashboard
    - Navigation menu
    - VPC netork
    - VPC network peering
    - Create connection
    - Continue
    - Name: vpc-a-b (vai de a para b, depois criar o vpc-b-a que vai de b para a)
    - vpc-a
    - In project
    - vpc-b
    - Apenas última opção habilitada (Export subnet routes with public IP)
    - Create
    
    Repetir o passo a passo com VPC B.
    

### **Criação de buckets**

- Criar um bucket:
    - GCP Dashboard
    - Navigation menu
    - Cloud Storage
    - Buckets
    - Create bucket
    - Name: o nome dos buckets tem que ser únicos no mundo
    - Region: escolher apenas **region**, pois é a opção mais barata
    - Class: Standart, para poder acessar os documentos durante o curso
    - Control access
        - Selecionar enforce public access (depois da para mudar)
        - Access control: fine-grained (da acesso apenas aos documentos autorizados e não à bucket toda)
    - Protection tools: none
    - Create
    - Confirm
    
    Dentro da bucket: 
    
    - CREATE FOLDER - cria pastas (clicar no nome para entrar dentro dela)
    - UPLOAD FILES - adiciona arquivos
    
    Em public access aparecerá que o arquivo está Not public, então vamos alterar essas configurações:
    
    - Ir na aba Permissions
    - Public Access: remove public access prevention
    - Confirm
    - Ir na aba Objects
    - 3 pontos no lado direito do arquivo
    - Edit Access
    - Add entry
    - Entity: public
    - Name: allUsers
    - Access: reader (a pessoa não vai poder mexer, apenas ler/ver o documento).
    - Save
    - Copy URL e teste acesso para ver se está publico (essa url é o caminho dessa imagem na internet, podemos usar esse caminho para anexar esta imagem ao nosso servidor web por ex.)

### Criação de Health Checks

- Criar um health check:
    - GCP Dashboard
    - Navigation menu
    - Compute Engine
    - Health checks
    - Create health check
    - Name: verificar-index (colocar nome de acordo com o objetivo)
    - Scope: global
    - Request Path: /index.html
    - Protocol: HTTP
    - Port: 80
    - Proxy protocol: none
    - Logs: desabilitar
    - Interval: 10s
    - Timeout: 5s
    - Healthy threshold: 2 successes
    - Unhealthy threshold: 3 failures
    - Create

### Criação de Load Balancing

- Criar um load balance:
    - GCP Dashboard
    - Navigation menu
    - Network services
    - Load balancing
    - Create load balancer
    - HTTPS
        - Start configuration
    - From internet to my VMs
    - Global HTTP(S) Load Balancer
    - Name: lb-webserver
    - Frontend configuration
        - Name: lb-webserver-front
        - Protocol: HTTP
        - IP version: IPv4
        - IP adress: Ephemeral
        - Port: 80
    - Backend configuration
        - Clicar em backend services
        - Create a backend service
        - Name: lb-webserver-back
        - Backend type: instance group
        - Protocol: HTTP
        - Named port: http
        - Timeout: 30
        - Backends
            - Instance group: selecionar o grupo criado
            - Port numbers: 80
            - Balancing mode: utilization - 80% - 100%
        - Desabilitar Cloud CDN
        - Não habilitar o logging
    - Pular parte de routing
    - Create
    
    Para confirmar que a troca de servidores está ocorrendo:
    
    - Ir na aba frontends
    - Procurar o IP externo (address)
    - Colar na barra de navegação e atualizar a página diversas vezes (fica melhor a visualização se cada servidor estiver com um texto diferente)
    

### Criar Instância MySQL

- Criando VM MYSQL - GCP:
    - Navigation Menu
    - SQL
    - + Create Instance
    - Choose MySQL
    - Criar nome e senha (root)
    - Database Version: escolher mais atual
    - Configuration: Development
    - Região: single zone - escolher qualquer uma
    - Customize your instance
        - Machine type: Lightweight (mais barata)
        - Data protection: desativar backup e proteção de deleção
- CONECTANDO NO MYSQL - GCP Shell:
    - gcloud sql connect <nomeservidor> --user=root
    - {ENTER}
    - y
    - {COLOCA A SENHA - ELA NÃO APARECE - Senha DB: root (você digita e não vê nada é normal)}
    - {ENTER}
    - mysql > SHOW DATABASES;
    - exit (para sair)
    - {seta pra cima - ele vai mostra o ultimo comando (gcloud sql connect aula-db --user=root) }

### Dataproc

- Criação de **DataProc**
    - Autoscaling Policies*
        - Create autoscaling policy
        - Não criaremos agora, mas se fosse criar a melhor opção é MapReduce
        - Name (minúsculo, sem separações e com nome compatível com uso)
        - Region (nem toda região tem as máquinas todas para dataproc)
        - YARN (deixar padrão)
        - Graceful decommissioning (deixar padrão)
        - Cooldown duration (deixar padrão)
        - Worker configuration (deixar padrão)
    - Cluster on Compute Engine
        
        **Set Up cluster**
        
        - Name (único dentro do projeto)
        - Location (evitar região us-central1)
        - Cluster type: Single Node
        - Autoscaling  (None)
        - Versioning: 2.0 (debian 10)
        - Components: NÃO ESQUECER DE ATIVAR Enable component gateway, Jupyter Notebook, Zeppelin Notebook
        
        **Configure Nodes**
        
        - Machine family: N1 - N1-standard-2
        - Primary disk size: 200 GB
        - Primary disk type: standard persistent disk
        - Sole-tenancy (não habilitar)
        - Shielded VM: Turn on Integrity Monitoring
        
        **Customize cluster**
        
        *Criar uma VPC apenas para o cluster*
        
        - Network Configuration: default
        - Internal IP only: desmarcado
        - Dataproc Metastore: None
        - Labels, properties, initialization actions e metadata: default
        - ****Scheduled deletion: NÃO ESQUECER DE selecionar as duas opções!!!****
            - Delete on a fixed time schedule - Delete after elapsed time since creation: 10 hours
            - Delete after a cluster idle time period without submitted jobs: 2 horas
            - Cloud Storage staging bucket: selecionar bucket para espaço de preparação
        
        **Manage Security**
        
        - Project access (habilitar)
        - Encryption: Google-managed encryption key
        - Personal Cluster Authentication (default)
        - Secure Multi Tenancy (default)
        - Kerberos and Hadoop Secure Mode (default)
        
        **Create**
        
    
    Para usar usar o Jupyter no cluster, clicar no cluster e ir em Web Interfaces