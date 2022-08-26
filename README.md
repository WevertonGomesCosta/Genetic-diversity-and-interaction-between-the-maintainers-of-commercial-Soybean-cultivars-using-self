# Genetic diversity and interaction between the maintainers of commercial Soybean cultivars using self-organizing maps

## Loading Libraries

```{r message=FALSE, warning=FALSE}
library("FactoMineR")
library("factoextra") #Correspondence Analysis
library(readxl) #Read data files in xlsx
library(randomForest) #Random Forest
library(tidyverse)
require(magrittr)
require(reshape2)
library(ggforce) #Data and graphics manipulation
library("kohonen") #Kohonen SOM
library(RColorBrewer) #Color palette
require("ggrepel")
library(gridExtra) #Graphs of Kohonen Analysis
```

## Defining colors

```{r}
qual_col_pals = brewer.pal.info[brewer.pal.info$category == 'qual', ]
col_vector = unlist(mapply(brewer.pal, qual_col_pals$maxcolors, rownames(qual_col_pals)))
```

# Reading the data

```{r}
data <- read_excel("data/data.xlsx")
```

## Data cleaning

```{r}
lower5 <-   data %>%
  group_by(Maintainer_coded) %>%
  count() %>%
  filter(n <= 5)

data <- data[!data$Maintainer_coded %in%
               lower5$Maintainer_coded,]
```

## Transforming variables into factors

```{r}
for (i in 1:ncol(data)) {
  data[[i]] = as.factor(data[[i]])
}
```

# Traits Importance - Random Forest

## Data manipulation

```{r}
data = data[, -c(1, 2, 5)] #Excluding Name, Maintainer and Year variables
```

## Processing

```{r}
ntraits <- (ncol(data) - 1) / 3 #Number of traits divided by 3
ntree <- 500 #Number of trees

rf1 <- randomForest(
  data$Maintainer_coded ~ .,
  data = data,
  mtry = ntraits,
  importance = TRUE,
  proximity = TRUE,
  ntree = ntree,
  na.action = na.roughfix
)
```

## Importance of traits based on accuracy index and Gine index

```{r}
imp <- as.data.frame(varImpPlot(rf1))
imp$traitnames <- rownames(imp)
imp_trait <- melt(imp, id.var = "traitnames")
imp_trait$traitnames <- as.factor(imp_trait$traitnames)
levels(imp_trait$traitnames) <- c(
  "Event Type",
  "Flower Color",
  "Growth Type",
  "Hilo Color",
  "Hypocotyl",
  "Legum Color",
  "Peroxidase Reaction",
  "Pubescence Color",
  "Pubescence Density",
  "Seed Brightness",
  "Seed Shape",
  "Tegument Color"
)
```

### Plot importance of traits

```{r}

imp_trait %>%
  mutate(traitnames = fct_reorder(traitnames, value)) %>%
  ggplot(aes(x = traitnames, y = value)) +
  geom_point(aes(colour = variable, shape = variable), size = 4) +
  scale_colour_discrete(labels = c("Accuracy", "Gini Indice")) +
  scale_shape_discrete(labels = c("Accuracy", "Gini Indice")) +
  geom_segment(aes(
    x = traitnames,
    xend = traitnames,
    y = 0,
    yend = value
  )) +
  ylab("Increase in node purity") +
  xlab("") +
  theme_minimal() +
  theme(
    legend.position = c(0.8, 0.15),
    legend.title = element_blank(),
    legend.background = element_rect(fill = "white", colour = "black"),
    legend.key.size = unit(0.5, "cm"),
    axis.text.y = element_text(
      angle = 15,
      vjust = 0.5,
      hjust = 1
    )
  ) +
  coord_flip()

```

## Excluding the 4 least important traits by RF

```{r}
x <- imp_trait %>% group_by(traitnames) %>%
  summarise(value = mean(value)) %>%
  slice_min(value, n = 4, with_ties = FALSE) %>%
  ungroup()

colnames(data) <- c(
  "Maintainer_coded",
  "Event Type",
  "Flower Color",
  "Legum Color",
  "Seed Shape",
  "Tegument Color",
  "Hypocotyl",
  "Pubescence Color",
  "Pubescence Density",
  "Hilo Color",
  "Peroxidase Reaction",
  "Growth Type",
  "Seed Brightness"
)

data <-
  data  %>% select(!any_of(x$traitnames)) #tegument color, flower color, peroxidase reaction, and peroxidase

```

# Multiple Correspondence Analysis

```{r}
res.mca <- MCA(data,
               quali.sup = 1,
               graph = FALSE,
               ncp = 54)

```

## Eigenvalues

```{r}
eig.val <- get_eigenvalue(res.mca)
eig.val

eig.val <-
  as.data.frame(eig.val) #creating data frame with eigenvalues
eig.val$Dimension <-
  as.numeric(str_replace(rownames(eig.val), "Dim.", ""))
```

## Graph of the percentage of explanation of the variance accumulated by the dimensions

```{r}
ggplot(data = eig.val, aes(x = Dimension, y = (cumulative.variance.percent))) +
  geom_bar(stat = "identity", fill = "#00AFBB", alpha = 0.5) +
  geom_point(colour = "#FC4E07") +
  geom_line(colour = "#FC4E07") +
  scale_x_continuous(breaks = c(1:nrow(eig.val)), expand = c(0.01, 0)) +
  scale_y_continuous(expand = expansion(mult = c(0, .05))) +
  theme_bw() +
  labs(x = "Dimensions",
       y = "Cumulative variance percent")

```

# Grouping and organizing via Kohonen's self-organizing maps

```{r}
# Network topology and parameters
map_dimension = 4

n_iterations = 2000

recalculate_map = T

recalculate_no_clusters = T

#Kohonen SOM
som_grid = kohonen::somgrid(
  xdim = map_dimension,
  ydim = map_dimension,
  topo = "hexagonal",
  toroidal = FALSE
)

b <- res.mca[["quali.sup"]][["coord"]] #COORDINATES OF THE MAINTAINERS IN THE 54 DIMENSIONS
```

## Processing

```{r}
set.seed(123)                                      #Seed for reproduction of results

m = kohonen::supersom(
  scale(b),
  grid = som_grid,
  rlen = n_iterations,
  alpha = c(0.05, 0.01),
  dist.fcts = 'euclidean'
)
```

## Interaction Graph

```{r}
progress <- as.data.frame(cbind(m$changes, 1:nrow(m$changes)))
colnames(progress) <- c("Mean distance to closest unit", "Iteration")

ggplot(progress, aes(Iteration, progress[[1]])) +
  geom_line(color = "#00AFBB", size = 1) +
  stat_smooth(color = "#FC4E07",
              fill = "#FC4E07",
              method = "loess") +
  theme_minimal() +
  labs(y = colnames(progress[1]))

```

## Trait Frequency

```{r}
grouping <- as_tibble(cbind(m$unit.classif, rownames(b)))
colnames(grouping) <- c("group", "Maintainer_coded")
grouping$group <- as.factor(grouping$group)
grouping$Maintainer_coded <- as.factor(grouping$Maintainer_coded)
data <- data %>% left_join(grouping)
```

## Distance graph between neighboring neurons

```{r}
som_coord <- m[[4]]$pts %>%
  as_tibble %>%
  mutate(group = as.integer(row_number()))

som_pts <- tibble(group = as.integer(m[[2]]),
                  dist = m[[3]])

ndist <- unit.distances(m$grid)
cddist <- as.matrix(object.distances(m, type = "codes"))
cddist[abs(ndist - 1) > .001] <- NA
neigh.dists <- colMeans(cddist, na.rm = TRUE)
som_coord <- som_coord %>% mutate(dist = neigh.dists)

neigdists <- som_coord %>%
  ggplot(aes(x0 = x, y0 = y)) +
  geom_regon(aes(
    r = 0.5,
    angle = 11,
    sides = 6,
    fill = dist
  ),
  expand = unit(0.1, 'cm')) +
  scale_fill_gradient(low = "red", high = "yellow", name = "Distance") +
  theme(
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    panel.grid = element_blank(),
    axis.text = element_blank(),
    axis.title = element_blank(),
    legend.position = "bottom"
  ) +
  geom_text_repel(
    data = som_coord,
    aes(x = x, y = y, label = group),
    hjust        = 0.5,
    force = 0,
    color = "black",
    show.legend = FALSE,
    segment.color = NA
  )

neigdists
```

## Graph distribution of maintainers in clusters

```{r}
p <- som_coord %>%
  ggplot(aes(x0 = x, y0 = y)) +
  geom_regon(aes(
    r = 0.5,
    angle = 11,
    sides = 6,
    fill = factor(group)
  ),
  expand = unit(0.1, 'cm')) +
  theme(
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    panel.grid = element_blank(),
    axis.text = element_blank(),
    axis.title = element_blank(),
    legend.position = "none"
  ) +
  scale_fill_manual(values = col_vector[40:56])

plotdata <- data %>%
  group_by(group, Maintainer_coded) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)

MC <- p +
  geom_label_repel(
    data = plotdata,
    aes(x = x, y = y, label = Maintainer_coded),
    max.overlaps = Inf,
    box.padding = 0.1,
    hjust        = 0.5,
    direction    = "y",
    color = "black",
    show.legend = FALSE,
    segment.color = NA
  ) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank())
MC
  
```


## Graphs Distribution of the categories of traits by cluster

```{r}
p <- som_coord %>%
  ggplot(aes(x0 = x, y0 = y)) +
  geom_regon(
    aes(r = 0.5, angle = 11, sides = 6),
    expand = unit(0.1, 'cm'),
    alpha = 0,
    color = "black"
  ) +
  theme(
    panel.background = element_blank(),
    axis.ticks = element_blank(),
    panel.grid = element_blank(),
    axis.text = element_blank(),
    axis.title = element_blank(),
    legend.position = "bottom"
  )

plotdata <- data %>%
  group_by(group, `Event Type`) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$`Event Type`) <-
  c(
    "Conventional",
    "CV127",
    "Intacta™ Roundup Ready™ 2 Pro",
    "Liberty Link",
    "Roundup Ready"
  )

EV <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = `Event Type`
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[1:5]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
EV

plotdata <- data %>%
  group_by(group, `Legum Color`) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$`Legum Color`) <-
  c(
    "Brown",
    "Clear",
    "Dark Brown",
    "Dark Gray",
    "Gray",
    "Light Brown",
    "Light Gray",
    "Medium Brown"
  )

CL <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = `Legum Color`
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[6:13]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
CL

plotdata <- data %>%
  group_by(group, `Seed Shape`) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$`Seed Shape`) <- c(
  "Elongated flat",
  "Elongated spherical",
  "Spherical flat",
  "Ovulated spherical",
  "Round",
  "Rounded",
  "Spherical",
  "Elongated"
)

FS <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = `Seed Shape`
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[14:21]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
FS

plotdata <- data %>%
  group_by(group, Hypocotyl) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$Hypocotyl) <- c(
  "Ausente",
  "Ausente/green",
  "Presente",
  "Presente/green",
  "Presente/white",
  "Presente/bronze" ,
  "Presente/purple"
)

HI <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = Hypocotyl
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[22:28]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
HI

plotdata <- data %>%
  group_by(group, `Pubescence Color`) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$`Pubescence Color`) <-
  c("Average",
    "Brown",
    "Dark Brown",
    "Gray",
    "Light Brown",
    "Light Gray",
    "Medium Brown")

CP <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = `Pubescence Color`
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[29:35]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
CP

plotdata <- data %>%
  group_by(group, `Hilo Color`) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$`Hilo Color`) <-
  c(
    "Black",
    "Brown",
    "Dark Brown",
    "Faded Black",
    "Gray",
    "Imperfect Black",
    "Light Brown",
    "Medium Brown",
    "Yellow"
  )

CH <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = `Hilo Color`
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[36:44]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
CH

plotdata <- data %>%
  group_by(group, `Growth Type`) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$`Growth Type`) <- c("Determined",
                                    "Undetermined",
                                    "Semi-determined")

TC <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = `Growth Type`
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[45:47]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
TC

plotdata <- data %>%
  group_by(group, `Seed Brightness`) %>%
  summarize(n = n()) %>%
  mutate(pct = (n / sum(n)),
         lbl = scales::percent(pct)) %>%
  ungroup()

plotdata$group <- as.integer(levels(plotdata$group))[plotdata$group]
plotdata <- plotdata %>% left_join(som_coord, by = "group")
plotdata <- na.omit(plotdata)
levels(plotdata$`Seed Brightness`) <-
  c("Average",
    "Bright",
    "Shiny",
    "High",
    "Low",
    "Low/Average",
    "Matte" ,
    "Opaque")

BS <- p +
  geom_arc_bar(
    data = plotdata,
    aes(
      x0 = x,
      y0 = y,
      r0 = 0,
      r = 0.4,
      amount = n,
      fill = `Seed Brightness`
    ),
    stat = 'pie',
    alpha = 0.8
  ) +
  scale_fill_manual(values = col_vector[48:55]) +
  geom_text(
    data = plotdata,
    aes(
      x = x,
      y = (y - 0.45),
      label = group
    ),
    color = "black",
    show.legend = FALSE
  ) +
  theme(plot.title = element_blank(), legend.title = element_blank())
BS

```

# Graphs of productivity according to event and growth type

```{r}
yield <- read_excel("data/yield.xlsx")
data <- read_excel("data/data.xlsx")

#Transforming variables into factors
yield$Year <- as.factor(yield$Year)
for (i in 1:ncol(data)) {
  data[[i]] = as.factor(data[[i]])
}

# Count graph relating event to year
levels(data$Event) <-
  c(
    "Conventional",
    "CV127",
    "Intacta™ Roundup Ready™ 2 Pro",
    "Liberty Link",
    "Roundup Ready"
  )

ggplot() +
  geom_bar(data = data, aes(Year, fill = Event)) +
  geom_point(data = yield,
             aes(x = Year, y = (Yield * (175 / 125000))),
             group = 1,
             color = "#a50026") +
  geom_line(
    data = yield,
    aes(x = Year, y = Yield * (175 / 125000)),
    group = 1,
    color = "#a50026"
  ) +
  scale_y_continuous(name = "Count",
                     sec.axis = sec_axis( ~ . * 125000 / 175, name = "Yield (Thousand Tons)")) +
  theme_bw() +
  xlab("Year") +
  scale_fill_brewer(palette = "Dark2") +
  labs(fill = "Event Type") +
  theme(
    legend.position = c(0.125, 0.85),
    legend.background = element_rect(fill = "white", colour = "black")
  )

# Count graph relating Growth type to year

levels(data$Growth_type) <- c("Determined",
                              "Undetermined",
                              "Semi-determined")

data %>%
  filter(Growth_type != "NA") %>%
  ggplot() +
  geom_bar(aes(Year, fill = Growth_type)) +
  theme_bw() +
  xlab("Year") +
  ylab("Count") +
  scale_fill_brewer(palette = "Set1") +
  labs(fill = "Growth type") +
  theme(
    legend.position = c(0.125, 0.85),
    legend.background = element_rect(fill = "white", colour = "black")
  )
```

