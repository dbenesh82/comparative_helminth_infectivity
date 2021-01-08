Biases
================

  - [Wrangle](#wrangle)
  - [Descriptives](#descriptives)
  - [Worm families overrepresented in life cycle database and recovery
    data relative to their overall
    diversity](#worm-families-overrepresented-in-life-cycle-database-and-recovery-data-relative-to-their-overall-diversity)
      - [Taxonomy](#taxonomy)
      - [Study effort](#study-effort)
      - [Habitat](#habitat)
      - [Host type](#host-type)
      - [Conclusions - biases in life cycle database relative to known
        helminth
        diversity](#conclusions---biases-in-life-cycle-database-relative-to-known-helminth-diversity)
      - [Plots for supplement](#plots-for-supplement)

In this notebook, we explore various kinds of biases in our comparative
data. We explore these biases at the parasite family level. That is, we
look for families that are more or less represented in the life cycle
database relative to their general diversity, and for families that are
over/underrepresented in the recovery rate data relative to the life
cycle database.

# Wrangle

In another [notebook](get_study_effort_family.Rmd), we generated a
family-level table that included study effort.

Then we add habitat from the life cycle database: freshwater, marine, or
terrestrial. In some families, the hosts recorded occupy different
habitats (e.g. some aquatic hosts, some terrestrial hosts). In those
cases where less than 75% of the hosts were from the same habitat
category, we considered the habitat as “mixed”.

Next, we added the typical final host type for a family: bird, mammal,
herp, fish, or invert. We only defined *final* host type, not
intermediate host type, because not all families have intermediate hosts
and some families have life cycles with multiple intermediate hosts.

# Descriptives

Here are some descriptive statistics. How many parasite families are
represented in the open tree taxonomy (ott), life cycle database (lcdb),
and recovery rate data (rr)?

<div class="kable-table">

| db   | num\_families |
| :--- | ------------: |
| ot   |           198 |
| lcdb |           124 |
| rr   |            54 |

</div>

Here are the number of orders:

<div class="kable-table">

| db   | num\_orders |
| :--- | ----------: |
| ot   |          43 |
| lcdb |          32 |
| rr   |          17 |

</div>

Number of classes:

<div class="kable-table">

| db   | num\_class |
| :--- | ---------: |
| ot   |          7 |
| lcdb |          6 |
| rr   |          6 |

</div>

we only have habitat and host type information for families represented
in the life cycle database. How many families are in each habitat in
each dataset? There are few marine families in the recovery rate data.

<div class="kable-table">

| habitat     | num\_fams\_lcdb | num\_fams\_rr |
| :---------- | --------------: | ------------: |
| freshwater  |              35 |            11 |
| marine      |              21 |             2 |
| mixed       |              13 |            10 |
| terrestrial |              55 |            31 |

</div>

There are also relatively few recovery experiments with herps (reptiles
or amphibians) and many with mammals.

<div class="kable-table">

| host\_type\_dh | num\_fams\_lcdb | num\_fams\_rr |
| :------------- | --------------: | ------------: |
| bird           |              22 |             7 |
| fish           |              41 |            11 |
| herptile       |               7 |             1 |
| invertebrate   |               2 |            NA |
| mammal         |              35 |            24 |
| mixed          |              17 |            11 |

</div>

# Worm families overrepresented in life cycle database and recovery data relative to their overall diversity

We mined our recovery data from studies in the life cycle database.
Therefore, we should consider taxonomic biases in the recovery data
*and* the life cycle database, relative to overall helminth diversity.
Presumably, some parasite families are overrepresented in the database,
because they are medically important or are easy to study. To test
whether some families are over or underrepesented, we calculated the
number of species in the life cycle database and number of species in
the open tree taxonomy from each family. Families that are
overrepresented in the life cycle database should have a larger percent
of their overall diversity included in the database. To model this, we
fit generalized linear mixed models (binomial errors) for the proportion
of overall family-level species diversity included in the recovery data
and life cycle database (i.e. number of species in family in data /
number of species in family, measured from ott).

Parasite family was a random effect - high and low values represent
families that are over and underrepresented in the data relative to the
open tree taxonomy.

## Taxonomy

We want to know if the same families are over/underrepresented in the
life cycle database and the recovery rate data. Therefore, we allowed a
random interaction between parasite family and dataset (lcdb vs rr).
This is a row-level effect and is therefore basically the residual
variance. Adding this effect is a clear improvement.

<div class="kable-table">

|      | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :--- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| mod  |    3 | 1206.480 | 1218.402 | \-600.2401 | 1200.480 |       NA | NA |          NA |
| mod0 |    5 | 1193.811 | 1213.680 | \-591.9057 | 1183.811 | 16.66876 |  2 |   0.0002401 |

</div>

The model included a fixed effect for “dataset”. It indicated that on
average just `round(df_int$prop[1] * 100, 2)`% of the species in a
family were included in the life cycle database and just
`round(df_int$prop[2] * 100, 2)`% of the species in the recovery data.
This was expected; more life cycles are known than infection rate
experiments. The low values overall are due to many families having zero
representation in the life cycle and recovery datasets, as seen in this
histogram.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

The model’s random effects quantify the over and under representation of
different taxa. Note the variance components in model summary.

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: cbind(n_spp, n_spp_ot - n_spp) ~ db + (db - 1 | parasite_family)
    ##    Data: filter(g_levx, n_spp_ot >= n_spp)
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   1193.8   1213.7   -591.9   1183.8      388 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -1.0750 -0.3702 -0.1616  0.1148  1.6735 
    ## 
    ## Random effects:
    ##  Groups          Name   Variance Std.Dev. Corr
    ##  parasite_family dblcdb 3.056    1.748        
    ##                  dbrr   4.191    2.047    0.92
    ## Number of obs: 393, groups:  parasite_family, 193
    ## 
    ## Fixed effects:
    ##             Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)  -3.1517     0.1597  -19.73   <2e-16 ***
    ## dbrr         -2.5541     0.2186  -11.69   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##      (Intr)
    ## dbrr -0.024

The family-level variance was a bit higher in the recovery rate data
than the life cycle data, which suggests that differences among families
in their representation is more extreme in the recovery data. This makes
sense. If the life cycle data is a subset of the total diversity, then
the recovery data is a subset of subset. Also note in the summary that
the random effect estimates were highly correlated across datasets
(\>0.9), indicating that families the were overrepresented in the life
cycle data were also overrepresented in the recovery data.

Here are the 20 families that are proportionally most over-represented
in both the life cycle and recovery data (the combined random effect for
each dataset). These tend to be either diverse, well-studied groups
(e.g. Ascarididae, Taeniidae), taxa with unique phylogenetic positions
(e.g. Spathebothriidae), or groups where the known diversity is probably
underestimated (e.g. Ophidascaridae, Fessisentidae).

<div class="kable-table">

|   dblcdb |     dbrr | parasite\_family  | comb\_re |    diff\_re | parasite\_phylum |
| -------: | -------: | :---------------- | -------: | ----------: | :--------------- |
| 4.377503 | 5.139085 | Toxocaridae       | 4.758294 |   0.7615816 | Nematoda         |
| 4.004335 | 5.080148 | Ascarididae       | 4.542242 |   1.0758129 | Nematoda         |
| 3.839645 | 4.487726 | Ancylostomatidae  | 4.163686 |   0.6480818 | Nematoda         |
| 4.363026 | 3.693655 | Subuluridae       | 4.028341 | \-0.6693712 | Nematoda         |
| 3.601559 | 4.026309 | Anguillicolidae   | 3.813934 |   0.4247503 | Nematoda         |
| 3.108700 | 3.783557 | Angiostrongylidae | 3.446129 |   0.6748568 | Nematoda         |
| 3.631605 | 3.160222 | Taeniidae         | 3.395913 | \-0.4713827 | Platyhelminthes  |
| 2.944309 | 3.737058 | Protostrongylidae | 3.340684 |   0.7927496 | Nematoda         |
| 2.860937 | 3.381268 | Acrobothriidae    | 3.121103 |   0.5203314 | Platyhelminthes  |
| 2.850489 | 3.246891 | Dracunculidae     | 3.048690 |   0.3964019 | Nematoda         |
| 2.426728 | 2.481252 | Fessisentidae     | 2.453990 |   0.0545241 | Acanthocephala   |
| 1.642349 | 3.160738 | Ascaridiidae      | 2.401543 |   1.5183884 | Nematoda         |
| 1.841392 | 2.879526 | Dioctophymidae    | 2.360459 |   1.0381339 | Nematoda         |
| 2.062885 | 2.514804 | Ophidascaridae    | 2.288844 |   0.4519187 | Nematoda         |
| 2.137373 | 2.290932 | Spathebothriidae  | 2.214152 |   0.1535580 | Platyhelminthes  |
| 1.553828 | 2.776115 | Haemonchidae      | 2.164971 |   1.2222872 | Nematoda         |
| 1.828159 | 2.495124 | Crenosomatidae    | 2.161641 |   0.6669650 | Nematoda         |
| 1.756601 | 2.553861 | Dictyocaulidae    | 2.155231 |   0.7972603 | Nematoda         |
| 1.993362 | 2.164151 | Gongylonematidae  | 2.078756 |   0.1707894 | Nematoda         |
| 1.846558 | 2.142519 | Gnathostomatidae  | 1.994539 |   0.2959602 | Nematoda         |

</div>

Here are the 20 families most overrepresented in the recovery data
compared to the life cycle database. They seem to biased towards highly
studied groups like ascarids, diphyllobothirds, haemonchids, etc.

<div class="kable-table">

|      dblcdb |      dbrr | parasite\_family      |  comb\_re |  diff\_re | parasite\_phylum |
| ----------: | --------: | :-------------------- | --------: | --------: | :--------------- |
|   1.6423491 | 3.1607375 | Ascaridiidae          | 2.4015433 | 1.5183884 | Nematoda         |
|   0.8552718 | 2.3254524 | Diphyllobothriidae    | 1.5903621 | 1.4701806 | Platyhelminthes  |
|   1.5538276 | 2.7761148 | Haemonchidae          | 2.1649712 | 1.2222872 | Nematoda         |
|   4.0043355 | 5.0801484 | Ascarididae           | 4.5422420 | 1.0758129 | Nematoda         |
|   1.8413924 | 2.8795264 | Dioctophymidae        | 2.3604594 | 1.0381339 | Nematoda         |
| \-0.0796404 | 0.9264741 | Cloacinidae           | 0.4234169 | 1.0061145 | Nematoda         |
|   0.5301110 | 1.5113562 | Acuariidae            | 1.0207336 | 0.9812451 | Nematoda         |
|   0.8754302 | 1.7787411 | Mesocestoididae       | 1.3270856 | 0.9033109 | Platyhelminthes  |
|   1.7566006 | 2.5538609 | Dictyocaulidae        | 2.1552307 | 0.7972603 | Nematoda         |
|   2.9443088 | 3.7370584 | Protostrongylidae     | 3.3406836 | 0.7927496 | Nematoda         |
|   4.3775033 | 5.1390848 | Toxocaridae           | 4.7582940 | 0.7615816 | Nematoda         |
|   1.5175553 | 2.2409825 | Heligmosomatidae      | 1.8792689 | 0.7234272 | Nematoda         |
| \-0.2009995 | 0.5044539 | Molineidae            | 0.1517272 | 0.7054534 | Nematoda         |
|   3.1087003 | 3.7835571 | Angiostrongylidae     | 3.4461287 | 0.6748568 | Nematoda         |
|   1.8281588 | 2.4951238 | Crenosomatidae        | 2.1616413 | 0.6669650 | Nematoda         |
|   3.8396448 | 4.4877265 | Ancylostomatidae      | 4.1636857 | 0.6480818 | Nematoda         |
|   0.6182456 | 1.1982392 | Oligacanthorhynchidae | 0.9082424 | 0.5799936 | Acanthocephala   |
|   1.0593698 | 1.6063692 | Tetrameridae          | 1.3328695 | 0.5469994 | Nematoda         |
|   2.8609370 | 3.3812684 | Acrobothriidae        | 3.1211027 | 0.5203314 | Platyhelminthes  |
|   1.0863348 | 1.5860473 | Moniliformidae        | 1.3361911 | 0.4997125 | Acanthocephala   |

</div>

Here are the 20 families most underrepresented in both datasets,
relative to overall diversity. There are mostly nematodes that are
parasitoids (Mermithidae), plant parasites (Longidoridae,
Hoplolaimidae), or entomopathogens (Steinernematidae), and therefore are
not represented in the life cycle database (which was focused on
trophically-transmitted parasites).

<div class="kable-table">

|     dblcdb |       dbrr | parasite\_family    |   comb\_re |    diff\_re | parasite\_phylum |
| ---------: | ---------: | :------------------ | ---------: | ----------: | :--------------- |
| \-3.183575 | \-3.476962 | Mermithidae         | \-3.330269 | \-0.2933879 | Nematoda         |
| \-3.119438 | \-3.407105 | Longidoridae        | \-3.263272 | \-0.2876674 | Nematoda         |
| \-3.018355 | \-3.296993 | Hoplolaimidae       | \-3.157674 | \-0.2786385 | Nematoda         |
| \-2.953489 | \-3.226325 | Aphelenchoididae    | \-3.089907 | \-0.2728361 | Nematoda         |
| \-2.940599 | \-3.212282 | Monhysteridae       | \-3.076440 | \-0.2716823 | Nematoda         |
| \-2.711889 | \-2.963054 | Cephalobidae        | \-2.837472 | \-0.2511647 | Nematoda         |
| \-2.360778 | \-2.580275 | Heteroderidae       | \-2.470526 | \-0.2194966 | Nematoda         |
| \-2.265455 | \-2.476317 | Anguinidae          | \-2.370886 | \-0.2108622 | Nematoda         |
| \-2.252097 | \-2.461748 | Allantonematidae    | \-2.356922 | \-0.2096510 | Nematoda         |
| \-2.059233 | \-2.286689 | Pharyngodonidae     | \-2.172961 | \-0.2274557 | Nematoda         |
| \-2.047051 | \-2.238069 | Steinernematidae    | \-2.142560 | \-0.1910177 | Nematoda         |
| \-2.035178 | \-2.225114 | Monticelliidae      | \-2.130146 | \-0.1899364 | Platyhelminthes  |
| \-1.829814 | \-2.001006 | Panagrolaimidae     | \-1.915410 | \-0.1711916 | Nematoda         |
| \-1.805764 | \-1.974755 | Lecanicephalidae    | \-1.890260 | \-0.1689911 | Platyhelminthes  |
| \-1.763845 | \-1.928998 | Strongyloididae     | \-1.846422 | \-0.1651531 | Nematoda         |
| \-1.662179 | \-1.818009 | Meloidogynidae      | \-1.740094 | \-0.1558303 | Nematoda         |
| \-1.566899 | \-1.713974 | Tetragonocephalidae | \-1.640437 | \-0.1470745 | Platyhelminthes  |
| \-1.504250 | \-1.682866 | Kathlaniidae        | \-1.593558 | \-0.1786155 | Nematoda         |
| \-1.457017 | \-1.593971 | Arhythmacanthidae   | \-1.525494 | \-0.1369537 | Acanthocephala   |
| \-1.326940 | \-1.451880 | Polypocephalidae    | \-1.389410 | \-0.1249403 | Platyhelminthes  |

</div>

This raises the question of whether nematode families are more
underrepresented than acanth or cestode families. When we add parasite
phylum as a fixed effect to the model, it is not an improvement. This
reminds us that nematode families are among those most over and
underrepresented in the datasets.

<div class="kable-table">

|           | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :-------- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| mod0      |    5 | 1193.811 | 1213.680 | \-591.9057 | 1183.811 |       NA | NA |          NA |
| mod0\_phy |    7 | 1194.031 | 1221.848 | \-590.0155 | 1180.031 | 3.780476 |  2 |   0.1510358 |

</div>

Another way to visualize disproportionate representation is by looking
at whether species number in the ott correlates with species number in
the datasets. In general, there is a correlation - worm families with
many species usually have more species in the recovery and life cycle
data - but some groups are over- or underrepresented as indicated by
color coding by the random effect estimate (red = overrepresented, blue
= underrepresented).

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->

The distribution of the random effects is not too bad, though there is
some bimodality from the many zero values (i.e. families not represented
in the lcdb).

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-25-1.png)<!-- -->

Since most of the underrepresented groups in the life cycle database
were taxa that were not targeted anyways, let’s try limiting the
analysis to the families in the life cycle database. This assumes that
most families containing trophically-transmitted animal parasites have
at least one representative in the database.

After excluding families not in the life cycle database, the average
percent of species from a family in the database jumped from
`round(df_int$prop[1] * 100, 2)`% to `round(df_int2$prop[1] * 100, 2)`%
for the life cycle database and from `round(df_int$prop[2] * 100, 2)`%
to `round(df_int2$prop[2] * 100, 2)`% for the recovery rate data. The
distribution of random effects also looks a little better (less
bimodal).

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-28-1.png)<!-- -->
The most overrepresented groups are the same.

<div class="kable-table">

|    dblcdb |     dbrr | parasite\_family  | comb\_re |    diff\_re | parasite\_phylum |
| --------: | -------: | :---------------- | -------: | ----------: | :--------------- |
| 3.2682427 | 4.131151 | Toxocaridae       | 3.699697 |   0.8629083 | Nematoda         |
| 2.6265674 | 3.905493 | Ascarididae       | 3.266030 |   1.2789252 | Nematoda         |
| 3.2533725 | 2.655523 | Subuluridae       | 2.954448 | \-0.5978496 | Nematoda         |
| 2.4500437 | 3.124591 | Ancylostomatidae  | 2.787317 |   0.6745473 | Nematoda         |
| 2.4499059 | 2.907547 | Anguillicolidae   | 2.678726 |   0.4576410 | Nematoda         |
| 2.1859192 | 2.840928 | Angiostrongylidae | 2.513424 |   0.6550086 | Nematoda         |
| 2.7342285 | 2.228337 | Taeniidae         | 2.481283 | \-0.5058916 | Platyhelminthes  |
| 2.0585576 | 2.802910 | Protostrongylidae | 2.430734 |   0.7443527 | Nematoda         |
| 1.9286167 | 2.300905 | Dracunculidae     | 2.114761 |   0.3722885 | Nematoda         |
| 1.8335136 | 2.342118 | Acrobothriidae    | 2.087816 |   0.5086040 | Platyhelminthes  |
| 0.7905947 | 2.234801 | Ascaridiidae      | 1.512698 |   1.4442059 | Nematoda         |
| 0.9765039 | 1.953682 | Dioctophymidae    | 1.465093 |   0.9771777 | Nematoda         |
| 1.4390202 | 1.458172 | Fessisentidae     | 1.448596 |   0.0191518 | Acanthocephala   |
| 1.1895292 | 1.586874 | Ophidascaridae    | 1.388202 |   0.3973450 | Nematoda         |
| 0.7051689 | 1.852523 | Haemonchidae      | 1.278846 |   1.1473542 | Nematoda         |
| 0.9629083 | 1.570855 | Crenosomatidae    | 1.266881 |   0.6079463 | Nematoda         |
| 0.8950284 | 1.632469 | Dictyocaulidae    | 1.263749 |   0.7374408 | Nematoda         |
| 1.1200434 | 1.237836 | Gongylonematidae  | 1.178940 |   0.1177924 | Nematoda         |
| 1.0865799 | 1.183427 | Spathebothriidae  | 1.135004 |   0.0968472 | Platyhelminthes  |
| 0.9846647 | 1.217361 | Gnathostomatidae  | 1.101013 |   0.2326963 | Nematoda         |

</div>

The 20 families most overrepresented in the recovery data compared to
the life cycle database also remain the same.

<div class="kable-table">

|      dblcdb |        dbrr | parasite\_family      |    comb\_re |  diff\_re | parasite\_phylum |
| ----------: | ----------: | :-------------------- | ----------: | --------: | :--------------- |
|   0.7905947 |   2.2348006 | Ascaridiidae          |   1.5126976 | 1.4442059 | Nematoda         |
|   0.0235618 |   1.3956983 | Diphyllobothriidae    |   0.7096301 | 1.3721365 | Platyhelminthes  |
|   2.6265674 |   3.9054926 | Ascarididae           |   3.2660300 | 1.2789252 | Nematoda         |
|   0.7051689 |   1.8525232 | Haemonchidae          |   1.2788461 | 1.1473542 | Nematoda         |
|   0.9765039 |   1.9536816 | Dioctophymidae        |   1.4650928 | 0.9771777 | Nematoda         |
| \-0.8807910 |   0.0075914 | Cloacinidae           | \-0.4365998 | 0.8883824 | Nematoda         |
| \-0.3038266 |   0.5792406 | Acuariidae            |   0.1377070 | 0.8830673 | Nematoda         |
|   3.2682427 |   4.1311510 | Toxocaridae           |   3.6996968 | 0.8629083 | Nematoda         |
|   0.0917328 |   0.9056447 | Mesocestoididae       |   0.4986888 | 0.8139120 | Platyhelminthes  |
|   2.0585576 |   2.8029103 | Protostrongylidae     |   2.4307339 | 0.7443527 | Nematoda         |
|   0.8950284 |   1.6324692 | Dictyocaulidae        |   1.2637488 | 0.7374408 | Nematoda         |
|   2.4500437 |   3.1245910 | Ancylostomatidae      |   2.7873174 | 0.6745473 | Nematoda         |
|   2.1859192 |   2.8409278 | Angiostrongylidae     |   2.5134235 | 0.6550086 | Nematoda         |
|   0.6713048 |   1.3258251 | Heligmosomatidae      |   0.9985649 | 0.6545203 | Nematoda         |
|   0.9629083 |   1.5708546 | Crenosomatidae        |   1.2668815 | 0.6079463 | Nematoda         |
| \-1.0013648 | \-0.4120376 | Molineidae            | \-0.7067012 | 0.5893272 | Nematoda         |
|   1.8335136 |   2.3421176 | Acrobothriidae        |   2.0878156 | 0.5086040 | Platyhelminthes  |
| \-0.1926566 |   0.2922633 | Oligacanthorhynchidae |   0.0498034 | 0.4849199 | Acanthocephala   |
|   0.2285083 |   0.6926184 | Tetrameridae          |   0.4605634 | 0.4641101 | Nematoda         |
|   2.4499059 |   2.9075469 | Anguillicolidae       |   2.6787264 | 0.4576410 | Nematoda         |

</div>

The most underrepresented groups change, though. They are no longer
predominantly nematodes. Some of the groups fit my expectations, like
shark cestodes (Onchobothriidae, Eutetrarhynchidae) and herp nematodes
(Cosmocercidae, Pharyngodonidae, Kathlaniidae).

<div class="kable-table">

|      dblcdb |        dbrr | parasite\_family   |    comb\_re |    diff\_re | parasite\_phylum |
| ----------: | ----------: | :----------------- | ----------: | ----------: | :--------------- |
| \-2.4434831 | \-2.7924109 | Pharyngodonidae    | \-2.6179470 | \-0.3489278 | Nematoda         |
| \-1.9287916 | \-2.3402520 | Onchobothriidae    | \-2.1345218 | \-0.4114603 | Platyhelminthes  |
| \-1.9172155 | \-2.2038399 | Kathlaniidae       | \-2.0605277 | \-0.2866243 | Nematoda         |
| \-1.8103474 | \-2.1182196 | Cosmocercidae      | \-1.9642835 | \-0.3078722 | Nematoda         |
| \-1.6861160 | \-1.9796654 | Eutetrarhynchidae  | \-1.8328907 | \-0.2935494 | Platyhelminthes  |
| \-1.6676136 | \-1.9243327 | Heligmosomidae     | \-1.7959731 | \-0.2567191 | Nematoda         |
| \-1.5582759 | \-1.9015135 | Thelastomatidae    | \-1.7298947 | \-0.3432376 | Nematoda         |
| \-1.4378194 | \-1.6667919 | Quimperiidae       | \-1.5523057 | \-0.2289725 | Nematoda         |
| \-1.4282523 | \-1.6560650 | Echeneibothriidae  | \-1.5421587 | \-0.2278127 | Platyhelminthes  |
| \-1.4087658 | \-1.6342150 | Tentaculariidae    | \-1.5214904 | \-0.2254492 | Platyhelminthes  |
| \-1.3020193 | \-1.6774560 | Oxyuridae          | \-1.4897377 | \-0.3754367 | Nematoda         |
| \-1.2755854 | \-1.6761728 | Phyllobothriidae   | \-1.4758791 | \-0.4005874 | Platyhelminthes  |
| \-1.2072917 | \-1.4452744 | Plagiorhynchidae   | \-1.3262830 | \-0.2379827 | Acanthocephala   |
| \-1.0599958 | \-1.3162838 | Quadrigyridae      | \-1.1881398 | \-0.2562880 | Acanthocephala   |
| \-0.9902716 | \-1.2387591 | Rhinebothriidae    | \-1.1145154 | \-0.2484875 | Platyhelminthes  |
| \-0.9666757 | \-1.1765235 | Tetrabothriidae    | \-1.0715996 | \-0.2098478 | Platyhelminthes  |
| \-0.8853761 | \-1.2484304 | Raphidascarididae  | \-1.0669032 | \-0.3630543 | Nematoda         |
| \-0.8771178 | \-1.0764586 | Giganthorhynchidae | \-0.9767882 | \-0.1993408 | Acanthocephala   |
| \-0.8634970 | \-1.0612381 | Pomphorhynchidae   | \-0.9623675 | \-0.1977411 | Acanthocephala   |
| \-1.0518906 | \-0.8327727 | Heterakidae        | \-0.9423317 |   0.2191179 | Nematoda         |

</div>

Adding phylum to this model is not an improvement, suggesting that
nematode, cestode, and acanth families are equally under and
overrepresented in the lcdb.

<div class="kable-table">

|               | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :------------ | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| mod0\_lc      |    5 | 1026.202 | 1043.749 | \-508.1010 | 1016.202 |       NA | NA |          NA |
| mod0\_lc\_phy |    7 | 1028.517 | 1053.083 | \-507.2586 | 1014.517 | 1.684764 |  2 |   0.4306834 |

</div>

Here is the correlation between the number of species in the life cycle
database and total species diversity, but just for families represented
in the life cycle database.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-33-1.png)<!-- -->

## Study effort

Are overrepresented families the most intensely studied? Study effort
was quantified by recording the number of hits in PubMed for all the
families in three datasets (ott, lcdb, and rr).

When we plot pubmed hits for each family, we see that families in the
life cycle database tend to be more intensely studied. Weirdly, though,
the trend is not seen in nematodes.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-35-1.png)<!-- -->

The nematode pattern could be an artifact, because when we make the same
plot using pubmed hits for genera, then there is a clear trend.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-36-1.png)<!-- -->

Notably, the number of hits per genus was much higher than for families,
probably because genus names are mentioned more in article titles than
family names, especially in the taxonomically fluid nematodes.
Therefore, let’s recalculate our study effort metric as the summed hits
for genera in the family.

Even with this metric of study effort, the pattern in nematodes is still
ambiguous.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-38-1.png)<!-- -->

We would expect families with more species to have more pubmed hits than
less diverse ones. Indeed, there is a correlation between study effort
and species diversity. Some families, though, appear more intensely
studied than we would expect from their species diversity.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-39-1.png)<!-- -->

Let’s again compare the datasets but with pubmed hits per species in the
family. The differences are much smaller, though the most intensely
studied families may be overrepresented in the recovery data.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-40-1.png)<!-- -->
Thus, parasite families in the recovery and life cycle data are more
intensely studied than an average worm family, but is at least partially
due to those families being more diverse with many species to study.

We test this statistically by adding study effort to our mixed model as
a fixed effect. We log transform it, because there were a few families
that were intensely studied and many that were little studied.

Adding a study effort main effect weakly improved the model, whereas
there was a clearer interaction effect.

<div class="kable-table">

|         | npar |      AIC |      BIC |     logLik | deviance |     Chisq | Df | Pr(\>Chisq) |
| :------ | ---: | -------: | -------: | ---------: | -------: | --------: | -: | ----------: |
| mod0alt |    5 | 1193.811 | 1213.680 | \-591.9057 | 1183.811 |        NA | NA |          NA |
| mod1    |    6 | 1189.419 | 1213.262 | \-588.7095 | 1177.419 |  6.392465 |  1 |   0.0114606 |
| mod1.1  |    7 | 1176.145 | 1203.961 | \-581.0723 | 1162.145 | 15.274370 |  1 |   0.0000930 |

</div>

Overall, there was a weak tendency for more intensely studied families
to be better represented in both datasets, but especially the recovery
dataset.

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: cbind(n_spp, n_spp_ot - n_spp) ~ db + (db - 1 | parasite_family) +  
    ##     log(pubmed_hits_gensum + 1) + db:log(pubmed_hits_gensum +      1)
    ##    Data: filter(g_levx, n_spp_ot >= n_spp)
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   1176.1   1204.0   -581.1   1162.1      386 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -1.0336 -0.3013 -0.0935  0.1200  1.9522 
    ## 
    ## Random effects:
    ##  Groups          Name   Variance Std.Dev. Corr
    ##  parasite_family dblcdb 3.122    1.767        
    ##                  dbrr   3.972    1.993    0.92
    ## Number of obs: 393, groups:  parasite_family, 193
    ## 
    ## Fixed effects:
    ##                                  Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                      -3.86796    0.39756  -9.729  < 2e-16 ***
    ## dbrr                             -4.80824    0.74511  -6.453  1.1e-10 ***
    ## log(pubmed_hits_gensum + 1)       0.16636    0.07969   2.088 0.036826 *  
    ## dbrr:log(pubmed_hits_gensum + 1)  0.40829    0.11796   3.461 0.000537 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr) dbrr   l(__+1
    ## dbrr        -0.079              
    ## lg(pbm__+1) -0.915  0.075       
    ## dbrr:(__+1)  0.077 -0.967 -0.078

The interaction was still important when we limited the analysis to just
the families in the life cycle database.

<div class="kable-table">

|            | npar |      AIC |      BIC |     logLik | deviance |      Chisq | Df | Pr(\>Chisq) |
| :--------- | ---: | -------: | -------: | ---------: | -------: | ---------: | -: | ----------: |
| mod0\_lc   |    5 | 1026.202 | 1043.749 | \-508.1010 | 1016.202 |         NA | NA |          NA |
| mod1\_lc   |    6 | 1028.079 | 1049.135 | \-508.0396 | 1016.079 |  0.1228298 |  1 |   0.7259853 |
| mod1.1\_lc |    7 | 1012.877 | 1037.443 | \-499.4385 |  998.877 | 17.2021290 |  1 |   0.0000336 |

</div>

Let’s plot these effects, first with all families and then with just the
lcdb families. The dashed lines are the average proportion
representation across all families. The life cycle database is not
obviously biased towards more intensely studied families, but the
recovery rate data seems to be.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-45-1.png)<!-- -->
The pattern is the same when focusing just on families in the life cycle
database.
![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-46-1.png)<!-- -->

Thus, both the recovery and life cycle data tends to include families
that are more intensely studied. However, only in the recovery data are
more intensely studied families proportionally overrepresented.

## Habitat

If we focus only on families in the life cycle database, we can also
examine how characteristics encoded in the lcdb, like habitat or host
type, might impact representation. So, are over/underrepresented
families likely to be from certain habitats? To test this, we add
habitat to the model as a fixed effect.

A LRT suggests that habitat matters and that the effect is relatively
consistent across datasets (no interaction).

<div class="kable-table">

|            | npar |      AIC |      BIC |     logLik | deviance |     Chisq | Df | Pr(\>Chisq) |
| :--------- | ---: | -------: | -------: | ---------: | -------: | --------: | -: | ----------: |
| mod0\_lc   |    5 | 1026.202 | 1043.749 | \-508.1010 | 1016.202 |        NA | NA |          NA |
| mod2\_lc   |    8 | 1018.998 | 1047.073 | \-501.4991 | 1002.998 | 13.203686 |  3 |   0.0042162 |
| mod2.1\_lc |   11 | 1018.761 | 1057.364 | \-498.3805 |  996.761 |  6.237242 |  3 |   0.1006214 |

</div>

The model parameters indicate that marine families are underrepresented.

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: cbind(n_spp, n_spp_ot - n_spp) ~ db + (db - 1 | parasite_family) +  
    ##     habitat
    ##    Data: filter(g_levx, n_spp_ot >= n_spp, parasite_family %in% fams_in_lcdb)
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   1019.0   1047.1   -501.5   1003.0      239 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -1.0925 -0.4043 -0.1223  0.3333  2.9389 
    ## 
    ## Random effects:
    ##  Groups          Name   Variance Std.Dev. Corr
    ##  parasite_family dblcdb 1.265    1.125        
    ##                  dbrr   2.066    1.437    0.84
    ## Number of obs: 247, groups:  parasite_family, 122
    ## 
    ## Fixed effects:
    ##                    Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)        -2.24057    0.22719  -9.862  < 2e-16 ***
    ## dbrr               -2.43831    0.16389 -14.878  < 2e-16 ***
    ## habitatmarine      -1.03475    0.38432  -2.692  0.00709 ** 
    ## habitatmixed        0.54930    0.39873   1.378  0.16832    
    ## habitatterrestrial  0.05503    0.28693   0.192  0.84790    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Here are the families split by habitat. The dashed lines are the overall
averages for each dataset.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-50-1.png)<!-- -->

When we revisit the most underrepresented groups, we see that the marine
families are almost all cestodes. Cestodes are diverse in sharks but
their life cycles are poorly known.

<div class="kable-table">

|      dblcdb |        dbrr | parasite\_family   |    comb\_re |    diff\_re | parasite\_phylum | habitat     |
| ----------: | ----------: | :----------------- | ----------: | ----------: | :--------------- | :---------- |
| \-2.4434831 | \-2.7924109 | Pharyngodonidae    | \-2.6179470 | \-0.3489278 | Nematoda         | freshwater  |
| \-1.9287916 | \-2.3402520 | Onchobothriidae    | \-2.1345218 | \-0.4114603 | Platyhelminthes  | marine      |
| \-1.9172155 | \-2.2038399 | Kathlaniidae       | \-2.0605277 | \-0.2866243 | Nematoda         | freshwater  |
| \-1.8103474 | \-2.1182196 | Cosmocercidae      | \-1.9642835 | \-0.3078722 | Nematoda         | terrestrial |
| \-1.6861160 | \-1.9796654 | Eutetrarhynchidae  | \-1.8328907 | \-0.2935494 | Platyhelminthes  | marine      |
| \-1.6676136 | \-1.9243327 | Heligmosomidae     | \-1.7959731 | \-0.2567191 | Nematoda         | terrestrial |
| \-1.5582759 | \-1.9015135 | Thelastomatidae    | \-1.7298947 | \-0.3432376 | Nematoda         | terrestrial |
| \-1.4378194 | \-1.6667919 | Quimperiidae       | \-1.5523057 | \-0.2289725 | Nematoda         | freshwater  |
| \-1.4282523 | \-1.6560650 | Echeneibothriidae  | \-1.5421587 | \-0.2278127 | Platyhelminthes  | marine      |
| \-1.4087658 | \-1.6342150 | Tentaculariidae    | \-1.5214904 | \-0.2254492 | Platyhelminthes  | marine      |
| \-1.3020193 | \-1.6774560 | Oxyuridae          | \-1.4897377 | \-0.3754367 | Nematoda         | terrestrial |
| \-1.2755854 | \-1.6761728 | Phyllobothriidae   | \-1.4758791 | \-0.4005874 | Platyhelminthes  | marine      |
| \-1.2072917 | \-1.4452744 | Plagiorhynchidae   | \-1.3262830 | \-0.2379827 | Acanthocephala   | terrestrial |
| \-1.0599958 | \-1.3162838 | Quadrigyridae      | \-1.1881398 | \-0.2562880 | Acanthocephala   | freshwater  |
| \-0.9902716 | \-1.2387591 | Rhinebothriidae    | \-1.1145154 | \-0.2484875 | Platyhelminthes  | marine      |
| \-0.9666757 | \-1.1765235 | Tetrabothriidae    | \-1.0715996 | \-0.2098478 | Platyhelminthes  | marine      |
| \-0.8853761 | \-1.2484304 | Raphidascarididae  | \-1.0669032 | \-0.3630543 | Nematoda         | mixed       |
| \-0.8771178 | \-1.0764586 | Giganthorhynchidae | \-0.9767882 | \-0.1993408 | Acanthocephala   | terrestrial |
| \-0.8634970 | \-1.0612381 | Pomphorhynchidae   | \-0.9623675 | \-0.1977411 | Acanthocephala   | freshwater  |
| \-1.0518906 | \-0.8327727 | Heterakidae        | \-0.9423317 |   0.2191179 | Nematoda         | terrestrial |

</div>

Do habitat differences depend on the parasite group? When we add a
habitat by parasite group interaction to the model, it is not better
than the habitat-only model. This shouldn’t be too surprising, because
this cuts the data fairly thin.

<div class="kable-table">

|            | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :--------- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| mod2.1\_lc |   11 | 1018.761 | 1057.364 | \-498.3805 | 996.7610 |       NA | NA |          NA |
| mod2.2\_lc |   19 | 1031.157 | 1097.835 | \-496.5784 | 993.1568 | 3.604256 |  8 |   0.8909494 |

</div>

We can visualize this by separating the plot by phyla. Marine families
seem underrepresented regardless of group.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-53-1.png)<!-- -->

## Host type

Are over/underrepresented families likely to infect certain kinds of
final hosts? Let’s add final host type (invert, bird, mammal, etc) to
the model as a fixed effect.

A LRT suggests there may be a marginal effect of host type, and its
effect may vary among datasets (weak interaction).

<div class="kable-table">

|            | npar |      AIC |      BIC |     logLik |  deviance |    Chisq | Df | Pr(\>Chisq) |
| :--------- | ---: | -------: | -------: | ---------: | --------: | -------: | -: | ----------: |
| mod0\_lc   |    5 | 1026.202 | 1043.749 | \-508.1010 | 1016.2020 |       NA | NA |          NA |
| mod3\_lc   |   10 | 1023.915 | 1059.009 | \-501.9575 | 1003.9150 | 12.28693 |  5 |   0.0310609 |
| mod3.1\_lc |   15 | 1021.585 | 1074.226 | \-495.7926 |  991.5851 | 12.32990 |  5 |   0.0305365 |

</div>

But that effect may overlap the habitat effect, since the LRT is weaker
when adding host type to a model with habitat.

<div class="kable-table">

|            | npar |      AIC |      BIC |     logLik |  deviance |    Chisq | Df | Pr(\>Chisq) |
| :--------- | ---: | -------: | -------: | ---------: | --------: | -------: | -: | ----------: |
| mod2\_lc   |    8 | 1018.998 | 1047.073 | \-501.4991 | 1002.9983 |       NA | NA |          NA |
| mod3.2\_lc |   13 | 1019.014 | 1064.636 | \-496.5070 |  993.0139 | 9.984322 |  5 |   0.0756807 |

</div>

The model parameters suggest herps are weakly underrepresented in the
lcdb, whereas fish and mammals are under and overrepresented in the
recovery data, respectively.

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: cbind(n_spp, n_spp_ot - n_spp) ~ db + (db - 1 | parasite_family) +  
    ##     host_type_dh + db:host_type_dh
    ##    Data: filter(g_levx, n_spp_ot >= n_spp, parasite_family %in% fams_in_lcdb)
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   1021.6   1074.2   -495.8    991.6      232 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -1.1203 -0.4106 -0.1319  0.3684  2.1830 
    ## 
    ## Random effects:
    ##  Groups          Name   Variance Std.Dev. Corr
    ##  parasite_family dblcdb 1.296    1.138        
    ##                  dbrr   1.762    1.328    0.84
    ## Number of obs: 247, groups:  parasite_family, 122
    ## 
    ## Fixed effects:
    ##                                Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                     -2.3125     0.2670  -8.660  < 2e-16 ***
    ## dbrr                            -2.5103     0.3167  -7.926 2.26e-15 ***
    ## host_type_dhfish                -0.3093     0.3406  -0.908    0.364    
    ## host_type_dhherptile            -0.9062     0.5978  -1.516    0.130    
    ## host_type_dhinvertebrate        -0.9247     1.0999  -0.841    0.401    
    ## host_type_dhmammal               0.2847     0.3435   0.829    0.407    
    ## host_type_dhmixed                0.6299     0.4133   1.524    0.128    
    ## dbrr:host_type_dhfish           -0.6790     0.4727  -1.436    0.151    
    ## dbrr:host_type_dhherptile       -0.3164     0.9673  -0.327    0.744    
    ## dbrr:host_type_dhinvertebrate  -13.9724  1692.8033  -0.008    0.993    
    ## dbrr:host_type_dhmammal          0.5898     0.3877   1.521    0.128    
    ## dbrr:host_type_dhmixed           0.2825     0.4380   0.645    0.519    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## convergence code: 0
    ## unable to evaluate scaled gradient
    ##  Hessian is numerically singular: parameters are not uniquely determined

Here are the families split by host type. The dashed line is the overall
average. No groups seem obviously over or underrepresented.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-58-1.png)<!-- -->

It is also not clear that host type differences depend on the parasite
group; adding a host type by parasite group interaction to the model is
not an improvement.

<div class="kable-table">

|            | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :--------- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| mod3.1\_lc |   15 | 1021.585 | 1074.226 | \-495.7926 | 991.5851 |       NA | NA |          NA |
| mod3.3\_lc |   26 | 1029.470 | 1120.714 | \-488.7350 | 977.4700 | 14.11509 | 11 |   0.2266906 |

</div>

Here is the above plot separated by phyla. Any group that looks under or
overrepresented includes only a few families, so there is not a clear
pattern suggesting certain host types in certain helminth groups are
overrepresented in the database.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-60-1.png)<!-- -->

## Conclusions - biases in life cycle database relative to known helminth diversity

These analyses confirmed that there are taxonomic biases in the life
cycle database and our recovery data - some groups are over or
underrepresented. More intensely studied families were overrepresented
in the recovery data. Families from marine habitat were most
underrepresented, which is primarily due to the diverse shark tapeworms.
Mammal parasites were a bit overrepresented in the recovery data.

## Plots for supplement

Let’s fit the same models with `MCMCglmm`, which makes it easier to
extract confidence intervals for plotting, particularly for the random
effects.

After fitting models, we get predictions and start plotting. On the
plots, blue represents the model predictions. We first plot the families
most over and underrepresented in our comparative datasets.

Here are the overrepresented taxa.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-66-1.png)<!-- -->
Here are the underrepresented taxa.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-67-1.png)<!-- -->

Now we go through our predictors for whether a family is over or
underrepresented, starting with study effort. The blue area is the
predicted credible interval.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-70-1.png)<!-- -->
Here are the predictions as a function of habitat…

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-72-1.png)<!-- -->
…and as a function of host type.

![](explore_biases_fam_lev_files/figure-gfm/unnamed-chunk-74-1.png)<!-- -->
