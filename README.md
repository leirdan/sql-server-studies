# OPERAÇÕES EM SQL SERVER 2022
* Primeiro, cria-se o banco de dados **SUCOS_VENDAS** através dos códigos SQL do arquivo "/src/Cria_Banco.sql";
* Após isso, carrega-se o arquivo "src/Carga_Cadastros.sql" para adicionar os vendedores e os produtos; * Em seguida, executa-se o arquivo "src/Carga_Notas.sql" para cadastrar todas as notas fiscais dos produtos. É um arquivo gigantesco, então pode demorar para acontecer todas as inserções.
* Por fim, executa-se também o arquivo "src/Carga_Itens_Notas.sql" para carregar os itens das notas fiscais. Novamente, um arquivo colossal, então aguarde.

## 1. FILTRAGEM
* Podemos utilizar dos filtros nos comandos SQL para escrever consultas melhores.
* Exemplos de filtros:
	* **Where [expressão lógica]**: aplica um filtro no SQL onde uma linha será exibida se, e somente se, ela atender à condição do where;
		* Por exemplo, vamos listar somente as informações de produtos vendidos pelo vendedor de matrícula "00236":
		```sql
		SELECT v.MATRICULA, nf.NUMERO, inf.CODIGO_DO_PRODUTO, inf.PRECO 
			from NOTAS_FISCAIS nf
			inner join ITENS_NOTAS_FISCAIS inf 
				on inf.NUMERO = nf.NUMERO
			inner join TABELA_DE_VENDEDORES v 
				on v.MATRICULA = nf.MATRICULA
			where v.MATRICULA = '00236'
		```
		* Outro exemplo, listar todas as informações de clientes que moram em ruas "maiores" que "R. Benicio de Abreu";
		```sql
		select * from TABELA_DE_CLIENTES where ENDERECO_1 >= 'R. Benicio de Abreu'
		```
		* Isso não significa que esse endereço é maior que os outros ou nada, mas sim que a consulta SQL retornará os valores que, em ordem alfabética, vêm depois do "R. Benício de Abreu";
	* **and** e **or**: expressões usadas para montar condições lógicas mais complexas, podem ser utilizadas junto com o *where*, para, por exemplo, delimitar ainda mais o alcance de uma consulta; 
		* o *and* retornará positivo e, portanto, uma linha será impressa, se e somente se as duas ou mais condições forem todas verdadeiras, enquanto o *or* o fará também mas desde que pelo menos um dos resultados for verdadeiro;
		* Por exemplo, liste todos os clientes de São Paulo e com o CEP "3123212"`. Perceba que a consulta não retornará nada, pois uma das condições (CEP 3123212) não é verdadeira e não há valores assim na tabela:
			```sql	
			select * from TABELA_DE_CLIENTES where estado = 'Sp' and cep = '3123212'
			```
		* Outro exemplo, liste todos os produtos que tenham embalagem PET ou tenham tamanho diferente de 700ml:
			```sql
			select * from TABELA_DE_PRODUTOS where EMBALAGEM = 'pet' or tamanho <> '700 ml'
			```
	* **not**: cláusula que nega uma expressão lógica e *inverte* seu resultado. Por exemplo, para realizar uma consulta de produtos que não são PET ou tem 700ml de tamanho, podemos fazer:
		```sql
		select * from TABELA_DE_PRODUTOS where not EMBALAGEM = 'pet' or tamanho = '700 ml'
		```
	* Observação: quando tivermos muitas expressões *or* podemos escrever a consulta utilizando o '*in*' para simplificar:
		* Suponha que quero selecionar todos os sucos que não sejam de uva, manga ou laranja;
			```sql
			select * from TABELA_DE_PRODUTOS where sabor not in ('uva', 'manga', 'LARANJA')
			```
	* **between**: podemos utilizar tal cláusula para abrigar um intervalo entre dois números durante uma consulta;
		* Para a consulta anterior, suponha que desejo listar os sucos que não são de uva nem de manga e também que tenham o preço entre 6 e 10 reais:
			```sql
			select * from TABELA_DE_PRODUTOS where sabor not in ('uva', 'manga') and (PRECO_DE_LISTA between 6 and 10)
			```
	* **like**: cláusula utilizada para procurar, dentro de uma string, por um valor específico;
		* Por exemplo, vamos procurar por vendedores que têm a letra "*m*" em seu nome:
			```sql
			select * from TABELA_DE_VENDEDORES where nome like '%m%'
			```
		* Outro exemplo, procuremos por notas fiscais que têm CPF que apenas iniciam com "943":
			```sql
			select * from NOTAS_FISCAIS where cpf like '943%'
			```
		* Vale salientar que o símbolo de porcentagem entra no comando como um "coringa" de textos.
	* **distinct**: cláusula utilizada junto do *select* para informar que apenas registros com valores diferentes sejam retornados na consulta;
		* Muito útil para quando desejamos selecionar apenas uma coluna de uma tabela que contém muitos valores repetidos;
		* Por exemplo, vamos listar apenas as cidades onde os clientes moram mas sem repeti-las:
			```sql
			select distinct cidade from TABELA_DE_CLIENTES 
			```	
	* **top**: cláusula utilizada para selecionar somente um número *x* de linhas a ser exibida na saída. É ideal para ser utilizado em tabelas com muitos registros.
		* Por exemplo, vamos selecionar apenas as 100 primeiras notas fiscais dos produtos onde o cpf do cliente começa com "192":
			```sql
			select top 100 * from NOTAS_FISCAIS where cpf like '192%'
			```
## 2. ORDENAÇÃO
* Podemos utilizar também alguns comandos para ordenar a nossa saída e limitar os resultados;
* Exemplos de comandos de ordenação:
	* **order by**: usada para ordenar em ordem alfabética/numérica de forma crescente ou decrescente a saída da consulta, a depender do que o usuário deseja ('*asc*' para crescente, '*desc*' para decrescente);
		* Por exemplo, liste, de forma crescente, as 5000 primeiras notas fiscais que terminam com o cpf '787':
			```sql
			select top 5000 * from NOTAS_FISCAIS  where cpf like '%787' order by CPF asc
			```	
	* **group by**: cláusula utilizada quando há, no mínimo, uma função de agregação na seleção (SUM, AVG, MIN ou MAX). Ela necessita que você declare os campos da consulta que não estão dentro dessas funções de agregação, para que seja possível uni-los.
		* Por exemplo, suponha que vamos verificar a média das idades dos clientes de cada estado do nosso banco de dados. Assim, temos:
			```sql
			select estado, avg(idade) as media_idade from TABELA_DE_CLIENTES group by estado
			```
			* Nós teremos que o resultado dessa consulta será:
		
				| | estado | media_idade |
				| --- | --- | --- |
				| 1 | RJ | 21| 
				| 2 | SP | 27| 
			* Ou seja, o SQL Server agrupou os estados (que se repetiam) em um local só e executou a função de média (AVG).
		* Em outro exemplo, vamos verificar o número total de produtos com o código 1101035 que foram vendidos e quantas vezes foi vendido:
			```sql
			select CODIGO_DO_PRODUTO as codigo, sum(QUANTIDADE) as qtd_total, count (codigo_do_produto) as vendas_totais from ITENS_NOTAS_FISCAIS where CODIGO_DO_PRODUTO = '1101035' group by CODIGO_DO_PRODUTO 
			```
			* Temos como resultado:
			
			| | codigo | qtd_total | vendas_totais
			| --- | --- | --- | --- |
			| 1 | 1101035 | 388042 | 7103
	* **having**: localizado após o *order by*, serve como um meio de testar os resultados das funções de agregação do *select*. Ele, ao contrário de comandos como *where*, não coleta os dados dos campos em si, mas sim o resultado que é dado após a execução de uma função de agregação em cima desse campo.
		* Digamos que, por exemplo, desejo verificar em quais estados o limite total de crédito dos clientes é maior que 900000:
			```sql
			select estado, sum(limite_de_credito) as crédito from TABELA_DE_CLIENTES group by estado having sum(limite_de_credito) > 900000;
			```
			* Tal consulta vai me exibir somente o estado do RJ, com 995000 de crédito total, já que SP tem menos que isso.
	* **case**: sim, esta cláusula nos permite gerar uma estrutura de classificação no SQL com condições e testes lógicos. Sua sintaxe padrão é:
		```sql
		CASE WHEN <CONDIÇÃO> THEN <VALOR>
			WHEN <CONDIÇÃO> THEN <VALOR>
			WHEN <CONDIÇÃO> THEN <VALOR>
			ELSE <VALOR> END
		```
		* Por exemplo, se quisermos pegar dados de vendedores e classificarmos eles em "vendedores antigos" e "recentes", podemos fazer:
			```sql
			select 
				matricula,
				nome,
				(case when year(data_admissao) < 2015 then 'vendedor antigo'
				else 'vendedor recente' end) as periodo_tempo
			from TABELA_DE_VENDEDORES
			```
		* E se quisermos classificar os sucos entre 1) caro, a partir dos 12 reais; 2) em conta, entre os 6 e 12 reais; ou 3) barato, abaixo de 6 reais? Teremos:
			```sql
			select NOME_DO_PRODUTO, TAMANHO, PRECO_DE_LISTA,
				(case when PRECO_DE_LISTA >= 12 then 'produto caro'
					  when PRECO_DE_LISTA >= 6 and PRECO_DE_LISTA < 12 then 'produto em conta'
					  else 'produto barato' end) as CLASSIFICACAO
			from TABELA_DE_PRODUTOS
			```
		* Podemos, ainda, contar a quantidade de produtos de cada classificação:
			```sql
			select 
				(case when PRECO_DE_LISTA >= 12 then 'produto caro'
					when PRECO_DE_LISTA >= 6 and PRECO_DE_LISTA < 12 then 'produto em conta'
				else 'produto barato' end) as CLASSIFICACAO, count(*) as NUMERO_DE_PRODUTOS
			from TABELA_DE_PRODUTOS
			group by
				(case when PRECO_DE_LISTA >= 12 then 'produto caro'
					when PRECO_DE_LISTA >= 6 and PRECO_DE_LISTA < 12 then 'produto em conta'
				else 'produto barato' end)
			```
			* Para agrupar uma condição, você deve repeti-la no *group by*.
			* Temos que existem, então, 9 produtos baratos, 11 caros e 11 em conta.
## 3. JOINS
    Consultar um diagrama de Venn pode ser uma boa opção.
### 3.1 CONCEITOS
Sejam A e B dois conjuntos distintos. Teremos que:
### Inner Join
* *Inner joins* retornam os **elementos de A com intersecção em B**, ou seja, elementos dos dois conjuntos que são comuns;
* Em conjuntos, seria o equivalente à $A \cap B$.
```sql
select *
    from TableA a
    inner join TableB b
    on a.pk = b.pk
```
* *pk* = primary key

### Left Outer Join
* *Left outer joins* retornam, normalmente, os **elementos de A + os elementos de A com intersecção em B**;


* Existem dois tipos de resultados para a consulta:
1. Consulta inclusiva:
* Em conjuntos, pode ser denotado por $(A \backslash B) \cup (A \cap B)$
```sql
select *
    from TableA a
    left join TableB b
        on a.pk = b.pk
```
2. Consulta exclusiva
* Em conjuntos, pode ser denotado por $A \backslash B$
```sql
select *
    from TableA a
    left join TableB b
        on a.pk = b.pk
    where b.pk is null
```
### Right Outer Join

* *Right outer joins* retornam, normalmente, os **elementos de B + os elementos de B com intersecção em A**;


* Existem dois tipos de resultados para a consulta:
1. Consulta inclusiva:
* Em conjuntos, pode ser denotado por $(B \backslash A) \cup (B \cap A)$
```sql
select *
    from TableA a
    right join TableB b
        on a.pk = b.pk
```
2. Consulta exclusiva
* Em conjuntos, pode ser denotado por $B \backslash A$
```sql
select *
    from TableA a
    right join TableB b
        on a.pk = b.pk
    where a.pk is null
```
### Full Outer Join
* *Full outer joins* retornam **A união B**, ou seja, todos os elementos das duas tabelas, em comum ou não.

* Existem dois tipos de resultados para a consulta:
1. Consulta inclusiva:
* Em conjuntos, pode ser denotado por $A \cup B$
```sql
select *
    from TableA a
    full outer join TableB b
        on a.pk = b.pk
```
2. Consulta exclusiva
* Em conjuntos, pode ser denotado por $(A \cup B) \backslash (A \cap B)$
```sql
select *
    from TableA a
    full outer join TableB b
        on a.pk = b.pk
    where a.pk is null or b.pk is null
```

## 4. UNIÃO DE CONSULTAS
* Ao contrário da junção de consultas, que retorna duas tabelas uma ao lado da outra, a *união de consultas* resulta em uma tabela com as linhas da primeira tabela e, em seguida, as linhas da segunda tabela.
* Para que tal aconteça, necessita-se de duas condições básicas:
	* Que o número de **campos** (colunas) selecionados da tabela A seja igual ao número de **campos** selecionados da tabela B;
	* Que estes campos sejam do mesmo tipo.
* Existem os seguintes comandos de união:
	* **union**: une as duas ou mais consultas, exibindo os nomes das colunas a partir da primeira tabela a ser consultada e também *sem repetição de valores*;
		* Por exemplo, vamos pegar a lista total de bairros das tabelas de vendedor e cliente:
			```sql
			select BAIRRO from TABELA_DE_CLIENTES 
			union 
			select BAIRRO from TABELA_DE_VENDEDORES
			```
		* Temos, como resultado, uma lista de 13 bairros, mas sem indicação de onde eles vieram.
	* **union all**: fará a mesma coisa que o comando union, com exceção de que ele *repetirá* os valores.
		* Por exemplo, vamos pegar a lista total de bairros das tabelas de vendedor e cliente mas sem impedir que os registros se repitam:
			```sql
			select BAIRRO from TABELA_DE_CLIENTES 
			union all
			select BAIRRO from TABELA_DE_VENDEDORES
			```
		* Temos, como resultado, uma lista de 16 bairros e, novamente, sem indicação de onde vieram.
	* Mas e se quisermos descobrir de quais tabelas cada dado veio? Podemos criar uma nova coluna em cada select, chamada "cliente" ou "vendedor" para indicar de onde está vindo esse dado:
		```sql
		select BAIRRO, 'cliente' as origem from TABELA_DE_CLIENTES 
		union 
		select BAIRRO, 'vendedor' as origem from TABELA_DE_VENDEDORES
		```
		* Como resultado, teremos 16 registros mesmo utilizando o union, pois, além dele não conseguir mais realizar a distinção de valores repetidos, o nosso intuito é realmente saber de onde veio cada dado, mesmo que este se repita.
## 5. VIEWS
* As **visões** (ou *views*) nada mais são que sub-consultas SQL que são frequentemente utilizadas e, por isso, podem ser armazenadas na memória em forma de tabela virtual (não física, pois ela não existe de fato, é somente um resultado de um comando SQL que pode ser acessado frequentemente sem precisarmos inserir manualmente o mesmo comando todas as vezes).
	* Uma sub-consulta SQL pode ser vista como uma consulta que está dentro de outra consulta. Por exemplo, visualize o seguinte código SQL: 
		```sql
		select distinct quantidade_produtos.* from  (
			select tp.CODIGO_DO_PRODUTO, tp.NOME_DO_PRODUTO, sum(inf.QUANTIDADE) as quantidade
			from TABELA_DE_PRODUTOS tp
			inner join ITENS_NOTAS_FISCAIS inf
				on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
			group by tp.CODIGO_DO_PRODUTO, tp.NOME_DO_PRODUTO
		) quantidade_produtos 
		where quantidade_produtos.quantidade > 394000 
		order by quantidade desc
		```
	* Aqui, temos a seguinte situação: 
		* A sub-query (que está entre parênteses) seleciona alguns campos das tabelas de produtos e notas fiscais e monta um resultado com o código do produto, seu nome e a soma de sua quantidade vendida.
		* Após isso, a sub-query é apelidada como *quantidade_produtos*.
		* Então, uma outra consulta SQL (fora dos parênteses) utiliza o resultado gerado pela sub-query para listar apenas os produtos que venderam mais de 394000 unidades.
	* Sabendo que o resultado dessa consulta pode ser utilizado diversas vezes por outras consultas, nós podemos criar uma *view* para armazenar esse resultado e agilizar o processo de escrita das querys.
* Podemos criar uma view a partir do comando **create view**:
	```sql
	create view quantidade_produtos_vendidos as
		select tp.CODIGO_DO_PRODUTO, tp.NOME_DO_PRODUTO, sum(inf.QUANTIDADE) as quantidade
			from TABELA_DE_PRODUTOS tp
			inner join ITENS_NOTAS_FISCAIS inf
				on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
			group by tp.CODIGO_DO_PRODUTO, tp.NOME_DO_PRODUTO
	```
	e usar essa view como se fosse uma tabela no código normal:
	```sql
	select distinct quantidade_produtos_vendidos.* from quantidade_produtos_vendidos 
	where quantidade_produtos_vendidos.quantidade > 394000 
	order by quantidade desc
	```
	* Estes comandos terão o mesmo efeito do primeiro mostrado, só que bem mais legíveis.
## 6. FUNÇÕES
* Funções nada mais são que procedimentos que recebem uma entrada, realizam uma operação e devolvem uma saída modificada. Existem diversas funções no SQL dos mais diversos tipos, e não há um padrão determinado pela ANSI, assim, cada SGBD tem funções próprias, apesar de muitas serem compartilhadas.
* Para testar uma função simples, basta escrever `select [Função] [Campo] from [Tabela]`. Vejamos adiante alguns tipos.
### 6.1 Funções de texto
* **lower**: transforma todos os caracteres de uma string em minúsculos;
* **upper**: transforma todos os caracteres de uma string em maiúsculos;
* **concat**: concatena duas ou mais cadeias de caracteres em uma única. Por exemplo: `select concat('its a', ' long way ', 'down.')`;
* **right**: coleta os últimos caracteres de uma *string* (ou os caracteres à direita). Por exemplo, o código `select right('i used to have a best friend [but then he gave me a std], 27')` imprime somente "[but then he gave me and std]";
* **left**: coleta os primeiros caracteres de uma *string* (ou os caracteres à esquerda). Por exemplo, o código `select left('garden in the bones', 6)` imprime somente "garden";
* **replicate**: repete *x* vezes a *string* informada. Por exemplo, o código `select replicate('better ', 3)` resultará em "better better better";
* **substring**: coleta, por meio de um intervalo informado pelo usuário, uma parte da cadeia de caracteres e retorna esta parte resultante. Por exemplo, o código `select substring('in another life', 4, 15)` retornará "another life";
* **trim**: remove os espaços em branco presentes no início/fim da *string*;
* **replace**: substitue um conjunto de caracteres por outro. Por exemplo, o código `select replace('vinum sabbathi', ' ', '_')` resulta em "vinum_sabbathi";
* **len**: retorna o número de caracteres totais do texto;
* **charindex**: procura por uma substring dentro de uma string e retorna a posição do primeiro elemento da substring. Por exemplo, o código `select charindex('you', 'im always gonna look for your face')` retorna "26" (a posição na string onde o caractere 'y' é detectado)
* Exemplo prático: **faça uma consulta SQL e retorne somente o primeiro nome de cada cliente**.
	* Resposta:
		```sql
		select substring(
			nome, 
			0, 
			charindex('_', replace(nome, ' ', '_'), 0)
			) 
			from TABELA_DE_CLIENTES
		```
	
### 6.2 Funções de data
* **getdate()**: retorna a data corrente no formato norte-americano;
* **convert()**: converte um valor de um tipo para outro;
    * Os formatos para data (113, 103, 1, etc.) estão disponíveis na tabela *ANSI SQL*.
* **day(*v*)**: retorna o dia de um valor *v* em formato de data;
* **month(*v*)**: retorna o mês de um valor *v* em formato de data; 
* **year(*v*)**: retorna o ano de um valor *v* em formato de data; 
* **dateadd**: permite alterar e manipular um valor de data;
* **datediff**: permite calcular a diferença entre duas datas;  
* **datetimefromparts**: retornará uma data baseada em valores passados como parâmetros, que são: **datetimefromparts**(*ano*, *mes*, *dia*, *hora*, *minuto*, *segundos*, *milissegundos*);
* **datename**: retorna um trecho de uma data a partir de uma data que foi inserida. Por exemplo, o código `select datename(month, '17-02-2004')` retorna "Fevereiro".

* Exemplo prático: **imprima, junto do nome de cada cliente, sua data de nascimento por extenso, incluindo o dia, dia da semana, mês e ano**.
	* Resposta:
		```sql
		select nome, 
			concat(datename(day, data_de_nascimento), ', ', 
			datename(weekday, data_de_nascimento), ', ', 
			datename(month, data_de_nascimento), ' de ', 
			datename(year, data_de_nascimento) )
		from TABELA_DE_CLIENTES
		```
### 6.3 Funções numéricas
* **square(*n*)**: eleva um número ao quadrado;
* **power(*n*, *m*)**: eleva um número n à potência m;
* **abs(*n*)**: retorna o módulo (valor absoluto) de um número;
* **sqrt(*n*)**: tira a raiz quadrada de um número n;
* **pi()**: retorna o número *pi*;
* **getdate()**: retorna a data atual com horário;
* **sign(*n*)**: retorna -1 para um número real negativo e 1 para um positivo;
* **sum(*c*)**: soma os valores de uma coluna no SQL;
* **count(*c.c*)**: conta o número de registros de uma coluna;
* **avg(c)**: calcula a média de uma coluna;
* **max(c)**: retorna o valor máximo de uma coluna;
* **min(c)**: retorna o valor mínimo de uma coluna.

* Exemplo prático: **Na tabela de notas fiscais, temos o valor do imposto. Já na tabela de itens, temos a quantidade e o faturamento. Calcule o valor do imposto pago no ano de 2016, arredondando para o menor inteiro.**
	* Resposta:
		```sql
		select 
			year(nf.DATA_VENDA) as Ano, 
			floor(sum((inf.quantidade * inf.preco) * nf.imposto )) as imposto_pago 
		from notas_fiscais nf
		inner join ITENS_NOTAS_FISCAIS inf
			on inf.NUMERO = nf.NUMERO
		where year(nf.DATA_VENDA) = 2016
		group by year(nf.DATA_VENDA)
		```
## 7. DESAFIOS FINAIS
### a) Verificar se as compras de cada cliente a cada mês ultrapassam o valor máximo do campo "limite de compra"
Para cumprir este desafio, temos que realizar duas grandes consultas principais: 
* A primeira, uma sub-query, envolve as tabelas *ITENS_NOTAS_FISCAIS*, *NOTAS_FISCAIS* e *TABELA_DE_PRODUTOS*, para podermos recuperar a relação de compra de cada cliente a cada mês por meio do CPF contido em *NOTAS_FISCAIS* e da soma de suas compras em cada mês. O resultado serão 3 colunas: o cpf de cada cliente, o mês e ano que o cliente comprou e o volume total de sua compra. Em suma, a consulta será: 
	```sql
	select nf.CPF,
	convert(varchar(7), nf.DATA_VENDA, 120) as mes_ano, 
	sum(inf.quantidade) as volume_total from NOTAS_FISCAIS nf
	inner join ITENS_NOTAS_FISCAIS inf
		on nf.NUMERO = inf.NUMERO
	inner join TABELA_DE_PRODUTOS tp
		on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
	group by nf.CPF, convert(varchar(7), nf.DATA_VENDA, 120)
	```
* Já a segunda é necessária para relacionar os resultados desta sub-query (por meio de um *join*) com uma consulta em *TABELA_DOS_CLIENTES*, para verificar se o campo *volume_total* ultrapassa ou não o valor estabelecido no campo *volume_de_compra* de cada cliente. Teremos, ao final, esta consulta:

	```sql
	select tc.NOME, tc.VOLUME_DE_COMPRA, volume_venda.mes_ano, volume_venda.volume_total, 
	(case when volume_venda.volume_total > tc.VOLUME_DE_COMPRA then 'Vendas inválidas'
	else 'Vendas válidas' end) as diagnóstico
	from TABELA_DE_CLIENTES tc
	inner join (
		select nf.CPF,
		convert(varchar(7), nf.DATA_VENDA, 120) as mes_ano, 
		sum(inf.quantidade) as volume_total from NOTAS_FISCAIS nf
		inner join ITENS_NOTAS_FISCAIS inf
			on nf.NUMERO = inf.NUMERO
		inner join TABELA_DE_PRODUTOS tp
			on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
		group by nf.CPF, convert(varchar(7), nf.DATA_VENDA, 120)
	) volume_venda
	on volume_venda.CPF = tc.cpf
	```
Portanto, esta última consulta retorna o nome e volume de compra máximo de cada cliente, cada período em que ele comprou, o volume de sua compra e se sua compra ultrapassou o limite (sendo inválida) ou não (sendo válida).
### b) Com base na consulta anterior, selecione somente as vendas inválidas e mostre a diferença em %.
Para encontrar os registros que são inválidos, devemos utilizar a cláusula **having** (e, adicionalmente, a **group by**).
Após isso, criamos uma nova coluna que mostra a diferença em % e executamos a operação matemática responsável por isso. Assim, teremos o código final:
```sql
select tc.NOME, tc.VOLUME_DE_COMPRA, volume_venda.mes_ano, volume_venda.volume_total, 
(case when volume_venda.volume_total > tc.VOLUME_DE_COMPRA then 'Vendas inválidas'
else 'Vendas válidas' end) as diagnóstico,
-- calcular porcentagem e exibir o número com duas casas decimais
convert(decimal(10,2), (1 - (tc.VOLUME_DE_COMPRA/volume_venda.volume_total) ) * 100) as ultrapassou_qnts_porcent
from TABELA_DE_CLIENTES tc
inner join (
	select nf.CPF,
	convert(varchar(7), nf.DATA_VENDA, 120) as mes_ano, 
	sum(inf.quantidade) as volume_total from NOTAS_FISCAIS nf
	inner join ITENS_NOTAS_FISCAIS inf
		on nf.NUMERO = inf.NUMERO
	inner join TABELA_DE_PRODUTOS tp
		on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
	group by nf.CPF, convert(varchar(7), nf.DATA_VENDA, 120)
) volume_venda
on volume_venda.CPF = tc.cpf
group by volume_venda.volume_total, tc.VOLUME_DE_COMPRA, tc.NOME, volume_venda.mes_ano
-- selecionando os registros que são classificados como inválidos
having volume_venda.volume_total > tc.VOLUME_DE_COMPRA
```

### c) Elabore um relatório anual sobre as vendas dos sucos de frutas por sabor e o percentual de cada venda em relação à venda total
Para elaborar esse relatório, podemos, primeiramente, consultar as tabelas *TABELA_DE_PRODUTOS*, *NOTAS_FISCAIS* e *ITENS_NOTAS_FISCAIS* e atrelar seus dados por *joins*.	Nesta consulta, selecionamos o sabor do suco, o ano da venda e a soma de todas as quantidades que foram vendidas. Assim, temos:
```sql
select tp.SABOR,
year(nf.DATA_VENDA) as ANO,
sum(inf.QUANTIDADE) as LITROS_VENDIDOS
from TABELA_DE_PRODUTOS tp
	inner join ITENS_NOTAS_FISCAIS inf
		on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
	inner join NOTAS_FISCAIS nf
		on nf.NUMERO = inf.NUMERO
group by tp.SABOR, year(nf.DATA_VENDA)
```
Esta consulta nos retorna uma tabela com as vendas por sabor em cada ano. Entretanto, também precisamos saber quais foram as vendas totais de cada ano se queremos saber a porcentagem de cada venda sobre o total. Então, podemos consultar as tabelas *ITENS_NOTAS_FISCAIS* (recuperar a quantidade) e *NOTAS_FISCAIS* (recuperar o ano de venda) e somar as quantidades por ano, resultando no seguinte código:
```sql
select year (nf.DATA_VENDA) as ANO,
sum(inf.quantidade) as VOLUME_ANO
from ITENS_NOTAS_FISCAIS inf
	inner join NOTAS_FISCAIS nf
		on nf.NUMERO = inf.NUMERO
group by year (nf.DATA_VENDA)
```
Porém, se são consultas separadas, como podemos relacioná-las? Simples: crie uma consulta maior, insira a primeira consulta na forma de sub-query e relacione-a com a segunda a partir do *inner join*. Com o acesso ao volume total vendido por ano e por fruta, na consulta principal, selecionamos os campos *ano* e *volume_ano* da 2ª consulta (VENDA_ANO) e *sabor* e *litros_vendidos_por_ano* da 1ª consula (VENDA_SABOR) e podemos listar tais dados.

Por fim, para calcular a porcentagem das vendas de cada fruta em relação ao total, criemos, na seleção da consulta principal, uma expressão matemática que divida o campo *litros_vendidos_por_ano* pelo *volume_ano* e multiplicamos por 100 para resultar em um valor percentual. Atribuímos um nome para a coluna da porcentagem e executamos. Como resultado, vamos ter as colunas *ano*, *sabor*, *volume_ano*, *litros_vendidos_por_ano* e *percentual*, o que vai gerar o relatório requerido (calcular quanto cada suco vendeu por ano e seu percentual em relação à quantidade vendida).

Desse modo, temos o seguinte código:

```sql
select VENDA_ANO.ANO, 
VENDA_SABOR.SABOR,
VENDA_ANO.VOLUME_ANO,
VENDA_SABOR.LITROS_VENDIDOS_POR_ANO,
round((convert(FLOAT, VENDA_SABOR.LITROS_VENDIDOS_POR_ANO) / convert(FLOAT, VENDA_ANO.VOLUME_ANO)) * 100, 2) as percentual
from (
	-- primeira query: calcular o total de vendas por sabor
	select tp.SABOR,
	year(nf.DATA_VENDA) as ANO,
	sum(inf.QUANTIDADE) as LITROS_VENDIDOS_POR_ANO
	from TABELA_DE_PRODUTOS tp
		inner join ITENS_NOTAS_FISCAIS inf
			on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
		inner join NOTAS_FISCAIS nf
			on nf.NUMERO = inf.NUMERO
	group by tp.SABOR, year(nf.DATA_VENDA)
) VENDA_SABOR
inner join (
	-- segunda query, calcular o total de vendas por ano
	select year (nf.DATA_VENDA) as ANO,
	sum(inf.quantidade) as VOLUME_ANO
	from ITENS_NOTAS_FISCAIS inf
	inner join NOTAS_FISCAIS nf
		on nf.NUMERO = inf.NUMERO
	group by year (nf.DATA_VENDA)
) VENDA_ANO
on VENDA_ANO.ANO = VENDA_SABOR.ANO
order by LITROS_VENDIDOS_POR_ANO desc;
```

### d) Modifique o relatório anterior de modo a ver o ranking das vendas por tamanho
Para obtermos este relatório, basta somente modificarmos os campos *sabor* por *tamanho*, para classificarmos as vendas a partir do tamanho.
Isso resulta, desse modo, no código:
```sql
select VENDA_ANO.ANO, 
VENDA_SABOR.TAMANHO, 
VENDA_ANO.VOLUME_ANO,
VENDA_SABOR.LITROS_VENDIDOS_POR_ANO,
round((convert(FLOAT, VENDA_SABOR.LITROS_VENDIDOS_POR_ANO) / convert(FLOAT, VENDA_ANO.VOLUME_ANO)) * 100, 2) as percentual
from (
	-- primeira query: calcular o total de vendas por sabor
	select tp.TAMANHO,
	year(nf.DATA_VENDA) as ANO,
	sum(inf.QUANTIDADE) as LITROS_VENDIDOS_POR_ANO
	from TABELA_DE_PRODUTOS tp
		inner join ITENS_NOTAS_FISCAIS inf
			on inf.CODIGO_DO_PRODUTO = tp.CODIGO_DO_PRODUTO
		inner join NOTAS_FISCAIS nf
			on nf.NUMERO = inf.NUMERO
	group by tp.TAMANHO, year(nf.DATA_VENDA)
) VENDA_SABOR
inner join (
	-- segunda query, calcular o total de vendas por ano
	select year (nf.DATA_VENDA) as ANO,
	sum(inf.quantidade) as VOLUME_ANO
	from ITENS_NOTAS_FISCAIS inf
	inner join NOTAS_FISCAIS nf
		on nf.NUMERO = inf.NUMERO
	group by year (nf.DATA_VENDA)
) VENDA_ANO
on VENDA_ANO.ANO = VENDA_SABOR.ANO
order by LITROS_VENDIDOS_POR_ANO desc;
```

## 8.TRANSAÇÕES, COMMIT E ROLLBACK
* Uma transação no SQL Server é uma *unidade lógica de processamento* que tem por objetivo preservar a consistência e integridade dos dados, ou seja, permite ao usuário salvar o estado atual do banco de dados como uma versão que pode ser recuperada posteriormente.
* Quando queremos iniciar uma transação, devemos utilizar a expressão **begin transaction**. Tal comando salva, temporariamente, o estado atual do banco de dados, e nenhum outro usuário pode, durante esse momento, consultar a tabela que está sob esse efeito.
* Para confirmar as operações realizadas e salvar o estado atual do banco de dados de forma definitiva, utiliza-se o comando **commit**. Então, outros usuários vão poder acessar a tabela com as novas modificações.
* Para ignorar as operações que foram realizadas após o *begin transaction* e retornar ao ponto onde executamos esse comando, utilizamos o comando **rollback**, que desfaz essas operações e retorna ao "checkpoint" do database. Então, outros usuários vão poder acessar a tabela da forma que ela estava antes.

* Para fins didáticos, tais comandos funcionam de forma similar ao **git** e GitHub: 
    * em um repositório do GitHub, podemos ter um projeto em uma versão 1.0; 
    * fase **begin transaction**: ao acrescentarmos operações, códigos e arquivos neste projeto na nossa máquina, estamos criando um novo estado para esse projeto, **mas que ainda não foi registrado no repositório remoto** e, portanto, nenhum outro desenvolvedor ainda tem acesso ao seu novo código;
    * fase **commit**: se fizermos um **git commit**, significa que estaremos atualizando o estado do projeto, onde **todos os outros desenvolvedores poderão ter acesso ao seu código**;
    * fase **rollback**: entretanto, se executarmos um **git reset** ou um **git revert** neste projeto, retornaremos o projeto a um estado específico, descartando os commits feitos durante este processo.

## 9. TRIGGERS/GATILHOS
* Um *trigger* é um procedimento especial do SQL Server que é ativado automaticamente quando determinado evento ocorrer dentro do banco de dados. Por exemplo, ao inserir um registro em uma tabela, um trigger pode ser acionado e inserir, em outra tabela de *logs*, uma mensagem contendo detalhes dessa inserção.
* Para definir um trigger, usamos:
```sql
CREATE TRIGGER [Nome do gatilho]
ON [Tabela-alvo]
[FOR/AFTER/INSTEAD OF] [INSERT/UPDATE/DELETE]
AS
BEGIN
[Corpo do Gatilho]
END
```
* Componentes da trigger:
    * **for**: trigger será executada *simultaneamente* ao comando na tabela;
    * **after**: trigger será executada *logo após* ao comando na tabela;
    * **instead of**: trigger será executada *no lugar* do comando na tabela;
    * **begin - end**: espaço onde os comandos da trigger serão executados.
