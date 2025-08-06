# Pipeline de Dados - Acidentes de Trânsito (2019-2021) 🚗💥

## 📋 Descrição do Projeto

Este projeto implementa um pipeline completo de integração e consolidação de dados de acidentes de trânsito de três anos consecutivos (2019, 2020 e 2021). O objetivo principal é criar uma base de dados única, coesa e padronizada que permita análises aprofundadas, consultas complexas e geração de insights relevantes sobre acidentes de trânsito.

## 🎯 Objetivos

- **Integração de Dados**: Consolidar três conjuntos de dados distintos de diferentes períodos temporais
- **Padronização**: Criar uma estrutura de dados uniforme e consistente
- **Análise Temporal**: Facilitar comparações e análises ao longo do tempo
- **Implementação de ETL/ELT**: Aplicar técnicas modernas de processamento de dados

## 🗂️ Estrutura do Projeto

```
pipeline-data-db/
├── ETL_Crash.ipynb          # Notebook principal com pipelines ETL e ELT
├── README.md                # Documentação do projeto
├── data 
├──── acidentes-2019.csv  
├──── acidentes_2020-novo.csv 
├──── acidentes2021.csv        
├── banco_etl.db             # Banco SQLite resultante do processo ETL
├── banco_elt.db             # Banco SQLite resultante do processo ELT
└── acidentes_unificados_2019-2021.csv  # CSV consolidado
```

## 🔧 Tecnologias Utilizadas

- **Python**: Linguagem principal do projeto
- **Pandas**: Manipulação e análise de dados
- **NumPy**: Operações numéricas
- **SQLAlchemy**: Interface para bancos de dados
- **SQLite**: Banco de dados relacional
- **Jupyter Notebook**: Ambiente de desenvolvimento interativo

## 📊 Bases de Dados

### Fonte dos Dados
- **Origem**: Dados de acidentes de trânsito de órgãos públicos
- **Período**: 2019, 2020 e 2021
- **Formato Original**: CSV com separador ';'
- **Codificação**: UTF-8

### Estrutura das Bases Originais

#### Colunas Principais:
- **Temporais**: `data`, `hora`, `DATA` (variações entre anos)
- **Localização**: `endereco`, `bairro`, `numero`, `complemento`
- **Veículos Envolvidos**: `auto`, `moto`, `ciclom`, `ciclista`, `pedestre`, `onibus`, `caminhao`, `viatura`, `outros`
- **Vítimas**: `vitimas`, `vitimasfatais`
- **Infraestrutura**: `num_semaforo`, `situacao_semaforo`, `sinalizacao`
- **Condições**: `natureza_acidente`, `tempo_clima`, `condicao_via`

## 🔄 Processos Implementados

### 1. Pipeline ETL (Extract, Transform, Load)

#### **Extração (Extract)**
```python
# Carregamento dos três conjuntos de dados
df_2019 = pd.read_csv('acidentes-2019.csv', delimiter=';')
df_2020 = pd.read_csv('acidentes_2020-novo.csv', delimiter=';')
df_2021 = pd.read_csv('acidentes2021.csv', delimiter=';')
```

#### **Transformação (Transform)**
1. **Padronização de Colunas**:
   - Renomeação de `DATA` para `data` (2019)
   - Remoção de colunas inconsistentes entre anos

2. **Tratamento Temporal**:
   - Correção de horários inválidos (`24:00:00` → `00:00:00`)
   - Criação de coluna `timestamp` unificada
   - Adição de coluna `ano`

3. **Padronização de Tipos**:
   - Conversão de colunas numéricas para inteiros
   - Tratamento de valores nulos/vazios
   - Normalização de endereços

#### **Carga (Load)**
- Unificação dos DataFrames
- Exportação para CSV consolidado
- Inserção em banco SQLite

### 2. Pipeline ELT (Extract, Load, Transform)

#### **Extração e Carga**
```python
# Carregamento direto para banco de dados
for arquivo in lista_arquivos_csv:
    df_temp = pd.read_csv(arquivo, sep=';', encoding='utf-8')
    df_temp['ano_do_dado'] = extrair_ano(arquivo)
    df_temp.to_sql('dados_brutos_sinistros', engine, if_exists='append')
```

#### **Transformação no Banco**
```sql
-- Criação de coluna timestamp
ALTER TABLE dados_brutos_sinistros ADD COLUMN TimeStamp TEXT;

-- Tratamento de dados temporais
UPDATE dados_brutos_sinistros 
SET TimeStamp = coalesce(data, DATA) || ' ' || 
    CASE WHEN hora = '24:00:00' THEN '00:00:00' ELSE hora END;

-- Padronização de campos numéricos
UPDATE dados_brutos_sinistros 
SET numero = CASE 
    WHEN numero IS NULL OR TRIM(numero) = '' THEN -1 
    ELSE numero 
END;
```

## 📈 Comparativo ETL vs ELT

| Aspecto | ETL | ELT |
|---------|-----|-----|
| **Processamento** | Transformação antes da carga | Transformação após a carga |
| **Performance** | Melhor para datasets pequenos | Melhor para grandes volumes |
| **Flexibilidade** | Menor flexibilidade posterior | Maior flexibilidade para ajustes |
| **Recursos** | Usa recursos da aplicação | Usa recursos do banco de dados |
| **Manutenção** | Código Python mais complexo | SQL mais direto e legível |

### Vantagens Observadas

**ETL:**
- ✅ Controle granular das transformações
- ✅ Validação prévia dos dados
- ✅ Melhor para lógicas complexas em Python

**ELT:**
- ✅ Performance superior para grandes volumes
- ✅ Aproveitamento da otimização do SGBD
- ✅ Maior flexibilidade para ajustes posteriores
- ✅ Rastreabilidade completa dos dados brutos

## 🚀 Como Executar o Projeto

### Pré-requisitos
```bash
pip install pandas numpy sqlalchemy openpyxl
```

### Execução

1. **Clone o repositório**:
```bash
git clone https://github.com/EsdrasAlbino/pipeline-data-db.git
cd pipeline-data-db
```

2. **Prepare os dados**:
   - Coloque os arquivos CSV na pasta raiz do projeto
   - Certifique-se de que os nomes dos arquivos correspondem aos esperados

3. **Execute o notebook**:
   - Abra `ETL_Crash.ipynb` no Jupyter Notebook ou Google Colab
   - Execute as células sequencialmente

### Estrutura de Execução

1. **Célula 1**: Instalação e importação das dependências
2. **Célula 2**: Pipeline ETL completo
3. **Célula 3**: Pipeline ELT completo
4. **Célula 4**: Geração de arquivo Excel consolidado

## 📊 Resultados Esperados

### Arquivos Gerados
- `banco_etl.db`: Banco SQLite com dados processados via ETL
- `banco_elt.db`: Banco SQLite com dados processados via ELT
- `acidentes_unificados_2019-2021.csv`: CSV consolidado
- `dados_brutos_sinistros_consolidados.xlsx`: Excel com análises

### Estrutura Final dos Dados
- **Registros**: ~45.000+ acidentes consolidados
- **Período**: 2019-2021
- **Colunas**: ~25 campos padronizados
- **Qualidade**: Dados limpos e consistentes

## 🔍 Análises Possíveis

1. **Análise Temporal**: Evolução dos acidentes ao longo dos anos
2. **Análise Geográfica**: Distribuição por bairros e endereços
3. **Análise de Gravidade**: Relação entre tipos de veículos e fatalidades
4. **Análise de Infraestrutura**: Impacto de semáforos e sinalização
5. **Análise Meteorológica**: Influência das condições climáticas

## 👥 Contribuições

### Como Contribuir
1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add: nova funcionalidade'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

### Padrões de Commit
- `Add:` para novas funcionalidades
- `Fix:` para correções de bugs
- `Update:` para atualizações e melhorias
- `Remove:` para remoções de código
- `Docs:` para atualizações de documentação

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo `LICENSE` para mais detalhes.


**Universidade**: [Nome da Universidade]  
**Disciplina**: Pipeline de Dados  
**Semestre**: [Semestre/Ano]

---

⭐ **Se este projeto foi útil para você, considere dar uma estrela!**
