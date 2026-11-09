# Prática NoSQL - Banco de Dados Colunar (Apache Cassandra)

## 1. Ambiente
* **Banco Escolhido:** Apache Cassandra.
* **Justificativa de Escolha:** O Cassandra foi escolhido entre as opções colunares por ser a solução líder de mercado para lidar com grande volume de escritas em sequências temporais e alta disponibilidade sem ponto único de falha, atendendo perfeitamente ao cenário de sensores IoT.
* **Como Subir o Ambiente:**
  * **Opção 1 (Nuvem - Utilizada):** Banco configurado no DataStax Astra DB (serviço gratuito em nuvem do Cassandra). O acesso é feito via painel web acessando o *CQL Console*.
  * **Opção 2 (Docker Local):** Execute `docker-compose up -d` na raiz do repositório e acesse com `docker exec -it cassandra_iot cqlsh`.

## 2. Modelagem
A tabela `telemetria` foi modelada com uma chave primária composta:
* **Partition Key (`device_id`, `data`):** Garante que todas as leituras de um mesmo sensor em um dia específico fiquem gravadas juntas fisicamente no mesmo nó de armazenamento.
* **Clustering Key (`hora DESC`):** Organiza as leituras cronologicamente dentro do disco em ordem decrescente.

**Perguntas que a modelagem responde de forma otimizada:**
1. Qual o histórico completo de medições de um sensor em um dia específico?
2. Qual foi a última medição recebida do sensor?
3. Quantas medições foram registradas hoje por determinado dispositivo?
4. Qual o histórico de variação apenas de temperatura de um sensor?
5. Quais foram as leituras registradas dentro de um intervalo de tempo exato (ex: das 09h às 10h)?

## 3. Carga e Operações
O script completo contendo as operações de CRUD e a carga realista de 30 registros encontra-se no arquivo [`script.cql`](./script.cql).

## 4. Consultas do Estudo de Caso

### Consulta 1: Histórico completo do dispositivo no dia
`SELECT * FROM telemetria WHERE device_id = 'sensor_01' AND data = '2026-10-26';`
![Consulta 1](Consulta_1.png)

### Consulta 2: Leitura mais recente do sensor
`SELECT * FROM telemetria WHERE device_id = 'sensor_01' AND data = '2026-10-26' LIMIT 1;`
![Consulta 2](Consulta_2.png)

### Consulta 3: Total de leituras geradas no dia
`SELECT COUNT(*) FROM telemetria WHERE device_id = 'sensor_01' AND data = '2026-10-26';`
![Consulta 3](Consulta_3.png)

### Consulta 4: Projeção de dados específicos (Hora e Temperatura)
`SELECT hora, temperatura FROM telemetria WHERE device_id = 'sensor_02' AND data = '2026-10-26';`
![Consulta 4](Consulta_4.png)

### Consulta 5 (Ponto Forte): Busca por faixa de tempo
`SELECT * FROM telemetria WHERE device_id = 'sensor_01' AND data = '2026-10-26' AND hora >= '2026-10-26 09:00:00' AND hora <= '2026-10-26 10:00:00';`
![Consulta 5](Consulta_5.png)

## 5. Comparativo com o Modelo Relacional (SQL)

### Modelo SQL Equivalente (Tabela e Índice)
```sql
CREATE TABLE telemetria_relacional (
    id SERIAL PRIMARY KEY,
    device_id VARCHAR(50) NOT NULL,
    data DATE NOT NULL,
    hora TIMESTAMP NOT NULL,
    temperatura NUMERIC(5,2),
    umidade NUMERIC(5,2)
);

CREATE INDEX idx_telemetria_busca ON telemetria_relacional (device_id, data, hora DESC);