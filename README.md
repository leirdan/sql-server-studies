# CONSULTAS AVAN�ADAS EM SQL SERVER 2022 E SQL SERVER MANAGEMENT STUDIO 2019 
* Primeiro, cria-se o banco de dados **SUCOS_VENDAS** atrav�s dos c�digos SQL do arquivo "/src/Cria_Banco.sql";
* Ap�s isso, carrega-se o arquivo "src/Carga_Cadastros.sql" para adicionar os vendedores e os produtos;
* Em seguida, executa-se o arquivo "src/Carga_Notas.sql" para cadastrar todas as notas fiscais dos produtos. � um arquivo gigantesco, ent�o pode demorar para acontecer todas as inser��es.
* Por fim, executa-se tamb�m o arquivo "src/Carga_Itens_Notas.sql" para carregar os itens das notas fiscais. Novamente, um arquivo colossal, ent�o aguarde.

## 1. FILTRAGEM
* Podemos utilizar dos filtros nos comandos SQL para escrever consultas melhores.
* Exemplos de filtros:
	* **Where [express�o l�gica]**: aplica um filtro no SQL onde uma linha ser� exibida se, e somente se, ela atender � condi��o do where;
		* Por exemplo, vamos listar somente as informa��es de produtos vendidos pelo vendedor de matr�cula "00236":
		```sql
		SELECT v.MATRICULA, nf.NUMERO, inf.CODIGO_DO_PRODUTO, inf.PRECO 
			from NOTAS_FISCAIS nf
			inner join ITENS_NOTAS_FISCAIS inf 
				on inf.NUMERO = nf.NUMERO
			inner join TABELA_DE_VENDEDORES v 
				on v.MATRICULA = nf.MATRICULA
			where v.MATRICULA = '00236'
		```
		* Outro exemplo, listar todas as informa��es de clientes que moram em ruas "maiores" que "R. Benicio de Abreu";
		```sql
		select * from TABELA_DE_CLIENTES where ENDERECO_1 >= 'R. Benicio de Abreu'
		```
		* Isso n�o significa que esse endere�o � maior que os outros ou nada, mas sim que a consulta SQL retornar� os valores que, em ordem alfab�tica, v�m depois do "R. Ben�cio de Abreu";
	* **and** e **or**: express�es usadas para montar condi��es l�gicas mais complexas, podem ser utilizadas junto com o *where*, para, por exemplo, delimitar ainda mais o alcance de uma consulta; 
		* o *and* retornar� positivo e, portanto, uma linha ser� impressa, se e somente se as duas ou mais condi��es forem todas verdadeiras, enquanto o *or* o far� tamb�m mas desde que pelo menos um dos resultados for verdadeiro;
		* Por exemplo, liste todos os clientes de S�o Paulo e com o CEP "3123212"`. Perceba que a consulta n�o retornar� nada, pois uma das condi��es (CEP 3123212) n�o � verdadeira e n�o h� valores assim na tabela:
			```sql	
			select * from TABELA_DE_CLIENTES where estado = 'Sp' and cep = '3123212'
			```
		* Outro exemplo, liste todos os produtos que tenham embalagem PET ou tenham tamanho diferente de 700ml:
			```sql
			select * from TABELA_DE_PRODUTOS where EMBALAGEM = 'pet' or tamanho <> '700 ml'
			```
	* **not**: cl�usula que nega uma express�o l�gica e *inverte* seu resultado. Por exemplo, para realizar uma consulta de produtos que n�o s�o PET ou tem 700ml de tamanho, podemos fazer:
		```sql
		select * from TABELA_DE_PRODUTOS where not EMBALAGEM = 'pet' or tamanho = '700 ml'
		```
	* Observa��o: quando tivermos muitas express�es *or* podemos escrever a consulta utilizando o '*in*' para simplificar:
		* Suponha que quero selecionar todos os sucos que n�o sejam de uva, manga ou laranja;
			```sql
			select * from TABELA_DE_PRODUTOS where sabor not in ('uva', 'manga', 'LARANJA')
			```
	* **between**: podemos utilizar tal cl�usula para abrigar um intervalo entre dois n�meros durante uma consulta;
		* Para a consulta anterior, suponha que desejo listar os sucos que n�o s�o de uva nem de manga e tamb�m que tenham o pre�o entre 6 e 10 reais:
			```sql
			select * from TABELA_DE_PRODUTOS where sabor not in ('uva', 'manga') and (PRECO_DE_LISTA between 6 and 10)
			```
	* **like**: cl�usula utilizada para procurar, dentro de uma string, por um valor espec�fico;
		* Por exemplo, vamos procurar por vendedores que t�m a letra "*m*" em seu nome:
			```sql
			select * from TABELA_DE_VENDEDORES where nome like '%m%'
			```
		* Outro exemplo, procuremos por notas fiscais que t�m CPF que apenas iniciam com "943":
			```sql
			select * from NOTAS_FISCAIS where cpf like '943%'
			```
		* Vale salientar que o s�mbolo de porcentagem entra no comando como um "coringa" de textos.
	* **distinct**: cl�usula utilizada junto do *select* para informar que apenas registros com valores diferentes sejam retornados na consulta;
		* Muito �til para quando desejamos selecionar apenas uma coluna de uma tabela que cont�m muitos valores repetidos;
		* Por exemplo, vamos listar apenas as cidades onde os clientes moram mas sem repet�-las:
			```sql
			select distinct cidade from TABELA_DE_CLIENTES 
			```	
	* **top**: cl�usula utilizada para selecionar somente um n�mero *x* de linhas a ser exibida na sa�da. � ideal para ser utilizado em tabelas com muitos registros.
		* Por exemplo, vamos selecionar apenas as 100 primeiras notas fiscais dos produtos onde o cpf do cliente come�a com "192":
			```sql
			select top 100 * from NOTAS_FISCAIS where cpf like '192%'
			```
## 2. ORDENA��O
* Podemos utilizar tamb�m alguns comandos para ordenar a nossa sa�da e limitar os resultados;
* Exemplos de comandos de ordena��o:
	* **order by**: usada para ordenar em ordem alfab�tica/num�rica de forma crescente ou decrescente a sa�da da consulta, a depender do que o usu�rio deseja ('*asc*' para crescente, '*desc*' para decrescente);
		* Por exemplo, liste, de forma crescente, as 5000 primeiras notas fiscais que terminam com o cpf '787':
			```sql
			select top 5000 * from NOTAS_FISCAIS  where cpf like '%787' order by CPF asc
			```	
	* **group by**: cl�usula utilizada quando h�, no m�nimo, uma fun��o de agrega��o na sele��o (SUM, AVG, MIN ou MAX). Ela necessita que voc� declare os campos da consulta que n�o est�o dentro dessas fun��es de agrega��o, para que seja poss�vel uni-los.
		* Por exemplo, suponha que vamos verificar a m�dia das idades dos clientes de cada estado do nosso banco de dados. Assim, temos:
			```sql
			select estado, avg(idade) as media_idade from TABELA_DE_CLIENTES group by estado
			```
			* N�s teremos que o resultado dessa consulta ser�:
		
				| | estado | media_idade |
				| --- | --- | --- |
				| 1 | RJ | 21| 
				| 2 | SP | 27| 
			* Ou seja, o SQL Server agrupou os estados (que se repetiam) em um local s� e executou a fun��o de m�dia (AVG).
		* Em outro exemplo, vamos verificar o n�mero total de produtos com o c�digo 1101035 que foram vendidos e quantas vezes foi vendido:
			```sql
			select CODIGO_DO_PRODUTO as codigo, sum(QUANTIDADE) as qtd_total, count (codigo_do_produto) as vendas_totais from ITENS_NOTAS_FISCAIS where CODIGO_DO_PRODUTO = '1101035' group by CODIGO_DO_PRODUTO 
			```
			* Temos como resultado:
			
			| | codigo | qtd_total | vendas_totais
			| --- | --- | --- | --- |
			| 1 | 1101035 | 388042 | 7103