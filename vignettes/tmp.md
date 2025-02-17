# Pre-Workshop Instructions

Please perform the following steps *before the workshop*.
These steps take a few minutes
and you may have issues specific to your machine.

## STEP 1: Install the latest Cytoscape into your Desktop environment(3.9.0)
- If you have Cytoscape installed before 3.9.0, please update it.
- [Download Cytoscape](https://cytoscape.org/download.html).
- Open and follow installation steps.
  - If you don't have Java in your environment, Cytoscape will ask you if you want to download it. Please accept it and download Java.

Mac users need to be careful about
![image](https://user-images.githubusercontent.com/12192/139439069-dba3e46a-8fe2-414c-98fe-55d02ba39e32.png)

- Launch Cytoscape 3.9.0 from start menu or Desktop shortcut

![image](https://user-images.githubusercontent.com/12192/139441980-5d73579f-39dd-45da-916b-759eff99474d.png)

- A message like the above image will appear, so press the OK button to restart Cytoscape.
- and keep Cytoscape up and running.

## STEP 2: Install Google Chrome and the RCy3 package in Google Colab
- You will need to get a Google Account to use Colab
- You will need to install Google Chrome. Our Notebook does not work with Safari. Firefox may work, but I haven't confirmed it.
- Open this Google Colab link in Chrome while logged in to Google.
  - Simply run the following code cell

  ```
  devtools::install_github("cytoscape/RCy3")
  ```
- It takes a few minutes for install_github to finish.
  - Let's leave it alone and move onto the main workshop.

# Main workshop (primer)

## Self-introduction

Kozo Nishida, RIKEN, Japan
- A member of Bioconductor Community Advisory Board (CAB)
- Author of a Bioconductor package based on RCy3 (transomics2cytoscape)
- Cytoscape community contributor (Google Summer of Code, Google Season of Docs)
- Author of KEGGscape Cytoscape App

## What is Cytoscape?

![image](https://user-images.githubusercontent.com/12192/139426468-915e9a76-7e4e-4a37-aee9-3d0e344f551e.png)

- Open source, cross platform Java desktop GUI app.
- for network visualization.

### Core concepts

**Networks and Tables**: Network nodes and edges have annotation tables.

![image](https://user-images.githubusercontent.com/12192/139427094-bfd9a839-dabf-468d-8f28-6458443c8e61.png)

![image](https://user-images.githubusercontent.com/12192/139427149-4f0fe568-3851-4de6-834e-2e809e85f1be.png)

Color, shape, size, or ... according to the annotation table can be mapped to nodes and edges.

## Why do we need to automate?
Why automate Cytoscape when I could just use the GUI directly?

- For things you want to do multiple times, e.g., loops
- For things you want to repeat in the future
- For things you want to share with colleagues or publish
- For things you are already working on in R or Python, etc
  - To prepare data for collaborators

In short, for "reproducibility", "data sharing", "the use of R or Python".

## How can Cytoscape GUI operations be automated?

![image](https://user-images.githubusercontent.com/12192/139397677-80076550-e458-4bd4-9ab5-ba48ef6843b9.png)

- Cytoscape makes that possible with the REST API.
- Today Cytoscape is not only a Desktop application but also a REST server.
- You can check if Cytoscape is now working as a server with the command below.

  ```
  curl localhost:1234
  ```

- Now Cytoscape has REST API for almost every GUI operation.
  - RCy3 or py4cytoscape is R or Python wrapper of the REST API
  - py4cytoscape is Python clone of RCy3, py4cytoscape has same function specifications with RCy3
- Since table operations are essential for Bioinformatics, it is convenient to be able to operate them with R[dplyr] or Python[pandas].

[CyREST: Turbocharging Cytoscape Access for External Tools via a RESTful API. F1000Research 2015.](https://dx.doi.org/10.12688%2Ff1000research.6767.1)

[Cytoscape Automation: empowering workflow-based network analysis. Genome Biology 2019.](https://doi.org/10.1186/s13059-019-1758-4)

## Automation with RCy3

![image](https://user-images.githubusercontent.com/12192/139400142-8a2a764b-dbbe-4e47-9d3c-d4cc07602468.png)

[RCy3: Network biology using Cytoscape from within R. F1000Research 2019.](https://f1000research.com/articles/8-1774)

## Translating R data into a Cytoscape network using RCy3

Networks offer us a useful way to represent our biological data.
But how do we seamlessly translate our data from R into Cytoscape?

![image](https://user-images.githubusercontent.com/12192/139404069-536a67a2-e8fe-4072-bc42-74bfb060f924.png)

From here it finally becomes hands-on using Google Colab.
Aside from the details, let's connect Google Colab to local Cytoscape.

Make sure your local Cytoscape is fully up and running before running the code below.
It will take some time for Cytoscape to start up and its REST server to start up completely.
(Please wait for about 10 seconds.)

```{r}
library(RCy3)
browserClientJs <- getBrowserClientJs()
IRdisplay::display_javascript(browserClientJs)
cytoscapePing()
```

### Why was the remote Google Colab able to communicate with the local Cytoscape REST service?

We need a detailed description of what happened in

```
browserClientJs <- getBrowserClientJs()
IRdisplay::display_javascript(browserClientJs)
```

We used a technology called **Jupyter Bridge** in the above code.
Jupyter Bridge is a JavaScript implementation that makes HTTP requests from a remote REST client look like local requests.

![image](https://user-images.githubusercontent.com/12192/139530994-8afd99b2-1175-46b3-9ad7-166d8ba78f2a.png)

Since it is difficult to access Cytoscape in the desktop environment from a remote environment, we use Jupyter Bridge.

And since I couldn't get Jupyter Bridge to work in the Orchestra environment,
this workshop is exceptionally using Google Colab instead of Orchestra.

If you have RCy3 installed locally instead of remotely like Google Colab,
you don't need to use this Jupyter Bridge technology.

### (Then) Why use Jupyter Bridge?

- Users do not need to worry about dependencies and environment.
- Easily share notebook-based workflows and data sets
- Workflows can reside in the cloud, access cloud resources, and yet still use Cytoscape features.

## Let's go back to how to translate R data into a Cytoscape network...

Create a Cytoscape network from some basic R objects

```{r}
nodes <- data.frame(id=c("node 0","node 1","node 2","node 3"),
    group=c("A","A","B","B"), # categorical strings
    score=as.integer(c(20,10,15,5)), # integers
    stringsAsFactors=FALSE)
```

```{r}
edges <- data.frame(source=c("node 0","node 0","node 0","node 2"),
    target=c("node 1","node 2","node 3","node 3"),
    interaction=c("inhibits","interacts","activates","interacts"),  # optional
    weight=c(5.1,3.0,5.2,9.9), # numeric
    stringsAsFactors=FALSE)
```

#### Data frame used to create Network

![image](https://user-images.githubusercontent.com/12192/139534280-0c569dfd-d66d-4054-9b58-becce79225bc.png)

#### Create Network

```{r}
createNetworkFromDataFrames(nodes, edges, title="my first network", collection="DataFrame Example")
```

#### Export an image of the network

Remember.
All networks we make are created in Cytoscape so get an image of the resulting network and include it in your current analysis if desired.

```{r}
exportImage("my_first_network", type = "png")
```

Initial simple network

![image](https://user-images.githubusercontent.com/12192/139537190-1f79f871-5dbd-4779-9a4f-7c67f263101b.png)

# Main workshop (more practical)

## Example Use Case 1

**Omics data** - I have a ----------- fill in the blank (microarray, RNASeq, Proteomics, ATACseq, MicroRNA, GWAS …) dataset. I have normalized and scored my data. How do I overlay my data on existing interaction data?

## The example data set

We downloaded gene expression data from the Ovarian Serous Cystadenocarcinoma project of The Cancer Genome Atlas (TCGA)(International Genome et al.),
http://cancergenome.nih.gov via the Genomic Data Commons (GDC) portal(Grossman et al.) on 2017-06-14 using [TCGABiolinks Bioconductor package(Colaprico et al.)](http://bioconductor.org/packages/release/bioc/html/TCGAbiolinks.html).

- 300 samples available as RNA-seq data
- 79 classified as Immunoreactive, 72 classified as Mesenchymal, 69 classified as Differentiated, and 80 classified as Proliferative samples
- RNA-seq read counts were converted to CPM values and genes with CPM > 1 in at least 50 of the samples are retained for further study
- The data was normalized and differential expression was calculated for each cancer class relative to the rest of the samples.

We will use the following table as a result of the analysis to integrate it into a interaction network:

- [Gene ranks](https://cytoscape.org/cytoscape-tutorials/presentations/modules/RCy3_ExampleData/data/TCGA_OV_RNAseq_All_edgeR_scores.txt) - containing the p-values, FDR and foldchange values for the 4 comparisons (mesenchymal vs rest, differential vs rest, proliferative vs rest and immunoreactive vs rest)

```{r}
library(RCurl)
matrix <- getURL("https://raw.githubusercontent.com/cytoscape/cytoscape-tutorials/gh-pages/presentations/modules/RCy3_ExampleData/data/TCGA_OV_RNAseq_All_edgeR_scores.txt")
RNASeq_gene_scores <- read.table(text=matrix, header = TRUE, sep = "\t", quote="\"", stringsAsFactors = FALSE)
```

```{r}
RNASeq_gene_scores
```

Get a subset of genes of interest from the scored data:

```{r}
top_mesenchymal_genes <- RNASeq_gene_scores[which(RNASeq_gene_scores$FDR.mesen < 0.05 & RNASeq_gene_scores$logFC.mesen > 2),]
head(top_mesenchymal_genes)
```

## Use Case - How are my top genes related?

There are endless amounts of databases storing interaction data.

![image](https://user-images.githubusercontent.com/12192/139541346-9e223e88-e6df-4e4d-b7f2-a5836f6e97eb.png)

We are going to query the STRING Database to get all interactions found for our set of top Mesenchymal genes.

### Cytoscape Apps with network data
Thankfully we don't have to query each independently.
In addition to many specialized (for example, for specific molecules, interaction type, or species) interaction databases there are also databases that collate these databases to create a broad resource that is easier to use. For example:

- [stringApp](https://apps.cytoscape.org/apps/stringapp) - is a protein-protein and protein-chemical database that imports data from [STRING(Szklarczyk et al.)](https://doi.org/10.1093/nar/gkaa1074), [STITCH] into a unified, queriable database.

You can install apps in Cytoscape directly from R.

```{r}
installApp("stringApp")
```

You can check it was successfully installed with:

```{r}
getAppStatus("stringApp")
```

### Help on specific cytoscape command
To get information about an individual command from the R environment you can also use the commandsHelp function.
Simply specify what command you would like to get information on by adding its name to the command.
For example “commandsHelp("help string”)“

```{r}
commandsHelp("help")
```

```{r}
commandsHelp("help string")
```

```{r}
commandsHelp("help string protein query")
```

```{r}
mesen_string_interaction_cmd <- paste('string protein query taxonID=9606 limit=150 cutoff=0.9 query="',paste(top_mesenchymal_genes$Name, collapse=","),'"',sep="")
commandsGET(mesen_string_interaction_cmd)
```

- cutoff: The confidence score reflects the cumulated evidence that this interaction exists. Only interactions with scores greater than this cutoff will be returned.
- limit: The maximum number of proteins to return in addition to the query set.
- query: Comma separated list of protein names or identifiers.
- taxonID: The species taxonomy ID. See the NCBI taxonomy home page for IDs. If both species and taxonID are set to a different species, the taxonID has priority.

Please see Cytoscape menubar "Help -> Automation -> CyREST Commands API" for details.

```{r}
exportImage("initial_string_network", type = "png")
```

![initial_string_network](https://user-images.githubusercontent.com/12192/139543384-b19ac0f3-1dc9-4e6d-a212-af4f19ab6529.png)

### Layouts
Layout the network

```{r}
layoutNetwork('force-directed')
```

Check what other layout algorithms are available to try out

```{r}
getLayoutNames()
```

### Layouts - cont'd

Get the parameters for a specific layout

```{r}
getLayoutPropertyNames(layout.name='force-directed')
```

Re-layout the network using the force directed layout but specify some of the parameters

```{r}
layoutNetwork('force-directed defaultSpringCoefficient=0.0000008 defaultSpringLength=70')
```

Get a screenshot of the re-laid out network

```{r}
response <- exportImage("relayout_string_network", type = "png")
```

```{r}
response
```

String network with new layout

![relayout_string_network](https://user-images.githubusercontent.com/12192/139554068-55353d6a-62e2-4956-b2c6-cda4cc5a28d8.png)

## Overlay our expression analysis data on the STRING network

To do this we will be using the loadTableData function from RCy3.
It is important to make sure that that your identifiers types match up.
You can check what is used by STRING by pulling in the column names of the node attribute table.

```{r}
getTableColumnNames('node')
```

## Overlay our expression data on the String network - cont'd

If you are unsure of what each column is and want to further verify the column to use you can also pull in the entire node attribute table.

```{r}
node_attribute_table_topmesen <- getTableColumns(table="node")
head(node_attribute_table_topmesen[,3:7])
```

![image](https://user-images.githubusercontent.com/12192/139554236-dc19aaf7-8186-40f0-9ed3-d50a46c64d27.png)

The column “display name” contains HGNC gene names which are also found in our Ovarian Cancer dataset.

To import our expression data we will match our dataset to the “display name” node attribute.

```{r}
loadTableData(RNASeq_gene_scores, table.key.column = "display name", data.key.column = "Name")  #default data.frame key is row.names
```

## Visual Style

Modify the visual style Create your own visual style to visualize your expression data on the String network.
Start with a default style

**Formatted String network**

![mesen_string_network](https://user-images.githubusercontent.com/12192/139555494-3d46d339-153d-4c93-bf06-c393375ba2be.png)

