# CONSULTAS AVANÇADAS EM SQL SERVER 2022 E SQL SERVER MANAGEMENT STUDIO 2019 
* Primeiro, cria-se o banco de dados **SUCOS_VENDAS** através dos códigos SQL do arquivo "/src/Cria_Banco.sql";
* Após isso, carrega-se o arquivo "src/Carga_Cadastros.sql" para adicionar os vendedores e os produtos;
* Em seguida, executa-se o arquivo "src/Carga_Notas.sql" para cadastrar todas as notas fiscais dos produtos. É um arquivo gigantesco, então pode demorar para acontecer todas as inserções.
* Por fim, executa-se também o arquivo "src/Carga_Itens_Notas.sql" para carregar os itens das notas fiscais. Novamente, um arquivo colossal, então aguarde.

## 1. FILTRAGEM
* Podemos utilizar dos filtros nos comandos SQL para escrever consultas melhores.
* Exemplos de filtros:
	* **Where [expressão lógica]**: aplica um filtro no SQL onde uma linha será exibida se, e somente se, ela atender à condição do where.
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
	* **and** e **or**: expressões usadas para montar condições lógicas mais complexas, podem ser utilizadas junto com o *where*, para, por exemplo, delimitar ainda mais o alcance de uma consulta; o *and* retornará positivo e, portanto, uma linha será impressa, se e somente se as duas ou mais condições forem todas verdadeiras, enquanto o *or* o fará também mas desde que pelo menos um dos resultados for verdadeiro.
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
	* **like**: cláusula utilizada para procurar, dentro de uma string, por um valor específico.
		* Por exemplo, vamos procurar por vendedores que têm a letra "*m*" em seu nome:
			```sql
			select * from TABELA_DE_VENDEDORES where nome like '%m%'
			```
		* Outro exemplo, procuremos por notas fiscais que têm CPF que apenas iniciam com "943":
			```sql
			select * from NOTAS_FISCAIS where cpf like '943%'
			```
		* Vale salientar que o símbolo de porcentagem entra no comando como um "coringa" de textos.