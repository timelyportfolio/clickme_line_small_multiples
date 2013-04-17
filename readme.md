# d3 Line in Small Multiples Ractive


## JGB Yields by Maturity since Jan 2012 using R and clickme
### [Live Example](http://bl.ocks.org/timelyportfolio/5407807)

To recreate in R, copy/paste this code.

```{r,results="asis",message=FALSE,error=FALSE,warning=FALSE}
#get Japan yield data from the Ministry of Finance Japan
#data goes back to 1974

require(xtsExtra)
require(clickme)

url <- "http://www.mof.go.jp/english/jgbs/reference/interest_rate/"
filenames <- paste("jgbcme",c("","_2010","_2000-2009","_1990-1999","_1980-1989","_1974-1979"),".csv",sep="")

#load all data and combine into one jgb data.frame
jgb <- read.csv(paste(url,filenames[1],sep=""),stringsAsFactors=FALSE)
for (i in 2:length(filenames)) {
  jgb <- rbind(jgb,read.csv(paste(url,"/historical/",filenames[i],sep=""),stringsAsFactors=FALSE))
}

#now clean up the jgb data.frame to make a jgb xts
jgb.xts <- as.xts(data.matrix(jgb[,2:NCOL(jgb)]),order.by=as.Date(jgb[,1]))
colnames(jgb.xts) <- paste0(gsub("X","JGB",colnames(jgb.xts)),"Y")

#get Yen from the Fed
#getSymbols("DEXJPUS",src="FRED")

xtsMelt <- function(data) {
    require(reshape2)
    
    #translate xts to time series to json with date and data
    #for this behavior will be more generic than the original
    #data will not be transformed, so template.rmd will be changed to reflect
    
    
    #convert to data frame
    data.df <- data.frame(cbind(format(index(data),"%Y-%m-%d"),coredata(data)))
    colnames(data.df)[1] = "date"
    data.melt <- melt(data.df,id.vars=1,stringsAsFactors=FALSE)
    colnames(data.melt) <- c("date","indexname","value")
    #remove periods from indexnames to prevent javascript confusion
    #these . usually come from spaces in the colnames when melted
    data.melt[,"indexname"] <- apply(matrix(data.melt[,"indexname"]),2,gsub,pattern="[.]",replacement="")
    return(data.melt)
    #return(df2json(na.omit(data.melt)))
    
  }
  
  jgb.melt <- xtsMelt(jgb.xts["2012::",])

  set_root_path("c:/users/kent.tleavell_nt/dropbox/development/r")
  clickme(jgb.melt,"clickme_line_small_multiples")
```