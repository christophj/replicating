---
date: '2013-02-03'
title: How to produce nice tables in PDFs using knitr/Sweave and R
categories: ['R', 'tutorial']
tags: [R, tutorial]
---




In this post, I want to show you how to produce nice tables in PDFs, even if you use [knitr](http://yihui.name/knitr/) or [Sweave](http://www.stat.uni-muenchen.de/~leisch/Sweave/) to produce your reports dynamically. Why should you use tools for  reproducible research in the first place? Well, it guarantees that you always know how you did your analysis. I mean if someone came up to me today and asked me how I computed the mean on page 52 of my diploma thesis, it would take me probably hours to figure that out (or maybe I couldn't figure it out anymore at all). When someone asks me how I computed the mean of one of my papers written during my PhD, I have a look at my knitr document and could tell him in minutes. That's the beauty of it, so you should definitely check it out if you don't use such a tool so far.

However, one thing that bothered me for a while was that the tables produced didn't really look great. I mean they had all necessary information in it, but I just like tables that look good and with LaTex (knitr or Sweave are just built on top of LaTex, so you still use that) it is normally quite easy to make tables look great, for instance by using the package **booktabs**. In my early knitr days, I just edited the .tex file produced by knitr, but this seemed like a quick and dirty hack that was prone to non-reproducible errors (for instance, you delete one row in the table). That's what you want to get rid of when using those tools, so I figured out how to edit the tables in the source .Rnw file. This is what I want to show you here with a small minimal example.

There are two key tricks that we have to use:

1. The option *add.to.row* in the function *print.xtable* allows us to enter strings before or after certain rows in your table. 
2. The backslash is a special character in R. For instance, if you want to get a line break you type "\n", which does not actually print that string, but inserts a line break. However, in tables we actually want to enter backslashes at the end of rows because two backslashes break a row there. So how do we do that? We just write four backslashes: the first backslash is then considered as a special character, telling R that the next character should be considered as a normal character, not as a special character. So in this case, the backslash should just be printed. Since we need two backslashes, we have to do that twice. I know, it sounds complicated, but it's quite similar to the percentage sign in LaTex. You can't just write % because this tells LaTex that it should be a comment. To actually get the percentage sign, you have to write \%.

OK, now we have the basics, so let's actually produce a nice table. In R, you need to load the **xtable** package and in LaTex, you need to load the **booktabs** package. Also, I use the package **caption**; otherwise, the caption is too close to the table. 

Now imagine we want to compare three different regression models (rows) and want to print in the columns the $\alpha$, the $\beta$, the t-value of the $\beta$ coefficient, and the adjusted $R^2$. With randomly drawn data, our minimal example looks like this. 

Here is the source code of the minimal example. Save it as a .Rnw file, *knit* that file and you should get a nice [PDF]({{urls.theme}}/media/Tables_in_R/Minimal_example.pdf).


```r
\begin{document}

Here is our minimal example:

<<Code_chunk_Minimal_example, results='asis', echo=FALSE>>=
library(xtable)
#Just some random data
x1 <- rnorm(1000); x2 <- rnorm(1000); x3 <- rnorm(1000)
y  <- 2 + 1 *x1 + rnorm(1000)
#Run regressions
reg1 <- summary(lm(y ~ x1))
reg2 <- summary(lm(y ~ x2))
reg3 <- summary(lm(y ~ x3))
#Create data.frame
df <- data.frame(Model = 1:3,
                 Alpha = c(reg1$coef[1,1], reg2$coef[1,1], reg3$coef[1,1]),
                 Beta  = c(reg1$coef[2,1], reg2$coef[2,1], reg3$coef[2,1]),
                 tV    = c(reg1$coef[2,2], reg2$coef[2,2], reg3$coef[2,2]),
                 AdjR  = c(reg1$adj.r.s,  reg2$adj.r.s,   reg3$adj.r.s))
strCaption <- paste0("\\textbf{Table Whatever} This table is just produced with some",
                     "random data and does not mean anything. Just to show you how ",
                     "things work.")
print(xtable(df, digits=2, caption=strCaption, label="Test_table"), 
      size="footnotesize", #Change size; useful for bigger tables
      include.rownames=FALSE, #Don't print rownames
      include.colnames=FALSE, #We create them ourselves
      caption.placement="top", 
      hline.after=NULL, #We don't need hline; we use booktabs
      add.to.row = list(pos = list(-1, 
                                   nrow(df)),
                        command = c(paste("\\toprule \n",
                                          "Model & $\\alpha$ & $\\beta$ & t-value & $R^2$ \\\\\n", 
                                          "\\midrule \n"),
                                    "\\bottomrule \n")
                        )
      )

@

From table \ref{Test_table} you can't learn a lot, only how it looks is important here.

\end{document}
```


A few notes:

* The option *add.to.row* expects two inputs: a list of integers named *pos* and a list of strings named *command*. The latter keeps the  strings that should be entered, the former tells the function *print.xtable* at which row to enter the strings. In our example, we want to add the first string that keeps the column names before everything else. That is why we use $-1$. The bottomrule should be at the very end of the table, that is why we use *nrow(df)*.
* We have to set *hline.after* to NULL because we are not using hlines, but the lines provided by the **booktabs** package (toprule, midrule, and bottomrule).
* The example also shows how to write formulas in the table. Again, the only trick is to know that the backslash is a special character, so we have to write two backslashes.


