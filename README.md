# Compass Uol

## Linux Aws + NFS Client + ApacheServer

<p>
    <img src = "https://d25lcipzij17d.cloudfront.net/badge.svg?id=gh&r=r&type=6e&v=3.0.0&x2=0" alt = "Version">
    <img src = "https://img.shields.io/badge/License-MIT-blue.svg" alt = "Licença MIT">
    <img src = "https://img.shields.io/badge/Made%20by-Lucas%20Emanoel-purple" alt = "Lucas Emanoel">
       <img src = "https://img.shields.io/badge/Project%20Lang-Portugueses%20BR-lightgreen" alt = "Project Lang">
       <img src = "https://badgen.net/badge/icon/github?icon=github&label" alt = "Github">
       <img src = "https://svgshare.com/i/Zhy.svg" alt = "Linux">
</p>

<br />
<br />
<div align="center">
  <a href="https://github.com/LucasEmanoel/Compass-Aws.git">
    <img src="https://github.com/LucasEmanoel/assets/blob/master/uol-logo.svg" alt="Logo" height="200">
  </a>

  <h2 align="center">Atividade Prática</h2>

  <p align="center">
    Iniciar uma instância na aws, transformar ela em um servidor apache com status check automatizado!
    <br />
    <a href="#"><strong>Explore a documentação »</strong></a>
    <br />
    <br />
  </p>
</div>
<br />
<br />

<details>
  <summary>Conteúdos</summary>
    <li>
      <a href="#sobre-o-projeto">Sobre o Projeto</a>
    </li>
    <li>
      <a href="#preparando-ambiente">Preparando Ambiente</a>
      <ul>
        <li><a href="#configurando-o-acesso-local-para-aws">Configurando o acesso local para Aws</a></li>
        <li><a href="#configurando-vpc">Configurando VPC</a></li>
        <li><a href="#configurando-ec2">Configurando EC2</a></li>
      </ul>
    </li>
    <li>
      <a href="#usando-ssh">Usando SSH</a>
    </li>
    <li>
      <a href="#release-100">Release 1.0.0</a>
      <ul>
        <li><a href="#transforme-sua-instancia-em-um-servidor-apache">Transforme sua instância em um servidor apache</a></li>
      </ul>
    </li>
    <li><a href="#release-200">Release 2.0.0</a></li>
    <li><a href="#release-300">Release 3.0.0</a></li>
    <li><a href="#uso">Recomendações de Uso</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contato">Contato</a></li>
</details>
<br />
<br />

# Sobre o projeto

Existem muitas razões para utilizarmos a plataforma cloud computing da amazon em trabalhos onde é requisitado servidores.
Aqui estão algumas:

* Aumento da segurança - de acordo com o modelo de responsabilidade.
* Estabilidade
* Flexibilidade

Para o momento iremos usar Aws Cloud Computing e é claro adotar boas práticas para aproveitarmos os benefícios da cloud, recomendo uma rápida leitura no [Modelo De Responsabilidade.](https://docs.aws.amazon.com/cli/index.html)

# Preparando Ambiente

De início vamos utilizar dois serviços disponibilizados pela aws e configurar com a [CLI](https://docs.aws.amazon.com/cli/index.html).

* Amazon [EC2](https://docs.aws.amazon.com/ec2/index.html)
* Amazon [VPC](https://docs.aws.amazon.com/vpc/)

## Configurando o acesso local para Aws

Para seguir o tutorial, você deve acessar o [serviço IAM](https://aws.amazon.com/pt/iam/) no `Console AWS`, criar um novo usuário ou usar um existente, ir na aba do **security credentials** e criar uma access key do tipo **command line interface**, por fim, faça o download das informações e vá para o terminal na sua maquina local:
<div align="center"  >
  <img src="https://github.com/LucasEmanoel/assets/blob/f267f64860716691a97714f2baac12ddca5ff920/access-keys.png" alt="img" height=50rem/>
  <img src="https://github.com/LucasEmanoel/assets/blob/f267f64860716691a97714f2baac12ddca5ff920/access-keys2.png" alt="img" height=450rem />
</div>

* Na maquina local, use os comando abaixo para instalar o cli, para instalar em windows ou mac veja: [Iniciando com CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip";
unzip awscliv2.zip;
sudo ./aws/install
```

* por fim use o comando aws configure.

```bash
sudo aws configure
```

```bash
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

1. Copie e cole o AWS Access Key ID.
2. Copie e cole o AWS Secret Access Key.
3. Defina uma região.
4. Escolha o tipo de saída dos comandos do CLI.

<br/>

Veja um exemplo:

![image](https://github.com/LucasEmanoel/assets/blob/f267f64860716691a97714f2baac12ddca5ff920/aws.png)

## Configurando VPC

1. caso não possua uma VPC, crie uma vpc com o comando abaixo.

* Caso tenha uma VPC, Copie o id na console aws e cole aqui.

```bash
VpcId="Cole o Id da vpc"
```

* Caso não, use o comando abaixo para criar.

```bash
VpcId=$(aws ec2 create-vpc --cidr-block 172.31.0.0/16 --query Vpc.VpcId --output text)
```

* Para associar outra VPC, copie o id dela no console aws.
* Note que usamos $() para executar um comando e armazenar o resultado na variável VpcId, devemos fazer o filtro dessa informação usando o --query e exportando o resultado em formato de texto usando o --output text.


2. Para criar as sua subnet, utilize os comandos abaixo.

* Caso tenha uma Subnet configurada, Copie o id na console aws e cole aqui.

```bash
SubnetId01="Cole o Id da subnet"
```

```bash
SubnetId01=`aws ec2 create-subnet --vpc-id $VpcId --cidr-block 172.64.1.0/20 --query Subnet.SubnetId --output text`
```

* Note que é preciso alterar os finais pois estamos fazendo uma divisão da rede, assim disponibilizando 256 ip's, para cada uma uma pois usamos /24.

* Por fim use `modify-subnet-attribute`, para a instancia ser criada automaticamente com ip publico.

```bash
aws ec2 modify-subnet-attribute --subnet-id $SubnetId01 --map-public-ip-on-launch
```

3. Criando Internet Gateway(IGW).

```bash
IgwId=`aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text`
```

* basicamente o IGW, é responsável por conectar a instancia com a internet, logo é necessário um em nosso ambiente.

4. Vinculando o IGW na nossa VPC.

```bash
aws ec2 attach-internet-gateway --vpc-id $VpcId --internet-gateway-id $IgwId
```

* É necessário fazer isso pois o IGW, deve está associado a VPC onde a instancia será criada.

5. Criando um Route Table(RT).

```bash
RouteTableId=`aws ec2 create-route-table --vpc-id $VpcId --query RouteTable.RouteTableId --output text`
```

* O route table será responsável por direcionar o trafego nossa instancia, nesse caso iremos configurar ele para o igw criado.

6. Crie uma rota no route table direcionando para o igw

```bash
aws ec2 create-route --route-table-id $RouteTableId --destination-cidr-block 0.0.0.0/0 --gateway-id $IgwId
```

* Após criar a rota, associe a subnet utilizando `associate-route-table`.

```bash
aws ec2 associate-route-table --subnet-id $SubnetId01 --route-table-id $RouteTableId
```

* Pronto, agora você pode criar uma instancia dentro dessa subnet e conectar na internet

2. Criando um Security Group para ter acesso a porta TCP/22, porta padrão do SSH.

* Caso tenha uma Security Group configurado, Copie o id na console aws e cole aqui.

```bash
SgId="Cole o Id do security group"
```

```bash
SgId=`aws ec2 create-security-group --group-name AwsCompassSG --description "Security group for SSH access" --vpc-id $VpcId --query GroupId --output text`
```

* Permitir porta 22, para poder usar login SSH.

```bash
aws ec2 authorize-security-group-ingress --group-id $SgId --protocol tcp --port 22 --cidr 0.0.0.0/0
```

* Permitir porta 2049, para porta usada pelo NFS.

```bash
#TCP: 2049
aws ec2 authorize-security-group-ingress --group-id $SgId --protocol tcp --port 2049 --cidr 0.0.0.0/0
#UDP: 2049
aws ec2 authorize-security-group-ingress --group-id $SgId --protocol udp --port 2049 --cidr 0.0.0.0/0
```

* Permitir porta 111, Utilizadas pelo NFS.

```bash
#TCP: 111
aws ec2 authorize-security-group-ingress --group-id $SgId --protocol tcp --port 111 --cidr 0.0.0.0/0
#UDP: 111
aws ec2 authorize-security-group-ingress --group-id $SgId --protocol udp --port 111 --cidr 0.0.0.0/0
```

* Permitir porta 80 e 443, Utilizadas pelo Apache.

```bash
#TCP: 80
aws ec2 authorize-security-group-ingress --group-id $SgId --protocol tcp --port 80 --cidr 0.0.0.0/0
#TCP: 443
aws ec2 authorize-security-group-ingress --group-id $SgId --protocol tcp --port 443 --cidr 0.0.0.0/0
```

## Configurando EC2

1. Criando a SSH da instancia.

* Crie primeiramente um SSH para poder logar na instancia.

```bash
aws ec2 create-key-pair --key-name MySSH --query "KeyMaterial" --output text > MySSH.pem
```

* Utilizando o --query "KeyMaterial", podemos visualizar apenas o valor da chave ssh, assim usando >, para salvar o output da query num arquivo .pem, por fim altere as permissões usando chmod, para leitura e escrita.

```bash
chmod 400 MySSH.pem
```

5. Iniciando a instancia.

* Abra o console aws, e copie as seguintes informações.
  * ImageId - Vamos especificar uma id de imagem retirada da aws console.
  * SubnetId - Podemos escolher uma das 3 subnets criadas.
  * Device - Para especificar o armazenamento, no nosso caso 16gb.

```bash
ImageId="ami-0b5eea76982371e91"
```

* Especificando device de armazenamento.

```json
  {
    "DeviceName": "/dev/xvda",
    "Ebs":{ "VolumeSize":16,
            "VolumeType":"gp2" }
  }
```

* Iremos passar esse json para dentro de um comando `run-instances`

```bash
InstanceId=`aws ec2 run-instances --image-id $ImageId --count 1 --instance-type t3.small --key-name MySSH --security-group-ids $SgId --subnet-id $SubnetId01 --block-device-mappings "[{\"DeviceName\":\"/dev/xvda\",\"Ebs\":{\"VolumeSize\":16,\"VolumeType\":\"gp2\"}}]" --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=LUCAS EMANOEL SILVA BARROS},{Key=Project,Value=PB},{Key=CostCenter,Value=PBCompass}]' --query Instances.InstanceId --output text`
```

6. Criando e Associando Elastic IP a Instancia Criada.

* Esse comando irá criar um Elastic IP!

```bash
AllocationId=`aws ec2 allocate-address --query AllocationId --output text`
```

* Para associar com a instancia usamos o comando abaixo.

```bash
aws ec2 associate-address --allocation-id $AllocationId --instance-id $InstanceId
```

# Usando SSH

* Abra o console aws, e copie a seguinte informação:
  * Elastic Ip - Ip publico associado a instancia.

<br/>

<div align="center"  >
    <img src="https://github.com/LucasEmanoel/assets/blob/f267f64860716691a97714f2baac12ddca5ff920/elastic-ip.png" alt="img" height=100rem/>
</div>

<br/>

```bash
ssh -i "MySSH.pem" ec2-user@[Elastic-IP]
```

# Release 1.0.0

## Transforme sua instancia em um servidor apache

<br/>

* Comando para instalar o apache, em distribuições que usam o package manager yum. caso esteja usando dpkg tente: `sudo apt install apache2`.

```bash
sudo yum -y update;
sudo yum -y install httpd
```

* Agora você precisa utilizar o comando `systemctl start` iniciar o servidor apache!

```bash
sudo systemctl start httpd
```

* Verifique se está tudo configurado e rodando com o comando `status`!

```bash
sudo systemctl status httpd
```

* Caso precise, habilite o httpd para iniciar com a instancia usando `enable`.

```bash
sudo systemctl enable httpd
```

# Release 2.0.0

## Conectar [Elastic File System (EFS).](https://aws.amazon.com/pt/efs/)

* O EFS permite que os possamos escalar nosso File System, ele funciona similar a um nfs onde podemos acessar os arquivos na rede em mais de uma instancia, assim provendo escalabilidade e permitindo espaço adicional.

## Instalar Pacotes necessários

```bash
sudo yum -y update;
sudo yum -y install amazon-efs-utils
```

* Basicamente poderemos fazer um mount e acessar os arquivos em nosso maquina.

```bash
sudo mkdir -p /mnt/efs
```

* Para fazer o mount na pasta criada basta usar o comando `mount -t efs`, em seguida, especificar o `<dns-efs>:<path-efs> <path-local>`

```bash
mount -t efs fs-00551b6438692354b.efs.us-east-1.amazonaws.com:/ /mnt/efs
```

# Release 3.0.0

## Criando um ShellScript para status check do servidor apache

<br/>

* Utilize novamente o gerenciador de pacotes `yum`, agora para instalar o cronie serviço que faremos o agendamento do script.

```bash
sudo yum -y update;
sudo yum install -y cronie
```

* Inicie o serviço e verifique se está correto.

```bash
sudo systemctl start crond
```

```bash
sudo systemctl status crond
```

* Caso necessário habilite o serviço para iniciar com o sistema

```bash
sudo systemctl enable crond
```

1. Criando nosso primeiro script.

* Iremos criar uma pasta especifica para armazenar os shell scripts criados.

```bash
sudo mkdir -p scripts/status_apache
```

```bash
sudo vim scripts/status_apache/script.sh
```

* Começamos o script especificando o bash.
* Note que é uma boa pratica especificar qual script atual você está trabalhando e fazer uma breve descrição.

```sh
#!/bin/bash
# path: scripts/status_apache/script.sh
# description: exemplo para verificar status de serviço no linux.

SERVICE="httpd"
STATUS="$(systemctl status $SERVICE | awk '/Active:/ {print $2 FS $3}')"
DATE=`date +%F-%T`

if [[ "${STATUS}" == "active (running)" ]]; then
        $(mkdir -p /mnt/efs/lucas-emanoel/log) echo "Data e Hora: $DATE - Serviço: $SERVICE | Status: $STATUS | Sucesso: O Servidor Está ONLINE! " > /mnt/efs/lucas-emanoel/log/SERVICO-ONLINE-$DATE.txt
else
        $(mkdir -p /mnt/efs/lucas-emanoel/log) echo "Data e Hora: $DATE - Serviço: $SERVICE | Status: $STATUS | Erro: O Servidor Está OFFLINE! " > /mnt/efs/lucas-emanoel/log/SERVICO-OFFLINE-$DATE.txt
        exit 1
fi
```

2. Precisamos alterar as permissões dos scripts para poder executa-los e das pastas para gravar os resultados dos logs.

```bash
sudo chmod +x scripts/status_apache/script.sh    
```

* Para o crontab poder escrever os arquivos de log vamos alterar as permissões de leitura e escrita.

```bash
sudo chmod go+rw /mnt/efs/lucas-emanoel
```

* Configurando Crond

3. Utilize o comando crontab para abrir o arquivo de agendamentos do usuário atual.

```bash
sudo crontab -e
```

* Dentro do arquivo você poderá configurar o tempo em que cada script será executado.
* Observe que cada * representa uma unidade de medida:
  * 1º - Minutos
  * 2º - Horas
  * 3º - Dias
  * 4º - Meses
  * 5º - Dia da Semana

```sh
*/5 * * * * /scripts/status_apache/script.sh 
```

* Para verificar se as configurações deram certo, use o comando abaixo e verifique se o arquivo de log foi criado na pasta de logs.

```sh
sudo crontab -l
```

```sh
sudo ls -lh /mnt/efs/lucas-emanoel/log
```

# Uso

Para usar em seu ambiente fique ciente que deve seguir os passos adaptando para seu console aws, configurando sua cli, caso necessário consultado informações diretamente no console, alguns comandos você aplicará diretamente no seu terminal então recomendo uma visualização mais detalhada na documentação do [CLI](https://docs.aws.amazon.com/cli/index.html).

<p align="right"><a href="#compass-uol">volte pra o inicio</a></p>

# Roadmap

- [x] Inicie uma Instancia Aws - t3.small - 16gb SSD
- [x] Associe um Elastic Ip
- [x] Subir um Servidor Apache
- [x] Fazer um Mount no EFS Disponibilizado Pela Compass
- [x] Criar um Script de Status
  - [x] Data + Hora
  - [x] Nome do Serviço
  - [x] Status do Serviço
  - [x] Mensagem Online ou Offline
  - [x] Gerar 1 Arquivo para Cada Status
  - [x] Execução a Cada 5 Minutos
  - [x] Enviar os Arquivos para EFS

<p align="right"><a href="#compass-uol">volte pra o inicio</a></p>

# Contato

Lucas Emanoel - [@Lucas Barros](https://www.linkedin.com/in/lucas-barros-979011135) - lucas2014.barros@gmail.com

Project Link: [Linux Aws Pratica](https://github.com/LucasEmanoel/Linux-Aws-Pratica.git)

<p align="right"><a href="#compass-uol">volte pra o inicio</a></p>

