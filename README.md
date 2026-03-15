# Case Técnico: Analista de Integração de Dados (Implantação)

## Script SQL para criação das tabelas
```sql
-- Criação do banco
CREATE DATABASE public;

-- Criação da tabela fornecedor
CREATE TABLE public.fornecedor(
	idfornecedor  varchar(25) NOT NULL,
	razao_social varchar(255) NOT NULL,
	CONSTRAINT fornecedor_pkey PRIMARY KEY (idfornecedor) -- Correção: 'idproduto' não pode ser atribuído como chave porque não existe na tabela.
);


CREATE TABLE public.produtos_filial(
	filial_id int4 NOT NULL, -- Campos chaves não podem ser nulos, então foi corrigido para NOT NULL
	produto_id varchar(255) NOT NULL,
	descricao varchar(255) NOT NULL,
	estoque float8 DEFAULT 0 NOT NULL,
	preco_unitario float8 DEFAULT '0' NOT NULL,
	preco_compra float8 DEFAULT '0' NOT NULL,
	preco_venda float8 DEFAULT '0' NOT NULL,
	idfornecedor varchar(25) NOT NULL, -- Este campo receberá valores alfanuméricos, então não pode ser int.
	CONSTRAINT produtos_filial_pkey PRIMARY KEY (filial_id, produto_id) -- O campo 'idproduto' não existe na tabela, foi corrigido.
);


-- Criação da tabela vendas
CREATE TABLE public.venda(
	venda_id int8 NOT NULL,
	data_emissao date NOT NULL,
	horariomov varchar(8) DEFAULT '00:00:00'::character varying NOT NULL,
	produto_id varchar(255) DEFAULT ''::character varying NOT NULL, -- Alterado para varchar(255) conforme a tabela produtos.
	qtde_vendida float8 NULL,
	valor_unitario numeric(12, 4) DEFAULT 0 NOT NULL,
	filial_id int8 DEFAULT 1 NOT NULL,
	item int4 DEFAULT 0 NOT NULL,
	unidade_medida varchar(3) NULL,
	CONSTRAINT pk_consumo PRIMARY KEY (filial_id, venda_id, data_emissao, produto_id, item, horariomov)
)

-- Criação da tabela pedidos_compra
CREATE TABLE public.pedido_compra(
	pedido_id int8 DEFAULT 0 NOT NULL, -- Receberá dados do tipo inteiro
	data_pedido date NOT NULL, --
	item int8 DEFAULT 0 NOT NULL, -- Receberá dados do tipo inteiro
	produto_id varchar(255) DEFAULT '0' NOT NULL, -- Mesmo padrão que a tabela produtos
	descricao_produto varchar(255) NULL,
	ordem_compra int8 DEFAULT 0 NOT NULL, -- Receberá dados do tipo inteiro
	qtde_pedida float8 NULL,
	filial_id int4 NULL,
	data_entrega date NULL,
	qtde_entregue float8 DEFAULT 0 NOT NULL,
	qtde_pendente float8 DEFAULT 0 NOT NULL,
	preco_compra float8 DEFAULT 0 NULL,
	idfornecedor  varchar(25) DEFAULT 0 NOT NULL, -- Mesmo tipo de dados da tabela de fornecedor, não poderá receber nulos por conta das relações.
	CONSTRAINT pedido_compra_pkey PRIMARY KEY (pedido_id , produto_id, item)
);

-- Criação da tabela entrada_mercadoria
CREATE TABLE public.entradas_mercadoria (
	data_entrada date NULL,
	nro_nfe varchar(255) NOT NULL,
	item int8 DEFAULT 0 NOT NULL, -- Receberá dados do tipo inteiro
	produto_id varchar(255) DEFAULT '0' NOT NULL, -- Mantendo o mesmo formato da tabela de produtos
	descricao_produto varchar(255) NULL,
	ordem_compra int8 DEFAULT 0 NOT NULL, -- Criando o capo ordem_compra para se relacionar com pedido_compra
	qtde_recebida float8 NULL, 
	filial_id int4 NOT NULL, -- Não pode ser nulo devido aos relacionamentos
	custo_unitario numeric(12, 4) DEFAULT 0 NOT NULL,
	CONSTRAINT entradas_mercadoria_pkey PRIMARY KEY (ordem_compra, item, produto_id, nro_nfe)
);

```

## Importação de dados
O arquivo de importação se chama **base_teste_systock.xlsx**. Antes de fazer a carga no banco é necessário fazer tratar e normalizar os dados.
Analisarei e importarei uma tabela por vez.

### Tabela "venda":
Importado no DBeaver usando um arquivo .csv de entrada, configurando o delimitador de colunas para " ; " e codifiação para utf-8.   
Tratamento realizado: Nenhum, os dados de origem estavam de acordo com o tipo de dados esperado e a importação ocorreu sem erros.
<details>
  <summary>Clique aqui para ver a planilha completa</summary>

| venda_id | data_emissao | horariomov | produto_id | qtde_vendida | valor_unitario | filial_id | item | unidade_medida |
|----------|--------------|------------|------------|--------------|----------------|-----------|------|----------------|
| 1        | 1/11/2025    | 08:00:00   | P1         | 5            | 78,93          | 1         | 1    | UN             |
| 2        | 3/2/2025     | 08:00:00   | P2         | 7            | 92,96          | 1         | 1    | UN             |
| 3        | 1/28/2025    | 08:00:00   | P3         | 9            | 197,61         | 1         | 1    | UN             |
| 4        | 1/10/2025    | 08:00:00   | P4         | 38,6         | 139,71         | 1         | 1    | UN             |
| 5        | 1/11/2025    | 08:00:00   | P5         | 3            | 126,79         | 1         | 1    | UN             |
| 6        | 1/24/2025    | 08:00:00   | P6         | 2            | 36,83          | 1         | 1    | UN             |
| 7        | 2/22/2025    | 08:00:00   | P7         | 5            | 40,75          | 1         | 1    | UN             |
| 8        | 1/26/2025    | 08:00:00   | P8         | 20,04        | 51,37          | 1         | 1    | UN             |
| 9        | 1/17/2025    | 08:00:00   | P9         | 6            | 172,55         | 1         | 1    | UN             |
| 10       | 1/3/2025     | 08:00:00   | P10        | 90           | 44,22          | 1         | 1    | UN             |
| 11       | 1/8/2025     | 08:00:00   | P11        | 6            | 190,37         | 1         | 1    | UN             |
| 12       | 1/21/2025    | 08:00:00   | P12        | 2,86         | 136,4          | 1         | 1    | UN             |
| 13       | 1/24/2025    | 08:00:00   | P13        | 13           | 61,85          | 1         | 1    | UN             |
| 14       | 2/7/2025     | 08:00:00   | P14        | 53           | 106,3          | 1         | 1    | UN             |
| 15       | 2/20/2025    | 08:00:00   | P15        | 27           | 43,4           | 1         | 1    | UN             |
| 16       | 2/17/2025    | 08:00:00   | P16        | 37,11        | 14,41          | 1         | 1    | UN             |
| 17       | 2/22/2025    | 08:00:00   | P17        | 3            | 139,8          | 1         | 1    | UN             |
| 18       | 2/18/2025    | 08:00:00   | P18        | 5            | 185,23         | 1         | 1    | UN             |
| 19       | 2/20/2025    | 08:00:00   | P19        | 10           | 182,51         | 1         | 1    | UN             |
| 20       | 2/28/2025    | 08:00:00   | P20        | 2            | 68,54          | 1         | 1    | UN             |
| 21       | 1/24/2025    | 08:00:00   | P21        | 25           | 61,85          | 1         | 1    | UN             |
| 22       | 2/7/2025     | 08:00:00   | P22        | 6            | 106,3          | 1         | 1    | UN             |
| 23       | 2/20/2025    | 08:00:00   | P23        | 7            | 43,4           | 1         | 1    | UN             |
| 24       | 2/17/2025    | 08:00:00   | P24        | 4            | 14,41          | 1         | 1    | UN             |
| 25       | 2/22/2025    | 08:00:00   | P25        | 8            | 139,8          | 1         | 1    | UN             |
| 26       | 2/18/2025    | 08:00:00   | P26        | 3,11         | 185,23         | 1         | 1    | UN             |
| 27       | 2/20/2025    | 08:00:00   | P27        | 3            | 182,51         | 2         | 1    | UN             |
| 28       | 3/28/2025    | 08:00:00   | P28        | 6            | 68,54          | 3         | 1    | UN             |
| 29       | 3/17/2025    | 08:00:00   | P24        | 5            | 14,41          | 1         | 1    | UN             |
| 30       | 3/22/2025    | 08:00:00   | P25        | 3            | 139,8          | 1         | 1    | UN             |
| 31       | 3/18/2025    | 08:00:00   | P26        | 4            | 185,23         | 1         | 1    | UN             |
| 32       | 3/20/2025    | 08:00:00   | P27        | 2            | 182,51         | 2         | 1    | UN             |
| 33       | 3/28/2025    | 08:00:00   | P28        | 1            | 68,54          | 3         | 1    | UN             |



</details>

### Tabela "pedido_compra":
Importado no DBeaver usando um arquivo .csv de entrada, configurando o delimitador de colunas para " ; " e codifiação para utf-8.   
Tratamento realizado:

- Divergências de datas do pedido e entrega: 20 dos 29 pedidos possuem a data de entrega inferior a data do pedido. Isso é um erro, porque não há como existir entrega sem um pedido previamente realizado, o mais breve que um fornecedor pode entregar é na mesma data do pedido.  
Será necessário consultar o ERP do cliente para validar se as datas estão invertidas ou se de fato é um erro, antes da importação.

- Deslocamento de dados: Existem dados na planilha de origem das colunas M até W que estão fora da estrutura padrão. É possível analisar os padrões e deduzir a sua posição, porém para as colunas R, U e V a decisão fica em aberto.
Para evitar divergência, esses dados não serão importados. O correto é fazer uma nova consulta no ERP do cliente.

- Campo "fornecedor_id": Foi alterado para "idfornecedor" conforme as tabelas do banco e adicionado o sufixo "F" que não estava presente nos valores.

<details>
  <summary>Clique aqui para ver a planilha completa antes do tratamento</summary>

| pedido_id | data_pedido | item | produto_id | descricao_produto | ordem_compra | qtde_pedida | filial_id | data_entrega | qtde_entregue | preco_compra | fornecedor_id | col1       | col2 | col3 | col4       | col5 | col6 | col7 | col8       | col9 | col10 | col11 |
|-----------|-------------|------|------------|-------------------|--------------|-------------|-----------|--------------|---------------|--------------|---------------|------------|------|------|------------|------|------|------|------------|------|-------|-------|
| 1         | 2025-01-02  | 1    | P1         | Produto 1         | 1            | 96          | 1         | 2025-02-27   | 10            | 46,67        | 1             |            |      |      |            |      |      |      |            |      |       |       |
| 2         | 2025-01-07  | 1    | P2         | Produto 2         | 2            | 14          | 1         | 2025-01-07   | 7             | 77,32        | 2             |            |      |      |            |      |      |      |            |      |       |       |
| 3         | 2025-01-05  | 1    | P3         | Produto 3         | 3            | 12          | 1         | 2025-01-03   | 2             | 47,82        | 3             |            |      |      |            |      |      |      |            |      |       |       |
| 4         | 2025-01-22  | 1    | P4         | Produto 4         | 4            | 27          | 1         | 2025-01-28   | 3             | 49,57        | 4             |            |      |      |            |      |      |      |            |      |       |       |
| 5         | 2025-01-28  | 1    | P5         | Produto 5         | 5            | 35          | 1         | 2025-02-28   | 12            | 57,18        | 5             |            |      |      |            |      |      |      |            |      |       |       |
| 6         | 2025-02-22  | 1    | P6         | Produto 6         | 6            | 98          | 1         | 2025-01-05   | 55            | 59,96        | 6             |            |      |      |            |      |      |      |            |      |       |       |
| 7         | 2025-03-01  | 1    | P7         | Produto 7         | 7            | 34          | 1         | 2025-02-01   | 29            | 49,22        | 7             |            |      |      |            |      |      |      |            |      |       |       |
| 8         | 2025-02-02  | 1    | P8         | Produto 8         | 8            | 29          | 1         | 2025-02-14   | 24            | 35,88        | 8             |            |      |      |            |      |      |      |            |      |       |       |
| 9         | 2025-01-15  | 1    | P9         | Produto 9         | 9            | 57          | 1         | 2025-01-28   | 34            | 28,48        | 9             |            |      |      |            |      |      |      |            |      |       |       |
| 10        | 2025-01-09  | 1    | P10        | Produto 10        | 10           | 49          | 1         | 2025-02-09   | 4             | 42,86        | 10            |            |      |      |            |      |      |      |            |      |       |       |
| 11        | 2025-02-22  | 1    | P11        | Produto 11        | 11           | 24          | 1         | 2025-01-08   | 12            | 14,82        | 11            |            |      |      |            |      |      |      |            |      |       |       |
| 12        | 2025-02-25  | 1    | P12        | Produto 12        | 12           | 91          | 1         | 2025-02-20   | 48            | 6,92         | 12            |            |      |      |            |      |      |      |            |      |       |       |
| 13        | 2025-02-23  | 1    | P13        | Produto 13        | 13           | 99          | 1         | 2025-02-02   | 91            | 65,44        | 13            |            |      |      |            |      |      |      |            |      |       |       |
| 14        | 2025-01-21  | 1    | P14        | Produto 14        | 14           | 96          | 1         | 2025-01-01   | 27            | 21,91        | 14            |            |      |      |            |      |      |      |            |      |       |       |
| 15        | 2025-02-04  | 1    | P15        | Produto 15        | 15           | 45          | 1         | 2025-01-04   | 1             | 85,04        | 15            |            |      |      |            |      |      |      |            |      |       |       |
| 16        | 2025-02-27  | 1    | P16        | Produto 16        | 16           | 84          | 1         | 2025-01-14   | 51            | 64,17        | 16            |            |      |      |            |      |      |      |            |      |       |       |
| 17        | 2025-01-08  | 1    | P17        | Produto 17        | 17           | 22          | 1         | 2025-01-19   | 7             | 74,55        | 17            |            |      |      |            |      |      |      |            |      |       |       |
| 18        | 2025-02-17  | 1    | P18        | Produto 18        | 18           | 63          | 1         | 2025-01-02   | 17            | 24,94        | 18            |            |      |      |            |      |      |      |            |      |       |       |
| 19        | 2025-02-19  | 1    | P19        | Produto 19        | 0            | 20          | 1         | 2025-01-08   | 0             | 22,21        | 19            |            |      |      |            |      |      |      |            |      |       |       |
| 20        | 2025-02-10  | 1    | P20        | Produto 20        | 0            | 25          | 1         | 2025-01-15   | 0             | 38,51        | 20            |            |      |      |            |      |      |      |            |      |       |       |
| 21        | 2025-02-25  | 1    | P12        | Produto 12        | 0            | 12          | 1         | 2025-02-20   | 0             | 6,92         | 12            | 2025-02-25 | 1    | P12  | Produto 12 | 12   | 91   | 1    | 2025-02-20 | 48   | 43    | 6,92  |
| 22        | 2025-02-23  | 1    | P13        | Produto 13        | 0            | 4           | 1         | 2025-02-02   | 0             | 65,44        | 13            | 2025-02-23 | 1    | P13  | Produto 13 | 13   | 99   | 1    | 2025-02-02 | 91   | 8     | 65,44 |
| 23        | 2025-01-21  | 1    | P14        | Produto 14        | 0            | 6           | 1         | 2025-01-01   | 0             | 21,91        | 14            | 2025-01-21 | 1    | P14  | Produto 14 | 14   | 96   | 1    | 2025-01-01 | 27   | 69    | 21,91 |
| 24        | 2025-02-04  | 1    | P15        | Produto 15        | 0            | 8           | 1         | 2025-01-04   | 0             | 85,04        | 15            | 2025-02-04 | 1    | P15  | Produto 15 | 15   | 45   | 1    | 2025-01-04 | 1    | 44    | 85,04 |
| 25        | 2025-02-27  | 1    | P16        | Produto 16        | 0            | 9           | 1         | 2025-01-14   | 0             | 64,17        | 16            | 2025-02-27 | 1    | P16  | Produto 16 | 16   | 84   | 1    | 2025-01-14 | 51   | 33    | 64,17 |
| 26        | 2025-01-08  | 1    | P17        | Produto 17        | 0            | 4           | 1         | 2025-01-19   | 0             | 74,55        | 17            | 2025-01-08 | 1    | P17  | Produto 17 | 17   | 22   | 1    | 2025-01-19 | 7    | 15    | 74,55 |
| 27        | 2025-02-17  | 1    | P18        | Produto 18        | 0            | 3           | 1         | 2025-01-02   | 0             | 24,94        | 18            | 2025-02-17 | 1    | P18  | Produto 18 | 18   | 63   | 1    | 2025-01-02 | 17   | 46    | 24,94 |
| 28        | 2025-02-19  | 1    | P19        | Produto 19        | 0            | 3           | 1         | 2025-01-08   | 0             | 22,21        | 19            | 2025-02-19 | 1    | P19  | Produto 19 | 19   | 40   | 1    | 2025-01-08 | 0    | 40    | 22,21 |
| 29        | 2025-02-10  | 1    | P20        | Produto 20        | 0            | 2           | 1         | 2025-01-15   | 0             | 38,51        | 20            | 2025-02-10 | 1    | P20  | Produto 20 | 20   | 86   | 1    | 2025-01-15 | 78   | 8     | 38,51 |


</details>  
  

<details>
  <summary>Clique aqui para ver a planilha completa após o tratamento</summary>

| pedido_id | data_pedido | item | produto_id | descricao_produto | ordem_compra | qtde_pedida | filial_id | data_entrega | qtde_entregue | preco_compra | idfornecedor |
|-----------|-------------|------|------------|-------------------|--------------|-------------|-----------|--------------|---------------|--------------|--------------|
| 1         | 02/01/2025  | 1    | P1         | Produto 1         | 1            | 96          | 1         | 27/02/2025   | 10            | 46,67        | F1           |
| 2         | 07/01/2025  | 1    | P2         | Produto 2         | 2            | 14          | 1         | 07/01/2025   | 7             | 77,32        | F2           |
| 4         | 22/01/2025  | 1    | P4         | Produto 4         | 4            | 27          | 1         | 28/01/2025   | 3             | 49,57        | F4           |
| 5         | 28/01/2025  | 1    | P5         | Produto 5         | 5            | 35          | 1         | 28/02/2025   | 12            | 57,18        | F5           |
| 8         | 02/02/2025  | 1    | P8         | Produto 8         | 8            | 29          | 1         | 14/02/2025   | 24            | 35,88        | F8           |
| 9         | 15/01/2025  | 1    | P9         | Produto 9         | 9            | 57          | 1         | 28/01/2025   | 34            | 28,48        | F9           |
| 10        | 09/01/2025  | 1    | P10        | Produto 10        | 10           | 49          | 1         | 09/02/2025   | 4             | 42,86        | F10          |
| 17        | 08/01/2025  | 1    | P17        | Produto 17        | 17           | 22          | 1         | 19/01/2025   | 7             | 74,55        | F17          |
| 26        | 08/01/2025  | 1    | P17        | Produto 17        | 0            | 4           | 1         | 19/01/2025   | 0             | 74,55        | F17          |


</details>

### Tabela "entradas_mercadoria":
Importado no DBeaver usando um arquivo .csv de entrada, configurando o delimitador de colunas para " ; " e codifiação para utf-8.   
Tratamento realizado: 
- Não detectei indícios de dados sujos, porém como a tabela pedido_compra não foi importada por completo devidos as falhas apontadas, a análise dos valores de entrada_mercadorias ficará impactada até a correção, pois essas tabelas estarão relacionadas no banco.

<details>
  <summary>Clique aqui para ver a planilha completa</summary>

| data_entrada | nro_nfe | item | produto_id | descricao_produto | ordem_compra | qtde_recebida | filial_id | custo_unitario |
|--------------|---------|------|------------|-------------------|--------------|---------------|-----------|----------------|
| 2025-02-27   | NFE1    | 1    | P1         | Produto 1         | 1            | 77            | 1         | 84,35          |
| 2025-01-20   | NFE2    | 1    | P2         | Produto 2         | 2            | 64            | 1         | 16,66          |
| 2025-02-18   | NFE3    | 1    | P3         | Produto 3         | 3            | 88            | 1         | 90,36          |
| 2025-02-12   | NFE4    | 1    | P4         | Produto 4         | 4            | 4             | 1         | 84,68          |
| 2025-02-19   | NFE5    | 1    | P5         | Produto 5         | 5            | 95            | 1         | 98,99          |
| 2025-02-08   | NFE6    | 1    | P6         | Produto 6         | 6            | 41            | 1         | 90,29          |
| 2025-01-03   | NFE7    | 1    | P7         | Produto 7         | 7            | 75            | 1         | 27,22          |
| 2025-02-21   | NFE8    | 1    | P8         | Produto 8         | 8            | 25            | 1         | 71,1           |
| 2025-02-13   | NFE9    | 1    | P9         | Produto 9         | 9            | 57            | 1         | 19,55          |
| 2025-03-01   | NFE10   | 1    | P10        | Produto 10        | 10           | 7             | 1         | 54,39          |
| 2025-01-23   | NFE11   | 1    | P11        | Produto 11        | 11           | 85            | 1         | 91,89          |
| 2025-01-02   | NFE12   | 1    | P12        | Produto 12        | 12           | 12            | 1         | 38,53          |
| 2025-02-20   | NFE13   | 1    | P13        | Produto 13        | 13           | 7             | 1         | 60,86          |
| 2025-01-10   | NFE14   | 1    | P14        | Produto 14        | 14           | 92            | 1         | 38,48          |
| 2025-01-13   | NFE15   | 1    | P15        | Produto 15        | 15           | 68            | 1         | 95,58          |
| 2025-01-22   | NFE16   | 1    | P16        | Produto 16        | 16           | 89            | 1         | 39,46          |
| 2025-02-24   | NFE17   | 1    | P17        | Produto 17        | 17           | 10            | 1         | 10,32          |
| 2025-01-31   | NFE18   | 1    | P18        | Produto 18        | 18           | 48            | 1         | 62,56          |
| 2025-02-13   | NFE19   | 1    | P19        | Produto 19        | 19           | 64            | 1         | 84,54          |
| 2025-01-01   | NFE20   | 1    | P20        | Produto 20        | 20           | 6             | 1         | 65,7           |


</details>

### Tabela "produtos_filial:
Importado no DBeaver usando um arquivo .csv de entrada, configurando o delimitador de colunas para " ; " e codifiação para utf-8.   
Tratamento realizado: 
- Nome de colunas alterados: "idproduto" para "produto_id".

<details>
  <summary>Clique aqui para ver a planilha completa antes do tratamento</summary>

| filial_id | idproduto | descricao  | estoque | preco_unitario | preco_compra | preco_venda | idfornecedor |
|-----------|-----------|------------|---------|----------------|--------------|-------------|--------------|
| 1         | P1        | Produto 1  | 88      | 42,65          | 144,13       | 40,79       | F8           |
| 1         | P2        | Produto 2  | 28      | 79,52          | 103,56       | 174,18      | F9           |
| 1         | P3        | Produto 3  | 40      | 119,5          | 24,14        | 60,69       | F10          |
| 1         | P4        | Produto 4  | 73      | 89,67          | 7,75         | 226,5       | F11          |
| 1         | P5        | Produto 5  | 97      | 135,99         | 36,18        | 89,92       | F12          |
| 1         | P6        | Produto 6  | 38      | 161,31         | 55,37        | 95,6        | F13          |
| 1         | P7        | Produto 7  | 131     | 153,82         | 14,04        | 46,64       | F7           |
| 1         | P8        | Produto 8  | 71      | 140,57         | 149,5        | 95,28       | F17          |
| 1         | P9        | Produto 9  | 2       | 30,88          | 137          | 164,32      | F18          |
| 1         | P10       | Produto 10 | 38      | 115,71         | 27,77        | 87,7        | F19          |
| 1         | P11       | Produto 11 | 154     | 147,99         | 29,39        | 44,95       | F1           |
| 1         | P12       | Produto 12 | 78      | 32,47          | 64,63        | 276,58      | F2           |
| 1         | P13       | Produto 13 | 79      | 194,04         | 58,3         | 99,05       | F3           |
| 1         | P14       | Produto 14 | 9       | 199,56         | 56,8         | 80,74       | F4           |
| 1         | P15       | Produto 15 | 131     | 101,15         | 107,6        | 29,24       | F5           |
| 1         | P16       | Produto 16 | 177     | 24,64          | 75,94        | 278,88      | F6           |
| 1         | P17       | Produto 17 | 105     | 195,63         | 126,25       | 183,92      | F7           |
| 1         | P18       | Produto 18 | 198     | 162,2          | 134,12       | 105,61      | F18          |
| 1         | P19       | Produto 19 | 148     | 184,36         | 121,69       | 234,58      | F19          |
| 1         | P20       | Produto 20 | 196     | 52,04          | 124,87       | 157,93      | F20          |


</details>

<details>
  <summary>Clique aqui para ver a planilha após o tratamento</summary>

| filial_id | produto_id | descricao  | estoque | preco_unitario | preco_compra | preco_venda | idfornecedor |
|-----------|------------|------------|---------|----------------|--------------|-------------|--------------|
| 1         | P1         | Produto 1  | 88      | 42,65          | 144,13       | 40,79       | F8           |
| 1         | P2         | Produto 2  | 28      | 79,52          | 103,56       | 174,18      | F9           |
| 1         | P3         | Produto 3  | 40      | 119,5          | 24,14        | 60,69       | F10          |
| 1         | P4         | Produto 4  | 73      | 89,67          | 7,75         | 226,5       | F11          |
| 1         | P5         | Produto 5  | 97      | 135,99         | 36,18        | 89,92       | F12          |
| 1         | P6         | Produto 6  | 38      | 161,31         | 55,37        | 95,6        | F13          |
| 1         | P7         | Produto 7  | 131     | 153,82         | 14,04        | 46,64       | F7           |
| 1         | P8         | Produto 8  | 71      | 140,57         | 149,5        | 95,28       | F17          |
| 1         | P9         | Produto 9  | 2       | 30,88          | 137          | 164,32      | F18          |
| 1         | P10        | Produto 10 | 38      | 115,71         | 27,77        | 87,7        | F19          |
| 1         | P11        | Produto 11 | 154     | 147,99         | 29,39        | 44,95       | F1           |
| 1         | P12        | Produto 12 | 78      | 32,47          | 64,63        | 276,58      | F2           |
| 1         | P13        | Produto 13 | 79      | 194,04         | 58,3         | 99,05       | F3           |
| 1         | P14        | Produto 14 | 9       | 199,56         | 56,8         | 80,74       | F4           |
| 1         | P15        | Produto 15 | 131     | 101,15         | 107,6        | 29,24       | F5           |
| 1         | P16        | Produto 16 | 177     | 24,64          | 75,94        | 278,88      | F6           |
| 1         | P17        | Produto 17 | 105     | 195,63         | 126,25       | 183,92      | F7           |
| 1         | P18        | Produto 18 | 198     | 162,2          | 134,12       | 105,61      | F18          |
| 1         | P19        | Produto 19 | 148     | 184,36         | 121,69       | 234,58      | F19          |
| 1         | P20        | Produto 20 | 196     | 52,04          | 124,87       | 157,93      | F20          |

</details>

### Tabela "fornecedor:
Importado no DBeaver usando um arquivo .csv de entrada, configurando o delimitador de colunas para " ; " e codifiação para utf-8.   
Tratamento realizado: Nenhum, os dados estavam limpos e normalizados. A importação ocorreu seu erros.

<details>
  <summary>Clique aqui para ver a planilha completa</summary>

| idfornecedor | razao_social       |
|--------------|--------------------|
| F1           | Fornecedor 1 LTDA  |
| F2           | Fornecedor 2 LTDA  |
| F3           | Fornecedor 3 LTDA  |
| F4           | Fornecedor 4 LTDA  |
| F5           | Fornecedor 5 LTDA  |
| F6           | Fornecedor 6 LTDA  |
| F7           | Fornecedor 7 LTDA  |
| F8           | Fornecedor 8 LTDA  |
| F9           | Fornecedor 9 LTDA  |
| F10          | Fornecedor 10 LTDA |
| F11          | Fornecedor 11 LTDA |
| F12          | Fornecedor 12 LTDA |
| F13          | Fornecedor 13 LTDA |
| F14          | Fornecedor 14 LTDA |
| F15          | Fornecedor 15 LTDA |
| F16          | Fornecedor 16 LTDA |
| F17          | Fornecedor 17 LTDA |
| F18          | Fornecedor 18 LTDA |
| F19          | Fornecedor 19 LTDA |
| F20          | Fornecedor 20 LTDA |


</details>


## Consultas SQL Básicas

>**Consumo por produto e mês**  
Monte uma consulta que traga o total de vendas, em quantidade e em valores (R$), de cada produto, no mês de fevereiro de 2025.

```sql
select produto_id as Produto,
SUM(qtde_vendida) as Quantidade_Vendida,
TO_CHAR(SUM(qtde_vendida * valor_unitario), 'LFM999G999G990D00')  as Faturamento
from public.venda 
where EXTRACT(YEAR FROM data_emissao) = 2025 
and EXTRACT(MONTH FROM data_emissao) = 2  
group by produto_id;
```

<details>
  <summary>Resultado</summary>

| produto | quantidade_vendida | faturamento |
|---------|--------------------|-------------|
| P14     | 53                 | R$5.633,90  |
| P15     | 27                 | R$1.171,80  |
| P16     | 37,11              | R$534,76    |
| P17     | 3                  | R$419,40    |
| P18     | 5                  | R$926,15    |
| P19     | 10                 | R$1.825,10  |
| P20     | 2                  | R$137,08    |
| P22     | 6                  | R$637,80    |
| P23     | 7                  | R$303,80    |
| P24     | 4                  | R$57,64     |
| P25     | 8                  | R$1.118,40  |
| P26     | 3,11               | R$576,07    |
| P27     | 3                  | R$547,53    |
| P7      | 5                  | R$203,75    |

</details>

>**Produtos com requisição pendente**  
Crie uma consulta para listar os produtos que foram requisitados, mas não recebidos.
```sql
select descricao_produto as produto,
SUM(qtde_pedida - qtde_entregue) as qtde_nao_recebidas
from pedido_compra
where (qtde_pedida - qtde_entregue) > 0
group by produto
order By qtde_nao_recebidas desc;
```
<details>
  <summary>Resultado</summary>

| produto    | qtde_nao_recebidas |
|------------|--------------------|
| Produto 1  | 86                 |
| Produto 10 | 45                 |
| Produto 4  | 24                 |
| Produto 5  | 23                 |
| Produto 9  | 23                 |
| Produto 17 | 19                 |
| Produto 2  | 7                  |
| Produto 8  | 5                  |


</details>

***Observação:*** Essa análise foi impactada pela divergência pontuada durante a importação da tabela "pedido_compra".`

## Transformações de Dados
>Crie consultas SQL ou funções com os seguintes requisitos para as pedidos de compra e vendas e produtos:
>1. Concatenar os campos produto_id e descricao_produto (onde houver) no formato;
>2. Transformar o campo de datas para o formato `DD/MM/YYYY`;
>3. Retornar os dados filtrando apenas os produtos requisitados mais de 10 vezes no período.

```sql
select
concat(pc.produto_id,' - '|| pf.descricao) as produto,
qtde_pedida as qtde_requisitada,
TO_CHAR(data_pedido, 'DD/MM/YYYY') as data_solicitacao
FROM pedido_compra pc
INNER JOIN produtos_filial pf ON pc.produto_id = pf.produto_id
where pc.qtde_pedida > 10;
```
<details>
  <summary>Resultado</summary>

| produto          | qtde_requisitada | data_solicitacao |
|------------------|------------------|------------------|
| P1 - Produto 1   | 96               | 01/02/2025       |
| P2 - Produto 2   | 14               | 01/07/2025       |
| P4 - Produto 4   | 27               | 01/10/2026       |
| P5 - Produto 5   | 35               | 01/04/2027       |
| P8 - Produto 8   | 29               | 02/02/2025       |
| P9 - Produto 9   | 57               | 01/03/2026       |
| P10 - Produto 10 | 49               | 01/09/2025       |
| P17 - Produto 17 | 22               | 01/08/2025       |
</details>  

>Criar uma trigger que gere automaticamente um novo `idfornecedor` numérico na tabela de produtos que se relacione com a tabela de fornecedor.
```sql
-- A função irá preencher o idfornedor com o valor mais recente da tabela de fornecedores

-- Criação do trigger
CREATE OR REPLACE FUNCTION fn_vincular_fornecedor_automatico()
RETURNS TRIGGER AS $$
BEGIN
    -- Busca o ID do fornecedor mais recente na tabela 'fornecedor'
    SELECT idfornecedor INTO NEW.idfornecedor 
    FROM fornecedor 
    ORDER BY idfornecedor DESC 
    LIMIT 1;

    -- Caso não exista nenhum fornecedor cadastrado, o banco cancela a inserção
    IF NEW.idfornecedor IS NULL THEN
        RAISE EXCEPTION 'Erro: Não existe nenhum fornecedor cadastrado para vincular ao produto.';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Aplicando a função na tabela produtos_filial
-- A regra será validada antes do insert
CREATE TRIGGER tg_gerar_idfornecedor_produto
BEFORE INSERT ON produtos_filial
FOR EACH ROW
EXECUTE FUNCTION fn_vincular_fornecedor_automatico();

```
Testando trigger:  
Estarei importando um registro com o campo idfornecedo vazio.
| filial_id | produto_id | descricao                  | estoque | preco_unitario | preco_compra | preco_venda | idfornecedor |
|-----------|------------|----------------------------|---------|----------------|--------------|-------------|--------------|
| 1         | P30        | Produto 30 (Teste Trigger) | 196     | 52,04          | 124,87       | 157,93      |              |

Após a importação, ao realizar o comando insert o banco preencheu o campo idoforncedor automaticamente.
| filial_id | produto_id | descricao                  | estoque | preco_unitario | preco_compra | preco_venda | idfornecedor |
|-----------|------------|----------------------------|---------|----------------|--------------|-------------|--------------|
| 1         | P30        | Produto 30 (Teste Trigger) | 196     | 52,04          | 124,87       | 157,93      | F9           |


## Estratégia de Validação com o Cliente

### Quais seriam os **principais pontos que você validaria com o cliente?**
1. Apontaria em seguida validaria as diversas divergências de datas e registros com o campos zerados da tabela 'pedido_compra'.  
2. Apontaria que existe divergência nos dados de venda, existem registros com produtos em filiais que não foram cadastradas na tabela 'produtos_filial'. O banco retorná erro ao importar esses dados devido ao relacionamento das tabelas 'venda' e 'produto_filial'.

| venda_id | data_emissao | horariomov | produto_id | qtde_vendida | valor_unitario | filial_id | item | unidade_medida |
|----------|--------------|------------|------------|--------------|----------------|-----------|------|----------------|
| 28       | 3/28/2025    | 08:00:00   | P28        | 6            | 68,54          | 3         | 1    | UN             |
| 33       | 3/28/2025    | 08:00:00   | P28        | 1            | 68,54          | 3         | 1    | UN             |
| 27       | 2/20/2025    | 08:00:00   | P27        | 3            | 182,51         | 2         | 1    | UN             |
| 32       | 3/20/2025    | 08:00:00   | P27        | 2            | 182,51         | 2         | 1    | UN             |


Quais técnicas utilizaria para garantir a **exatidão e a precisão** dos dados?  
1. Se for dados números de ponto flutuando podemos atribuir o tipo 'numeric' para configurar a precisão dos dados.
2. Campos com chaves primárias e estrangeiras não podem ter duplicadas ou valores nulos.

Quais **consultas você deixaria prontas** para usar na reunião de validação?
```sql
-- Para exibir os registros cadastrados, evidenciando que muitos ficaram de fora por conta das divergências
select count(*) as qtde_registros 
from pedido_compra;

-- Exibia os produtos que estão cadastrados por filial
select produto_id, filial_id as filial 
from produtos_filial 
group by produto_id, filial_id;
```