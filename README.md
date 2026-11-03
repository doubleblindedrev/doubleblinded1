# Data Architecture for Process Mining in SEI

### From Web Scraping Extraction to Performance Analysis

Repository containing the source code and documentation of the data architecture developed for extracting, processing, reconstructing, and analyzing transaction records from Brazil's Electronic Information System (*Sistema Eletrônico de Informações* – SEI).

The project integrates Data Engineering, Web Scraping, SQL, and Process Mining techniques to transform SEI administrative histories into analytical structures that support the examination of process duration, residence time within organizational units, and routing patterns between units.

The architecture was developed and demonstrated in the context of the Federal University of Sergipe (UFS), using the assignment of civil servants to other public agencies as its empirical application.

The development was guided by the **Design Science Research (DSR)** paradigm, while the **Goal–Question–Metric (GQM)** model was used to define the research objectives and analytical indicators.

---

## 1. Project Objective

SEI records information about administrative cases, including timestamps, organizational units, and descriptions of administrative movements. However, these records are not directly structured in a format suitable for process-oriented analysis.

The objective of this project is to provide a data architecture capable of transforming these histories into structured and traceable information, enabling:

- Extraction of administrative histories through authenticated and authorized access to SEI.
- Storage and organization of original records.
- Reconstruction of process residence episodes within organizational units.
- Identification of directed origin–destination relationships between units.
- Classification of organizational units into analytical macro-stages.
- Calculation of process and stage duration indicators.
- Examination of the frequency and distribution of movements between units.
- Exploratory identification of stages with comparatively longer recorded durations.

The research focuses on the **Enhancement perspective of Process Mining**, emphasizing performance analysis.

The objective is not to develop a new mining algorithm, automatically discover a complete process model, or perform automated regulatory compliance assessment. The contribution lies in the architecture for acquiring, transforming, organizing, and analyzing administrative data.

---

## 2. Solution Architecture

The solution is organized into integrated components that transform SEI transaction records into structures suitable for performance analysis.

The general workflow comprises:

1. Extraction of administrative histories.
2. Persistence of records in a structured format.
3. Data processing and transformation through SQL queries.
4. Reconstruction of residence episodes within organizational units.
5. Reconstruction of directed movements between units.
6. Classification of organizational units into analytical macro-stages.
7. Calculation of temporal and routing indicators.

The architecture maintains two separate but complementary representations:

- **Residence episodes:** represent the time interval during which a process remained associated with a given organizational unit.
- **Directed movements:** represent explicit relationships between an origin unit and a destination unit.

This separation is important because a single movement may have multiple recipients without representing multiple independent residence episodes.

---

## 3. Repository Structure

The repository is organized into the following directories:

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

Contains the component responsible for the automated extraction of SEI administrative histories.

The `sei_scraper.py` script uses:

- Python;
- Selenium;
- Pandas;
- SQLAlchemy;
- python-dotenv.

The routine authenticates through the SEI interface, searches for processes within the specified period, navigates through the results, and extracts their transaction histories.

The collected fields include:

- Process identifier;
- Process type;
- Marker;
- Event date and time;
- Organizational unit;
- Description;
- Movement observation.

The extracted records are organized into a DataFrame and exported to a CSV file. Optionally, they can also be persisted in a relational database table.

Data extraction requires valid institutional credentials and authorization to access the selected processes.

### 3.2. `02_tratamento_sql/`

Contains the SQL queries responsible for reconstructing the analytical structures used in the research.

**`vw_cessao.sql`**

Reconstructs process residence episodes within organizational units.

The resulting structure includes:

- Processo;
- Unidade;
- StayID;
- Data_Inicio;
- Data_Fim;
- Status_Etapa;
- Duracao_Decimal_Dias.

The `StayID` identifier distinguishes successive residence episodes of the same process within the same organizational unit.

The query considers the following events as starting markers:

- Public process generated;
- Process received by the unit;
- Process reopened in the unit.

The following events are considered episode-ending markers:

- Process concluded in the unit;
- Process forwarded by the unit;
- Attached to the process.

Episodes without a recorded ending event are identified as open.

**`vw_cessao_fluxo.sql`**

Reconstructs the directed movement relationships between organizational units.

The resulting structure includes:

- Processo;
- Unidade_Origem;
- Unidade_Destino;
- Data_Transicao;
- Horas_de_Transicao.

The query identifies explicitly recorded movements in the process history and recovers the originating unit from the movement description when available.

Artificial start and end markers are also represented to support the structuring of process flows. These markers are distinguished from actual inter-unit transfers when calculating movement frequencies.

### 3.3. `05_bpmn/`

Contains the BPMN modeling materials used to represent and contextualize the institutional procedure for assigning civil servants to other public agencies.

The procedural flow representation supports the understanding of organizational responsibilities and the participation of different administrative units.

It is important to distinguish the institutional process model from the level of information actually recorded by SEI: transaction histories reveal the participation of organizational units but do not necessarily identify every activity or task performed internally by their staff.

### 3.4. `06_parametrizacao/`

Contains the institutional parameterization materials used to classify and harmonize the organizational units identified in SEI.

The mapping relates the original unit identifiers to the analytical macro-stages used in temporal comparisons.

Original unit names are preserved to ensure traceability and allow verification against the source records.

---

## 4. Data Model for Process Analysis

The architecture produces two distinct and complementary analytical structures.

### 4.1. Residence Episodes

The `vw_cessao` structure represents the residence intervals of processes within organizational units.

Each episode is associated with:

- A process;
- An organizational unit;
- A residence identifier (`StayID`);
- A start date;
- An end date, when recorded;
- An episode status;
- A chronological duration.

This representation makes it possible to distinguish successive returns of the same process to the same unit, avoiding the treatment of all participation periods as a single occurrence.

These episodes constitute the primary basis for calculating durations associated with the analytical macro-stages.

### 4.2. Directed Movements

The `vw_cessao_fluxo` structure records movements between organizational units through explicit origin–destination relationships.

Each relationship identifies the process, origin unit, destination unit, and recorded movement timestamp.

This model supports the examination of:

- Frequency of movements between pairs of units;
- Number of processes associated with each relationship;
- Recurrence of routing relationships;
- Simultaneous dispatches to different units.

When a single dispatch has multiple recipients at the same timestamp, each recipient is preserved as a separate directed relationship.

This approach avoids imposing an artificial chronological sequence on units that received the same process simultaneously.

---

## 5. Macro-Stage Classification

To enable comparisons between processes with different administrative trajectories, organizational units were grouped into ten analytical categories.

The categories used in the research are:

1. DIMOP;
2. CMOP;
3. PROGEP;
4. GR;
5. CP;
6. DIAPC;
7. DICAF;
8. DIPES;
9. SUPERIOR UNIT;
10. ORIGIN UNIT.

The classification harmonizes DCMOP with CMOP and groups hierarchically superior units into the SUPERIOR UNIT category.

All remaining units are assigned to the residual ORIGIN UNIT category, according to the mapping rule adopted for the secondment procedure.

This residual category does not necessarily represent a formally homogeneous administrative unit, nor does it imply that all grouped units have identical institutional responsibilities.

Original SEI identifiers are preserved to ensure classification traceability.

---

## 6. Performance Indicators

The analysis uses indicators defined according to the Goal–Question–Metric (GQM) model.

### 6.1. Operational Lead Time

Represents the observed chronological duration of each process, considering the operational start and end markers and, for cases that remain open, the analysis reference timestamp.

The following measures are used:

- Mean;
- Median;
- Standard deviation;
- Number of processes with open episodes.

This indicator represents elapsed calendar time, not actual working time.

### 6.2. Episode Duration by Macro-Stage

Represents the elapsed time between the beginning and end of each residence episode within an organizational unit.

For episodes that remain open, the observed age is calculated up to the analysis reference timestamp.

The median observed duration is adopted as the main criterion for comparing macro-stages and identifying those with longer typical recorded residence times.

Mean, standard deviation, number of episodes, and process coverage are used as complementary measures.

### 6.3. Routing Indicators

Directed movements allow the calculation of:

- Frequency of each origin–destination relationship;
- Number of distinct processes associated with each relationship;
- Percentage coverage of each relationship within the analyzed dataset;
- Number of processes involving simultaneous dispatches.

Movements are analyzed separately from residence episodes to avoid duplicating durations when multiple destinations are involved.

---

## 7. Empirical Demonstration

The architecture was demonstrated using the assignment of civil servants from the Federal University of Sergipe (UFS) to other public agencies.

The empirical dataset used in the article comprises 56 cases, selected according to the start-date field used in the extraction, covering the period from February 2023 to December 2025.

Subsequent movements of the selected cases were also retained.

For calculating durations of open episodes and processes, the following reference timestamp was adopted:

**October 23, 2026, at 10:54:46 (UTC−03:00).**

The dataset presented the following composition:

| Analyzed element | Quantity |
|---|---:|
| Secondment cases | 56 |
| Original movement records | 3,905 |
| Reconstructed residence episodes | 1,584 |
| Completed episodes | 1,564 |
| Open episodes | 20 |
| Directed movement records | 2,039 |
| Original organizational unit identifiers | 73 |
| Analytical macro-stage categories | 10 |

The 2,039 directed movement records include 1,600 inter-unit dispatches and 439 artificial start and end relationships used to structure the flow. Artificial relationships are not considered actual dispatches when calculating inter-unit movement frequencies.

Simultaneous dispatches were identified in 42 of the 56 analyzed cases.

---

## 8. Main Results

The empirical demonstration made it possible to calculate temporal indicators and examine the recorded movements within the secondment procedure.

The main results reported in the article are:

- The median operational case lead time was **211.21 days**.
- The DIPES macro-stage presented the highest median observed duration, at **15.79 days**.
- DIPES presented 68 residence episodes across 42 cases, including 11 episodes that remained open at the reference timestamp.
- The SUPERIOR UNIT category presented a median duration of 0.59 days.
- The residual ORIGIN UNIT category presented a median duration of 0.20 days.

Considering only completed episodes, the median residence time at DIPES was 10.08 days. This difference results from the inclusion, in the primary analysis, of the observed ages of episodes that remained open.

DIPES was identified as the main relative temporal priority among the ten compared macro-stages, according to the criterion of the highest median observed duration.

This result does not, by itself, demonstrate operational inefficiency, insufficient staffing, or a specific cause of delay. Rather, it identifies a stage that deserves further administrative investigation.

---

## 9. Interpretation and Analytical Limitations

The indicators produced must be interpreted in light of the characteristics of SEI records.

The duration of an episode represents the chronological time during which a process remained associated with an organizational unit. It does not directly distinguish between:

- Actual working time;
- Waiting time;
- Interruption time;
- Time spent on internal activities not recorded in the system history.

Likewise:

- A return movement does not necessarily indicate rework.
- A movement in the opposite direction does not, by itself, demonstrate a process deviation.
- Simultaneous movements do not establish that internal tasks were performed simultaneously.
- Residence within a unit does not demonstrate that all recorded days corresponded to active processing.
- A conclusion record within a particular unit does not automatically represent the overall administrative conclusion of the procedure.

The classification of units into macro-stages is an analytical abstraction specific to the studied procedure and must be validated before being applied in other institutional contexts.

The empirical results refer to a specific procedure within a single institution and are exploratory in nature. Transferring the architecture to other public agencies requires adapting the extraction rules, institutional mappings, and interpretation criteria.

---

## 10. Reproducibility and Execution

### 10.1. Requirements

To execute the extraction component, the following are required:

- Python 3.10 or higher;
- Google Chrome;
- Authorized access to SEI;
- Valid institutional credentials;
- Dependencies specified in `01_extracao/requirements.txt`.

To execute the available SQL queries, a PostgreSQL database containing the source records in a table named `historico` is required.

### 10.2. Installing Dependencies

From the repository root:

```bash
cd 01_extracao

python -m venv .venv
```

Activate the virtual environment.

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows:

```bash
.venv\Scripts\activate
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

If direct persistence to PostgreSQL is required, also install the database driver:

```bash
pip install "psycopg[binary]>=3.1"
```

### 10.3. Extraction Configuration

The script uses environment variables to configure access to SEI and, optionally, the database connection.

The main variables are:

| Variable | Purpose |
|---|---|
| `SEI_BASE_URL` | SEI base URL |
| `SEI_USERNAME` | Institutional username |
| `SEI_PASSWORD` | Institutional password |
| `DATABASE_URL` | Database connection string |
| `DATABASE_TABLE` | Destination table; default: `historico` |
| `OUTPUT_DIR` | Directory for extracted files |
| `SELENIUM_TIMEOUT` | Maximum Selenium wait time |

These settings can be provided through a local `.env` file or directly through the system environment variables.

Credentials must remain outside version control.

### 10.4. Running the Extraction

Run the script by specifying the desired period:

```bash
python sei_scraper.py \
  --inicio 01/02/2023 \
  --fim 31/12/2025
```

Dates must be provided in DD/MM/YYYY format.

If dates are not provided, the script will request them interactively.

The extraction generates a CSV file containing the collected histories. When `DATABASE_URL` is configured, records are also persisted in the table specified by `DATABASE_TABLE`.

Database persistence uses incremental insertion (`append`). Therefore, appropriate duplicate-control mechanisms should be considered when executing the extraction repeatedly.

### 10.5. Data Processing

After making the records available in the PostgreSQL `historico` table, execute the SQL queries in the following order:

```sql
-- Reconstruction of residence episodes
\i 02_tratamento_sql/vw_cessao.sql

-- Reconstruction of directed movements
\i 02_tratamento_sql/vw_cessao_fluxo.sql
```

The queries produce the structures required for temporal and routing analyses.

The `vw_cessao` structure is used to analyze residence episodes, while `vw_cessao_fluxo` is used to examine relationships between organizational units.

The institutional mapping available in `06_parametrizacao/` should be used to classify organizational units and generate the analytical macro-stages.

---

## 11. Security, Privacy, and Data Availability

The repository provides methodological and computational components for demonstrating and adapting the architecture.

The following are not publicly distributed:

- SEI access credentials;
- Institutional passwords;
- Authentication cookies or tokens;
- Original administrative histories;
- Institutional databases containing process information.

Data collection must be performed exclusively by authorized users and must comply with institutional permissions, SEI access rules, and applicable data protection requirements.

Process identifiers are retained internally when necessary for record linkage and verification but are omitted from the public results presented in the article.

Full reproduction of the data acquisition process requires authorized access to the source system. SQL queries, transformation rules, and analytical procedures can be independently examined without access to the original administrative histories.
