Biodiversity is the Way, Don’t Let Nature Go Astray
================
Kristin Liu, Briana Huynh

## Final Project

*Proposal:*

We hope to obtain data via DOI and analyze a research paper about
biodiversity loss for multiple species over a certain length of time.

-   We would like to replicate results of the existing study that was
    done in the research paper.  
-   We aim to create our own graph to compare trends between different
    species.  
-   Analyze how biodiversity is measured. What are possible causes of
    biodiversity loss and how do they relate to the trends in the graphs
    we created? What are the future effects of the loss of species
    diversity?

## Background

The graph below is from an article written by Ripple and colleagues
titled “Are we eating the world’s megafauna to extinction?”. Megafauna
extinction risk and trends are shown in percentages of species
classified as threatened by the IUCN Red List Category Vulnerable,
Endangered, or Critically Endangered and with decreasing population
trend. Ripple and colleagues obtained their data from the Amniote
database for mammals, reptiles, and birds (Myhrvold et al., 2015). The
complete data sets from Myhrvold et al. are located in the Ecological
Archives at <http://esapubs.org/archive> with accession number E096-269.

We hope to replicate these results for mammals, birds, and reptiles, as
well as perform our own further analyses with the data.

![](https://conbio.onlinelibrary.wiley.com/cms/asset/c5172cfc-2ad7-4881-bc30-2f41bb250cab/conl12627-fig-0001-m.png)

## Additional References

-   [Ripple et al (2019)](https://doi.org/10.1111/conl.12627)
-   [Myhrvold et al(2015)](https://doi.org/10.1890/15-0846R.1)

## We first replicate the results from Ripple and colleagues, as shown in the background info.

#### Importing data; filtering for megafauna and count the number of megafauna in each class.

``` r
suppressPackageStartupMessages({
  library("dplyr")
  library("ggplot2")
  library("jsonlite")
  library("tidyverse")
  library("httr")
  library("tidyr")
})
```

``` r
species_data <- read.csv("https://esapubs.org/archive/ecol/E096/269/Data_Files/Amniote_Database_Aug_2015.csv")

megafauna <- species_data %>%
  filter((class == "Mammalia" & adult_body_mass_g > 100000) | 
           (class %in% c("Aves", "Reptilia") & adult_body_mass_g > 40000))

megafauna_count <- megafauna %>%
  count(class) %>%
  left_join(megafauna, by = "class") %>%
  rename("total_megafauna" = n) %>%
  unite(scientific_name, genus:species, sep = " ", remove = TRUE, na.rm = FALSE) %>%
  select(class, total_megafauna, scientific_name, adult_body_mass_g) %>%
  distinct(scientific_name, .keep_all = TRUE)
head(megafauna_count)
```

    ##      class total_megafauna            scientific_name adult_body_mass_g
    ## 1     Aves               4        Casuarius casuarius           44000.0
    ## 2     Aves               4 Casuarius unappendiculatus           47300.0
    ## 3     Aves               4           Struthio camelus          109250.0
    ## 4     Aves               4     Struthio molybdophanes          109250.0
    ## 5 Mammalia             177      Alcelaphus buselaphus          159000.0
    ## 6 Mammalia             177           Alcelaphus caama          159968.9

#### Importing data from the IUCN red list api and making a data frame for all species.

``` r
base <- "https://apiv3.iucnredlist.org/api/v3/species/"
page <- "page/"
page_number <- 0:14
query <- "?token="
token <- "9bb4facb6d23f48efbf424bb05c0c1ef1cf6f468393bc745d42179ac4aca5fee"
  
urls <- paste0(base, page, page_number, query, token)

if(!file.exists("species_request.rds")){
  species_request <- map(urls, GET)
  saveRDS(species_request, "species_request.rds")
}

species_request <- readRDS("species_request.rds")
```

``` r
contents <- map(species_request, content, as = "text")
species <- map(contents, fromJSON)
species_df <- map_dfr(species, "result")
```

#### Filtering IUCN species data for mammals, birds, and reptiles; counting the number of species in each class.

``` r
vertebrates <- species_df %>%
  distinct(scientific_name, .keep_all = TRUE) %>%
  filter(class_name %in% c("AVES", "REPTILIA", "MAMMALIA"))

vertebrates_count <- vertebrates %>%
  count(class_name) %>% 
  left_join(vertebrates, by = "class_name") %>%
  rename("total_vertebrates" = n) %>%
  select(taxonid, class_name, total_vertebrates, scientific_name, category)
head(vertebrates_count)
```

    ##    taxonid class_name total_vertebrates            scientific_name category
    ## 1 22678073       AVES             11158             Rhea americana       NT
    ## 2 22678108       AVES             11158        Casuarius casuarius       LC
    ## 3 22678111       AVES             11158         Casuarius bennetti       LC
    ## 4 22678114       AVES             11158 Casuarius unappendiculatus       LC
    ## 5 22678117       AVES             11158   Dromaius novaehollandiae       LC
    ## 6 22678122       AVES             11158          Apteryx australis       VU

#### Filtering, counting, and finding percentages of threatened megafauna/vertebrates.

``` r
vertebrates_filtered <- vertebrates_count %>%
  filter(category %in% c("CR", "EN", "VU")) 

vertebrates_threatened_percentage <- vertebrates_filtered %>%
  count(class_name) %>%
  left_join(vertebrates_filtered, by = "class_name") %>%
  rename("threatened_vertebrates" = n) %>%
  mutate("percentage_threatened" = threatened_vertebrates / total_vertebrates) %>%
  select(class_name, threatened_vertebrates, total_vertebrates, 
         percentage_threatened, scientific_name, category) %>%
  mutate("classification" = "all vertebrates") %>%
  rename("class" = class_name) %>%
  distinct(scientific_name, .keep_all = TRUE)
head(vertebrates_threatened_percentage)
```

    ##   class threatened_vertebrates total_vertebrates percentage_threatened
    ## 1  AVES                   1481             11158             0.1327299
    ## 2  AVES                   1481             11158             0.1327299
    ## 3  AVES                   1481             11158             0.1327299
    ## 4  AVES                   1481             11158             0.1327299
    ## 5  AVES                   1481             11158             0.1327299
    ## 6  AVES                   1481             11158             0.1327299
    ##            scientific_name category  classification
    ## 1        Apteryx australis       VU all vertebrates
    ## 2          Apteryx haastii       VU all vertebrates
    ## 3              Tinamus tao       VU all vertebrates
    ## 4          Tinamus osgoodi       VU all vertebrates
    ## 5     Crypturellus kerriae       VU all vertebrates
    ## 6 Nothoprocta taczanowskii       VU all vertebrates

``` r
megafauna_filtered <- megafauna_count %>%
  left_join(species_df, by = "scientific_name") %>%
  filter(category %in% c("CR", "EN", "VU"))

megafauna_threatened_percentage <- megafauna_filtered %>%
  count(class) %>%
  left_join(megafauna_filtered, by = "class") %>%
  rename("threatened_megafauna" = n) %>%
  mutate("percentage_threatened" = threatened_megafauna / total_megafauna) %>%
  select(class, threatened_megafauna, total_megafauna, 
         percentage_threatened, scientific_name, category) %>%
  mutate(class = toupper(class)) %>%
  mutate("classification" = "megafauna") %>%
  distinct(scientific_name, .keep_all = TRUE)
head(megafauna_threatened_percentage)
```

    ##      class threatened_megafauna total_megafauna percentage_threatened
    ## 1     AVES                    1               4             0.2500000
    ## 2 MAMMALIA                   82             177             0.4632768
    ## 3 MAMMALIA                   82             177             0.4632768
    ## 4 MAMMALIA                   82             177             0.4632768
    ## 5 MAMMALIA                   82             177             0.4632768
    ## 6 MAMMALIA                   82             177             0.4632768
    ##          scientific_name category classification
    ## 1 Struthio molybdophanes       VU      megafauna
    ## 2      Beatragus hunteri       CR      megafauna
    ## 3          Bos javanicus       EN      megafauna
    ## 4            Bos sauveli       CR      megafauna
    ## 5 Bubalus depressicornis       EN      megafauna
    ## 6    Bubalus mindorensis       CR      megafauna

#### Combining data for threatened megafauna and vertebrates data to be used to make a plot.

``` r
combined_percent <- bind_rows(megafauna_threatened_percentage, vertebrates_threatened_percentage)
head(combined_percent)
```

    ##      class threatened_megafauna total_megafauna percentage_threatened
    ## 1     AVES                    1               4             0.2500000
    ## 2 MAMMALIA                   82             177             0.4632768
    ## 3 MAMMALIA                   82             177             0.4632768
    ## 4 MAMMALIA                   82             177             0.4632768
    ## 5 MAMMALIA                   82             177             0.4632768
    ## 6 MAMMALIA                   82             177             0.4632768
    ##          scientific_name category classification threatened_vertebrates
    ## 1 Struthio molybdophanes       VU      megafauna                     NA
    ## 2      Beatragus hunteri       CR      megafauna                     NA
    ## 3          Bos javanicus       EN      megafauna                     NA
    ## 4            Bos sauveli       CR      megafauna                     NA
    ## 5 Bubalus depressicornis       EN      megafauna                     NA
    ## 6    Bubalus mindorensis       CR      megafauna                     NA
    ##   total_vertebrates
    ## 1                NA
    ## 2                NA
    ## 3                NA
    ## 4                NA
    ## 5                NA
    ## 6                NA

#### Plotting threatened vertebrates and megafauna to replicate graph Ripple and colleagues’ “Are we eating the world’s megafauna to extinction?”

``` r
ggplot(combined_percent, mapping = aes(x = class, y = percentage_threatened, fill = classification, 
                                       label = scales::percent(percentage_threatened))) + 
  geom_bar(stat = "identity", position = "dodge") +
  geom_text(position = position_dodge(width = .9), vjust = -0.5, size = 3) +
  labs(title = "Percentages of Threatened Megafauna and Vertebrate Species", 
       x = "Class", y = "Percentage Threatened")
```

![](Final-Project_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

## We next create our own original graph to compare trends between megafauna.

#### Extracting population trends from the IUCN red list api.

``` r
base2 <- "https://apiv3.iucnredlist.org/api/v3/species/narrative/"
sci_name <- megafauna_threatened_percentage$scientific_name
token2 <- "?token=9bb4facb6d23f48efbf424bb05c0c1ef1cf6f468393bc745d42179ac4aca5fee"
url2 <- paste0(base2, sci_name, token2)

if(!file.exists("population_trend.rds")){
  population_trend <- map(url2, GET)
  saveRDS(population_trend, "population_trend.rds")
}

population_trend <- readRDS("population_trend.rds")
```

#### Sorting by class and counting the number of species for each population trend.

``` r
text_population_trend <- map(population_trend, content, as = "text")
json <- map_dfr(text_population_trend, fromJSON)

population_trend_data <- json %>% 
  left_join(megafauna_threatened_percentage, by = c("name" = "scientific_name")) %>%
  mutate(trend = json$result$populationtrend) %>%
  count(trend, class)
head(population_trend_data)
```

    ## # A tibble: 6 × 3
    ##   trend      class        n
    ##   <chr>      <chr>    <int>
    ## 1 decreasing AVES         1
    ## 2 decreasing MAMMALIA    41
    ## 3 decreasing REPTILIA     9
    ## 4 increasing MAMMALIA    10
    ## 5 increasing REPTILIA     6
    ## 6 stable     MAMMALIA     5

#### Plotting our sorted data for threatened megafauna.

``` r
population_trend_graph <- 
  ggplot(data = population_trend_data, aes(x = n, y = trend, fill = class)) +
  geom_bar(stat = "identity", position = "dodge") +
  labs(title = "Population Trends in Threatened Megafauna", 
       x = "Number of Species", y = "Trend")
population_trend_graph
```

![](Final-Project_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

## Explanation of Replicated and Original Results

Ripple and colleagues considered megafauna as mammalian species with a
body mass of over 100 kg, and bird or reptile species with a body mass
of over 40 kg. Our replicated plot is almost identical to the plot from
the original article. However, we have a more updated version of the
data from IUCN in comparison to the data from the study, hence any
slight deviations in our data from the study data. We calculated the
percentages of megafauna and all vertebrates that were categorized as
vulnerable, endangered, or critically endangered by the IUCN red list
and grouped the species by class for the following classes: ray-finned
fish (Actinopterygii), cartilaginous fish (Chondrichthyes), birds,
mammals, reptiles, and amphibians. Other minor fish classes contained no
species with masses ≥100 kg and thus were only included in the results
for all vertebrates together. We determined the percentages of species
threatened and decreasing for species classified as megafauna and for
all vertebrates with available data. We can see in the replicated graph
that megafauna species are more threatened and have a relatively higher
percentage of decreasing populations than all vertebrates together,
especially in reptiles and mammals.

For our original graph, we extracted the population trends from the
species narratives of the IUCN red list api and combined them with our
threatened megafauna data. We then sorted the data by class and
population trend, and counted the number of species for each trend. Some
of the species had unknown or non-applicable population trends, but we
can see clearly that a majority of threatened megafauna mammals have a
decreasing population. The only threatened megafauna Aves species also
fell into the decreasing population category compared to the other
trends, and there were more threatened megafauna reptiles that are
decreasing in population than increasing or stable.

## Analysis of Biodiversity Loss: Analyze how biodiversity is measured. What are possible causes of biodiversity loss and how do they relate to the trends in the graphs we created? What are the future effects of the loss of species diversity?

Ripple and colleagues suggest that threats to large species may be due
to a combination of hunting and habitat degradation; excessive hunting
occurs for the acquisition of meat or for body parts such as skins,
horns, organs, and antlers for traditional medicine or trophies. To
assess this, we used species lists from IUCN to estimate biodiversity
loss. As shown in our data, megafauna populations are more likely to be
threatened compared to their smaller counterparts. Most megafauna tend
to be K-selected species and are extremely vulnerable to fishing,
trapping, and hunting pressures (Johnson, 2002). The megafauna are
especially targeted given their size, as hunting these larger animals
will produce more profit when being sold, as compared to non-megafauna,
smaller animals. Decreasing population trends, especially in mammals,
are due to slower reproduction rates that are unable to compete with the
rate of hunting and habitat loss. In addition to intentional harvesting,
many megafauna species are possibly simultaneously affected by various
types of habitat degradation from anthropogenic activities, such as
agriculture and deforestation. When taken together, these threats can
have major negative cumulative effects on vertebrate species (Betts et
al., 2017; Shackelford, Standish, Ripple, & Starzomski, 2018).
Consistent with our results, overexploitation and habitat loss are
considered major twin threats to biodiversity in general (Maxwell et
al., 2016). Biodiversity loss negatively affects the ability of
ecosystems to function properly and therefore undermines their ability
to support a healthy environment. Consequently, it can directly affect
human health as well if ecosystem services are unable to meet social
needs (Roe, 2019).
