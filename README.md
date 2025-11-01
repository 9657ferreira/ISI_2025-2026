# Integração de Sistemas de Informação

Pipeline **ETL** para integração e análise de dados de **moradores**, **pagamentos** e **meteorologia** no contexto de gestão de condomínios, usando **Pentaho**, **base de dados relacional** e **Grafana** para visualização.

> Projeto desenvolvido para a UC **Integração de Sistemas de Informação (ISI)**. Inclui limpeza/normalização com *regex*, *jobs* integração com API externa (Open-Meteo), exportação **XML**, logging e dashboards.

---

## 📦 Stack Técnica

- **Pentaho Data Integration (PDI CE)** 10.2.0.0-222 – Transformações (`*.ktr`) e Jobs (`*.kjb`)
- **Base de Dados**: PostgreSQL 
- **Open-Meteo API** – dados meteorológicos diários (JSON)
- **Grafana (Community)** – dashboards e alertas
- **Ficheiros de entrada**: `moradores.csv` e `pagamentos.csv`

---

## 🧱 Arquitetura (alto nível)

```
CSV → Pentaho PDI (Transformações) → BD SQL → Exportação XML
                               ↘ Open-Meteo API ↗
                                 Grafana (dashboards)
```

---

## 🗂️ Estrutura do Repositório

```
/data
  /staging
    moradores.csv
    pagamentos.xlsx
  /out
    /xml
      meteo_export.xml
    /logs
/kettle
  /transforms
    stg_moradores.ktr
    stg_pagamentos.ktr
    stg_meteo_export.ktr
  /jobs
    job_Condominio.kjb
/docs
  Relatorio_ISI.pdf
```

---

## 📊 Modelo de Dados (resumo)

- **dim_morador**: dados descritivos (nome, NIF, contacto, condomínio)
- **fact_pagamento**: eventos de pagamento (data, valor, estado, FK morador)
- **dim_meteo_dia**: meteorologia diária (data, localização, condição, temperatura, precipitação)

Relações principais:
- `fact_pagamento.id_morador → dim_morador.id_morador`
- `fact_pagamento.data_pagamento ↔ dim_meteo_dia.data`

---

## ✅ Pré-requisitos

1. **Java** (JRE/JDK 8+) – confirmar com `java -version`
2. **Pentaho PDI CE** 10.2.0.0-222
3. **Base de Dados** criada (schema + tabelas) e *Database Connection* configurada no Spoon
4. **Grafana** com *Data Source* para a BD
5. **Acesso à Internet** (para chamadas à Open-Meteo)

---

## 🚀 Como Executar

### 1) Via Spoon (GUI)

1. Abrir **Spoon** e configurar a conexão à BD.
2. Executar, por ordem:
   - `kettle/transforms/stg_moradores.ktr`
   - `kettle/transforms/stg_pagamentos.ktr`
   - `kettle/transforms/stg_meteo_export.ktr` *(opcional: gera `meteo_export.xml`)*
3. Verificar **logs** em `data/out/logs/`.

### 2) Via Kitchen (CLI)

```bash
Kitchen.bat /file:"C:/.../kettle/jobs/job_Condominio.kjb" /level:Basic
# ou em Linux/Mac:
./kitchen.sh -file="/path/to/kettle/jobs/job_Condominio.kjb" -level=Basic
```

O *job* orquestra todo o pipeline, com *Pass log to parent* ativo e ramos de erro.

---

## 🌤️ Integração com a API Open-Meteo

- **REST Client** chama o endpoint com `latitude`, `longitude`, `start_date`, `end_date`
- **JSON Input** mapeia `temperature_2m_max`, `precipitation_sum`, `weathercode`
- Tradução do **weathercode** para descrição textual (e.g., `0 → "Céu limpo"`)

---

## 🧾 Exportação XML

- Geração de `data/out/xml/meteo_export.xml` com codificação UTF-8
- Estrutura base:
  ```xml
  <meteorologia>
    <dia>
      <localizacao>...</localizacao>
      <dados>...</dados>
    </dia>
  </meteorologia>
  ```

---

## 📈 Dashboards no Grafana

Exemplos de métricas/queries:
- Total de moradores
- Pagamentos por mês
- Média de temperatura por data de pagamento

Sugestões:
- Variáveis de filtro (Condomínio, Mês, Ano)
- Alertas por limiares de negócio

---

## 🧪 Matriz de Evidências do Enunciado

- **Regex / Normalização** (Replace in String, String Operations)
- **Import/Export XML**
- **Jobs e controlo de processos**
- **Lookups/Junções**, **operações em valores e datas**
- **Logs**
- **API externa**
- **BD relacional**
- **Visualização (Grafana)**

---

## 🩺 Troubleshooting (rápido)

Problema | Causa provável | Solução
---|---|---
Datas trocadas | Formato `dd-MM-yyyy` | Regex `^(\d{2})-(\d{2})-(\d{4})$` → `$3-$2-$1`
Valores com vírgulas | Decimal “,” | Substituir vírgula por ponto antes de converter
Duplicados | Entradas repetidas | `Unique Rows` por `id_morador` / `id_pagamento`
Falha na API | Rede / rate limit | Repetição com backoff ou modo offline (cache)
Exportação XML | Permissões/codificação | Garantir escrita em `data/out/xml/` e UTF-8

---

## 🧭 Boas Práticas

- **Idempotência** e UPSERT onde aplicável  
- **Versionamento Git** de `*.ktr`, `*.kjb` e SQL  
- **Naming** consistente: `stg_*`, `dim_*`, `fact_*`  
- **Logs centralizados** com *timestamp*  
- **Variáveis de ambiente/parametrização** no *job* principal

---

## 🗺️ Roadmap

- Enriquecer meteorologia (vento, humidade)
- Agendamento automático (cron/Windows Task Scheduler)
- Alertas Grafana por regras de negócio
- Testes de regressão às transformações

---

## 📜 Licença

Define aqui a tua licença (ex.: MIT).

---

## 🙌 Agradecimentos

- **IPCA — Escola Superior de Tecnologia**  
- **Professor Luís Ferreira**  
- Documentação **Pentaho PDI**, **Open-Meteo** e **Grafana**



