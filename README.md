# Comandos_basicos_en_Linux_2

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
