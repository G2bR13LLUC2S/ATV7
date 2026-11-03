# 1. Diagnóstico do problema

A View criada pelo desenvolvedor júnior apresenta dados incorretos porque não possui filtros para selecionar somente os pedidos enviados (`Shipped`) no ano de 2004.

O código original não utiliza a cláusula `WHERE` para verificar o status do pedido nem o ano da data do pedido. Dessa forma, pedidos cancelados (`Cancelled`), retidos (`On Hold`) e pedidos de outros anos também podem aparecer no relatório.

Além disso, o `GROUP BY` agrupa somente pelo número do pedido. Para deixar a consulta mais clara e compatível com os campos selecionados, o nome do cliente também deve ser incluído no agrupamento.

# 2. Refatoração da View

```sql
1. CREATE OR REPLACE VIEW view_relatorio_financeiro AS
SELECT
    c.customerName,
    o.orderNumber,
    SUM(od.quantityOrdered * od.priceEach) AS valor_total
FROM
    customers c
JOIN
    orders o ON c.customerNumber = o.customerNumber
JOIN
    orderdetails od ON o.orderNumber = od.orderNumber
WHERE
    o.status = 'Shipped'
    AND YEAR(o.orderDate) = 2004
GROUP BY
    c.customerName,
    o.orderNumber;
```

## Teste da View

```sql
2. SELECT *
FROM view_relatorio_financeiro;
```

Para verificar se a View não retorna pedidos cancelados:

```sql
3. SELECT *
FROM view_relatorio_financeiro
WHERE orderNumber IN (
    SELECT orderNumber
    FROM orders
    WHERE status = 'Cancelled'
);
```

O resultado esperado é vazio, pois a View deve conter apenas pedidos com status `Shipped`.

# 3. Síntese teórica: Segurança e abstração

## Segurança de dados

Uma View pode limitar quais colunas e registros ficam disponíveis para determinado usuário. Por exemplo, um usuário pode consultar uma View que apresenta o nome do cliente e o valor do pedido sem receber acesso direto a todas as colunas das tabelas originais.

Para que essa restrição funcione, os privilégios devem ser configurados corretamente. No MySQL, a segurança de uma View também depende do contexto `SQL SECURITY`, que pode utilizar os privilégios do criador (`DEFINER`) ou do usuário que a consulta (`INVOKER`).

## Complexidade de abstração

Uma View permite salvar uma consulta SQL como um objeto nomeado no banco de dados. Assim, o usuário pode consultar o relatório com um comando simples, como `SELECT * FROM view_relatorio_financeiro`, sem precisar escrever novamente os `JOINs`, filtros e cálculos.

Isso facilita a reutilização da consulta, reduz a repetição de código e esconde parte da complexidade das tabelas base. A View funciona como uma tabela virtual: os dados consultados são obtidos a partir da consulta que a define.

# Referências

ORACLE. **MySQL 8.4 Reference Manual: CREATE VIEW Statement**. Disponível em: https://dev.mysql.com/doc/refman/8.4/en/create-view.html. Acesso em: 3 nov. 2026.

LEARN WITH RISHI. **Views in SQL Explained | Complete Theory + MySQL Practical | MySQL Complete Course**. Vídeo. 10 out. 2026. Disponível em: https://zolotube.com/watch?v=hapB-gml5lc. Acesso em: 3 nov. 2026.

MYSQLTUTORIAL. **MySQL Views**. Disponível em: https://www.mysqltutorial.org/mysql-views/. Acesso em: 3 nov. 2026.
