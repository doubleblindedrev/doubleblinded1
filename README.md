# Arquitetura de Dados para Mineração de Processos no SEI

### Da extração com Web Scraping à análise de desempenho

Repositório de código e documentação da arquitetura de dados desenvolvida para extração, tratamento, reconstrução e análise de registros de tramitação do Sistema Eletrônico de Informações (SEI).

O projeto integra técnicas de Engenharia de Dados, Web Scraping, SQL e Mineração de Processos (*Process Mining*) para transformar históricos administrativos do SEI em estruturas analíticas que permitam examinar a duração dos processos, a permanência nas unidades organizacionais e os padrões de movimentação entre unidades.

A arquitetura foi desenvolvida e demonstrada no contexto da Universidade Federal de Sergipe (UFS), utilizando como aplicação empírica o processo de cessão de servidores para outros órgãos públicos.

O desenvolvimento foi orientado pelo paradigma da **Design Science Research (DSR)**, enquanto o modelo **Goal–Question–Metric (GQM)** foi utilizado para definir os objetivos e indicadores da análise.

---

## 1. Objetivo do projeto

O SEI registra informações sobre processos administrativos, incluindo datas, unidades organizacionais e descrições das movimentações. Entretanto, esses registros não estão diretamente organizados em um formato adequado à análise orientada a processos.

O objetivo deste projeto é apresentar uma arquitetura de dados capaz de transformar esses históricos em informações estruturadas e rastreáveis, permitindo:

- Extrair históricos de tramitação mediante acesso autenticado e autorizado ao SEI.
- Armazenar e organizar os registros originais.
- Reconstruir episódios de permanência dos processos nas unidades organizacionais.
- Identificar as relações direcionadas de origem e destino das movimentações.
- Classificar unidades organizacionais em macroetapas analíticas.
- Calcular indicadores de duração dos processos e das etapas.
- Examinar a frequência e a distribuição das movimentações entre unidades.
- Apoiar a identificação exploratória de etapas com durações registradas comparativamente elevadas.

A pesquisa concentra-se na perspectiva de **Enhancement da Mineração de Processos**, com ênfase na análise de desempenho temporal.

O objetivo não é desenvolver um novo algoritmo de mineração, descobrir automaticamente um modelo completo do processo ou realizar uma avaliação automatizada de conformidade normativa. A contribuição está na arquitetura de aquisição, transformação, organização e análise dos dados administrativos.

---

## 2. Arquitetura da solução

A solução é organizada em componentes integrados que transformam os registros de tramitação do SEI em estruturas adequadas à análise de desempenho.

O fluxo geral compreende:

1. Extração dos históricos de tramitação.
2. Persistência dos registros em formato estruturado.
3. Tratamento e transformação dos dados por meio de consultas SQL.
4. Reconstrução dos episódios de permanência nas unidades.
5. Reconstrução das movimentações direcionadas entre unidades.
6. Classificação das unidades em macroetapas analíticas.
7. Cálculo dos indicadores temporais e de movimentação.

A arquitetura mantém separadas duas representações complementares:

- **Episódios de permanência:** representam o intervalo de tempo durante o qual um processo permaneceu associado a determinada unidade organizacional.
- **Movimentações direcionadas:** representam as relações explícitas entre uma unidade de origem e uma unidade de destino.

Essa separação é importante porque uma movimentação pode possuir múltiplos destinatários, sem que isso represente múltiplos episódios independentes de permanência.

---

## 3. Estrutura do repositório

O repositório está organizado nos seguintes diretórios:

```text
doubleblinded1/
│
├── 01_extracao/
│   ├── sei_scraper.py
│   └── requirements.txt
│
├── 02_tratamento_sql/
│   ├── vw_cessao.sql
│   └── vw_cessao_fluxo.sql
│
├── 05_bpmn/
│
├── 06_parametrizacao/
│
├── .gitignore
└── README.md
```

### 3.1. `01_extracao/`

Contém o componente responsável pela extração automatizada dos históricos de tramitação do SEI.

O script `sei_scraper.py` utiliza:

- Python;
- Selenium;
- Pandas;
- SQLAlchemy;
- python-dotenv.

A rotina realiza autenticação na interface do SEI, consulta os processos no período informado, percorre os resultados e extrai seus históricos de tramitação.

Entre os campos coletados estão:

- Identificador do processo;
- Tipo de processo;
- Marcador;
- Data e horário do evento;
- Unidade organizacional;
- Descrição;
- Observação da movimentação.

Os registros extraídos são organizados em um DataFrame e exportados para arquivo CSV. Opcionalmente, podem ser persistidos em uma tabela de banco de dados relacional.

A extração depende de credenciais institucionais válidas e de autorização para consultar os processos selecionados.

### 3.2. `02_tratamento_sql/`

Contém as consultas SQL responsáveis pela reconstrução das estruturas analíticas utilizadas na pesquisa.

**`vw_cessao.sql`**

Reconstrói os episódios de permanência dos processos nas unidades organizacionais.

A estrutura resultante contempla:

- Processo;
- Unidade;
- StayID;
- Data_Inicio;
- Data_Fim;
- Status_Etapa;
- Duracao_Decimal_Dias.

O identificador `StayID` diferencia episódios sucessivos de permanência de um mesmo processo em uma mesma unidade.

A consulta considera os seguintes eventos como marcadores de início:

- Processo público gerado;
- Processo recebido na unidade;
- Reabertura do processo na unidade.

São considerados marcadores de encerramento do episódio:

- Conclusão do processo na unidade;
- Processo remetido pela unidade;
- Anexado ao processo.

Os episódios ainda sem evento de encerramento são identificados como em andamento.

**`vw_cessao_fluxo.sql`**

Reconstrói as relações direcionadas de movimentação dos processos.

A estrutura resultante contempla:

- Processo;
- Unidade_Origem;
- Unidade_Destino;
- Data_Transicao;
- Horas_de_Transicao.

A consulta identifica as movimentações explícitas registradas no histórico e recupera a unidade de origem das remessas a partir da descrição do andamento, quando disponível.

Também são representados os marcadores artificiais de início e fim necessários à estruturação das relações. Esses marcadores são diferenciados das remessas efetivas nas análises de frequência entre unidades.

### 3.3. `05_bpmn/`

Contém os materiais de modelagem BPMN utilizados para representar e contextualizar o procedimento institucional de cessão de servidores.

A representação do fluxo procedimental oferece suporte à compreensão das responsabilidades e da participação das unidades organizacionais.

É importante distinguir o modelo institucional do nível de informação efetivamente registrado pelo SEI: o histórico de tramitação permite observar a participação das unidades, mas não necessariamente identificar cada atividade ou tarefa executada internamente por seus servidores.

### 3.4. `06_parametrizacao/`

Contém os materiais de parametrização institucional utilizados para classificar e harmonizar as unidades organizacionais observadas no SEI.

O mapeamento permite relacionar os identificadores originais das unidades às macroetapas analíticas utilizadas na comparação temporal.

A classificação preserva os nomes originais das unidades para assegurar rastreabilidade e permitir a conferência dos registros de origem.

---

## 4. Modelo de dados para análise de processos

A arquitetura produz duas estruturas analíticas distintas e complementares.

### 4.1. Episódios de permanência

A estrutura `vw_cessao` permite observar os intervalos de permanência dos processos nas unidades organizacionais.

Cada episódio está associado a:

- Um processo;
- Uma unidade organizacional;
- Um identificador de permanência (`StayID`);
- Uma data de início;
- Uma data de término, quando registrada;
- Uma situação do episódio;
- Uma duração cronológica.

Essa representação permite distinguir retornos sucessivos de um mesmo processo à mesma unidade, evitando que todas as participações sejam tratadas como uma única ocorrência.

Os episódios constituem a principal base para o cálculo das durações associadas às macroetapas analíticas.

### 4.2. Movimentações direcionadas

A estrutura `vw_cessao_fluxo` registra as movimentações entre unidades por meio de relações explícitas de origem e destino.

Cada relação identifica o processo, a unidade de origem, a unidade de destino e o momento registrado da movimentação.

O modelo permite examinar:

- Frequência das movimentações entre pares de unidades;
- Quantidade de processos que percorreram determinada relação;
- Recorrência das relações de encaminhamento;
- Encaminhamentos simultâneos para diferentes unidades.

Quando uma mesma remessa apresenta múltiplos destinatários no mesmo instante, cada destinatário é preservado como uma relação direcionada.

Assim, evita-se impor uma sequência cronológica artificial entre unidades que receberam o mesmo processo simultaneamente.

---

## 5. Classificação das macroetapas

Para permitir comparações entre processos com diferentes trajetórias administrativas, as unidades organizacionais foram agrupadas em dez categorias analíticas.

As categorias utilizadas na pesquisa são:

1. DIMOP;
2. CMOP;
3. PROGEP;
4. GR;
5. CP;
6. DIAPC;
7. DICAF;
8. DIPES;
9. UNIDADE SUPERIOR;
10. UNIDADE DE ORIGEM.

A classificação harmoniza DCMOP com CMOP e agrupa unidades hierarquicamente superiores na categoria UNIDADE SUPERIOR.

As demais unidades são classificadas na categoria residual UNIDADE DE ORIGEM, conforme a regra de mapeamento adotada para o procedimento de cessão.

Essa categoria residual não representa necessariamente uma unidade administrativa formalmente homogênea, tampouco significa que todas as unidades agrupadas possuam as mesmas atribuições institucionais.

Os identificadores originais do SEI são preservados para permitir a rastreabilidade das classificações.

---

## 6. Indicadores de desempenho

A análise utiliza indicadores definidos a partir do modelo Goal–Question–Metric (GQM).

### 6.1. Lead time operacional

Representa a duração cronológica observada de cada processo, considerando os marcadores operacionais de início e término e, nos casos ainda abertos, o instante de referência da análise.

São utilizadas medidas como:

- Média;
- Mediana;
- Desvio-padrão;
- Quantidade de processos com episódios em aberto.

O indicador representa tempo decorrido em calendário, não tempo efetivo de trabalho.

### 6.2. Duração dos episódios por macroetapa

Representa o tempo decorrido entre o início e o encerramento de cada episódio de permanência em uma unidade.

Para episódios ainda em andamento, considera-se a idade observada até o instante de referência da análise.

A mediana da duração observada é adotada como principal critério para comparar as macroetapas e identificar aquelas que apresentam maior permanência típica registrada.

Média, desvio-padrão, quantidade de episódios e cobertura de processos são utilizados como medidas complementares.

### 6.3. Indicadores de movimentação

As movimentações direcionadas permitem calcular:

- Frequência de cada relação origem–destino;
- Quantidade de processos distintos associados a cada relação;
- Cobertura percentual de cada relação no conjunto analisado;
- Quantidade de processos com encaminhamentos simultâneos.

As movimentações são analisadas separadamente dos episódios de permanência, evitando a duplicação dos tempos quando há múltiplos destinatários.

---

## 7. Demonstração empírica

A arquitetura foi demonstrada utilizando o processo de cessão de servidores da Universidade Federal de Sergipe (UFS) para outros órgãos públicos.

O conjunto empírico utilizado no artigo compreende 56 processos, selecionados de acordo com o campo de data de início utilizado na extração, no período entre fevereiro de 2023 e dezembro de 2025.

As movimentações posteriores dos processos selecionados foram preservadas.

Para o cálculo das durações de episódios e processos ainda em aberto, foi adotado o instante de referência de:

**23 de outubro de 2026, às 10:54:46 (UTC−03:00).**

O conjunto de dados apresentou a seguinte composição:

| Elemento analisado | Quantidade |
|---|---:|
| Processos de cessão | 56 |
| Registros originais de movimentação | 3.905 |
| Episódios de permanência reconstruídos | 1.584 |
| Episódios encerrados | 1.564 |
| Episódios em aberto | 20 |
| Registros de movimentação direcionada | 2.039 |
| Identificadores originais de unidades organizacionais | 73 |
| Categorias analíticas de macroetapas | 10 |

Os 2.039 registros de movimentação direcionada incluem 1.600 remessas entre unidades e 439 relações artificiais de início e fim, utilizadas na estruturação do fluxo. As relações artificiais não são consideradas remessas efetivas no cálculo das frequências entre unidades.

Foram identificados encaminhamentos simultâneos em 42 dos 56 processos analisados.

---

## 8. Principais resultados

A demonstração empírica permitiu calcular indicadores temporais e examinar as movimentações registradas no procedimento de cessão.

Os principais resultados apresentados no artigo são:

- A mediana do lead time operacional dos processos foi de **211,21 dias**.
- A macroetapa DIPES apresentou a maior mediana de duração observada, de **15,79 dias**.
- A DIPES apresentou 68 episódios de permanência distribuídos em 42 processos, incluindo 11 episódios ainda em aberto no instante de referência.
- A categoria UNIDADE SUPERIOR apresentou mediana de 0,59 dia.
- A categoria residual UNIDADE DE ORIGEM apresentou mediana de 0,20 dia.

Considerando somente os episódios encerrados, a mediana de permanência na DIPES foi de 10,08 dias. A diferença decorre da inclusão, na análise principal, da idade observada dos episódios ainda em andamento.

A DIPES foi identificada como a principal prioridade temporal relativa entre as dez macroetapas comparadas, segundo o critério de maior mediana de duração observada.

Esse resultado não demonstra, isoladamente, ineficiência operacional, insuficiência de pessoal ou causa específica de atraso. Ele indica uma etapa que merece investigação administrativa complementar.

---

## 9. Interpretação e limites da análise

Os indicadores produzidos devem ser interpretados considerando as características dos registros do SEI.

A duração de um episódio representa tempo cronológico de permanência de um processo em uma unidade organizacional. Ela não permite distinguir diretamente:

- Tempo de trabalho efetivo;
- Tempo de espera;
- Tempo de interrupção;
- Tempo destinado a atividades internas não registradas no histórico.

Da mesma forma:

- Uma movimentação de retorno não significa necessariamente retrabalho.
- Uma relação de encaminhamento em sentido inverso não comprova, por si só, um desvio processual.
- Movimentações simultâneas não permitem afirmar que as tarefas internas foram executadas simultaneamente.
- A permanência em uma unidade não demonstra que todos os dias registrados corresponderam a atividades efetivas de processamento.
- Um registro de conclusão em determinada unidade não representa automaticamente a conclusão administrativa global do procedimento.

A classificação das unidades em macroetapas é uma abstração analítica específica do procedimento estudado e deve ser validada antes de sua aplicação em outros contextos institucionais.

Os resultados empíricos referem-se a um procedimento específico de uma instituição e possuem caráter exploratório. A transferência da arquitetura para outros órgãos requer adequação das regras de extração, dos mapeamentos institucionais e dos critérios de interpretação.

---

## 10. Reprodutibilidade e execução

### 10.1. Requisitos

Para executar o componente de extração, são necessários:

- Python 3.10 ou superior;
- Google Chrome;
- Acesso autorizado ao SEI;
- Credenciais institucionais válidas;
- Dependências especificadas em `01_extracao/requirements.txt`.

Para executar as consultas SQL disponibilizadas, é necessário um banco de dados PostgreSQL com os registros de origem organizados na tabela `historico`.

### 10.2. Instalação das dependências

A partir da raiz do repositório:

```bash
cd 01_extracao

python -m venv .venv
```

Ative o ambiente virtual.

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows:

```bash
.venv\Scripts\activate
```

Instale as dependências:

```bash
pip install -r requirements.txt
```

Caso seja utilizada a persistência direta em PostgreSQL, instale também o driver:

```bash
pip install "psycopg[binary]>=3.1"
```

### 10.3. Configuração da extração

O script utiliza variáveis de ambiente para configurar o acesso ao SEI e, opcionalmente, a conexão com o banco de dados.

As principais variáveis são:

| Variável | Finalidade |
|---|---|
| `SEI_BASE_URL` | Endereço base do SEI |
| `SEI_USERNAME` | Usuário institucional |
| `SEI_PASSWORD` | Senha institucional |
| `DATABASE_URL` | String de conexão com o banco de dados |
| `DATABASE_TABLE` | Tabela de destino; padrão: `historico` |
| `OUTPUT_DIR` | Diretório dos arquivos extraídos |
| `SELENIUM_TIMEOUT` | Tempo máximo de espera do Selenium |

Essas configurações podem ser fornecidas por meio de um arquivo `.env` local ou diretamente pelas variáveis de ambiente do sistema.

As credenciais devem permanecer fora do controle de versão.

### 10.4. Execução da extração

Execute o script informando o período desejado:

```bash
python sei_scraper.py \
  --inicio 01/02/2023 \
  --fim 31/12/2025
```

As datas devem ser informadas no formato DD/MM/AAAA.

Caso não sejam informadas, o script solicitará as datas interativamente.

A extração gera um arquivo CSV com os históricos coletados. Quando `DATABASE_URL` estiver configurada, os registros também serão persistidos na tabela indicada em `DATABASE_TABLE`.

A persistência no banco utiliza inserção incremental (`append`). Portanto, em execuções repetidas, devem ser considerados controles de duplicidade adequados à base utilizada.

### 10.5. Tratamento dos dados

Após disponibilizar os registros na tabela `historico` do PostgreSQL, execute as consultas SQL na seguinte ordem:

```sql
-- Reconstrução dos episódios de permanência
\i 02_tratamento_sql/vw_cessao.sql

-- Reconstrução das movimentações direcionadas
\i 02_tratamento_sql/vw_cessao_fluxo.sql
```

As consultas produzem as estruturas necessárias às análises temporais e de movimentação.

A estrutura `vw_cessao` é utilizada para análise dos episódios de permanência, enquanto `vw_cessao_fluxo` é utilizada para examinar as relações entre unidades.

O mapeamento institucional disponibilizado em `06_parametrizacao/` deve ser utilizado na classificação das unidades organizacionais e na produção das macroetapas analíticas.

---

## 11. Segurança, privacidade e disponibilidade dos dados

O repositório disponibiliza componentes metodológicos e computacionais para demonstração e adaptação da arquitetura.

Não são disponibilizados:

- Credenciais de acesso ao SEI;
- Senhas institucionais;
- Cookies ou tokens de autenticação;
- Históricos administrativos originais;
- Bases de dados institucionais contendo informações dos processos.

A coleta deve ser realizada exclusivamente por usuários autorizados e respeitando as permissões institucionais, as regras de acesso ao SEI e os requisitos aplicáveis de proteção de dados.

Os identificadores dos processos são mantidos internamente quando necessários à vinculação e à verificação dos registros, mas não integram os resultados públicos apresentados no artigo.

A reprodução integral da coleta depende de acesso autorizado ao sistema de origem. As consultas SQL, as regras de transformação e os procedimentos analíticos podem ser examinados independentemente da disponibilização dos históricos administrativos originais.

---

## 12. Limitações técnicas

O componente de extração utiliza automação da interface web do SEI por meio do Selenium.

Consequentemente, alterações na estrutura das páginas, nos seletores HTML, nos mecanismos de autenticação ou nas políticas de segurança do sistema podem exigir adequações no código.

A reprodução dos resultados também depende da correspondência entre as descrições dos eventos, os registros originais, as regras de reconstrução dos episódios e o mapeamento das unidades organizacionais.

A arquitetura deve ser adaptada e validada antes de sua utilização em outras instituições ou outros tipos de processos administrativos.

---

## 13. Contribuição científica

A principal contribuição deste projeto consiste em uma arquitetura de dados auditável para transformar históricos administrativos do SEI em informações estruturadas para análise orientada a processos.

A integração da extração autenticada, persistência dos registros, reconstrução SQL de episódios e movimentações direcionadas, classificação institucional e cálculo de indicadores temporais oferece uma alternativa para instituições que enfrentam restrições de acesso direto aos bancos de dados do sistema.

O trabalho demonstra que é possível desenvolver análises de desempenho temporal a partir dos registros operacionais disponíveis, preservando a rastreabilidade dos dados e reconhecendo os limites de interpretação associados à granularidade das informações.

A contribuição é metodológica e arquitetural: não representa a comprovação de uma intervenção de melhoria organizacional nem a identificação causal das razões dos atrasos observados.

---

## 14. Artigo relacionado

**A Data Architecture for Process Mining in Electronic Information System (SEI): From Web-Scraping Extraction to Performance Analysis**

O artigo apresenta os fundamentos metodológicos, a especificação da arquitetura, os procedimentos de transformação dos dados, os indicadores utilizados e os resultados da demonstração empírica realizada na Universidade Federal de Sergipe.

**Repositório do projeto:**

https://github.com/doubleblindedrev/doubleblinded1
