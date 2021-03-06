---
title: "FoF - DATA"
author: Rafael Cattan
output:
  prettydoc::html_pretty:
    theme: leonids
    fig_width: 8
    fig_height: 6
geometry: "left=3cm,right=3cm,top=2cm,bottom=2cm"
---

```{r include=FALSE}
library(readxl)
library(plotly)
library(kableExtra)
library(likert) 
library(RColorBrewer)
library(shiny)
library(dplyr)
library(magrittr)
library(knitr)
library(prettydoc)
library(tibble)


```


## Table of Contents
1. [Consumption](#consumption)
2. [Intermediary Consumption](#intermediary)
3. [Imports](#imports)
4. [Exports](#exports)
5. [Gov.Transfers](#transfers)

    5.1.[Social Benefits](#socben)
    
    5.2.[Cash Transfers](#cash)
    
    5.3[Employers' social contributions](#employ)
    
6. [Investment](#investment)
7. [Stocks Variation](#stock)
8. [Taxes](#taxes)
9. [Wages](#wages)
10.[Property Income](#property)
    
    10.1. [Interests](#interest)
    
    10.2. [Dividends](#dividends)
    
    10.3. [Others](#others)
    
11. [Interests](#interests)
12. [Dividends](#dividends)
13. [Adjusting variables](#adjusting)




## Data Gathering <a name="data"></a>

As IBGE's data is organized in MS Excel format the first task was downloading CEI database. After dowloanding it to own directory, they were changed in order to make them readable on R programming. For that, the following steps were taken:

a) Gathering all excel sheets (one sheet per year) on a single excel file;

b) Cleaning the input formats, keeping all cells on 'number' format;

c) Deleting empty rows and columns to avoid N.A's problems;

With all excel sheets ready to read in R, we have used a function to read all sheets at once, creating a nested list object of length 15, each one representing an year comprising the period 2000-2014. The following code was used to do the reading and create the nested list object:


```{r echo=TRUE, message=FALSE, warning=FALSE, paged.print=FALSE}


# function to read all sheets
read_excel_allsheets <- function(filename, tibble = FALSE) {

  sheets <- readxl::excel_sheets(filename)
  x <- lapply(sheets, function(X) readxl::read_excel(filename, sheet = X))
  if(!tibble) x <- lapply(x, as.data.frame)
  names(x) <- sheets
  x
}

# reading all excel sheets

mysheets <- read_excel_allsheets("C:/Users/Helena Souto/Documents/CEI/cei_clean/teste1.xlsx")


```


Having this object created, the next step was separating Resources (each institutional agent income sources) from Uses (agents' expenditures), in order to have their net position for each account described by the CEI.

```{r message=FALSE, warning=FALSE, include=FALSE}


# separating uses and resouces for each list
u<-list(NA)
r<-list(NA)


u<-lapply(mysheets,"[",1:9)
r<-lapply(mysheets,"[",12:20)

```


In order to proceed with the net values calculation operation, the lists were transformed into matrix. To facilitate the matricial operation, we had to reverse Resources' columns order and, then, subtracted Resources from Uses to get each institutional sector net position. 


```{r message=FALSE, warning=FALSE, include=FALSE}
# reversing resources order to matrix operation
inv_r<-list()

for (i in 1:length(r)){
  
inv_r[[i]]<-rev(r[[i]])

}

```

Besides the six institutional sectors - Families, Financial firms, Non-Financial firms (hereafter Firms), Government, Rest of the World (Row), and Non-governmental Institutions (NGOs) - these new matrices' columns are composed of Rest of the world's Goods and Services, Total of the economy and Domestic totals, totalizing nine columns. All these sectoral cathegories are standardly displayed on CEI data-set. 

```{r message=FALSE, warning=FALSE, include=FALSE}

# creating net position matrix

s<-list()

for(i in 1:length(u)){
  
  s[[i]]<-matrix(as.numeric(unlist(inv_r[[i]])),ncol=9)-matrix(as.numeric(unlist(u[[i]])),ncol=9)
  

}



```

The resulting object is a list of matrices, each one with `r nrow(s[[1]])` or `r nrow(s[[15]])` rows (from 2010 until 2014 there are more accounts included) and `r ncol(s[[1]])` columns (sectors). As this process was done for each year (2000 to 2010), we have produced `r length(s)` different matrices. 


Keeping net values matrices nested on a list object was helpful to visualize the data strucure and validate the values according to the MS Excel sheets. However, in order to better manipulate the values and proceed with data analysis, data frames objects are better suited, as there are multiple functions available for its manipulation. Before proceeding with the data-frame transformation all rows (CEI's accounts) and columns (sectors) were named, facilitating the code reading process.


```{r message=FALSE, warning=FALSE, include=FALSE}

#col names
coln<-(c("Total",
"Goods_Services",
"RoW",
"Total_Income",
"NGOs",
"Households",
"Government",
"Banks",
"Firms"))



#naming
s <-lapply(s, function(x) {
  colnames(x) <- coln
  return(x)
})






# row names

ms1<-list()

  
  for (i in 1:length(s)){
    
    ms1[[i]]<-mysheets[[i]][,11]
    
}
  




# complete data set named
s2<-list()

for(i in 1:length(s)){
    
    s2[[i]]<-`rownames<-`(s[[i]],ms1[[i]])
    
}


```


Next, the data frame object was created, together with a "years" column, proving a panel dataset.


```{r echo=FALSE, message=FALSE, warning=FALSE}


# data.framing the list objetc
df_s2<-do.call(rbind.data.frame, s2)



# creating years col (ps:brute force by now)

# 2000-09
y00<-rep(2000,58)
y01<-rep(2001,58)
y02<-rep(2002,58)
y03<-rep(2003,58)
y04<-rep(2004,58)
y05<-rep(2005,58)
y06<-rep(2006,58)
y07<-rep(2007,58)
y08<-rep(2008,58)
y09<-rep(2009,58)

# 2010-2014
y10<-rep(2010,146)
y11<-rep(2011,146)
y12<-rep(2012,146)
y13<-rep(2013,146)
y14<-rep(2014,146)


ys<-(cbind(y00,
           y01,
           y02,
           y03,
           y04,
           y05,
           y06,
           y07,
           y08,
           y09
))



ys2<-cbind(y10,
           y11,
           y12,
           y13,
           y14)




# 2000-2014 years matrix
y<-matrix(c(ys,ys2),ncol=1)

colnames(y)<-"years"



# complete data-set

y_df<-data.frame(df_s2,y)



#

  
names(y_df)<-c(coln,"Years")




# cleaning names
rownames(y_df) <- gsub(x = rownames(y_df),
                        pattern = "\\.",
                        replacement = " ")




#removing ngos

Households<-y_df$NGOs+y_df$Households

ydf2<-y_df[,-c(which(colnames(y_df)=="Households"),
         which(colnames(y_df)=="NGOs"))]
       
y_df<-cbind(ydf2,Households)

y_df<-y_df[,c(names(y_df[,-which(colnames(y_df)=="Years")]),
        "Years")]
```


The resulting object is a data frame of dimension `r dim (y_df)[1]` (58 x 10 + 146 x 5) by `r dim (y_df)[2]`, ready to use and manipulate. In order to build a the data-based SFC model however, we had to choose the following transaction variables:


    1. Production  

    2. Intermediary Consumption

    3. Imports

    4. Exports

    5.Government Transfers

      5.1.Social Benefits
      5.2.Cash Transfers
      5.3.Employer's social contribution
    
    6. Investment

    7. Stock Variation

    8.Taxes

      8.1. Production Taxes
      8.2. Income Taxes 
      8.3. Social Contributions
      8.4. Other taxes
    
    9. Wages

    10. Profits

    11. Capital Depreciation

    12.Property Income

      12.1. Interests
      12.2. Dividends
      12.3. Others


    13. Adjusting accounts

      1. Pension funds's adjustments
      2. Net Capital Aquisitions
      3. Capital Transfers
    

These variables, together, are sufficient to build a consistent flow of funds for the Brazilian economy. In order to properly build a SFC model with this data, it is helpful to visualize these items in terms of the Flow of Funds' and the Balance Sheet matrices. The signs displayed are based on the real data. Upperscripts represent's the financial asset issuer. The holder is, naturally, within its own column, as one can see below

!(https://github.com/rafaelcattan/fofdata.git/fofv2.png)
    









## Production

```{r echo=FALSE}

# THE DATA
y1<-y_df[grep("^Produ??o",
          rownames(y_df)),][1:10,]




y2<-y_df[grep("^Produ??o1",
          rownames(y_df)),][-1,]


y<-rbind(y1,y2)




#THE TABLE
kable(round(y,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```



 
 
## Consumption <a name="consumption"></a>
```{r echo=FALSE}



# THE DATA
cons<-y_df[grep("^Despesa de consumo final",
          rownames(y_df)),]




#TABLE FORMAT
knitr::kable(round(cons,2))%>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)




```

## Intermediary Consumption <a name="intermediary"></a> 

```{r echo=FALSE}

# THE DATA
int_cons<-y_df[grep("^Consumo intermedi?rio",
          rownames(y_df)),]


#THE TABLE
kable(round(int_cons,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```



## Imports <a name="imports"></a> 

```{r echo=FALSE}

# THE DATA
M<-y_df[grep("^Importa??o",
          rownames(y_df)),]


#THE TABLE
knitr::kable(round(M,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```






## Exports <a name="exports"></a>

```{r echo=FALSE}

# THE DATA
X<-y_df[grep("^Exporta??o",
          rownames(y_df)),]


#THE TABLE
kable(round(X,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```




## Government Transfers <a name="transfers"></a>



### Social Benefits <a name="socben"></a>


```{r echo=FALSE}
t_soc_ben<-y_df[grep("^Benef?cios sociais",
          rownames(y_df)),]



kable(round(t_soc_ben,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)

```


### Cash transfers <a name="cash"></a>


```{r echo=FALSE}
t_cash<-y_df[grep("^Transfer?ncias sociais em esp?cie",
          rownames(y_df)),]



kable(round(t_cash,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)

```


### Employers' social contributions <a name="employ"></a>


```{r echo=FALSE}
t_employ<-y_df[grep("^Contribui??es sociais dos empregadores",
          rownames(y_df)),]



kable(round(t_employ,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)

```









## Investment <a name="investment"></a> 

```{r echo=FALSE}

# THE DATA
gfkf<-y_df[grep("^Forma??o bruta de capital",
          rownames(y_df)),]


#THE TABLE
kable(round(gfkf,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```





## Stock Variation <a name="stock"></a> 

```{r echo=FALSE}

# THE DATA
stocks<-y_df[grep("^Varia??o de estoques",
          rownames(y_df)),]


#THE TABLE
kable(round(stocks,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```







## Taxes <a name="taxes"></a>

### Production Taxes


Impostos l?quidos de subs?dios sobre produtos

```{r echo=FALSE}

# THE DATA
t_y<-y_df[grep("^Impostos  l?quidos de subs?dios  sobre a produ??o",
          rownames(y_df)),]



#THE TABLE
kable(round(t_y,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```

### Income Taxes 


```{r echo=FALSE}
t_inc<-y_df[grep("^Impostos correntes sobre a renda",
          rownames(y_df)),]

kable(round(t_inc,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)

```



### Social Contributions


```{r echo=FALSE}
t_s_cont<-y_df[grep("^Contribui??es sociais",
          rownames(y_df)),]


t_s_cont<-t_s_cont[seq(4,nrow(t_s_cont),7),]





kable(round(t_s_cont,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)

```


### Other transfers 


```{r echo=FALSE}
t_other<-y_df[grep("^Outras transfer?ncias",
          rownames(y_df)),]



kable(round(t_other,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)

```





## Wages <a name="wages"></a> 

```{r echo=FALSE}

# THE DATA
wages<-y_df[c(grep("^Ordenados e sal?rios",
          rownames(y_df)),
          grep("^Sal?rios",
               rownames(y_df))),]


#THE TABLE
kable(round(wages,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```







## Profits <a name="profits"></a>


```{r include=FALSE}
# 2 value added all

## [2] [Value Added]


# getting all resources from all sectors
mat_r<-list()

for(i in 1:length(inv_r)){
  
  mat_r[[i]]<-matrix(as.numeric(unlist(inv_r[[i]])),ncol=9)
  
}



#naming resources cols

mat_r <-lapply(mat_r, function(x) {
  colnames(x) <- coln
  return(x)
})



#namings rows
mat_r_2<-list()

for(i in 1:length(s)){
  
  mat_r_2[[i]]<-`rownames<-`(mat_r[[i]],ms1[[i]])
  
}




# complete resources (all sectors)
df_rsc<-do.call(rbind.data.frame, mat_r_2)



# cleaning names
rownames(df_rsc) <- gsub(x = rownames(df_rsc),
                        pattern = "\\.",
                        replacement = " ")







all_va<-df_rsc[grep("Valor adicionado bruto Produto interno bruto",
                    rownames(df_rsc)),]

hh_dfrsc<-df_rsc$Households+df_rsc$NGOs


dfrsc2<-df_rsc[,-c(which(colnames(df_rsc)=="Households"),
                   which(colnames(df_rsc)=="NGOs"))]


df_rsc<-cbind(dfrsc2,"Households"= hh_dfrsc)

```




```{r echo=FALSE}

# THE DATA

profits<-df_rsc[grep("^Excedente operacional bruto",
          rownames(df_rsc)),]




#THE TABLE
kable(round(profits,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```








## Capital Depreciation - not found

On Daniel's table but not found on CEI.






##Property Income <a name="property"></a>

### Interests  <a name="interest"></a>
```{r echo=FALSE}

# THE DATA

interests<-y_df[grep("^Juros",
          rownames(y_df)),]




#THE TABLE
kable(round(interests,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```

### Dividends  <a name="dividends"></a>
```{r echo=FALSE}

# THE DATA

dividends<-y_df[grep("^Dividendos",
          rownames(y_df)),]




#THE TABLE
kable(round(dividends,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)


```





### Other <a name="other"></a>

```{r echo=FALSE}

# THE DATA

prop_other1<-y_df[grep("^Desembolsos por rendas",
          rownames(y_df)),]




prop_other2<-y_df[grep("^Renda de recursos",
          rownames(y_df)),]



prop_other<-prop_other1+prop_other2


#THE TABLE
kable(round(prop_other,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)




```


## Adjusting variables <a name="adjusting"></a>


### Pension adjustment
```{r}
pens_adjust<-y_df[grep("^Ajustamento pela varia??o",
                       rownames(y_df)),]


```



### Net Capital Aquisitions

```{r}
net_aqu<-y_df[grep("^Aquisi??es l?quida",
                       rownames(y_df)),]

```






### Capital transfers


```{r}

k_inc<-y_df[grep("^Transfer?ncias de capital a receber",
                       rownames(y_df)),]

k_exp<-y_df[grep("^Transfer?ncias de capital a pagar",
                       rownames(y_df)),]


# Pension adjustment + capital transfers + net aqu

k_trans<-k_inc+k_exp+net_aqu+pens_adjust


kable(round(k_trans,2)) %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                font_size = 11)



```















## Checkings


```{r eval=FALSE, include=FALSE}

# [1] V.A TOTAL = production - CI - prod.taxes

va1<-matrix(y$Total_Income)+matrix(int_cons$Total_Income) +df_rsc$Total[5]

#result: ok!



# [2] NFF all balances:

# [2.1] V.A: OK!

nff_va<-y$Firms+int_cons$Firms
nff_va





# [2.2] EOB: VA+ W+ employers soc.cont + prod.taxes 
#result ok!


nff_eob<-nff_va+wages$Firms+t_employ$Firms+t_y$Firms




#[2.3] Prim.Inc.Balance:nffeob + interests +dividends+desembolsos + natural resources inc.
#result: ok!

nff_pincbal<-nff_eob+interests$Firms+dividends$Firms+prop_other$Firms



# [2.4] Gross Disp. INc = pincbal + inc.taxes + soc.cont + other transf 
#result:ok!

nff_gdi<-nff_pincbal+t_inc$Firms+t_s_cont$Firms+t_soc_ben$Firms+t_other$Firms


#[2.5] Net Financial needs/capacity: 
# gdi+c+fbkf+dstocks+ pension adjust +net aquisition of non financial assets + capital transfers to receive + capital transfers to pay

#result: ok!

nff_nfn<-nff_gdi+cons$Firms+gfkf$Firms+stocks$Firms+pens_adjust$Firms+k_trans$Firms

nff_nfn


```





