# Comandos básicos en Linux 2

## **Objetivo:**

En esta sesión, los participantes aprenderán a utilizar comandos avanzados de Linux aplicados a bioinformática, incluyendo `fastq-dump`, `efetch`, `bio`, `awk` y `sed`. Se buscará desarrollar habilidades para manipular, extraer y procesar datos biológicos de manera eficiente mediante herramientas de línea de comandos.

## **Creación de la estructura del proyecto:**

Para organizar el trabajo bioinformático, se creará una estructura de proyecto utilizando los siguientes comandos:

```bash
# Crear el directorio principal del proyecto
mkdir Proyecto_NGS

# Moverse al directorio del proyecto
cd Proyecto_NGS

# Crear los subdirectorios necesarios para el flujo de trabajo
mkdir -p raw_data quality_control assembly annotation results scripts
```

---

## Uso del comando `fastq-dump`

Antes de utilizar el comando fastq-dump, es importante conocer un poco sobre la base de datos del Sequence Read Archive (SRA). 

### 1. ¿Qué es el Archivo de Secuencia de Lecturas (SRA)?

El Archivo de Secuencia de Lecturas (SRA, por sus siglas en inglés) es un servicio de almacenamiento de datos proporcionado por NCBI. 

### 2. ¿Cuáles son los esquemas de nomenclatura del SRA?

Los datos en el SRA están organizados bajo una estructura jerárquica donde cada categoría superior puede contener una o más subcategorías:

- **NCBI BioProject**: PRJN**** (ejemplo: PRJNA257197) contiene la descripción general de una sola iniciativa de investigación; un proyecto generalmente estará relacionado con múltiples muestras y conjuntos de datos.
- **NCBI BioSample**: SAMN**** o SRS**** (ejemplo: SAMN03254300) describe material biológico fuente; cada espécimen físicamente único se registra como una única BioSample con un conjunto único de atributos.
- **SRA Experiment**: SRX****, una biblioteca de secuenciación única para una muestra específica.
- **SRA Run**: SRR**** o ERR**** (ejemplo: SRR1553610) es un manifiesto de archivo(s) de datos vinculado a una biblioteca de secuenciación dada (experimento).

![](https://github.com/mramirezs/Accediendo-y-Automatizando-el-Acceso-al-Short-Read-Archive-SRA-/blob/main/figures/sra_categorias.png)

## 3. ¿Cómo descargamos datos del SRA?

Puedes descargar datos a través de un navegador web (https://www.ncbi.nlm.nih.gov/sra) o desde la línea de comandos mediante el paquete sratoolkit (https://github.com/ncbi/sra-tools), que se puede instalar a través de conda.

```
conda install bioconda::sra-tools
```

### 4. ¿Existen alternativas al SRA?

![](https://github.com/mramirezs/Accediendo-y-Automatizando-el-Acceso-al-Short-Read-Archive-SRA-/blob/main/figures/ENA_alnternativa.png)

### 5. ¿Donde está la documentación del SRA?

![](https://github.com/mramirezs/Accediendo-y-Automatizando-el-Acceso-al-Short-Read-Archive-SRA-/blob/main/figures/documentacion_sra.png)

### 6. ¿Como funciona el SRAtoolkit?

Existen varias herramientas y las más utilizadas son:

- **fastq-dump**, descarga datos en formato FASTQ.
- **sam-dump**, descarga datos en formato de alineación SAM.

Un caso de uso típico es `fastq-dump SRR1553607`, lo que crea el archivo: `SRR1553607.fastq`.

Por defecto, los datos del SRA concatenarán las lecturas pareadas. Para las lecturas pareadas, los datos deben separarse en diferentes archivos:

```bash
fastq-dump --split-files SRR1553607
```

Esto crea los archivos de lecturas pareadas:

```bash
SRR1553607_1.fastq
SRR1553607_2.fastq
```

El comando `fastq-dump` realiza dos cosas en secuencia. Primero descarga los datos en el formato SRA y luego los convierte al formato FASTQ. 

![](https://github.com/mramirezs/Accediendo-y-Automatizando-el-Acceso-al-Short-Read-Archive-SRA-/blob/main/figures/descarga_fastq.png)

Podemos pedirle a la herramienta que convierta solo un subconjunto de datos; por ejemplo, las primeras 10,000 lecturas:

```bash
fastq-dump --split-files -X 10000 SRR1553607
```

Pero notamos que incluso en ese caso, `fastq-dump` necesita primero descargar el archivo de datos completo.

### 7. ¿Existe una forma más sencilla de consultar números de SRA?

Philip Ewels creó un servicio llamado [SRA Explorer](https://sra-explorer.info/) que proporciona una interfaz incomparablemente mejor que la de NCBI.

Dado que es un servicio proporcionado por un científico independiente, puede no estar continuamente actualizado o soportado.

## Automatizando el acceso al SRA

- Demostramos cómo automatizar el acceso al Short Read Archive (SRA) para obtener datos depositados con publicaciones científicas.
   
- Mostramos cómo obtener información sobre todos los datos almacenados en SRA y compilar varias estadísticas sobre ellos. 

### ¿Cómo automatizar la descarga de múltiples ejecuciones de SRA?

La estrategia principal es obtener primero el número del proyecto PRJN, luego, a partir de los números del proyecto, obtener los números de ejecución SRR, y finalmente ejecutar `fastq-dump` en cada número SRR. 

Comenzaremos con el número del bioproyecto que se encuentra en la publicación. Por ejemplo, en *Genomic surveillance elucidates Ebola virus origin*, notamos que el número del proyecto es PRJNA257197.

```bash
esearch -db sra -query PRJNA257197
```

El comando anterior devuelve un “resultado de búsqueda” que nos indica que hay 891 ejecuciones de secuenciación para este ID de proyecto. El resultado de la búsqueda es un archivo XML que puedes canalizar a otros programas:

```xml
<ENTREZ_DIRECT>
  <Db>sra</Db>
  <WebEnv>MCID_66d63afd3456474c30759e40</WebEnv>
  <QueryKey>1</QueryKey>
  <Count>891</Count>
  <Step>1</Step>
</ENTREZ_DIRECT>
```

Ahora podemos usar `efetch` y formatear el resultado de la búsqueda como un tipo de datos llamado runinfo. El tipo runinfo está en formato CSV (valores separados por comas):

```bash
esearch -db sra -query PRJNA257197 | efetch -format runinfo > runinfo.csv
```

El archivo resultante `runinfo.csv` es bastante rico en metadatos y contiene una serie de atributos para la ejecución.

Al cortar la primera columna de este archivo:

```bash
cat runinfo.csv | cut -f 1 -d ',' | head
```

Obtendrás:

```
Run
SRR1972948
SRR1972956
SRR1972955
SRR1972969
SRR1972958
SRR1972943
SRR1972970
SRR1972960
SRR1972949
...
```

Para filtrar las líneas que coincidan con SRR y almacenar los primeros diez ID de ejecución en un archivo, podríamos escribir:

```bash
cat runinfo.csv | cut -f 1 -d ',' | grep SRR | head > runids.txt
```

Nuestro archivo `runids.txt` contiene:

```
SRR1972948
SRR1972956
SRR1972955
SRR1972969
SRR1972958
SRR1972943
SRR1972970
SRR1972960
SRR1972949
SRR1972964
```

Ahora tenemos acceso a los ID de ejecución SRR para el experimento. En el siguiente paso, deseamos generar una serie de comandos de la siguiente forma:

```bash
fastq-dump -X 10000 --split-files SRR1972948
fastq-dump -X 10000 --split-files SRR1972956
fastq-dump -X 10000 --split-files SRR1972955
...
```

> Nota cómo los comandos son casi iguales; solo sustituimos algunas partes de cada uno. En el mundo UNIX, se pueden utilizar diferentes técnicas para generar los comandos anteriores. Nuestra elección para la automatización utiliza la herramienta `parallel`, que ejecuta comandos en "paralelo", uno para cada núcleo de CPU que tiene tu sistema. Es un comando extremadamente versátil con una gran cantidad de características únicas, bien adecuado para la mayoría de las tareas que enfrentarás. Vale la pena dedicar tiempo a entender cómo funciona `parallel` para apreciar completamente las ganancias en productividad que puedes obtener en tu trabajo.

**Ejemplo:**

Ejecuta el comando `echo` en cada elemento del archivo:

```bash
cat runids.txt | parallel "echo This is number {}"
```

Producirá:

```
This is number SRR1972948
This is number SRR1972956
This is number SRR1972955
This is number SRR1972969
This is number SRR1972958
This is number SRR1972943
This is number SRR1972970
This is number SRR1972960
This is number SRR1972949
This is number SRR1972964
...
```

Podemos reemplazar el comando `echo` por `fastq-dump`:

```bash
cat runids.txt | parallel "echo fastq-dump -X 10000 --split-files {}"
```

```bash
cat runids.txt | parallel fastq-dump -X 10000 --split-files {}
```

El comando anterior generará veinte archivos (dos archivos para cada una de las diez ejecuciones).

---

## Uso del comando `efetch`

### **1. ¿Qué es Entrez?**
- **Entrez** es el sistema principal de búsqueda y recuperación de texto de NCBI.
- Integra **PubMed** (base de datos de literatura biomédica) con otras **39 bases de datos**, incluyendo secuencias de ADN, proteínas, genomas, variación genética y expresión génica.
- NCBI almacena una gran cantidad de datos, desde información cruda hasta resultados curados, incluyendo publicaciones científicas.
- El acceso a la información puede ser complicado debido al tamaño y la complejidad de las interfaces, lo que hace que el acceso por línea de comandos sea una herramienta valiosa.

### **2. Automatización del acceso a Entrez**
- NCBI ofrece una **API web** llamada **Entrez E-utils** y una suite de herramientas llamada **Entrez Direct**.
- **Entrez Direct** permite acceder a las bases de datos de NCBI desde la línea de comandos de UNIX, facilitando la descarga y el análisis de datos.

### **3. Organización de los datos en NCBI**
- Los datos en NCBI están organizados con **números de acceso** (accession numbers) y **números de versión**.
  - **Número de acceso**: Identificador único y estable para un registro (ejemplo: AF086833).
  - **Número de versión**: Identificador único para la secuencia dentro de un registro (ejemplo: AF086833.2). Se incrementa si la secuencia cambia.
- Los **números GI** (enteros asociados a registros) están siendo eliminados desde 2015.

### **4. Uso de Entrez E-utils**

- Permite consultar bases de datos de NCBI mediante URLs específicos.
- Ejemplo de uso con `curl`:
  ```bash
  curl -s 'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?id=AF086833.2'
  ```
- Es necesario proteger caracteres especiales como `&` en la línea de comandos.

### **5. ¿Qué es Entrez Direct?**

- Es una suite de herramientas que simplifica el acceso a las bases de datos de NCBI.
- **Herramientas principales**:
  - `efetch`: Recupera datos en varios formatos (ejemplo: GenBank, FASTA).
  - `esearch`: Realiza búsquedas en las bases de datos.
    
- Ejemplos de uso:
  ```bash
  efetch -db nuccore -format fasta -id AF086833 > AF086833.fa
  esearch -db nucleotide -query PRJNA257197 | efetch -format fasta > genomes.fa
  ```

### **6. Búsquedas con Entrez Direct**
- `esearch` genera un "entorno" que puede pasarse a otras herramientas como `efetch`.
- Ejemplo:
  ```bash
  esearch -db nucleotide -query PRJNA257197 | efetch -format fasta > genomes.fa
  ```

### **7. Información de proyectos y secuencias**
- Se puede obtener información detallada sobre proyectos (ejemplo: `PRJNA257197`) y secuencias.
- Ejemplo:
  ```bash
  esearch -db sra -query PRJNA257197 | efetch -format runinfo > runinfo.csv
  ```

### **8. Extracción de información taxonómica**
- `efetch` permite obtener información taxonómica en formato XML.
- Ejemplo:
  ```bash
  efetch -db taxonomy -id 9606,7227,10090 -format xml > output.xml
  ```
- Herramientas como `xtract` ayudan a procesar y extraer datos específicos.

---

## **Uso del comando `bio`**

El paquete `bio` incluye dos comandos principales: `bio fetch` para descargar datos y `bio search` para buscar información en diversas bases de datos biológicas. A continuación, se presenta un resumen estructurado de ambos comandos:

---

### **1. Instalación**
- Para instalar o actualizar la herramienta `bio`, ejecuta:
  ```bash
  pip install bio --upgrade
  ```
- La documentación completa se encuentra en: [https://www.bioinfo.help/](https://www.bioinfo.help/).

---

### **2. `bio fetch`: Descarga de datos**

#### **2.1 Funcionalidad principal**
- **Descarga automática de datos**: `bio fetch` identifica automáticamente el tipo de dato basado en el número de acceso y lo descarga en el formato más adecuado.
- **Fuentes soportadas**: GenBank, Ensembl, Short Read Archive (SRA), entre otros.

#### **2.2 Ejemplos de uso**
- **Descarga de datos desde GenBank**:
  - **Nucleótidos**:
    ```bash
    bio fetch NC_045512
    ```
  - **Proteínas**:
    ```bash
    bio fetch YP_009724390
    ```
  - **Múltiples accesiones**:
    ```bash
    bio fetch NC_045512 MN996532 > genomes.gb
    ```
  - **Formatos alternativos**:
    - **FASTA**:
      ```bash
      bio fetch NC_045512 --format fasta
      ```
    - **GFF**:
      ```bash
      bio fetch NC_045512 --format gff
      ```
      
- **Descarga de datos desde Ensembl**:
  - **Genes**:
    ```bash
    bio fetch ENSG00000157764
    ```
  - **Transcriptos**:
    - **Contexto genómico**:
      ```bash
      bio fetch ENST00000288602
      ```
    - **ADN complementario (cDNA)**:
      ```bash
      bio fetch ENST00000288602 --type cdna
      ```
    - **Secuencia codificante (CDS)**:
      ```bash
      bio fetch ENST00000288602 --type cds
      ```
    - **Proteína**:
      ```bash
      bio fetch ENST00000288602 --type protein
      ```

---

### **3. `bio search`: Búsqueda de información**

#### **3.1 Funcionalidad principal**
- **Búsqueda unificada**: `bio search` proporciona una interfaz de línea de comandos para buscar en múltiples fuentes de datos, como **GenBank**, **SRA** y **MyGene**.
- **Reconocimiento automático**: Identifica automáticamente el tipo de búsqueda basado en el término proporcionado (por ejemplo, números de acceso, símbolos de genes, etc.).

#### **3.2 Ejemplos de uso**
- **Búsqueda en GenBank**:
  - **Nucleótidos**:
    ```bash
    bio search AF086833
    ```
  - **Proteínas**:
    ```bash
    bio search NP_000509
    ```

- **Búsqueda en SRA**:
  - **Información de bioproyectos**:
    ```bash
    bio search PRJNA257197 --limit 10 -tab
    ```
  - **Información de runs**:
    ```bash
    bio search SRR14575325
    ```

- **Búsqueda en MyGene**:
  - **Símbolos de genes**:
    ```bash
    bio search symbol:HBB --limit 1
    ```
  - **Anotación con IDs de Ensembl**:
    ```bash
    bio search symbol:HBB --species human --fields ensembl
    ```

- **Búsqueda en ensamblados de NCBI**:
  - **Por número de ensamblado**:
    ```bash
    bio search GCA_000002415
    ```
  - **Por nombre de organismo**:
    ```bash
    bio search plasmodium -tab
    ```

## **Uso de `sed` y `awk` para manipulación de archivos biológicos**

A continuación, se presenta una guía práctica para el uso de los comandos `sed` y `awk` en la manipulación de archivos biológicos como FASTA, FASTQ, entre otros. Incluye tablas de opciones y órdenes comunes, así como ejercicios prácticos.

---

### **1. Comando `sed` (Stream Editor)**

#### **1.1 Características principales**
- **Editor de flujo de texto**: Realiza transformaciones en texto sin modificar el archivo original (a menos que se indique).
- **Operación línea por línea**: Procesa el archivo línea a línea.
- **Soporte para expresiones regulares**: Permite búsquedas y sustituciones avanzadas.

#### **1.2 Sintaxis básica**
```bash
sed [opcion(es)] ['orden(es)'] [archivo(s)]
```

#### **1.3 Opciones comunes de `sed`**

| **Opción** | **Descripción**                                                                 |
|------------|---------------------------------------------------------------------------------|
| `-e`       | Especifica múltiples órdenes.                                                   |
| `-f`       | Lee las órdenes desde un archivo.                                               |
| `-n`       | Suprime la salida automática (útil con la orden `p`).                           |
| `-i`       | Edita el archivo directamente (in-place).                                       |
| `-r`       | Usa expresiones regulares extendidas (ERE).                                     |

#### **1.4 Órdenes comunes de `sed`**

| **Orden** | **Descripción**                                                                 |
|-----------|---------------------------------------------------------------------------------|
| `s`       | Sustituye texto. Ejemplo: `s/old/new/`.                                         |
| `p`       | Imprime líneas.                                                                 |
| `d`       | Borra líneas.                                                                   |
| `a`       | Añade texto después de una línea.                                               |
| `i`       | Inserta texto antes de una línea.                                               |
| `c`       | Cambia una línea por otro texto.                                                |
| `q`       | Sale del script después de procesar una línea.                                  |

---

### **1.3. Ejercicios con `sed`**

#### **1.3.1 Ejercicio: Extraer encabezados de un archivo FASTA**
- **Archivo**: `secuencia.fasta` (obtenido con `bio fetch` o `efetch`).
- **Objetivo**: Extraer solo los encabezados de las secuencias.
- **Comando**:
  ```bash
  sed -n '/^>/p' NC_045512.2.fasta
  ```
- **Salida**:
  ```
  >NC_045512.2 Severe acute respiratory syndrome coronavirus 2 isolate Wuhan-Hu-1, complete genome
  ```

#### **1.3.2 Ejercicio: Eliminar líneas vacías en un archivo FASTQ**
- **Archivo**: `SRR1972917_1.fastq` (obtenido con `bio fetch` o `fastq-dump`).
- **Objetivo**: Eliminar líneas vacías.
- **Comando**:
  ```bash
  sed '/^$/d' SRR1972917_1.fastq
  ```
- **Salida**: Archivo FASTQ sin líneas vacías.

#### **1.3.3 Ejercicio: Cambiar el formato de un archivo FASTA**
- **Archivo**: `secuencia.fasta`.
- **Objetivo**: Cambiar el formato del encabezado.
- **Comando**:
  ```bash
  sed 's/>.*/>new_variant/' NC_045512.2.fasta
  ```
- **Salida**:
  ```
  >new_variant
  ATTAAAGGTTTATACCTTCCCAGGTAACAAACCAACCAACTTTCGATCTCTTGTAGATCTGTTCTCTAAAC
  ```

### **2. Comando `awk`**

#### **2.1 Características principales**
- **Lenguaje de programación**: Permite manipular y analizar datos de texto.
- **Operación línea por línea**: Procesa el archivo línea a línea.
- **Excelente para análisis de columnas**: Ideal para archivos tabulares.

#### **2.2 Sintaxis básica**
```bash
awk '/patron_busqueda/{accion_a_realizar_coincidencia;otra_accion}' archivo
```

#### **2.3 Variables comunes de `awk`**

| **Variable** | **Descripción**                                                                 |
|--------------|---------------------------------------------------------------------------------|
| `$0`         | Representa toda la línea actual.                                                |
| `$1, $2, ...`| Representa la primera, segunda, etc., columna de la línea.                      |
| `NR`         | Número de línea actual.                                                         |
| `NF`         | Número de campos (columnas) en la línea actual.                                 |

---

#### **2.4 Ejercicios con `awk`**

##### **2.4.1 Ejercicio: Extraer secuencias de un archivo FASTA**
- **Archivo**: `NC_045512.2.fasta`.
- **Objetivo**: Extraer solo las secuencias (sin encabezados).
- **Comando**:
  ```bash
  awk '/^[^>]/ {print}' NC_045512.2.fasta
  ```
- **Salida**:
  ```
  ATTAAAGGTTTATACCTTCCCAGGTAACAAACCAACCAACTTTCGATCTCTTGTAGATCTGTTCTCTAAAC
  ```

##### **2.4.2 Ejercicio: Contar el número de secuencias en un archivo FASTA**
- **Archivo**: `NC_045512.2.fasta`.
- **Objetivo**: Contar cuántas secuencias hay.
- **Comando**:
  ```bash
  awk '/^>/ {count++} END {print count}' NC_045512.2.fasta
  ```
- **Salida**:
  ```
  1
  ```

##### **2.4.3 Ejercicio: Filtrar líneas de calidad en un archivo FASTQ**
- **Archivo**: `SRR1972917_1.fastq`.
- **Objetivo**: Extraer solo las líneas de calidad.
- **Comando**:
  ```bash
  awk 'NR % 4 == 0' SRR1972917_1.fastq
  ```
- **Salida**:
  ```
  @CCFDDFFHGHHHGIJJIIJJJJGJJJJGIJIJJFIJJJIIJJJJHHFHHFFFFAADEFEDDDDDDDDDDDD??CCCDDDDDDCCCDDCCCDD:?CABDDB
   @CCFFFFFHHHHHJJJJJJJJGIJJJIJJJJJJJJJIJJIJJJJJJJJIHIHHHFFDDDDDDDDDDDDDBDBDCDDDDDDDDDDD@C@DCDDDDDDCCDD:
   @@CFFDDDHHHHHJJIJJJJJIHIIJJJJIJJHHHHHFFFDDD>BDDDDDDDDDDDDB?8AB:@>BBBDDDD@CDCAAADDDCDCACDD-)99<889@9B<
   BBBFFFFFHHHHHJJJJJJJIJJJJJIJJJJIJJJJJJJJJJJJJJJIJIIIHIIIIIJJJIIJIJIIJIIJJJJIHHFFFDDEEECDD;?CDEEDDDDDC
   CCCFFFFFHHHGHHHIIJIIIJJIJJJJGHIJJIHHIJJJBHJJIJIIEGHGHFFDDBBBD?BBC>CDD@?B99>&5955ACDCBD<A3>9A<<BDBDD?<
   CCCFFFFFHHHHHJJJJJIGHHIHIJIJJJFFHHJJJGIHHEHFFFDDDDDDDDDCDDDDDDDDDDDDBDBBABAACDDDDDCDBB>CAACDBDDDD<>4:
   CCCFFFFFHHHHHJJIJJJJJJFHIIHIJJJGJJHHEIJJJIIJJJJGEDGGHJHHHFEBDDFDB=B##################################

  ```

##### **2.4.3 Ejercicio: Extraer IDs de un archivo FASTQ**
- **Archivo**: `SRR1972917_1.fastq`.
- **Objetivo**: Extraer solo los IDs de las secuencias.
- **Comando**:
  ```bash
  awk 'NR % 4 == 1' SRR1972917_1.fastq
  ```
- **Salida**:
  ```
  @SRR1972976.1
  ```

##### **4.2 Ejercicio: Convertir un archivo FASTQ a FASTA**
- **Archivo**: `SRR1972917_1.fastq`.
- **Objetivo**: Convertir el archivo FASTQ a formato FASTA.
- **Comando**:
  ```bash
  awk 'NR % 4 == 1 {print ">" substr($0, 2)} NR % 4 == 2 {print}' SRR1972917_1.fastq > SRR1972917_1_converted.fasta
  ```
- **Salida** (`SRR1972917_1_converted.fasta`):
  ```fasta
  >SRR1972976.1
  ATCGATCGATCGATCG
  ```
