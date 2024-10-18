tela dividida em 2 quadrantes
ele vai selecionar a ordem de carga e entao vai bipando
lado esquerdo -> bipando codigos de barras cada bipe é um volume
cada bipe vai atualizar o lado direito e volta pro lado esquerdo pra bipar.

reuniao
- [ ] vai colcoar trava pra conferir os volumes?
- [x] só carrega parte de cima quando preencher os dois camps
- [x] entender codigo de barras só numero da conferencia e seqvol?? nao vai ter nunota nao!?
- [x] faltou jogar em produção entao a rotina e também o relatorio formatado
- [x] quando alterar a oredm carga, seja pelo popup precisa atualizar O GRID!

---
SELECT cab.nunota,cab.tipmov,
  LPAD(CAB.NUNOTA,9,0)||LPAD('1', 3, 0) AS CODBARRAS
  
  from TGFCAB CAB where tipmov ='P'
  order by 1 desc
  
