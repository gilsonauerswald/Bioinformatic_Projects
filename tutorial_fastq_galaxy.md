# Tutorial: Download de Arquivos FASTQ de Bancos Públicos e Análise no Galaxy

## Sumário
1. [Introdução](#introdução)
2. [Principais Bancos de Dados](#principais-bancos-de-dados)
3. [Preparação do Ambiente](#preparação-do-ambiente)
4. [Download do SRA Toolkit](#download-do-sra-toolkit)
5. [Busca e Download de Dados do NCBI SRA](#busca-e-download-de-dados-do-ncbi-sra)
6. [Download de Dados do TCGA](#download-de-dados-do-tcga)
7. [Upload para o Galaxy](#upload-para-o-galaxy)
8. [Análises Básicas no Galaxy](#análises-básicas-no-galaxy)
9. [Troubleshooting](#troubleshooting)

## Introdução

Este tutorial apresenta métodos para obter dados de sequenciamento de tecidos normais e tumorais de repositórios públicos e utilizá-los para análises moleculares na plataforma Galaxy. Focaremos principalmente no NCBI SRA (Sequence Read Archive) e TCGA (The Cancer Genome Atlas).

## Principais Bancos de Dados

### 1. NCBI SRA (Sequence Read Archive)
- **URL**: https://www.ncbi.nlm.nih.gov/sra
- **Conteúdo**: Dados brutos de sequenciamento (RNA-seq, DNA-seq, ChIP-seq, etc.)
- **Formatos**: SRA, FASTQ
- **Acesso**: Gratuito

### 2. TCGA (The Cancer Genome Atlas)
- **URL**: https://portal.gdc.cancer.gov/
- **Conteúdo**: Dados genômicos de câncer (RNA-seq, DNA-seq, metilação, etc.)
- **Formatos**: BAM, FASTQ, VCF
- **Acesso**: Gratuito (dados controlados requerem aprovação)

### 3. ENA (European Nucleotide Archive)
- **URL**: https://www.ebi.ac.uk/ena
- **Conteúdo**: Dados de sequenciamento europeus
- **Formatos**: FASTQ, SRA
- **Acesso**: Gratuito

## Preparação do Ambiente

### Requisitos de Sistema
- Sistema operacional: Linux, macOS ou Windows (com WSL)
- Espaço em disco: Mínimo 50GB livres
- Conexão de internet estável
- Python 3.6+ (para ferramentas auxiliares)

### Instalação de Dependências

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install wget curl python3 python3-pip

# CentOS/RHEL
sudo yum install wget curl python3 python3-pip

# macOS (com Homebrew)
brew install wget curl python3
```

## Download do SRA Toolkit

O SRA Toolkit é essencial para baixar dados do NCBI SRA.

### Instalação no Linux

```bash
# Download da versão mais recente
wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz

# Extração
tar -xzf sratoolkit.current-ubuntu64.tar.gz

# Adicionar ao PATH
export PATH=$PATH:$PWD/sratoolkit.3.0.7-ubuntu64/bin

# Tornar permanente (adicionar ao ~/.bashrc)
echo 'export PATH=$PATH:$PWD/sratoolkit.3.0.7-ubuntu64/bin' >> ~/.bashrc
source ~/.bashrc
```

### Configuração Inicial

```bash
# Configurar o SRA Toolkit
vdb-config --interactive

# Teste da instalação
fastq-dump --help
```

## Busca e Download de Dados do NCBI SRA

### 1. Busca de Datasets

**Acesse**: https://www.ncbi.nlm.nih.gov/sra

**Termos de busca sugeridos para câncer:**
- `"breast cancer"[Title] AND "RNA-seq"[Strategy]`
- `"lung cancer"[Title] AND "tumor"[Title] AND "normal"[Title]`
- `"colorectal cancer"[Title] AND "transcriptome"[Strategy]`

### 2. Seleção de Amostras

1. Filtrar por:
   - **Organism**: Homo sapiens
   - **Strategy**: RNA-Seq
   - **Source**: TRANSCRIPTOMIC
   - **Layout**: PAIRED ou SINGLE

2. Identificar pares tumor/normal
3. Anotar os números SRR (ex: SRR1234567)

### 3. Download via SRA Toolkit

```bash
# Download de uma amostra específica
fastq-dump --split-files --gzip SRR1234567

# Download de múltiplas amostras
for srr in SRR1234567 SRR1234568 SRR1234569; do
    echo "Baixando $srr..."
    fastq-dump --split-files --gzip $srr
done

# Download com verificação de integridade
prefetch SRR1234567
fastq-dump --split-files --gzip ~/ncbi/public/sra/SRR1234567.sra
```

### 4. Verificação dos Arquivos

```bash
# Listar arquivos baixados
ls -lh *.fastq.gz

# Verificar qualidade básica
zcat SRR1234567_1.fastq.gz | head -20

# Contar reads
echo $(zcat SRR1234567_1.fastq.gz | wc -l)/4 | bc
```

## Download de Dados do TCGA

### 1. Acesso ao GDC Portal

**Acesse**: https://portal.gdc.cancer.gov/

### 2. Busca de Dados

1. **Repository** → **Cases**
2. **Filtros**:
   - Primary Site: selecionar órgão de interesse
   - Project: TCGA-BRCA (câncer de mama), TCGA-LUAD (adenocarcinoma pulmonar), etc.
   - Data Category: Transcriptome Profiling
   - Data Type: Gene Expression Quantification
   - Experimental Strategy: RNA-Seq

### 3. Download via GDC Client

```bash
# Instalação do GDC Client
wget https://gdc.cancer.gov/files/public/file/gdc-client_v1.6.1_Ubuntu_x64.zip
unzip gdc-client_v1.6.1_Ubuntu_x64.zip
sudo cp gdc-client /usr/local/bin/

# Download usando manifest
gdc-client download -m gdc_manifest.txt

# Download de arquivos específicos
gdc-client download 00a5e471-a79f-4d56-8a4c-4847ac037400
```

### 4. Conversão BAM para FASTQ (se necessário)

```bash
# Instalar samtools
sudo apt install samtools

# Converter BAM para FASTQ
samtools sort -n input.bam | samtools fastq -1 output_R1.fastq -2 output_R2.fastq -
```

## Upload para o Galaxy

### 1. Acesso ao Galaxy

**Opções de servidores públicos:**
- Galaxy Main (usegalaxy.org)
- Galaxy Europe (usegalaxy.eu)
- Galaxy Australia (usegalaxy.org.au)

### 2. Criação de Conta

1. Acesse o servidor Galaxy escolhido
2. Clique em "Login or Register"
3. Crie uma nova conta com email válido

### 3. Upload de Arquivos

#### Método 1: Upload Direto

1. **Get Data** → **Upload File**
2. **Choose local files** → selecionar arquivos FASTQ
3. **Type**: fastqsanger (para Illumina)
4. **Genome**: Human (GRCh38/hg38)
5. **Start Upload**

#### Método 2: Upload via FTP

```bash
# Configurar FTP (para arquivos grandes)
# Usar credenciais do Galaxy
ftp uploads.usegalaxy.org
# Username: seu_email_do_galaxy
# Password: sua_senha_do_galaxy

# Upload dos arquivos
put arquivo1.fastq.gz
put arquivo2.fastq.gz
```

#### Método 3: Upload via SRA

1. **Get Data** → **SRA Tools** → **Faster Download and Extract Reads in FASTQ**
2. **Accession**: inserir número SRR
3. **Select how to split the spots**: Single-end ou Paired-end
4. **Execute**

### 4. Organização em Collections

1. Selecionar múltiplos datasets
2. **For all selected** → **Build Dataset List**
3. Nomear a collection (ex: "Tumor_Samples", "Normal_Samples")

## Análises Básicas no Galaxy

### 1. Controle de Qualidade

#### FastQC
1. **NGS: QC and manipulation** → **FastQC**
2. **Short read data from your current history**: selecionar arquivos FASTQ
3. **Execute**

#### MultiQC (para múltiplas amostras)
1. **NGS: QC and manipulation** → **MultiQC**
2. **Results**: selecionar saídas do FastQC
3. **Execute**

### 2. Trimming de Qualidade

#### Trimmomatic
1. **NGS: QC and manipulation** → **Trimmomatic**
2. **Single-end or paired-end reads**: conforme seus dados
3. **Input FASTQ file**: selecionar arquivo(s)
4. **Trimmomatic operations**:
   - LEADING: 3
   - TRAILING: 3
   - SLIDINGWINDOW: 4:15
   - MINLEN: 36
5. **Execute**

### 3. Alinhamento ao Genoma de Referência

#### HISAT2
1. **NGS: Mapping** → **HISAT2**
2. **Source for the reference genome**: Built-in genome
3. **Select a reference genome**: Human (GRCh38/hg38)
4. **Is this single or paired-end**: conforme seus dados
5. **FASTQ file**: selecionar arquivo(s) trimados
6. **Execute**

### 4. Quantificação de Expressão Gênica

#### featureCounts
1. **NGS: RNA Analysis** → **featureCounts**
2. **Alignment file**: selecionar saída do HISAT2
3. **Gene annotation file**: Built-in annotation
4. **Using built-in annotation**: Human (GRCh38)
5. **Feature type**: exon
6. **Attribute type**: gene_id
7. **Execute**

### 5. Análise Diferencial

#### DESeq2
1. **NGS: RNA Analysis** → **DESeq2**
2. **Count matrix**: combinar saídas do featureCounts
3. **Factor**: criar grupos (Tumor vs Normal)
4. **Execute**

### 6. Visualização de Resultados

#### Volcano Plot
1. **Graph/Display Data** → **Volcano Plot**
2. **Differential expression file**: saída do DESeq2
3. **Configure parameters** conforme necessário
4. **Execute**

#### Heatmap
1. **Graph/Display Data** → **heatmap2**
2. **Input should have column headers**: Yes
3. **Data transformation**: Log2 plus one
4. **Execute**

## Troubleshooting

### Problemas Comuns e Soluções

#### 1. Erro no SRA Toolkit
```bash
# Reconfigurar
vdb-config --restore-defaults
vdb-config --interactive

# Limpar cache
cache-mgr --clear
```

#### 2. Arquivos Corrompidos
```bash
# Verificar integridade
fastq-dump --validate SRR1234567

# Re-download se necessário
prefetch --force SRR1234567
```

#### 3. Problemas de Memória no Galaxy
- Use datasets menores para testes
- Considere subsampling dos dados
- Use servidores Galaxy com mais recursos

#### 4. Formato de Arquivo Incorreto
- Verifique se o formato está correto (fastqsanger vs fastq)
- Use **NGS: QC and manipulation** → **FASTQ Groomer** se necessário

### Dicas de Otimização

1. **Paralelização**: Use `parallel-fastq-dump` para downloads mais rápidos
2. **Compressão**: Mantenha arquivos comprimidos (.gz) para economizar espaço
3. **Metadados**: Sempre mantenha registro dos metadados das amostras
4. **Backup**: Faça backup dos arquivos brutos antes das análises

## Recursos Adicionais

### Documentação Oficial
- [Galaxy Training Materials](https://training.galaxyproject.org/)
- [NCBI SRA Handbook](https://www.ncbi.nlm.nih.gov/books/NBK242621/)
- [GDC User Guide](https://docs.gdc.cancer.gov/)

### Cursos Online
- Galaxy Training Network
- TCGA Training Materials
- EBI Training Online

### Comunidade
- Galaxy Community Hub
- Biostars Forum
- Reddit r/bioinformatics

---

**Nota**: Este tutorial serve como guia inicial. Sempre verifique as versões mais recentes das ferramentas e adapte os parâmetros conforme suas necessidades específicas de pesquisa.