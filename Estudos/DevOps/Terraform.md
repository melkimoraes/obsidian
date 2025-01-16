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

# Providers
Plugin no terrafo