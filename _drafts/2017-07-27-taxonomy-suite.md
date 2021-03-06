---
name: taxonomy-suite
layout: post_discourse
title: The rOpenSci Taxonomy Suite
date: 2017-07-27
authors:
  - name: Scott Chamberlain
categories:
  - blog
topicid: xxx
tags:
- R
- taxonomy
---




## What is Taxonomy?

Taxonomy in its most general sense is [the practice and science of classification](https://en.wikipedia.org/wiki/Taxonomy_(general)). It can refer to many things. You may have heard or used the word _taxonomy_ used to indicate any sort of classification of things, whether it be companies or widgets. Here, we're talking about [biological taxonomy](https://en.wikipedia.org/wiki/Taxonomy_(biology)), the science of defining and naming groups of biological organisms.

In case you aren't familiar with the terminology, here's a brief intro.

* `species` - the term you are likely most familiar with, usually a group of individuals in which any 2 individuals can produce fertile offspring (definitions can vary)
* `taxon` - a species, e.g., _Homo sapiens_
* `taxa` - a grouping of species (plural of `taxon`)
* `taxonomic hierarchy` (or `taxonomic classification`) - all the higher level taxonomic names for a taxon

## Ubiquity and Importance of Taxonomic Names

We put a lot of time into our suite of taxonomic software for a good reason - probably all naturalists/biologists/environmental consultants/etc. will be confronted with taxonomic names in their research/work/surveys/etc. at some point or all along the way. Some people study a single species their entire career, likely having little trouble with taxonomic names - while others study entire communities or ecosystems, dealing with thousands of taxonomic names.

Taxonomic names are not only ubiquitous but are incredibly important to get right. Just as the URL points to the correct page you want to view on the internet (an incorrect URL will not get you where you want to go), taxonomic names point to the right definition/description of a taxon, leading to lots of resources increasingly online including text, images, sounds, etc. If you get the taxonomic name wrong, all information downstream is likely to be wrong.

## Why R for taxonomic names?

R is widely used in academia, and particuarly widely used in biology within academia. At least in my graduate school experience ('06 - '12), most graduate students used R - despite their bosses often using other things, not to be named.

Given that R is widely used among biologists that have to deal with taxonomic names, it makes a lot of sense to build taxonomic tools in R.

## rOpenSci Taxonomy Suite

We have an ever-growing suite of taxonomy packages.

* `taxize` - taxonomic data from many sources
* `taxizedb` - work with taxonomic SQL databases locally
* `taxa` - taxonomic classes for R
* `binomen` - (getting folded into `taxa`, will be archived on CRAN soon)
* `wikitaxa` - taxonomic data from Wikipedia/Wikidata/Wikispecies
* `ritis` - get ITIS (Integrated Taxonomic Information Service) taxonomic data
* `worrms` - get WORMS (World Register of Marine Species) taxonomic data
* `pegax` - taxonomy PEG (Parsing Expression Grammar)

<br><br>

For each package below, there are 2-3 badges. One for whether the package is on CRAN
or not (<span class="label label-warning">cran</span> if on CRAN, <span class="label label-default">cran</span>
if not), and for link to source on GitHub (<span class="label label-info">github</span>).

For each package we show a very brief example - all packages have much more functionality - check them out on CRAN or GitHub.

## taxize

<a href="https://cran.rstudio.com/web/packages/taxize/"><span class="label label-warning">cran</span></a> <a href="https://github.com/ropensci/taxize"><span class="label label-info">github</span></a>

This was our first package for taxonomy. It is a one stop shop for lots of different taxonomic data sources online, including NCBI, ITIS, GBIF, EOL, IUCN, and more - up to 22 data sources now.

The canonical reference for `taxize` is the paper we published in 2013:

> Chamberlain, S. A., & Szöcs, E. (2013). taxize: taxonomic search and retrieval in R. F1000Research.

Check it out at <https://doi.org/10.12688/f1000research.2-191.v1>

We released a new version (`v0.8.8`) about a month ago (a tiny bug fix was pushed more recently (`v0.8.9`)) with some new features requested by users:

- You can now get downstream taxa from NCBI, see `ncbi_downstream` and `downstream`
- Wikipedia/Wikidata/Wikispecies are now data sources! via the `wikitaxa` package
- Now you can get IUCN IDs for taxa, see `get_iucn`
- `tax_rank` now works with many more data sources: ncbi, itis, eol, col, tropicos, gbif, nbn,
worms, natserv, and bold
- Many improvements and bug fixes

### Example

A quick example of the power of `taxize`


```r
install.packages("taxize")
```


```r
library("taxize")
```

Get WORMS identifiers for three taxa:


```r
ids <- get_wormsid(c("Platanista gangetica", "Lichenopora neapolitana", 'Gadus morhua'))
```

Get classifications for each taxon


```r
clazz <- classification(ids, db = 'worms')
```

Combine all three into a single data.frame


```r
head(rbind(clazz))
#>            name       rank     id  query
#> 1      Animalia    Kingdom      2 254967
#> 2      Chordata     Phylum   1821 254967
#> 3    Vertebrata  Subphylum 146419 254967
#> 4 Gnathostomata Superclass   1828 254967
#> 5     Tetrapoda Superclass   1831 254967
#> 6      Mammalia      Class   1837 254967
```


## taxizedb

<a href="https://cran.rstudio.com/web/packages/taxizedb/"><span class="label label-warning">cran</span></a> <a href="https://github.com/ropensci/taxizedb"><span class="label label-info">github</span></a>

`taxizedb` is a relatively new package. We just released a new version (`v0.1.4`) about one month ago, with fixes for the new `dplyr` version.

The sole purpose of `taxizedb` is to solve the use case where a user has a lot of taxonomic names, and thus using `taxize` is too slow. Although `taxize` is a powerful tool, for all data requests it's making another request to get data from the web, and the speed of that transaction can vary from very fast to very slow, depending on three factors: data provider speed (including many things), your internet speed, and how much data you requested. `taxizedb` gets around this problem by using a local SQL database of the same stuff the data providers have, so you can get things done much faster.

The trad-off with `taxizedb` is that the interface is quite different from `taxize`.  So there is a learning curve. There are two options in `taxizedb`: you can use SQL syntax, or `dplyr` commands. Clearly more and more people know the latter, but the former I imagine not so many.

### Example

Install `taxizedb`


```r
install.packages("taxizedb")
```


```r
library("taxizedb")
```

Here, we show working with the ITIS SQL database. Other sources work with the same workflow of function calls.

Download ITIS SQL database


```r
x <- db_download_itis()
#> downloading...
#> unzipping...
#> cleaning up...
#> [1] "/Users/sacmac/Library/Caches/R/taxizedb/ITIS.sql"
```

`db_load_tpl()` loads the SQL database into Postgres. Data sources vary in the SQL database used, see help for more.


```r
db_load_tpl(x, "<your Postgresql user name>", "your Postgresql password, if any")
```

Create a `src` object to connect to the SQL database.


```r
src <- src_itis("<your Postgresql user name>", "your Postgresql password, if any")
```

Query!


```r
library(dbplyr)
library(dplyr)
tbl(src, sql("select * from taxonomic_units limit 10"))
# Source:   SQL [?? x 26]
# Database: postgres 9.6.0 [sacmac@localhost:5432/ITIS]
     tsn unit_ind1                          unit_name1 unit_ind2 unit_name2 unit_ind3 unit_name3 unit_ind4 unit_name4
   <int>     <chr>                               <chr>     <chr>      <chr>     <chr>      <chr>     <chr>      <chr>
 1    50      <NA> Bacteria                                 <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 2    51      <NA> Schizomycetes                            <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 3    52      <NA> Archangiaceae                            <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 4    53      <NA> Pseudomonadales                          <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 5    54      <NA> Rhodobacteriineae                        <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 6    55      <NA> Pseudomonadineae                         <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 7    56      <NA> Nitrobacteraceae                         <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 8    57      <NA> Nitrobacter                              <NA>       <NA>      <NA>       <NA>      <NA>       <NA>
 9    58      <NA> Nitrobacter                              <NA>     agilis      <NA>       <NA>      <NA>       <NA>
10    59      <NA> Nitrobacter                              <NA>     flavus      <NA>       <NA>      <NA>       <NA>
# ... with more rows, and 17 more variables: unnamed_taxon_ind <chr>, name_usage <chr>, unaccept_reason <chr>,
#   credibility_rtng <chr>, completeness_rtng <chr>, currency_rating <chr>, phylo_sort_seq <int>, initial_time_stamp <dttm>,
#   parent_tsn <int>, taxon_author_id <int>, hybrid_author_id <int>, kingdom_id <int>, rank_id <int>, update_date <date>,
#   uncertain_prnt_ind <chr>, n_usage <chr>, complete_name <chr>
```


## taxa

<a href="https://cran.rstudio.com/web/packages/taxa/"><span class="label label-warning">cran</span></a> <a href="https://github.com/ropensci/taxa"><span class="label label-info">github</span></a>

`taxa` is our newest entry (hit CRAN just a few weeks ago) into the taxonomic R package space. It defines taxonomic classes for R, and basic, but powerful manipulations on those classes.

It defines two broad types of classes: those with just taxonomic names data, and those with taxonomic names data plus other data (such as traits, environmental data, etc.) - so called `taxmap` classes.

With these taxonomic classes, you can do various operations on them. With the taxonomic classes, you can filter out or keep taxa based on various criteria, and with the `taxmap` classes when you filter on taxa, the associated data is filtered the same way so taxa and data are in sync.

A manuscript about `taxa` is being prepared at the moment - so look out for that.

Most of the hard work in `taxa` has been done by my co-maintainer [Zachary Foster](https://github.com/zachary-foster)!

### Example

A quick example of the power of `taxa`


```r
install.packages("taxa")
```


```r
library("taxa")
```

An example `Hierarchy` data object that comes with the package:


```r
ex_hierarchy1
#> <Hierarchy>
#>   no. taxon's:  3
#>   Poaceae / family / 4479
#>   Poa / genus / 4544
#>   Poa annua / species / 93036
```

We can remove taxa like the following, combining criteria targeting ranks, taxonomic names, or IDs:


```r
ex_hierarchy1 %>% pop(ranks("family"), ids(4544))
#> <Hierarchy>
#>   no. taxon's:  1
#>   Poa annua / species / 93036
```

An example `taxmap` class:


```r
ex_taxmap
#> <Taxmap>
#>   17 taxa: b. Mammalia ... q. lycopersicum, r. tuberosum
#>   17 edges: NA->b, NA->c, b->d ... j->o, k->p, l->q, l->r
#>   4 data sets:
#>     info:
#>       # A tibble: 6 x 4
#>           name n_legs dangerous taxon_id
#>         <fctr>  <dbl>     <lgl>    <chr>
#>       1  tiger      4      TRUE        m
#>       2    cat      4     FALSE        n
#>       3   mole      4     FALSE        o
#>       # ... with 3 more rows
#>     phylopic_ids:  e148eabb-f138-43c6-b1e4-5cda2180485a ... 63604565-0406-460b-8cb8-1abe954b3f3a
#>     foods: a list with 6 items
#>     And 1 more data sets: abund
#>   1 functions:
#>  reaction
```

Here, filter by taxonomic names to those starting with the letter `t` (notice the taxa, edgelist, and datasets have changed)


```r
filter_taxa(ex_taxmap, startsWith(taxon_names, "t"))
#> <Taxmap>
#>   3 taxa: m. tigris, o. typhlops, r. tuberosum
#>   3 edges: NA->m, NA->o, NA->r
#>   4 data sets:
#>     info:
#>       # A tibble: 3 x 4
#>           name n_legs dangerous taxon_id
#>         <fctr>  <dbl>     <lgl>    <chr>
#>       1  tiger      4      TRUE        m
#>       2   mole      4     FALSE        o
#>       3 potato      0     FALSE        r
#>     phylopic_ids:  e148eabb-f138-43c6-b1e4-5cda2180485a ... 63604565-0406-460b-8cb8-1abe954b3f3a
#>     foods: a list with 3 items
#>     And 1 more data sets: abund
#>   1 functions:
#>  reaction
```

## wikitaxa

<a href="https://cran.rstudio.com/web/packages/wikitaxa/"><span class="label label-warning">cran</span></a> <a href="https://github.com/ropensci/wikitaxa"><span class="label label-info">github</span></a>

`wikitaxa` is a client that allows you to get taxonomic data from four different Wiki-* sites:

* Wikipedia
* Wikispecies
* Wikidata
* Wikicommons

The only site of the four above that's only taxonomy is Wikispecies - for the others you could use `wikitaxa` to do any searches, but we look for and parse out taxonomic specific items in the wiki objects that are returned.

We released a new version (`v0.1.4`) earlier this year. Big thanks to [Ethan Welty](https://github.com/ezwelty) for help on this package.

`wikitaxa` is used in `taxize` to get Wiki* data.

### Example

A quick example of the power of `wikitaxa`


```r
install.packages("wikitaxa")
```


```r
library("wikitaxa")
```

Search for _Malus domestica_ (apple):


```r
res <- wt_wikispecies(name = "Malus domestica")
# links to language sites for the taxon
res$langlinks
#> # A tibble: 12 x 5
#>     lang                                                   url    langname
#>  * <chr>                                                 <chr>       <chr>
#>  1   ast        https://ast.wikipedia.org/wiki/Malus_domestica    Asturian
#>  2    es         https://es.wikipedia.org/wiki/Malus_domestica     Spanish
#>  3    hu              https://hu.wikipedia.org/wiki/Nemes_alma   Hungarian
#>  4    ia         https://ia.wikipedia.org/wiki/Malus_domestica Interlingua
#>  5    it         https://it.wikipedia.org/wiki/Malus_domestica     Italian
#>  6   nds              https://nds.wikipedia.org/wiki/Huusappel  Low German
#>  7    nl           https://nl.wikipedia.org/wiki/Appel_(plant)       Dutch
#>  8    pl https://pl.wikipedia.org/wiki/Jab%C5%82o%C5%84_domowa      Polish
#>  9   pms        https://pms.wikipedia.org/wiki/Malus_domestica Piedmontese
#> 10    pt         https://pt.wikipedia.org/wiki/Malus_domestica  Portuguese
#> 11    sk https://sk.wikipedia.org/wiki/Jablo%C5%88_dom%C3%A1ca      Slovak
#> 12    vi         https://vi.wikipedia.org/wiki/Malus_domestica  Vietnamese
#> # ... with 2 more variables: autonym <chr>, `*` <chr>
# any external links on the page
res$externallinks
#> [1] "https://web.archive.org/web/20090115062704/http://www.ars-grin.gov/cgi-bin/npgs/html/taxon.pl?104681"
# any common names, and the language they are from
res$common_names
#> # A tibble: 19 x 2
#>               name   language
#>              <chr>      <chr>
#>  1          Ябълка  български
#>  2    Poma, pomera     català
#>  3           Apfel    Deutsch
#>  4     Aed-õunapuu      eesti
#>  5           Μηλιά   Ελληνικά
#>  6           Apple    English
#>  7         Manzano    español
#>  8           Pomme   français
#>  9           Melâr     furlan
#> 10        사과나무     한국어
#> 11          ‘Āpala    Hawaiʻi
#> 12            Melo   italiano
#> 13           Aapel Nordfriisk
#> 14  Maçã, Macieira  português
#> 15 Яблоня домашняя    русский
#> 16   Tarhaomenapuu      suomi
#> 17            Elma     Türkçe
#> 18  Яблуня домашня українська
#> 19          Pomaro     vèneto
# the taxonomic hierarchy - or classification
res$classification
#> # A tibble: 8 x 2
#>          rank          name
#>         <chr>         <chr>
#> 1 Superregnum     Eukaryota
#> 2      Regnum       Plantae
#> 3      Cladus   Angiosperms
#> 4      Cladus      Eudicots
#> 5      Cladus Core eudicots
#> 6      Cladus        Rosids
#> 7      Cladus    Eurosids I
#> 8        Ordo       Rosales
```



## ritis

<a href="https://cran.rstudio.com/web/packages/ritis/"><span class="label label-warning">cran</span></a> <a href="https://github.com/ropensci/ritis"><span class="label label-info">github</span></a>

`ritis` is a client for ITIS (Integrated Taxonomic Information Service), part of [USGS](https://www.usgs.gov/).

There's a number of different ways to get ITIS data, one of which (local SQL dump) is available in `taxizedb`, while the others are covered in `ritis`:

- SOLR web service <https://www.itis.gov/solr_documentation.html>
- RESTful web service <https://www.itis.gov/web_service.html>

The functions that use the SOLR service are: `itis_search`, `itis_facet`, `itis_group`, and `itis_highlight`.

All other functions interact with the RESTful web service.

We released a new version (`v0.5.4`) late last year.

`ritis` is used in `taxize` to get ITIS data.

### Example

A quick example of the power of `ritis`


```r
install.packages("ritis")
```


```r
library("ritis")
```

Search for blue oak ( _Quercus douglasii_ )


```r
search_scientific("Quercus douglasii")
#> # A tibble: 1 x 12
#>         author      combinedName kingdom   tsn unitInd1 unitInd2 unitInd3
#> *        <chr>             <chr>   <chr> <chr>    <lgl>    <lgl>    <lgl>
#> 1 Hook. & Arn. Quercus douglasii Plantae 19322       NA       NA       NA
#> # ... with 5 more variables: unitInd4 <lgl>, unitName1 <chr>,
#> #   unitName2 <chr>, unitName3 <lgl>, unitName4 <lgl>
```

Get taxonomic hierarchy down from the Oak genus - that is, since it's a genus, get all species in the Oak genus


```r
res <- search_scientific("Quercus")
hierarchy_down(res[1,]$tsn)
#> # A tibble: 207 x 5
#>    parentname parenttsn rankname          taxonname   tsn
#>  *      <chr>     <chr>    <chr>              <chr> <chr>
#>  1    Quercus     19276  Species    Quercus falcata 19277
#>  2    Quercus     19276  Species     Quercus lyrata 19278
#>  3    Quercus     19276  Species  Quercus michauxii 19279
#>  4    Quercus     19276  Species      Quercus nigra 19280
#>  5    Quercus     19276  Species  Quercus palustris 19281
#>  6    Quercus     19276  Species    Quercus phellos 19282
#>  7    Quercus     19276  Species Quercus virginiana 19283
#>  8    Quercus     19276  Species Quercus macrocarpa 19287
#>  9    Quercus     19276  Species   Quercus coccinea 19288
#> 10    Quercus     19276  Species  Quercus agrifolia 19289
#> # ... with 197 more rows
```



## worrms

<a href="https://cran.rstudio.com/web/packages/worrms/"><span class="label label-warning">cran</span></a> <a href="https://github.com/ropensci/worrms"><span class="label label-info">github</span></a>

`worrms` is a client for working with data from World Register of Marine Species (WoRMS).

WoRMS is the most authoritative list of names of all marine species globally.

We released our first version (`v0.1.0`) earlier this year.

`worrms` is used in `taxize` to get WoRMS data.

### Example

A quick example of the power of `worrms`


```r
install.packages("worrms")
```


```r
library("worrms")
```

Get taxonomic name synonyms for salmon ( _Oncorhynchus_ )


```r
xx <- wm_records_name("Oncorhynchus", fuzzy = FALSE)
wm_synonyms(id = xx$AphiaID)
#> # A tibble: 4 x 25
#>   AphiaID                                                           url
#> *   <int>                                                         <chr>
#> 1  296858 http://www.marinespecies.org/aphia.php?p=taxdetails&id=296858
#> 2  397908 http://www.marinespecies.org/aphia.php?p=taxdetails&id=397908
#> 3  397909 http://www.marinespecies.org/aphia.php?p=taxdetails&id=397909
#> 4  297397 http://www.marinespecies.org/aphia.php?p=taxdetails&id=297397
#> # ... with 23 more variables: scientificname <chr>, authority <chr>,
#> #   status <chr>, unacceptreason <chr>, rank <chr>, valid_AphiaID <int>,
#> #   valid_name <chr>, valid_authority <chr>, kingdom <chr>, phylum <chr>,
#> #   class <chr>, order <chr>, family <chr>, genus <chr>, citation <chr>,
#> #   lsid <chr>, isMarine <int>, isBrackish <lgl>, isFreshwater <lgl>,
#> #   isTerrestrial <int>, isExtinct <lgl>, match_type <chr>, modified <chr>
```


## pegax

<span class="label label-default">cran</span> <a href="https://github.com/ropenscilabs/pegax"><span class="label label-info">github</span></a>

`pegax` aims to be a powerful taxonomic name parser for R. This package started at [#runconf17](http://unconf17.ropensci.org/) - was made possible because the talented [Oliver Keyes](https://github.com/Ironholds/) created a [Parsing Expression Grammar](https://en.wikipedia.org/wiki/Parsing_expression_grammar) package for R: [piton](https://github.com/Ironholds/piton)

From `piton` PEGs are:

> a way of defining formal grammars for formatted data that allow you to identify matched structures and then take actions on them

Some great taxonomic name parsing does exist already. [Global Names Parser, gnparser](https://github.com/GlobalNamesArchitecture/gnparser) is a great effort by [Dmitry Mozzherin](https://github.com/dimus) and others. The only problem is Java does not play nice with R - thus `pegax`, implementing in C++. We'll definitely try to learn alot from the work they have done on `gnparser`.

`pegax` is not on CRAN yet.  The package is in very very early days, so expect lots of changes.

### Example

A quick example of the power of `pegax`


```r
devtools::install_github("ropenscilabs/pegax")
```


```r
library("pegax")
```

Parse out authority name


```r
authority_names("Linnaeus, 1758")
#> [1] "Linnaeus"
```

Parse out authority year


```r
authority_years("Linnaeus, 1758")
#> [1] "1758"
```

-------

## Feedback

What do you think about the taxonomic suite of packages?  Anything we're missing? Anything we can be doing better with any of the packages?  Are you working on a taxonomic R package? Consider [submitting to rOpenSci](https://github.com/ropensci/onboarding).
