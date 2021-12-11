---
title: "Tidyverse CREATE - Alec"
author: "Alec"
date: "10/24/2021"
output: 
  prettydoc::html_pretty:
    theme: cosmo
    highlight: github
    keep_md: true
    toc: true
    toc_depth: 2
    df_print: paged
---



## Introduction to Purrr

This vignette has been created to provide information on the tidyverse package: purrr. Purrr is a great package for analyzing and manipulating json data.

We will be using a League of Legends dataset, provided by Santiago Torres in project 2.

## Loading the Data


```r
library(rjson)
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──
```

```
## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
## ✓ tibble  3.1.5     ✓ dplyr   1.0.7
## ✓ tidyr   1.1.4     ✓ stringr 1.4.0
## ✓ readr   2.0.2     ✓ forcats 0.5.1
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(tidyr)
library(stringr)
library(jsonlite)
```

```
## 
## Attaching package: 'jsonlite'
```

```
## The following object is masked from 'package:purrr':
## 
##     flatten
```

```
## The following objects are masked from 'package:rjson':
## 
##     fromJSON, toJSON
```

```r
library(purrr)
```

# Load Data

Data is hosted on ddragon.leagueoflegends.com. We can use rjson::fromJSON to read and parse directly from the URL.

fromJSON will return a nested list structure containing all of the available json data


```r
champion_json <- rjson::fromJSON(file="https://ddragon.leagueoflegends.com/cdn/11.19.1/data/en_US/champion.json")
```


The champ_list is a list of 157 json objects. Each json object contains information about an available character in the online MOBA League of Legends. For instance, these json objects may hold the name of the character, descriptions of the character, or stats of the character. 


```r
champ_list <- champion_json$data
```

# Demonstrate value of Purrr package

We will be using the tidyverse "purrr" package to demonstrate how working with JSONs doesn't always need to be a chore.

## Extracting nested data - single values

One of the cool functions available in the purrr package is map(). Map() takes in two arguments:

- a list of jsons
- a key-value to extract

Below, we will extract all of the champion names into a new character vector called "champ_names"


```r
champ_names <- purrr::map(champ_list, "name")

champ_names[1:5]
```

```
## $Aatrox
## [1] "Aatrox"
## 
## $Ahri
## [1] "Ahri"
## 
## $Akali
## [1] "Akali"
## 
## $Akshan
## [1] "Akshan"
## 
## $Alistar
## [1] "Alistar"
```

## Extracting nested data - multiple values

What if we wanted to extract more than just the name of a champion? With purrr, we can do that as well. 

Say for instance we wanted to extract all champions' name, title, and blurb from the champion json. If we wanted to do this for a single json object, we could accomplish it like so:


```r
champ_list[[1]][c("name","title","blurb")]
```

```
## $name
## [1] "Aatrox"
## 
## $title
## [1] "the Darkin Blade"
## 
## $blurb
## [1] "Once honored defenders of Shurima against the Void, Aatrox and his brethren would eventually become an even greater threat to Runeterra, and were defeated only by cunning mortal sorcery. But after centuries of imprisonment, Aatrox was the first to find..."
```

In the above example, the square brackets surrounding the character vector is essentially a functional call, indexing the champlist object with the provided character vector. We can use this directly in purrr::map(.x, .f, ...). The function in this case is actually `[`!


```r
champ_list %>%
  purrr::map(`[`,c("name","title","blurb")) %>%
  .[2:3]
```

```
## $Ahri
## $Ahri$name
## [1] "Ahri"
## 
## $Ahri$title
## [1] "the Nine-Tailed Fox"
## 
## $Ahri$blurb
## [1] "Innately connected to the latent power of Runeterra, Ahri is a vastaya who can reshape magic into orbs of raw energy. She revels in toying with her prey by manipulating their emotions before devouring their life essence. Despite her predatory nature..."
## 
## 
## $Akali
## $Akali$name
## [1] "Akali"
## 
## $Akali$title
## [1] "the Rogue Assassin"
## 
## $Akali$blurb
## [1] "Abandoning the Kinkou Order and her title of the Fist of Shadow, Akali now strikes alone, ready to be the deadly weapon her people need. Though she holds onto all she learned from her master Shen, she has pledged to defend Ionia from its enemies, one..."
```

Make sure that when you enter the `[` you are NOT using quotes.


## Extracting data into a dataframe

map() is great for extracting data, but ultimately we likely will need to include that data into a more readable format than another list! That is where map_dfr (map dataframe) comes into play. Map_dfr() automatically converts the extracted nested list and converts to a dataframe


```r
champ_list %>%
  purrr::map_dfr(`[`,c("name","title","blurb")) %>%
  head()
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["name"],"name":[1],"type":["chr"],"align":["left"]},{"label":["title"],"name":[2],"type":["chr"],"align":["left"]},{"label":["blurb"],"name":[3],"type":["chr"],"align":["left"]}],"data":[{"1":"Aatrox","2":"the Darkin Blade","3":"Once honored defenders of Shurima against the Void, Aatrox and his brethren would eventually become an even greater threat to Runeterra, and were defeated only by cunning mortal sorcery. But after centuries of imprisonment, Aatrox was the first to find..."},{"1":"Ahri","2":"the Nine-Tailed Fox","3":"Innately connected to the latent power of Runeterra, Ahri is a vastaya who can reshape magic into orbs of raw energy. She revels in toying with her prey by manipulating their emotions before devouring their life essence. Despite her predatory nature..."},{"1":"Akali","2":"the Rogue Assassin","3":"Abandoning the Kinkou Order and her title of the Fist of Shadow, Akali now strikes alone, ready to be the deadly weapon her people need. Though she holds onto all she learned from her master Shen, she has pledged to defend Ionia from its enemies, one..."},{"1":"Akshan","2":"the Rogue Sentinel","3":"Raising an eyebrow in the face of danger, Akshan fights evil with dashing charisma, righteous vengeance, and a conspicuous lack of shirts. He is highly skilled in the art of stealth combat, able to evade the eyes of his enemies and reappear when they..."},{"1":"Alistar","2":"the Minotaur","3":"Always a mighty warrior with a fearsome reputation, Alistar seeks revenge for the death of his clan at the hands of the Noxian empire. Though he was enslaved and forced into the life of a gladiator, his unbreakable will was what kept him from truly..."},{"1":"Amumu","2":"the Sad Mummy","3":"Legend claims that Amumu is a lonely and melancholy soul from ancient Shurima, roaming the world in search of a friend. Doomed by an ancient curse to remain alone forever, his touch is death, his affection ruin. Those who claim to have seen him describe..."}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


## Extracting nested data, more than one level deep

Json objects may contain nested json objects. This nesting can theoretically continue down to many levels below the surface. In our dataset example, champions indeed have secondary levels to their data, including sub-lists such as "info", "image", and "stats".

In the below example, we will use the function map_chr() to extract the "HP" stat for each champion, which reside under the "stat" list for each champion json.


```r
champ_list %>%
  map_chr(c("stats", "hp")) %>%
  .[1:5]
```

```
##       Aatrox         Ahri        Akali       Akshan      Alistar 
## "580.000000" "526.000000" "500.000000" "560.000000" "600.000000"
```

## using keep(), the select_if for lists

Say you only wanted to analyze list objects based on some condition. purrr::keep() allows you to do this. You simply need to provide a list, and a conditional to follow.

In the below example, we will focus on analyzing only the champs that have a "HP" value above 500


```r
champ_list %>%
  map(c("stats","hp")) %>%
  keep(~ .x > 500) %>%
  .[1:5]
```

```
## $Aatrox
## [1] 580
## 
## $Ahri
## [1] 526
## 
## $Akshan
## [1] 560
## 
## $Alistar
## [1] 600
## 
## $Amumu
## [1] 615
```

## Reversing functions with negate()

This is a bit different than what we've explored so far, but purrr package also provides some useful functionality around customization of functions. 

Take for example the is.null() function. To create the opposite of this, we can use negate()


```r
lst <- list("a", 3, 22, NULL, "q", NULL)

is_not_null <- negate(is.null)

map_lgl(lst, is_not_null)
```

```
## [1]  TRUE  TRUE  TRUE FALSE  TRUE FALSE
```


## Using modify() and custom functions to clean data


The JSON file returned includes i18n columns that translates text. This function extracts the english version for each variable


```r
# extract .en version of a column and assign it to variable
translate <- function(.x) {
  ifelse((is.data.frame(.x) | is.list(.x)) && "en" %in% names(.x),
          return(.x$en),
          return(.x)
          )
}
```


Load data from the nobel prize api


```r
#load data from nobel prize api
raw_json = fromJSON("https://api.nobelprize.org/2.1/laureates",
                   simplifyDataFrame = TRUE
                   ) 
life_df  <- pluck(raw_json, "laureates") %>% as_tibble()
```


Call custom function to extract english column


```r
life_df <- modify(life_df, translate)
life_df$birth$place <- modify(life_df$birth$place, translate)
life_df$death$place <- modify(life_df$death$place, translate)
```


Clean-up and flatten data to create a tiddy record for an individual Nobel Laureates


```r
life_df <- life_df %>% select(id, knownName, familyName, gender, nobelPrizes, birth, death)

life_df <- life_df %>%
    mutate(
        birth_date = life_df$birth$date,
        birth_city = life_df$birth$place$cityNow,
        birth_country = life_df$birth$place$countryNow,
        death_date = life_df$death$date,
        death_city = life_df$death$place$cityNow,
        death_country = life_df$death$place$countryNow 
    )



life_df <- life_df %>%
    unnest_wider(nobelPrizes, simplify = TRUE) %>%
    unnest_wider(category, simplify = TRUE) %>%
    rename (
        award = en
    ) %>%
    select (-c(categoryFullName, motivation, affiliations, links, residences, birth, death, se, no))


life_df
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["id"],"name":[1],"type":["chr"],"align":["left"]},{"label":["knownName"],"name":[2],"type":["chr"],"align":["left"]},{"label":["familyName"],"name":[3],"type":["chr"],"align":["left"]},{"label":["gender"],"name":[4],"type":["chr"],"align":["left"]},{"label":["awardYear"],"name":[5],"type":["chr"],"align":["left"]},{"label":["award"],"name":[6],"type":["chr"],"align":["left"]},{"label":["sortOrder"],"name":[7],"type":["chr"],"align":["left"]},{"label":["portion"],"name":[8],"type":["chr"],"align":["left"]},{"label":["dateAwarded"],"name":[9],"type":["chr"],"align":["left"]},{"label":["prizeStatus"],"name":[10],"type":["chr"],"align":["left"]},{"label":["prizeAmount"],"name":[11],"type":["int"],"align":["right"]},{"label":["prizeAmountAdjusted"],"name":[12],"type":["int"],"align":["right"]},{"label":["birth_date"],"name":[13],"type":["chr"],"align":["left"]},{"label":["birth_city"],"name":[14],"type":["chr"],"align":["left"]},{"label":["birth_country"],"name":[15],"type":["chr"],"align":["left"]},{"label":["death_date"],"name":[16],"type":["chr"],"align":["left"]},{"label":["death_city"],"name":[17],"type":["chr"],"align":["left"]},{"label":["death_country"],"name":[18],"type":["chr"],"align":["left"]}],"data":[{"1":"745","2":"A. Michael Spence","3":"Spence","4":"male","5":"2001","6":"Economic Sciences","7":"2","8":"1/3","9":"2001-10-10","10":"received","11":"10000000","12":"12518033","13":"1943-00-00","14":"Montclair, NJ","15":"USA","16":"NA","17":"NA","18":"NA"},{"1":"102","2":"Aage N. Bohr","3":"Bohr","4":"male","5":"1975","6":"Physics","7":"1","8":"1/3","9":"1975-10-17","10":"received","11":"630000","12":"3465908","13":"1922-06-19","14":"Copenhagen","15":"Denmark","16":"2009-09-08","17":"Copenhagen","18":"Denmark"},{"1":"779","2":"Aaron Ciechanover","3":"Ciechanover","4":"male","5":"2004","6":"Chemistry","7":"1","8":"1/3","9":"2004-10-06","10":"received","11":"10000000","12":"11976161","13":"1947-10-01","14":"Haifa","15":"Israel","16":"NA","17":"NA","18":"NA"},{"1":"259","2":"Aaron Klug","3":"Klug","4":"male","5":"1982","6":"Chemistry","7":"1","8":"1","9":"1982-10-18","10":"received","11":"1150000","12":"3158777","13":"1926-08-11","14":"Zelvas","15":"Lithuania","16":"2018-11-20","17":"NA","18":"NA"},{"1":"1004","2":"Abdulrazak Gurnah","3":"Gurnah","4":"male","5":"2021","6":"Literature","7":"1","8":"1","9":"2021-10-07","10":"received","11":"10000000","12":"10000000","13":"1948-00-00","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA"},{"1":"114","2":"Abdus Salam","3":"Salam","4":"male","5":"1979","6":"Physics","7":"2","8":"1/3","9":"1979-10-15","10":"received","11":"800000","12":"3042231","13":"1926-01-29","14":"Jhang Maghiāna","15":"Pakistan","16":"1996-11-21","17":"Oxford","18":"United Kingdom"},{"1":"982","2":"Abhijit Banerjee","3":"Banerjee","4":"male","5":"2019","6":"Economic Sciences","7":"1","8":"1/3","9":"2019-10-14","10":"received","11":"9000000","12":"9000000","13":"1961-02-21","14":"Mumbai","15":"India","16":"NA","17":"NA","18":"NA"},{"1":"981","2":"Abiy Ahmed Ali","3":"Ahmed Ali","4":"male","5":"2019","6":"Peace","7":"1","8":"1","9":"2019-10-11","10":"received","11":"9000000","12":"9000000","13":"1976-08-15","14":"Beshasha","15":"Ethiopia","16":"NA","17":"NA","18":"NA"},{"1":"843","2":"Ada E. Yonath","3":"Yonath","4":"female","5":"2009","6":"Chemistry","7":"3","8":"1/3","9":"2009-10-07","10":"received","11":"10000000","12":"11157218","13":"1939-06-22","14":"Jerusalem","15":"Israel","16":"NA","17":"NA","18":"NA"},{"1":"866","2":"Adam G. Riess","3":"Riess","4":"male","5":"2011","6":"Physics","7":"3","8":"1/4","9":"2011-10-04","10":"received","11":"10000000","12":"10736783","13":"1969-12-16","14":"Washington, DC","15":"USA","16":"NA","17":"NA","18":"NA"},{"1":"199","2":"Adolf Butenandt","3":"Butenandt","4":"male","5":"1939","6":"Chemistry","7":"1","8":"1/2","9":"NA","10":"received","11":"148822","12":"4304564","13":"1903-03-24","14":"Bremerhaven-Lehe","15":"Germany","16":"1995-01-18","17":"Munich","18":"Germany"},{"1":"164","2":"Adolf von Baeyer","3":"von Baeyer","4":"male","5":"1905","6":"Chemistry","7":"1","8":"1","9":"NA","10":"received","11":"138089","12":"7753291","13":"1835-10-31","14":"Berlin","15":"Germany","16":"1917-08-20","17":"Starnberg","18":"Germany"},{"1":"185","2":"Adolf Windaus","3":"Windaus","4":"male","5":"1928","6":"Chemistry","7":"1","8":"1","9":"NA","10":"received","11":"156939","12":"4539342","13":"1876-12-25","14":"Berlin","15":"Germany","16":"1959-06-09","17":"Göttingen","18":"Germany"},{"1":"541","2":"Adolfo Pérez Esquivel","3":"Pérez Esquivel","4":"male","5":"1980","6":"Peace","7":"1","8":"1","9":"1980-10-27","10":"received","11":"880000","12":"2942067","13":"1931-11-26","14":"Buenos Aires","15":"Argentina","16":"NA","17":"NA","18":"NA"},{"1":"292","2":"Ahmed Zewail","3":"Zewail","4":"male","5":"1999","6":"Chemistry","7":"1","8":"1","9":"1999-10-12","10":"received","11":"7900000","12":"10231411","13":"1946-02-26","14":"Damanhur","15":"Egypt","16":"2016-08-02","17":"Pasadena, CA","18":"USA"},{"1":"853","2":"Akira Suzuki","3":"Suzuki","4":"male","5":"2010","6":"Chemistry","7":"3","8":"1/3","9":"2010-10-06","10":"received","11":"10000000","12":"11015580","13":"1930-09-12","14":"Mukawa","15":"Japan","16":"NA","17":"NA","18":"NA"},{"1":"978","2":"Akira Yoshino","3":"Yoshino","4":"male","5":"2019","6":"Chemistry","7":"3","8":"1/3","9":"2019-10-09","10":"received","11":"9000000","12":"9000000","13":"1948-01-30","14":"Suita","15":"Japan","16":"NA","17":"NA","18":"NA"},{"1":"819","2":"Al Gore","3":"Gore","4":"male","5":"2007","6":"Peace","7":"2","8":"1/2","9":"2007-10-12","10":"received","11":"10000000","12":"11506932","13":"1948-03-31","14":"Washington, DC","15":"USA","16":"NA","17":"NA","18":"NA"},{"1":"729","2":"Alan Heeger","3":"Heeger","4":"male","5":"2000","6":"Chemistry","7":"1","8":"1/3","9":"2000-10-10","10":"received","11":"9000000","12":"11538617","13":"1936-01-22","14":"Sioux City, IA","15":"USA","16":"NA","17":"NA","18":"NA"},{"1":"376","2":"Alan Hodgkin","3":"Hodgkin","4":"male","5":"1963","6":"Physiology or Medicine","7":"2","8":"1/3","9":"NA","10":"received","11":"265000","12":"2890771","13":"1914-02-05","14":"Banbury","15":"United Kingdom","16":"1998-12-20","17":"Cambridge","18":"United Kingdom"},{"1":"730","2":"Alan MacDiarmid","3":"MacDiarmid","4":"male","5":"2000","6":"Chemistry","7":"2","8":"1/3","9":"2000-10-10","10":"received","11":"9000000","12":"11538617","13":"1927-04-14","14":"Masterton","15":"New Zealand","16":"2007-02-07","17":"Drexel Hill, PA","18":"USA"},{"1":"11","2":"Albert A. Michelson","3":"Michelson","4":"male","5":"1907","6":"Physics","7":"1","8":"1","9":"NA","10":"received","11":"138796","12":"7161123","13":"1852-12-19","14":"Strzelno","15":"Poland","16":"1931-05-09","17":"Pasadena, CA","18":"USA"},{"1":"628","2":"Albert Camus","3":"Camus","4":"male","5":"1957","6":"Literature","7":"1","8":"1","9":"NA","10":"received","11":"208629","12":"2746709","13":"1913-11-07","14":"Mondovi","15":"Algeria","16":"1960-01-04","17":"Sens","18":"France"},{"1":"403","2":"Albert Claude","3":"Claude","4":"male","5":"1974","6":"Physiology or Medicine","7":"1","8":"1/3","9":"NA","10":"received","11":"550000","12":"3322627","13":"1898-08-24","14":"Longlier","15":"Belgium","16":"1983-05-22","17":"Brussels","18":"Belgium"},{"1":"26","2":"Albert Einstein","3":"Einstein","4":"male","5":"1921","6":"Physics","7":"1","8":"1","9":"NA","10":"received","11":"121573","12":"2578698","13":"1879-03-14","14":"Ulm","15":"Germany","16":"1955-04-18","17":"Princeton, NJ","18":"USA"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>



