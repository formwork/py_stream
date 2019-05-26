py\_stream
================

Setup reticulate from R

``` r
library(reticulate)
use_python("/anaconda3/bin/python", required = TRUE)
py_config()
```

    ## python:         /anaconda3/bin/python
    ## libpython:      /anaconda3/lib/libpython3.7m.dylib
    ## pythonhome:     /anaconda3:/anaconda3
    ## version:        3.7.3 (default, Mar 27 2019, 16:54:48)  [Clang 4.0.1 (tags/RELEASE_401/final)]
    ## numpy:          /anaconda3/lib/python3.7/site-packages/numpy
    ## numpy_version:  1.16.2
    ## 
    ## NOTE: Python version was forced by use_python function

``` python
import csv
import pandas as pd
from collections import defaultdict
```

### Sample to percentage of original lines

Function to sample line-by-line

``` python
def csv_sample(infile, outfile, prob, D = 2 ** 20):
    
    with open(infile, 'r') as f, open(outfile, 'w+') as o:
        for i, line in enumerate(f):
            if i == 0: 
                o.write(line)
            elif (hash(line) % D) / D < prob:
                o.write(line)
```

Try it

``` python
csv_sample(infile = "NSPL_FEB_2019_UK.csv",
           outfile = "nspl_py_sample.csv",
           prob = 1/200000)
```

Check results with pandas

``` python
pd.read_csv("nspl_py_sample.csv", usecols = ['pcds', 'ctry', 'imd'])
```

    ##         pcds       ctry    imd
    ## 0   AB52 6SA  S92000003   5730
    ## 1    B77 9JP  E92000001  20305
    ## 2   BD23 1NN  E92000001  28051
    ## 3   CH49 9AB  E92000001  10574
    ## 4   DY14 0TX  E92000001  16128
    ## 5   EX11 1BL  E92000001  20897
    ## 6    FK1 1BS  S92000003   4493
    ## 7   GU17 8DG  E92000001  32635
    ## 8    IG6 2RL  E92000001  15934
    ## 9    LS2 8DR  E92000001   8358
    ## 10  MK18 3PB  E92000001  27135
    ## 11  MK18 5NR  E92000001  19933
    ## 12  OX33 1UX  E92000001  28369
    ## 13   OX5 2UQ  E92000001  29171
    ## 14  PL26 8TG  E92000001   6337
    ## 15  SA62 5BN  W92000004   1165
    ## 16  WA14 1FX  E92000001  25782
    ## 17  WA14 3BU  E92000001  32404
    ## 18   WN1 2FA  E92000001   3645

### Sample to every nth line

``` python
def csv_modline(infile, outfile, n):
    
    with open(infile, 'r') as f, open(outfile, 'w+') as o:
        for i, line in enumerate(f):
            if (i % n) == 0: o.write(line)
```

``` python
csv_modline(infile = "NSPL_FEB_2019_UK.csv",
            outfile = "nspl_py_modline.csv",
            n = 200000)
```

``` python
pd.read_csv("nspl_py_modline.csv", usecols = ['pcds', 'ctry', 'imd'])
```

    ##         pcds       ctry    imd
    ## 0    BL3 2QZ  E92000001   4582
    ## 1   CF14 0XG  W92000004   1874
    ## 2    DD3 8LH  S92000003   5920
    ## 3    EN3 4XX  E92000001   4451
    ## 4    HG2 9PP  E92000001  32316
    ## 5    L63 3BQ  E92000001  31894
    ## 6   ME13 8NJ  E92000001  23546
    ## 7   NP24 6EP  W92000004     70
    ## 8    PL6 7NG  E92000001  24850
    ## 9    SA2 7NZ  W92000004   1820
    ## 10  SS12 0PP  E92000001  21092
    ## 11   TS7 0XU  E92000001  28535
    ## 12  YO19 6LR  E92000001  25852

### Tally field entries via defaultdict

``` python
def csv_histogram(infile, idx):

    hist = defaultdict(int)

    with open(infile, 'r') as f:
        f.readline()
        for i, line in enumerate(f):
            entry = line.split(",")[idx]
            hist[entry] += 1

    for item in hist:
        print(item, "=", hist[item])
```

``` python
csv_histogram(infile = "NSPL_FEB_2019_UK.csv", idx = 16)
```

    ## "S92000003" = 223270
    ## "" = 9808
    ## "E92000001" = 2181456
    ## "N92000002" = 59004
    ## "W92000004" = 138112
    ## "L93000001" = 6910
    ## "M83000003" = 6025

### Using csv.DictReader

``` python
def csv_histogram(infile, field):

    hist = defaultdict(int)
    f = csv.DictReader(open(infile, 'r'))
    
    for row in f:
        entry = row[field]
        hist[entry] += 1

    for item in hist:
        print(item, "=", hist[item])
```

``` python
csv_histogram(infile = "NSPL_FEB_2019_UK.csv", field = "ctry")
```

    ## S92000003 = 223270
    ##  = 9808
    ## E92000001 = 2181456
    ## N92000002 = 59004
    ## W92000004 = 138112
    ## L93000001 = 6910
    ## M83000003 = 6025

### Principles of KMV sketch for a chosen field

``` python
def csv_kmv(infile, idx, k = 256, D = 2 ** 20):

    kmv = set()

    with open(infile, 'r') as f:
        
        f.readline() #header
        
        for i, line in enumerate(f):
            entry = line.split(",")[idx]
            h_val = hash(entry) % D
            
            if len(kmv) <= k:
                kmv.add(h_val)
                maxhash = max(kmv)
            
            elif h_val < maxhash:
                if h_val not in kmv:
                    kmv.remove(maxhash)
                    kmv.add(h_val)
                    maxhash = max(kmv)
    
    return k / (maxhash / D)
            
```

``` python
csv_kmv("NSPL_FEB_2019_UK.csv", idx = 9)
```

    ## 220752.84210526315

``` python
nspl_oa = pd.read_csv("NSPL_FEB_2019_UK.csv", usecols = ['oa11'])

nspl_oa.oa11.nunique()
```

    ## 232034
