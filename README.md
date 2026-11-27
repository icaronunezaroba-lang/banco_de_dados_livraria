# banco_de_dados_livraria
# Sistema de Gerenciamento de Livraria (Banco de Dados Relacional)

Projeto de modelagem e implementação de um banco de dados relacional (`livraria_db`) desenvolvido em **MySQL** para controle de acervo, localização física, cadastro de pessoas/organizações e registro de vendas de uma livraria.

---

## Tecnologias e Ferramentas

* **SGBD:** MySQL
* **Modelagem Conceitual e Lógica:** BRModelo Web
* **Linguagem:** SQL (DDL, DML, DQL e rotinas armazenadas)

---

## Regras de Negócio e Arquitetura

O esquema foi projetado para garantir a integridade referencial e evitar redundância de dados através de 13 tabelas normalizadas:

* **Catálogo e Acervo:** Cada livro (`tb_livro`, identificado por `ISBN`) pertence a uma única categoria (`tb_categoria`), uma editora (`tb_editora`) e ocupa uma prateleira específica (`tb_prateleira`), que por sua vez está vinculada a uma seção física da loja (`tb_secao`).
* **Relacionamentos N:N:** Implementados via tabelas associativas com chaves primárias compostas:
  * `tb_livro_autor`: vincula obras a um ou mais autores.
  * `tb_livro_fornecedor`: mapeia os fornecedores de cada título.
  * `tb_item_compra`: registra os livros de cada venda, preservando a quantidade e o valor unitário praticado no momento da transação.
* **Atendimento e Vendas:** Cada cliente (`tb_cliente`) possui um funcionário de referência (`tb_funcionario`). As vendas (`tb_compra`) registram data, forma de pagamento, valor total, cliente comprador e funcionário responsável.

---

## Estrutura do Banco de Dados (`livraria_db`)

| Tabela | Descrição | Chaves e Restrições Principais |
| :--- | :--- | :--- |
| `tb_secao` | Setores físicos da livraria | `PK (id_secao)` |
| `tb_categoria` | Classificação editorial dos títulos | `PK (id_categoria)` |
| `tb_editora` | Cadastro de editoras | `PK (cnpj)` |
| `tb_autor` | Cadastro de autores | `PK (cpf)` |
| `tb_fornecedor` | Distribuidores e fornecedores | `PK (cnpj)` |
| `tb_funcionario` | Equipe de vendas e atendimento | `PK (id_funcionario)`, `UNIQUE (cpf)` |
| `tb_prateleira` | Localização interna nas seções | `PK (id_prateleira)`, `FK (id_secao)` |
| `tb_cliente` | Base de clientes | `PK (cpf)`, `FK (id_funcionario)` |
| `tb_livro` | Acervo, precificação e saldo em estoque | `PK (isbn)`, `FK (id_categoria, cnpj_editora, id_prateleira)` |
| `tb_compra` | Cabeçalho das transações de venda | `PK (id_compra)`, `FK (cpf_cliente, id_funcionario)` |
| `tb_livro_autor` | Tabela associativa Livro ↔ Autor | `PK (isbn, cpf_autor)` |
| `tb_livro_fornecedor` | Tabela associativa Livro ↔ Fornecedor | `PK (isbn, cnpj_fornecedor)` |
| `tb_item_compra` | Itens detalhados por compra | `PK (id_compra, isbn)` |

### Tipagem de Dados
* **Identificadores fixos:** `CHAR(11)` para CPF, `CHAR(14)` para CNPJ e `CHAR(13)` para ISBN.
* **Valores monetários:** `DECIMAL(10,2)` para evitar imprecisões de ponto flutuante.
* **Chaves internas e quantidades:** `INT` (com `AUTO_INCREMENT` em chaves substitutas).

---

## Recursos SQL Implementados

O script completo contempla desde a criação da estrutura e carga inicial de dados (*seed*) até consultas analíticas e automação:

1. **Consultas de Recuperação e Filtro (DQL):**
   * Filtros com operadores relacionais, `LIKE`, `BETWEEN`, `IN` e `DISTINCT`.
   * Agregações com `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, agrupamento via `GROUP BY` e filtragem de grupos com `HAVING`.
   * Subconsultas (*subqueries*) para análise comparativa de preços em relação à média do catálogo.
2. **Automação e Rotinas Armazenadas:**
   * **Trigger (`trg_baixa_estoque`):** Acionada `AFTER INSERT` na tabela `tb_item_compra` para decrementar automaticamente o saldo na `tb_livro`.
   * **Stored Procedure (`sp_compras_cliente`):** Recebe o `CPF` de um cliente como parâmetro (`IN p_cpf CHAR(11)`) e retorna o histórico consolidado de compras via `JOIN`.

---

## Como Executar

1. Certifique-se de ter um servidor **MySQL** (8.0+ recomendado) em execução.
2. Execute o script `.sql` disponível no repositório através do MySQL Workbench, DBeaver ou via CLI:
   ```bash
   mysql -u seu_usuario -p < script_livraria.sql
