---
title: Proyecto
author: Manuel Azaid Ordaz Arias
date: 28/2/2021
output: html_document
---
```{r}
library("recount3")
human_projects <- available_projects()

<<<<<<< HEAD:R/Proyecto.Rmd
rse_gene_SRP078152 <- create_rse(
    subset(
        human_projects,
        project == "SRP078152" & project_type == "data_sources"
    )
)
assay(rse_gene_SRP078152, "counts") <- compute_read_counts(rse_gene_SRP078152)
rse_gene_SRP078152$sra.sample_attributes

```
Analizando los atributos puedo que todos maejan el mismo tipo de informacion por lo que puedo pasar a usar expand_sra_attributes() sin problemas

```{r}
rse_gene_SRP078152 <- expand_sra_attributes(rse_gene_SRP078152)
colData(rse_gene_SRP078152)[
    ,
    grepl("^sra_attribute", colnames(colData(rse_gene_SRP078152)))
]
```
Asignaremos el tipo formato correcto a los atributos que los necesiten
```{r}
## rse_gene_SRP078152_unfiltered <- rse_gene_SRP078152
## rse_gene_SRP078152 <- rse_gene_SRP078152_unfiltered
rse_gene_SRP078152$sra_attribute.donor <- factor(rse_gene_SRP078152$sra_attribute.donor)
rse_gene_SRP078152$sra_attribute.treatment <- factor(rse_gene_SRP078152$sra_attribute.treatment)
rse_gene_SRP078152$`sra_attribute.hours_post-infection` <- as.numeric(rse_gene_SRP078152$`sra_attribute.hours_post-infection`)
summary(as.data.frame(colData(rse_gene_SRP078152)[
    ,
    grepl("^sra_attribute.[donor|treatment]", colnames(colData(rse_gene_SRP078152)))
]))
unique(rse_gene_SRP078152$`sra_attribute.hours_post-infection`)
```
Utilzaremo el tiempo transcurrido 12 o 24 pero primero cambiare el nombre de la columna para ingresar mas facil a ella
```{r}
rse_gene_SRP078152$hours <- factor(ifelse(rse_gene_SRP078152$`sra_attribute.hours_post-infection` <= 24, 24, 48))
table(rse_gene_SRP078152$hours)
```
Analizaremos la calidad de las muestras pra determinar si dejarlas o limpiarlas
```{r}
rse_gene_SRP078152$assigned_gene_prop <- rse_gene_SRP078152$recount_qc.gene_fc_count_all.assigned / rse_gene_SRP078152$recount_qc.gene_fc_count_all.total
summary(rse_gene_SRP078152$assigned_gene_prop)
```
Viendo que la mayoria de muestras tienen una calidad mala usare como punto de corte la mediana
```{r}
# Guardamos por si cambiara de opinion
rse_gene_SRP078152_unfiltered <- rse_gene_SRP078152
hist(rse_gene_SRP078152$assigned_gene_prop)
```
```{r}
table(rse_gene_SRP078152$assigned_gene_prop < 0.35)
```
```{r}
rse_gene_SRP078152 <- rse_gene_SRP078152[, rse_gene_SRP078152$assigned_gene_prop > 0.35]
```

Ahora que ya limpiamos las muestra, procedere a limpiar los genes
```{r}
gene_means <- rowMeans(assay(rse_gene_SRP078152, "counts"))
summary(gene_means)
```
Viendo que incluso en mi primer quantile tengo una expresion genetica muy mala, de 0, usare la meiana para determinarla como umbral para eliminar los genes
```{r}
rse_gene_SRP078152 <- rse_gene_SRP078152[gene_means > 0.3, ]
dim(rse_gene_SRP078152)
```
```{r}
round(nrow(rse_gene_SRP078152) / nrow(rse_gene_SRP078152_unfiltered) * 100, 2)
```
# Ahora normalizare los datos
```{r}
library("edgeR")
dge <- DGEList(
    counts = assay(rse_gene_SRP078152, "counts"),
    genes = rowData(rse_gene_SRP078152)
)
dge <- calcNormFactors(dge)
```

# Expresion diferencial
Pero primero haremos el modelo estadistico
```{r}
library("ggplot2")
ggplot(as.data.frame(colData(rse_gene_SRP078152)), aes(y = assigned_gene_prop, x = hours)) +
    geom_boxplot() +
    theme_bw(base_size = 20) +
    ylab("Assigned Gene Prop") +
    xlab("Hours Post Infection Group")
```
Ya que tengo el modelo estadistico puedo proceder a usar limma para el estudio de expresion diferencial
```{r}
app <- ExploreModelMatrix::ExploreModelMatrix(
  sampleData = colData(rse_gene_SRP078152)[, c("hours", "assigned_gene_prop",
                                           "sra_attribute.treatment"
           )],
  designFormula = ~ hours + assigned_gene_prop + sra_attribute.treatment
)
if (interactive()) shiny::runApp(app)
```

```{r}
mod <- model.matrix( ~ hours + assigned_gene_prop + sra_attribute.treatment,
                    data = colData(rse_gene_SRP078152))
colnames(mod)
```
```{r}
library("limma")
vGene <- voom(dge, mod, plot = TRUE)
```
# Contrastando primero con el de 24 hrs
```{r}
eb_results <- eBayes(lmFit(vGene))
=======
# Proyecto

\`\`\`{r setup, include=FALSE} knitr::opts\_chunk$set\(echo = TRUE\)

```text
## R Markdown
>>>>>>> cb8d1de45bc1be70cc7d26897c7611ba0acd219c:r/proyecto.md

de_results <- topTable(
    eb_results,
    coef = 2,
    number = nrow(rse_gene_SRP078152),
    sort.by = "none"
)
dim(de_results)
```
```{r}
head(de_results)
```

```{r}
table(de_results$adj.P.Val < 0.05)
```
```{r}
round(table(de_results$adj.P.Val < 0.05)[2] / length(de_results$adj.P.Val) * 100, 2)
```

```{r}
volcanoplot(eb_results, coef = 2, highlight = 5, names = de_results$gene_name)
```

# Visualizando


<<<<<<< HEAD:R/Proyecto.Rmd
```{r}
i <- which.min(de_results$adj.P.Val)
title <- paste("Expresion del gene top",de_results$gene_name[i])
## Or we can use ggplot2
## First we need to build a temporary data frame with
## the data for ggplot2
df_temp <- data.frame(
    Expression = vGene$E[i,],
    Hours = rse_gene_SRP078152$hours,
    Treatment = rse_gene_SRP078152$sra_attribute.treatment
)
## Next we can make the boxplot, we'll use "fill" to color
## the boxes by the primary diagnosis variable
ggplot(df_temp, aes(y = Expression, x = Hours, fill = Treatment)) +
    ggtitle(title) + 
    geom_boxplot() +
    theme_dark(base_size = 20)
```
Extrayendo el top 30 de expresion de mis genes
```{r}
library("pheatmap")

exprs_heatmap <- vGene$E[rank(de_results$adj.P.Val) <= 30, ]
df <- as.data.frame(colData(rse_gene_SRP078152)[, c("hours", "sra_attribute.treatment", "assigned_gene_prop")])
colnames(df) <- c("Hours Post Infection", "Treatment", "Assigned Genes")
rownames(exprs_heatmap) <- rowRanges(rse_gene_SRP078152)$gene_name[
    match(rownames(exprs_heatmap), rowRanges(rse_gene_SRP078152)$gene_id)
]
pheatmap(
    exprs_heatmap,
    cluster_rows = TRUE,
    cluster_cols = TRUE,
    show_rownames = TRUE,
    show_colnames = FALSE,
    annotation_col = df
)
```
```{r}
## Para colores
library("RColorBrewer")

## Conviertiendo los grupos de edad a colores
col.group <- df$`Hours Post Infection` 
levels(col.group) <- brewer.pal(nlevels(col.group), "Set1")
col.group <- as.character(col.group)

## MDS por grupos de Tratamiento
plotMDS(vGene$E, labels = df$Treatment, col = col.group)
```



=======
\`\`\`{r pressure, echo=FALSE} plot\(pressure\)

\`\`\`

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.

>>>>>>> cb8d1de45bc1be70cc7d26897c7611ba0acd219c:r/proyecto.md