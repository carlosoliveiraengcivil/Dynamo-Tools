# 🚀 Civil 3D— Automação Dynamo

> **Soluções avançadas em automação BIM, topografia computacional, processamento de dados de drones e modelagem de terraplenagem para Autodesk Civil 3D.**

Esta biblioteca reúne um conjunto de rotinas e algoritmos em **Dynamo (.dyn)** potencializados por scripts em **Python (CPython 3 / .NET API do AutoCAD e Civil 3D)**, desenvolvidos para eliminar tarefas manuais repetitivas, processar levantamentos complexos e garantir precisão cirúrgica em projetos de infraestrutura.

---

## 📋 Tabela Resumo das Ferramentas

| Ferramenta | Arquivo `.dyn` | Categoria | Descrição Rápida |
| :--- | :--- | :--- | :--- |
| **3DPoly Builder** | `3DPOLY BUILDER - VETORIZAÇÃO.dyn` | Topografia & Wireframe | Constrói polilinhas 3D longitudinais a partir de seções transversais com algoritmo auto-calibrável de conduíte elástico e detecção visual de falhas. |
| **Breakline Extractor** | `BREAKLINE EXTRACTOR - VETORIZAÇÃO DE TALUDES.dyn` | Terraplenagem & Drones | Fatiamento transversal e esqueletização para extração inteligente de cristas e pés de talude em superfícies TIN ruidosas. |
| **Cota x Área x Volume** | `COTA x ÁREA x VOLUME - TIN SURFACE.dyn` | Hidráulica & Cubagem | Fatiamento analítico por prismas 3D com "Tampa Inclinada" virtual e geração direta de tabela AutoCAD ABNT para cavas e reservatórios. |
| **Gerador de Locação** | `GERADOR DE LOCAÇÃO.dyn` | Campo & GNSS RTK | Criação contínua em massa de CogoPoints por estaqueamento e/ou vértices ao longo de múltiplos eixos, com filtro anti-duplicidade e padrão PNEZD. |
| **SpotBlock** | `SPOTBLOCK - INSERÇÃO DE BLOCOS.dyn` | Desenho Cadastral | Inserção em lote de blocos AutoCAD nas coordenadas 3D dos pontos com herança inteligente de propriedades (*Smart Eyedropper*). |
| **Surface Grid Builder** | `SURFACE GRID BUILDER - PRÉ-PROCESSAMENTO.dyn` | Fotogrametria & Drones | Resampler de superfícies pesadas de drones em malhas regulares ortogonais, eliminando ruídos e acelerando o Civil 3D. |
| **Z-Contours Editor** | `Z-CONTOURS - EDITOR DE CURVAS DE NÍVEL.dyn` | Cartografia & Apresentação | Classificação e separação automática de curvas de nível em Mestras (múltiplas de 5 m) e Intermediárias, criando e ajustando layers e cores ACI. |

---

## 🛠️ Detalhamento das Ferramentas

### 1. 3DPoly Builder — Vetorização Longitudinal
* **Arquivo:** `3DPOLY BUILDER - VETORIZAÇÃO.dyn`
* **Módulo Python:** *Auto-Calibração com Paredão de Borracha & Sensor de Erros*
* **Objetivo:**
  Transformar levantamentos de campo brutos (onde o topógrafo mediu seções transversais ponto a ponto) em aramados 3D (*wireframes*) contínuos e paralelos para estradas, canais, ferrovias e taludes.
* **Lógica do Algoritmo:**
  - O usuário fornece uma seção semente (`seed_section`), que dita a ordem e a topologia transversal dos pontos (ex: pé esquerdo, eixo, pé direito).
  - O algoritmo calcula um vetor guia elástico com largura auto-calibrável (conduíte mínimo de segurança para evitar colapsos em seções estreitas).
  - Ele rastreia a nuvem de pontos à frente, descobrindo as seções subsequentes e costurando as **Polyline3D** longitudinalmente.
  - **Fail-Safe:** Caso o levantamento de campo pule um ponto ou ocorra inconsistência, o script desenha todas as polilinhas até a última seção válida e gera um marcador visual na layer `ERRO_AUTOMACAO` exatamente onde a correspondência foi interrompida.
* **Entradas:**
  - `raw_pool`: Conjunto de pontos brutos selecionados no AutoCAD.
  - `seed_section`: Pontos da primeira seção transversal (ordem de conexão).
  - `layer_name`: Camada de destino das Polilinhas 3D criadas.

---

### 2. Breakline Extractor — Vetorização Inteligente de Taludes
* **Arquivo:** `BREAKLINE EXTRACTOR - VETORIZAÇÃO DE TALUDES.dyn`
* **Módulo Python:** *Smart Extractor V9 (Cross-Section & Skeletonization)*
* **Objetivo:**
  Extrair automaticamente linhas de quebra (*breaklines*) de crista e pé de talude em superfícies topográficas irregulares ou ruidosas (como as geradas por aerofotogrametria com drones).
* **Lógica do Algoritmo:**
  - Utiliza uma polilinha guia como eixo de varredura direcional do talude.
  - Executa fatiamentos transversais analíticos em intervalos regulares (`step_dist`) ao longo de uma faixa de largura definida (`sweep_width`).
  - Em cada fatia 2D, avalia derivadas de inclinação e curvatura para encontrar os pontos críticos de inflexão (bordo superior e inferior).
  - Filtra ruídos de vegetação e pedras através de corte de declividade (`slope_cut`), une cristas muito próximas (`merge_tol`) e suaviza o traçado final (`smooth_tol`).
  - Através do `bake_mode`, projeta e grava diretamente as Polilinhas 3D de crista e pé no banco de dados do desenho.
* **Entradas:**
  - `TIN Surface`: Superfície alvo no Civil 3D.
  - `Guia do Talude`: Polilinha orientativa do alinhamento do talude.
  - `Step Distance`: Intervalo longitudinal de amostragem transversal (ex: 2.0 m).
  - `Sweep Width`: Largura total de varredura lateral (ex: 300 m).
  - `Slope Cutoff`: Sensibilidade de detecção de rampa/inclinação (ex: 1.0).
  - `Merge Tolerance`: Distância para fusão de bicos de leira e bifurcações (ex: 25.0 m).
  - `Smooth Tolerance`: Fator de suavização geométrica (ex: 1.0 m).
  - `Bake Mode`: Booleano (`True` para gerar entidades definitivas no DWG).

---

### 3. Cota x Área x Volume — Fatiador Analítico de Superfícies TIN
* **Arquivo:** `COTA x ÁREA x VOLUME - TIN SURFACE.dyn`
* **Módulo Python:** *Nexus Table Generator V10.1 (Sólido Escavado & Tampa Inclinada)*
* **Objetivo:**
  Realizar o cálculo rigoroso de curva cota-área-volume e capacidade total de reservatórios, sumps de drenagem, bacias de contenção e cavas de mineração, gerando uma tabela de relatório nativa no AutoCAD formatada em normas brasileiras.
* **Lógica do Algoritmo:**
  - Supera as falhas do método convencional de curvas de nível (`ExtractContoursAt`), que "vazam" e abrem as linhas quando a crista do reservatório não é plana.
  - Utiliza decomposição em prismas triangulares 3D (`calculate_prism_properties`) integrando áreas e volumes analíticos sob planos de corte.
  - **Tampa Inclinada Dinâmica:** Rastreia as arestas da borda 3D da superfície e calcula a tampa virtual no centroide da crista, garantindo precisão matemática mesmo para geometrias inclinadas complexas.
  - **Tolerância Zero / Modo Expresso:** Se a cota máxima ou o espaçamento forem omitidos ou preenchidos com `0`, o algoritmo calcula automaticamente o volume total de capacidade da cava em uma fração de segundo.
  - Insere uma entidade `AcDb.Table` no Model Space com cabeçalhos e valores no formato PT-BR (`0.000,000`).
* **Entradas:**
  - `TIN Surface`: Superfície da cava / reservatório.
  - `Cota Máxima`: Elevação limite superior (opcional; `0` ou vazio = cota máxima real da borda).
  - `Espaçamento`: Passo vertical entre fatias (opcional; `0` ou vazio = capacidade total instantânea).
  - `Ponto de Inserção`: Coordenada de inserção da tabela no DWG.

---

### 4. Gerador de Locação — CogoPoints Contínuos em Massa
* **Arquivo:** `GERADOR DE LOCAÇÃO.dyn`
* **Módulo Python:** *Multi-Alignment Continuous Stationing Engine*
* **Objetivo:**
  Criar automaticamente centenas ou milhares de pontos topográficos do Civil 3D (`CogoPoints`) ao longo de múltiplos eixos e alinhamentos para exportação direta a coletores de dados GNSS RTK e Estações Totais.
* **Lógica do Algoritmo:**
  - Suporta múltiplos eixos/curvas em uma única execução.
  - Mantém o **contador sequencial de numeração contínuo** entre os diferentes eixos, eliminando conflitos de numeração de pontos em campo.
  - Modos de operação versáteis:
    1. Apenas estacas regulares (intervalo customizável, ex: a cada 10 ou 20 metros).
    2. Apenas vértices geométricos (PIs, começos e fins de curva).
    3. Híbrido: estaqueamento fixo intercalado com todos os vértices.
  - **Filtro Espacial Anti-Duplicidade:** Compara a distância 3D entre pontos sucessivos e descarta duplicidades dentro da tolerância milimétrica.
  - Formatação PNEZD profissional com concatenação de estacas no formato rodoviário (ex: `EST 12+15.00 - BORDO DIREITO`) e inclusão no grupo de pontos selecionado.
* **Entradas:**
  - `Eixos / Curvas`: Polilinhas ou geometrias de eixo.
  - `Somente Vértices`: Booleano para extrair apenas mudanças de direção.
  - `Intervalo de Estacas`: Espaçamento métrico (ex: 10 m ou 20 m).
  - `Base de Estaqueamento`: Padrão de estaca (geralmente 20 m para infraestrutura brasileira).
  - `Estaca Inicial`: Valor de partida do estaqueamento.
  - `Incluir Vértices`: Booleano para adicionar vértices junto às estacas regulares.
  - `Descrição Manual`: Prefixo ou descrição técnica dos pontos (ex: "PISTA", "EIXO_DRENAGEM").

---

### 5. SpotBlock — Inserção Automatizada de Blocos Topográficos
* **Arquivo:** `SPOTBLOCK - INSERÇÃO DE BLOCOS.dyn`
* **Módulo Python:** *Smart Eyedropper + CPython Safe (V2.2)*
* **Objetivo:**
  Substituir pontos de levantamento cadastral por blocos temáticos do AutoCAD (árvores, postes, caixas de passagem, poços de visita, estacas de fundação, marcos de divisa) com rapidez e padronização.
* **Lógica do Algoritmo:**
  - Desempacota coleções e listas complexas de pontos 3D e extrai suas coordenadas reais.
  - Insere as instâncias de `BlockReference` no espaço corrente do desenho (`ModelSpace`).
  - **Smart Eyedropper:** Identifica dinamicamente a camada padrão do bloco pré-carregado no DWG e direciona a inserção para a layer correta, preservando o padrão de layers do projeto.
  - Permite parametrização individual ou global de escala e ângulo de rotação.
* **Entradas:**
  - `Pontos`: Coordenadas ou objetos de pontos de campo.
  - `Nome do Bloco`: Identificador exato da definição do bloco carregado no DWG.
  - `Escala`: Fator multiplicador de escala (padrão: 1.0).
  - `Rotação`: Rotação angular em graus decimais (padrão: 0.0°).

---

### 6. Surface Grid Builder — Reamostrador e "Alisador" de Drones
* **Arquivo:** `SURFACE GRID BUILDER - PRÉ-PROCESSAMENTO.dyn`
* **Módulo Python:** *Surface Grid Resampler (V1.1)*
* **Objetivo:**
  Filtrar e pré-processar modelos digitais de terreno (MDT) oriundos de voos de drones e varreduras laser, que normalmente possuem excesso de ruído superficial (gramíneas, pedras, reflexos) e sobrecarregam o Civil 3D com milhões de triângulos desnecessários.
* **Lógica do Algoritmo:**
  - Acessa os limites geométricos nativos (`GeometricExtents` / Bounding Box) da superfície de entrada diretamente pela API do Civil 3D.
  - Constrói uma malha de amostragem ortogonal regular com passo paramétrico (ex: 0.5 m, 1.0 m ou 2.0 m).
  - Consulta a elevação real interpolada em cada nó do grid e compõe uma nova nuvem/superfície limpa, uniforme e altamente otimizada.
  - Reduz drasticamente o peso do arquivo DWG e estabiliza o modelo para extrações de taludes, seções e cálculo de terraplenagem.
* **Entradas:**
  - `TIN Surface`: Superfície densa/bruta do drone.
  - `Grid Size`: Tamanho do quadrilátero de amostragem em metros (ex: 0.5 m ou 1.0 m).

---

### 7. Z-Contours — Editor e Padronizador de Curvas de Nível
* **Arquivo:** `Z-CONTOURS - EDITOR DE CURVAS DE NÍVEL.dyn`
* **Módulo Python:** *Curvature & Z-Elevation Cartographic Dispatcher*
* **Objetivo:**
  Classificar, organizar e formatar desenhos topográficos que contenham curvas de nível soltas ou despadronizadas (2D ou 3D Polylines).
* **Lógica do Algoritmo:**
  - Analisa a elevação $Z$ de cada polilinha selecionada no desenho.
  - Aplica a regra cartográfica: curvas com cotas que são múltiplos exatos de **5,00 m** são catalogadas como **Curvas Mestras**, e as demais como **Curvas Intermediárias**.
  - Garante e cria automaticamente as camadas `CURVAS_MESTRAS` (cor ACI 10 - Vermelho) e `CURVAS_INTERMEDIARIAS` (cor ACI 34 - Marrom/Ocre) através do método `ColorMethod.ByAci` na tabela de layers do AutoCAD (`LayerTable`).
  - Transfere as entidades para suas respectivas camadas instantaneamente.
* **Entradas:**
  - `Objetos / Polilinhas`: Seleção de curvas de nível no AutoCAD.

---

## 💻 Requisitos e Configuração

* **Software Base:** Autodesk Civil 3D (versões 2022, 2023, 2024 ou 2025).
* **Ambiente de Execução:** Dynamo for Civil 3D (Core 2.12+).
* **Mecanismo Python Recomendado:** **CPython 3** (padrão nas versões modernas do Dynamo).
* **Bibliotecas .NET Utilizadas:**
  - `AcMgd.dll` (AutoCAD Managed Core)
  - `AcDbMgd.dll` (AutoCAD Database Services)
  - `AcCoreMgd.dll` (AutoCAD Core Engine)
  - `AeccDbMgd.dll` (Civil 3D Database Services)
  - `ProtoGeometry.dll` (Dynamo Geometry Engine)

---

## 🚀 Como Utilizar

1. **Via Dynamo Player:**
   - No Civil 3D, abra a aba **Manage** > **Dynamo Player**.
   - Aponte a pasta de busca para este repositório.
   - Clique no ícone de entradas (*Edit inputs*) da rotina desejada, selecione os objetos na tela e execute com um clique.

2. **Via Dynamo Editor:**
   - Abra o **Dynamo**, clique em **Open** e selecione o arquivo `.dyn`.
   - Certifique-se de que o modo de execução está em **Manual** (canto inferior esquerdo).
   - Conecte as seleções de tela do Civil 3D e clique em **Run**.

---

## 👨‍💻 Autor & Manutenção

* **Desenvolvedor:** Carlos Oliveira
* **Especialidade:** Engenharia de Infraestrutura, Automação BIM & Topografia Computacional
* **Contato / Repositório:** [GitHub - Carlos Oliveira](https://github.com/)

---
*Transformando horas de cliques repetitivos em segundos de processamento analítico com o Autodesk Civil 3D.*
