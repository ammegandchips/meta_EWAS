Add the module "random-metal", which is METAL adapted by Gibran Hemani (github.com/explodecomputer) to run random effects meta-analyses and/or print statistics to more than 4 decimal places
```
module add tools/git-1.8.4.2
git clone https://github.com/explodecomputer/random-metal
cd random-metal
make
random-metal/executables/metal
metal
```
To run (from home directory)
```
random-metal/executables/metal
```
An example running a fixed effects meta-analysis:
```
REMOVEFILTERS
SCHEME STDERR
USESTRAND OFF
GENOMICCONTROL OFF
AVERAGEFREQ OFF
MINMAXFREQ OFF
COLUMNCOUNTING LENIENT
SEPARATOR COMMA
PVALUELABEL p.pat
MARKER probe
EFFECTLABEL coef.pat
STDERRLABEL se.pat
OUTFILE /panfs/panasas01/sscm/gs8094/EWAS/pat_bmi/meta/full.pat .txt

PROCESSFILE      /panfs/panasas01/sscm/gs8094/EWAS/pat_bmi/alspac/results/prepared/ALSPAC.ewas.results.birth.full.csv
PROCESSFILE      /panfs/panasas01/sscm/gs8094/EWAS/pat_bmi/goya/results/prepared/GOYA.ewas.results.birth.full.csv

ANALYZE HETEROGENEITY
CLEAR
```
An example running a random effects meta-analysis:
```
REMOVEFILTERS
SCHEME STDERR
USESTRAND OFF
GENOMICCONTROL OFF
AVERAGEFREQ OFF
MINMAXFREQ OFF
COLUMNCOUNTING LENIENT
SEPARATOR COMMA
PVALUELABEL p.pat
MARKER probe
EFFECTLABEL coef.pat
STDERRLABEL se.pat
OUTFILE /panfs/panasas01/sscm/gs8094/EWAS/pat_bmi/meta/full.pat.random .txt

PROCESSFILE      /panfs/panasas01/sscm/gs8094/EWAS/pat_bmi/alspac/results/prepared/ALSPAC.ewas.results.birth.full.csv
PROCESSFILE      /panfs/panasas01/sscm/gs8094/EWAS/pat_bmi/goya/results/prepared/GOYA.ewas.results.birth.full.csv

ANALYZE RANDOM
CLEAR
```

