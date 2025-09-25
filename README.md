-- ====================================================
-- SCHEMA: E-COMMERCE (Modelo lógico)
-- Compatível com PostgreSQL (ajustável)
-- ====================================================

-- 1) Tabela de contas (clientes PJ ou PF)
CREATE TABLE accounts (
    account_id    SERIAL PRIMARY KEY,
    account_type  VARCHAR(2) NOT NULL, -- 'PF' or 'PJ'
    name          VARCHAR(200) NOT NULL,
    email         VARCHAR(200),
    cpf           VARCHAR(14),
    cnpj          VARCHAR(20),
    created_at    TIMESTAMP DEFAULT now(),
    CONSTRAINT chk_account_type CHECK (account_type IN ('PF', 'PJ')),
    CONSTRAINT chk_pf_pj_exclusive CHECK (
        (account_type = 'PF' AND cpf IS NOT NULL AND cnpj IS NULL)
     OR (account_type = 'PJ' AND cnpj IS NOT NULL AND cpf IS NULL)
    )
);

-- 2) Formas de pagamento (uma conta pode ter várias)
CREATE TABLE payment_methods (
    payment_id    SERIAL PRIMARY KEY,
    account_id    INTEGER NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    method_type   VARCHAR(50) NOT NULL, -- e.g. 'CARD', 'BOLETO', 'PIX'
    details       VARCHAR(500),
    is_default    BOOLEAN DEFAULT FALSE
);

-- 3) Produtos
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name       VARCHAR(200) NOT NULL,
    sku        VARCHAR(50) UNIQUE NOT NULL,
    price      NUMERIC(12,2) NOT NULL CHECK (price >= 0),
    stock      INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0)
);

-- 4) Fornecedores
CREATE TABLE suppliers (
    supplier_id SERIAL PRIMARY KEY,
    name        VARCHAR(200) NOT NULL,
    contact     VARCHAR(200),
    account_id  INTEGER REFERENCES accounts(account_id) -- optional link
);

-- 5) Associação produto - fornecedor (muitos-para-muitos)
CREATE TABLE product_suppliers (
    product_id  INTEGER REFERENCES products(product_id) ON DELETE CASCADE,
    supplier_id INTEGER REFERENCES suppliers(supplier_id) ON DELETE CASCADE,
    supplier_sku VARCHAR(100),
    cost         NUMERIC(12,2) CHECK (cost >= 0),
    PRIMARY KEY(product_id, supplier_id)
);

-- 6) Vendedores (pessoas que realizam vendas). Podem estar ligados a uma account opcional.
CREATE TABLE sellers (
    seller_id SERIAL PRIMARY KEY,
    name      VARCHAR(200) NOT NULL,
    account_id INTEGER REFERENCES accounts(account_id)
);

-- 7) Pedidos
CREATE TABLE orders (
    order_id   SERIAL PRIMARY KEY,
    account_id INTEGER NOT NULL REFERENCES accounts(account_id),
    seller_id  INTEGER REFERENCES sellers(seller_id),
    order_date TIMESTAMP NOT NULL DEFAULT now(),
    status     VARCHAR(50) NOT NULL DEFAULT 'PENDING' -- e.g. PENDING, PROCESSING, SHIPPED, CANCELLED
);

-- 8) Itens do pedido
CREATE TABLE order_items (
    order_id   INTEGER REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(product_id),
    qty        INTEGER NOT NULL CHECK (qty > 0),
    unit_price NUMERIC(12,2) NOT NULL CHECK (unit_price >= 0),
    PRIMARY KEY(order_id, product_id)
);

-- 9) Envio / rastreio (one-to-one com order)
CREATE TABLE shipments (
    shipment_id  SERIAL PRIMARY KEY,
    order_id      INTEGER UNIQUE REFERENCES orders(order_id) ON DELETE CASCADE,
    status        VARCHAR(50) NOT NULL DEFAULT 'CREATED', -- CREATED, IN_TRANSIT, DELIVERED
    tracking_code VARCHAR(200),
    shipped_date  TIMESTAMP
);

-- ====================================================
-- Dados de exemplo (inserts)
-- ====================================================

-- Accounts (clientes PF e PJ)
INSERT INTO accounts (account_type, name, email, cpf) VALUES
('PF','João Silva','joao@example.com','123.456.789-00'),
('PF','Maria Souza','maria@example.com','987.654.321-00'),
('PJ','LojaTop LTDA','contato@lojatop.com','00.000.000/0001-00');

-- Payment methods
INSERT INTO payment_methods (account_id, method_type, details, is_default) VALUES
(1,'CARD','Visa **** 1234', TRUE),
(1,'PIX','chave: joao@pix', FALSE),
(3,'BOLETO','Bank: 001, Agencia: 123', TRUE);

-- Products
INSERT INTO products (name, sku, price, stock) VALUES
('Camisa Polo', 'SKU-001', 79.90, 120),
('Calça Jeans',  'SKU-002', 149.90, 60),
('Tênis Corrida','SKU-003', 249.90, 30),
('Boné Trucker','SKU-004', 49.90, 200);

-- Suppliers
INSERT INTO suppliers (name, contact, account_id) VALUES
('Fornecedor A','forneA@email.com', NULL),
('Fornecedor B','forneB@email.com', NULL),
('Fornecedor LojaTop','supply@lojatop.com', 3); -- same account 3 is a PJ

-- Product-Suppliers
INSERT INTO product_suppliers (product_id, supplier_id, supplier_sku, cost) VALUES
(1,1,'A-001', 40.00),
(2,1,'A-002', 80.00),
(3,2,'B-003', 120.00),
(1,3,'LT-SKU001', 45.00);

-- Sellers
INSERT INTO sellers (name, account_id) VALUES
('Carlos Vendedor', NULL),
('LojaTop Vendedor','3'::text::int); -- seller linked to the PJ account (demonstration)

-- Orders + order_items + shipments
INSERT INTO orders (account_id, seller_id, order_date, status) VALUES
(1, 1, '2025-07-01 10:00:00', 'SHIPPED'),
(2, NULL, '2025-07-02 14:30:00', 'PROCESSING'),
(1, 2, '2025-07-03 09:20:00', 'PENDING');

INSERT INTO order_items (order_id, product_id, qty, unit_price) VALUES
(1, 1, 2, 79.90),
(1, 3, 1, 249.90),
(2, 2, 1, 149.90),
(3, 4, 3, 49.90);

INSERT INTO shipments (order_id, status, tracking_code, shipped_date) VALUES
(1, 'DELIVERED', 'TRACK123456', '2025-07-02'),
(2, 'IN_TRANSIT', 'TRACK987654', NULL);


