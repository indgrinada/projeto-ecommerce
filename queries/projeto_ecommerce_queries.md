# Projeto BI - E-Commerce


## Objetivo

Entender o comportamento dos clientes de um E-Commerce a partir das seguintes análises:
- Frequência de compra / Fidelidade.
- Segmentação geográfica.
- Última compra e valor.
- Produtos mais comprados.


## Tabelas

- cliente: id, nome, email, cidade, estado, data_cadastro.
- produto: id, nome, categoria, valor, estoque.
- pedido: id, id_cliente, id_produto, data_pedido, valor_pedido.


## Criação das Tabelas

```
CREATE DATABASE projeto_bi_ecommerce; # mesmo que CREATE SCHEMA

USE projeto_bi_ecommerce;

CREATE TABLE cliente (
id INT PRIMARY KEY AUTO_INCREMENT,
nome VARCHAR(255) NOT NULL,
email VARCHAR(255) UNIQUE NOT NULL,
cidade VARCHAR(100),
estado VARCHAR(2) COMMENT 'apenas a sigla do estado',
data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE produto (
id INT PRIMARY KEY AUTO_INCREMENT,
nome VARCHAR(255) NOT NULL,
categoria VARCHAR(100) NOT NULL,
valor DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
estoque INT DEFAULT 0
);

CREATE TABLE pedido (
id INT PRIMARY KEY AUTO_INCREMENT,
id_cliente INT NOT NULL,
id_produto INT NOT NULL,
data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
valor_pedido DECIMAL(10, 2) NOT NULL,
    
CONSTRAINT fk_pedido_cliente FOREIGN KEY (id_cliente) REFERENCES cliente(id)
ON DELETE CASCADE,

CONSTRAINT fk_pedido_produto FOREIGN KEY (id_produto) REFERENCES produto(id)
);
```

## Análises

1. Visão geral de clientes, pedidos e valor total dos pedidos: 

```
SELECT 
cliente.nome,
COUNT(pedido.id) AS total_pedidos,
SUM(pedido.valor_pedido) AS valor_total_gasto
FROM cliente
LEFT JOIN pedido ON cliente.id = pedido.id_cliente
GROUP BY cliente.nome # nesse caso, todos os meus nomes eram distintos, por isso não fiz por cliente.id
ORDER BY MAX(pedido.data_pedido);
```

Resultado CSV em projeto_ecommerce_analise_1


2. Visão geral dos clientes ordenados pela data do último pedido:

```
SELECT 
cliente.nome,
COUNT(pedido.id) AS total_pedidos,
MAX(pedido.data_pedido) AS data_ultimo_pedido,
SUM(pedido.valor_pedido) AS valor_total_gasto
FROM cliente
LEFT JOIN pedido ON cliente.id = pedido.id_cliente
GROUP BY cliente.id # melhorei aqui, com o agrupamento por id do cliente em vez do nome
ORDER BY MAX(pedido.data_pedido);
```

Resultado CSV em projeto_ecommerce_analise_2


3. Visão geral dos pedidos segmentados por estado do cliente: 

```
SELECT 
cliente.estado,
COUNT(pedido.id) AS total_pedidos,
SUM(pedido.valor_pedido) AS faturamento_total,
ROUND(AVG(pedido.valor_pedido), 2) AS valor_compra_media
FROM cliente
LEFT JOIN pedido ON cliente.id = pedido.id_cliente
GROUP BY cliente.estado;
```

Resultado CSV em projeto_ecommerce_analise_3


4. Top 5 estados com maior valor de compra:

```
SELECT 
cliente.estado,
COUNT(pedido.id) AS total_pedidos,
SUM(pedido.valor_pedido) AS faturamento_total,
ROUND(AVG(pedido.valor_pedido), 2) AS valor_compra_media
FROM cliente
LEFT JOIN pedido ON cliente.id = pedido.id_cliente
GROUP BY cliente.estado
ORDER BY faturamento_total DESC
LIMIT 5;
```

Resultado CSV em projeto_ecommerce_analise_4
