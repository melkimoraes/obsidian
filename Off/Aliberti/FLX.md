RHID → PAGINA 91 

vai inserir na TFPMOV → 

TFPFUN → DATA ADMISSAO(DTADM) E CPF(CPF)
	condição: situacao not in 0,8

pega a empresa na TFPFUN com essas condiçoes de cima!

eles fecham do dia 25 ao 26.

a referencia na TGFMOV é TRUNC, MM da DATAFIM!

eventos?

falta e atraso =soma as horas minutos segundos no caso 1 hora e 8 1,08

sequencia = 0
tipevento = desconto -1, provento = 1 (colocar no de/para)
vlrmov = null
indice = é a soma de horas 1.08 / soma dos dias
unidade = qtd (d ou h) de/para
tipmov = M
dtalter = sysdate

se tiver falta (dia) nao somar horas e atraso

---

- [x] login na RHID é por usuario ou posso criar preferecia global?