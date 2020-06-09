---
layout:     post
title:      FACETS Testing CNV
subtitle:   ASCN detection CNV principle
date:       2020-06-09
author:     soliva
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - 生信
---
# FACETS
facets is CNV detection software with using the ascn principle 
# input
first Let's look at the test data given 

![1591690799039](/img/facets/1591690799039.png)
```
Chromosome Chromosome of the SNP
Position Position of the SNP
File1R Read depth supporting the REF allele in normal sample
File1A Read depth supporting the ALT allele in normal sample
File2R Read depth supporting the REF allele in tumour sample
File2A Read depth supporting the ALT allele in tumour sample
```
The software comes with a comparison program, as shown in IGV

![1591691071264.png](/img/facets/1591691071264.png)
unlike other software ,facets calulates the depth of snp in each bin ,normal CNV software only calculates the depth of each bin then using CBS process.

### next it Count whether the sum of ref and alt of each snp is heterozygous


![1591691653200.png](/img/facets/1591691653200.png)
### the result is 

![1591691763518.png](/img/facets/1591691763518.png)
## summary 
```
1R = REF-N
1A = ALT-N
2R = REF-T
2A = ALT-T
```
### THE NEXT
```


1R=countN=N-RD
2R=countT=T-RD
N-DP=1R+1A
T-DP=2R+2A
varT=T-RD/T-DP
varN=N-RD/N-DP

```
### as the picture shows

![1591692248504.png](/img/facets/1591692248504.png)

Then the software starts to calculate logR and logOR and gcbias
the gcbias plaes look this blog http://www.zxzyl.com/archives/988
```math
the logR is 
    ncount <- tapply(rCountN, gcpct, sum)
    tcount <- tapply(rCountT, gcpct, sum)
    tsc1=sum(ncount)/sum(tcount)
    log2(1+rCountT*tscl) - log2(1+rCountN) - gcbias
the logOR is
    1=varT x countT
    2=(1-varT) x countT 
    3=varN x countN
    4=(1-varN) x countN
    than 
    ## log-odds-ratio (Haldane correction)
    logOR = log(1+0.5)-log(2+0.5)-log(3+0.5)+log(4+0.5) #using 
    ## variance of log-odds-ratio (Haldane; Gart & Zweifel Biometrika 1967)
    logOR = (1/( [,1]+0.5) + 1/( [,2]+0.5) + 1/( [,3]+0.5) + 1/( [,4]+0.5))
```
### The following steps are the EM algorithm, here they use Bayesian in step E to get the posterior probability,
```
####LogR mixture model parameter####
    gamma=2
    phi=2*(1-rho)+gamma*rho
    mu=log2(2*(1-rhov)+matrix(rhov,ncol=1)%*%t)-log2(phi)
    
    ####LogOR mixture model parameter####
    #allelic ratio
    k=(matrix(rhov,ncol=1)%*%major+1-rhov)/(matrix(rhov,ncol=1)%*%minor+1-rhov)
    logk=log(k)
    logk2=logk^2
    
    #posterior probability matrix
    #pmatrix=NULL
    pmatrix=matrix(NA,nrow=nrow(jointseg),ncol=ng)
    loglik=0
    
    clust=rep(segclust,nmark)
    segc=sort(unique(segclust[chr<=nX]))
    for(s in segc){
      idx=which(clust==s)
      x1ij=logR.adj[idx]
      upper=quantile(x1ij,0.95)
      lower=quantile(x1ij,0.05)
      x1ij[x1ij>upper]=NA
      x1ij[x1ij<lower]=NA
      mus=rep(mu[s,],each=length(idx))
      sd=sigma[s]
      if(rhov[s]<0.4){
        x1ij=rep(cnlr.median.clust[s]-dipLogR,length(idx))
        sd=0.1
        }
      #density for logR.adj (centered logR)
      d1=dnorm(x1ij,mean=mus,sd=sd)
      d1[d1==Inf]=NA
      
      #density for logOR, non-central chi-square
      nu=rep(logk2[s,],each=length(idx))
      lambda=nu/rep(logORvar[idx],ng)
      x2ij=logOR2var[idx]
      if(rhov[s]<0.4){
        x2ij=rep(mafR.clust[s]/logORvar.clust[s],length(idx))
        lambda=nu/logORvar.clust[s]
        }
      #d2=dchisq(x2ij+1,df=1,ncp=lambda)
      d2=dchisq(x2ij,df=1,ncp=lambda)
      d2=1/(abs(x2ij-lambda)+1e-6)
      d2[d2==Inf]=NA
      
      #likelihood
      d=d1*d2
      hetsum=d[rep(het[idx]==1,ng)]
      homsum=d1[rep(het[idx]==0,ng)]
      d=sum(hetsum[hetsum<Inf],na.rm=T)+sum(homsum[homsum<Inf],na.rm=T)
      if(!is.na(d)&d>0&d<Inf){loglik=loglik+log(d)}
      
      #heterozygous positions contribute to logR and logOR
      numerator1=matrix(d1*d2,nrow=length(idx),ncol=ng,byrow=F)
      numerator1=sweep(numerator1,MARGIN=2,prior[s,],`*`)
      
      #homozygous positions contribute to logR only
      numerator0=matrix(d1,nrow=length(idx),ncol=ng,byrow=F)
      numerator0=sweep(numerator0,MARGIN=2,prior[s,],`*`)
      
      numerator=numerator1
      numerator[het[idx]==0,]=numerator0[het[idx]==0,]
      
      tmp=apply(numerator,1,function(x)x/(sum(x,na.rm=T)+1e-5))
      #pmatrix=rbind(pmatrix,t(tmp))
      pmatrix[idx,]=t(tmp)
      
      #update prior
      prior[s,]=apply(t(tmp),2,function(x)mean(x,na.rm=T))
    }

```

![1591695237945.png](/img/facets/1591695237945.png)
```
###  the M step is 
#get CF per segments, pick mode close to 1 (favor high purity low cn solution)
    rhom=gammam=matrix(NA,nrow=nclust,ncol=ng)
    geno=matrix(0,nrow=nclust,ncol=ng)
    which.geno=posterior=rep(NA,nclust)
    for(i in segc){
    
      idx=which(clust==i)
      idxhet=which(clust==i&het==1)
      sump=apply(pmatrix[idx,,drop=F],2,function(x)sum(x,na.rm=T))
      
      #if probability is too small (highly uncertain), use lsd estimates for stability 
      if(all(is.na(prior[i,]))){
        prior[i,]=prior.old[i,]
        }else{
        if(sum(prior[i,],na.rm=T)==0)prior[i,]=prior.old[i,]
        }
      
      if(max(prior[i,],na.rm=T)>0.05){   
        
      ##calculate rho for the most likely genotype(s) for segment i
      ##if there more more than one likely candidates save two and pick one with higher CF
      #top2=sort(prior[i,],decreasing=T)[1:2]
      #if(top2[2]>0.05&abs(diff(top2))<0.0001){
      #    sump[prior[i,]<quantile(prior[i,],(ng-2)/ng)]=NA
      # }else{
      #    sump[prior[i,]<max(prior[i,])]=NA
      # }
      
      sump[prior[i,]<max(prior[i,])]=NA
      
        ##update k
        tmphet=pmatrix[idxhet,,drop=F]
        v1=as.vector((logOR[idxhet]^2-logORvar[idxhet])/logORvar[idxhet])
        v2=as.vector(1/logORvar[idxhet])
        sumdphet=apply(sweep(tmphet,MARGIN=1, v1, `*`), 2,function(x)sum(x,na.rm=T))
        sumphet=apply(sweep(tmphet,MARGIN=1,v2,`*`), 2,function(x)sum(x,na.rm=T))
        sumphet[is.na(sump)]=NA
                
        #CF from logOR    
        logk2hat=pmax(0,sumdphet/sumphet) #can be negative when k=1 logk=0 set to 0
        khat=exp(sqrt(logk2hat))
        a=(1-khat)/(khat*(minor-1)-(major-1))
        a[abs(a)==Inf]=NA
        a[a<=0]=NA
        a[a>1]=1
        if(all(nhet[segclust==i]<min.nhet))a=rep(NA,ng)
        
        #CF from logR
        tmp=pmatrix[idx,,drop=F]
        v=as.vector(logR.adj[idx])
        sumdp=apply(sweep(tmp,MARGIN=1,v,`*`),2,function(x)sum(x,na.rm=T))   
        mu.hat=sumdp/sump #mu.hat
        aa=2*(2^mu.hat-1)/(t-2)
        aa[abs(aa)==Inf]=NA
        aa[aa<=0]=NA
        aa[aa>1]=1
        
        aaa=pmax(a,aa,na.rm=T)
        #degenerate cases
        #homozygous deletion (0) and balanced gain (AABB, AAABBB), maf=0.5, purity information comes from logr only
        aaa[c(1,8,13)]=aa[c(1,8,13)]  
     
        #set upper bound at sample rho
        aaa=pmin(aaa,rho)
        
        #uniparental disomy (AA) CF information comes from logOR only.
        
        ##if there are two likely genotype, choose one with higher purity (e.g.,AAB 80% or AAAB 50%)
        ##if the higher CF exceeds sample purity, then the lower CF is the right one
        #if(all(is.na(aaa))){which.geno[i]=which.max(prior[i,])}else{
        #  which.geno[i]=ifelse(max(aaa,na.rm=T)<rho,which.max(aaa),which.min(aaa))
        #}
        
        which.geno[i]=which.max(prior[i,])
        
        postprob=pmatrix[idx,which.geno[i]]
        posterior[i]=mean(postprob[postprob>0],na.rm=T)
        
        #update sigma
        y=as.vector(logR.adj[idx])*pmatrix[idx,,drop=F]
        r=y-mu[i,]*pmatrix[idx,,drop=F]
        ss=sqrt(sum(r[,which.geno[i]]^2,na.rm=T)/sum(pmatrix[idx,which.geno[i]])) 
        sigma[i]=ifelse(is.na(ss),0.5,ss)
        
        aaa[setdiff(1:ng,which.geno[i])]=NA  
        
        #het dip (AB) seg has no information, set CF at a high value less than 1
        #if(any(which(!is.na(sump))==4)){aaa[4]=0.9}
        
        rhom[i,]=aaa 
        
      } #max prior
      
    }
```
![1591695253756.png](/img/facets/1591695253756.png)

### Loop until it converges

## Or simpler, also use the CBS algorithm instead it

# picard方式计算CNV
探针长度
```
chrom   start   end     length  
chr1    12080   12251   172 
chr1    12595   12802   208
```
normalized_coverage : for each target interval, the read depth (unique read starts) that correspond to a particular target interval is divided by the average number of read starts in all of the target intervals.
#### normal 
```
chrom   start   end     length   normalized_coverage    gene
chr1    12080   12251   172        0                    DDX11L1
chr1    12595   12802   208        0.002957             DDX11L1
```
#### tumor
```
chrom   start   end     length   normalized_coverage   gene
chr1    12080   12251   172        0                   DDX11L1
chr1    12595   12802   208        0.011583            DDX11L1
```

### 算法
例如gene  EGFR

片段长度：length = L1 L2  L3  L4  L5

归一化覆盖:Tn = Tn1 Tn2 Tn3 Tn4 Tn5
           Nn = Nn1 Nn2 Nn3 Nn4 Nn5

计算 :   ((Tn1/Nn1)xL1+(Tn2/Nn2)xL2+·····)/L1+···L5 =拷贝率  
        
