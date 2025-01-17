Usamos o terminal web pra fazer o AWS → isso é direto o site pra fazer.

Mas no dia a dia, e entrega automatizada, devOps → a melhor forma pra entregar isso é com Terraform, nao usando o terminal web!

tem também o uso de scripts/sdk → pra criar o ambiente, mas ainda sim nao é a melhor forma, isso é criar uma instrução de criação. Exemplo: cria esse cluster kubernets.
só que isso também nao é o melhor pq precisa sempre ter algo pra criar o proximo passo, tipo o EC2 precisa de uma VPC.

Terraform → cria de forma declarativa → voce vai colocar e ele vai criar. Pegar esse codigo que declaramos de infraestrutura e vai criar.
	→ nao precisamos nos importar com a ordem de execução
	→ pode usar isso como documentação do projeto
	→ da pra criar automação com pipeline e etc.
	→ por ser codigo, da pra versionar.
	→ criar usando terraform em qualquer cloud provider

	→ ele é gratuito na sua maioria -> só vai ser pago se for algo que concorre com a Hashicorp

## Providers
Plugin no terraform que vai ser o meio de campo com o serviço de cloud.
no site do terraform já da uma opção infinita de providers.


### Mãos a Obra
instalar o terraform → terraform.io da pra instalar em qualquer OS
professor ta com ele instalado no WSL, claro.

Faz uma pasta no linux e conectou no vscode mesmo com wsl →  fazer o arquivo main.tf
no registry.terraform.io → encontra o aws e lá tem tudo sobre o provider de cloud que vamos usar.
	→ no registry tem um botao Use provider e vai ter uma cola do codigo de como vai declarar !!

Ansible → ferramenta que complementa o terraform depois de um ambiente ja criado pelo terraform. pesquisar caso precise de algo!

depois de criar o arquivo até o vpc ali → ele ja tem um codigo que cria algo na AWS

Executar comando, que vai executar o tf
	→ terraform init → ele vai pegar o arquivo que fizemos e vai criar os arquivos hcl já do que feito no codigo.
	→ terraform plan → dá o plano de execução, analisar o codigo que fizemos e oq ele vai fazer, e no final do plan ele pergunta se vc quer executar → da pra ir ver na AWS até


### código do professor do tf feito

```js
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.83.1"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_subnet" "minha_subnet" {
  vpc_id     = aws_vpc.minha_vpc.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "minha-subnet"
  }
}

resource "aws_vpc" "minha_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "minha-vpc"
  }
}

resource "aws_internet_gateway" "meu_ig" {
  vpc_id = aws_vpc.minha_vpc.id

  tags = {
    Name = "meu-ig"
  }
}

resource "aws_route_table" "meu_rt" {
  vpc_id = aws_vpc.minha_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.meu_ig.id
  }

  tags = {
    Name = "meu-rt"
  }
}

resource "aws_route_table_association" "subnet_rt_a" {
  subnet_id      = aws_subnet.minha_subnet.id
  route_table_id = aws_route_table.meu_rt.id
}

resource "aws_key_pair" "meu_kp" {
  key_name   = "meu-kp"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_instance" "minha_ec2" {

  ami           = "ami-0e2c8caa4b6378d8c"
  instance_type = "t2.micro"
  key_name = aws_key_pair.meu_kp.key_name
  associate_public_ip_address = true
  subnet_id = aws_subnet.minha_subnet.id
  vpc_security_group_ids = [aws_security_group.allow_all.id]

  tags = {
    Name = "minha-ec2"
  }
}

resource "aws_security_group" "allow_all" {
  name        = "allow_all"
  description = "SG da EC2 da Live de Aquecimento."
  vpc_id      = aws_vpc.minha_vpc.id

  ingress {
    description = "Ingress Aberto"
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    description = "Ingress Aberto"
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_all"
  }
}

output "ec2_ip" {
  value = aws_instance.minha_ec2.public_ip
}
```