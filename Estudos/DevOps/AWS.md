![[Pasted image 20250115104558.png]]


Cloud Providers famoso: AWS, Azure e GCP (Google)


o servidor norte da virginia é mais barato principalmente pra quem ta estudando!

Professor deixa claro pra nao usar o root da criação da conta aws e sim criar um usuario com acesso total mas outro usuario, usando o serviço IAM dentro da aws.

# VPC

Pra tudo que for usar no AWS seja kubernets, EC2 e etc. Criar VPC pra um novo serviço sempre na AWS.

Nao usar VPC padrao que eles criam!

VPC → tem a CIDR Block que é o endereço ip do condominio

Subnet
vai criar uma subred→ queria seria o bloco do condominio
E na vdd aqui na criação vc vincula a AZ (que seria aquelas AZ’s que aparecem disponivel no mundo tod da AWS, no caso N. Virginia)
e aqui vai colocar o CIDR da subnet → ali seria quantos apartamentos vai ter condominio


e agora sim posso colocar algum serviço dentro da subnet!


VPC  → SubNet → EC2


EC2 → vai escolher qual OS ou imagem vai utilizar, no caso o professor selecionou Ubuntu.
na criação da EC2 depois vai pedir pra vc selecionar a instancia que dai tem a especificação da maquina e o preço.
se vai criar uma chave ali tb, e salvar ela no pc pq ela vai habilitar pra acesso via ssh (escolher o .pem)
na configuração de rede se vai colocar a vpc e subnet
habilita o acesso via ip publico
e ele desligou firewall por ser pra estudo né, produção no caso nao faria isso.
criado!

agora em EC2 → em instancias → vai ter a que criamos executando! o status check tem que só esperar ele sair de iniciando.

EKS no aws tem firewall. caso precise um dia.

na parte debaixo do EC2 já aparece tudo, ip publico, ip privado e por ai vai.

a chave .pem ele jogou na pasta de ssh no ubuntu do wsl, comando pra conectar via cmd do ubuntu:
ssh -i live.pem ubuntu@ip_publico
live.pem é o nome da chave na pasta ssh.

só que a subnet nao fizemos publica → nao deu certo acessar dessa forma, como fazer a subrede se tornar publica? 
	→ AQUI PRECISA DO IG → Internet Gateway na VPC!
		→ entrar na VPC → na aba do lado vai ter internet gateway
		→ e depois vai atacchar na VPC
		→ e agora Route tables → cria uma e seleciona a VPC → e depois associar essa RT na subnet, ali na RT tem associar Subnet

ai deu certo acessar o ubuntu que subiu no EC2 pelo AWS, ele até subiu um nginx e acessou no navegador mesmo com  o ip publico no browser!


POR ULTIMO → desligar a VPC pra parar de rodar o taximetro!
ec2 → Instances → actions → manage instance state → terminate → salvar!
vpc → route tables → desassociar a subnet nela
	deletar a route table
	internet gateways → desatachar → deletar dps