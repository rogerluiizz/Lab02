# Projeto: Modelo Lógico - E-commerce (SQL)

## Objetivo
Implementar o modelo lógico de um esquema de e-commerce com:
- **Contas** (clientes PF e PJ) com regra de exclusividade PF/PJ;
- **Formas de pagamento** (múltiplas por conta);
- **Produtos**, **Fornecedores** e relação M:N entre eles;
- **Vendedores**, **Pedidos**, **Itens do pedido** e **Envio / rastreio**.

## Requisitos modelados
- Uma **account** pode ser **PF** (cpf preenchido) ou **PJ** (cnpj preenchido), nunca ambos.
- **Uma conta pode ter várias formas de pagamento** (tabela payment_methods).
- **Entrega** possui `status` e `tracking_code` (tabela shipments).
- Relacionamentos M:N: products ↔ suppliers via `product_suppliers`.
- Uso de **PK**, **FK** e **CHECK** para integrity constraints.

## Conteúdo do repositório
- `schema.sql` — script de criação do esquema e inserts de exemplo.
- `queries.md` — consultas complexas com explicações e perguntas respondidas.
- `README.md` — contexto do projeto (este arquivo).

## Como executar
1. Criar um banco PostgreSQL (ou ajustar tipos/funcs para outro SGBD).
2. Executar `psql -U <user> -d <db> -f schema.sql`.
3. Rodar as queries do arquivo `queries.md` para validação.

## Observações
- O script e as queries usam funções PostgreSQL (ex.: `STRING_AGG`) — adaptar caso necessário.
- Os inserts fornecem dados de teste básicos para validação das queries.
