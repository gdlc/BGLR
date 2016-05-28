
### Various Ways of fitting a 'GBLUP' model using BGLR

**(i) Providing the markers, using `model='BRR'`**

In this case BGLR asigns iid normal priors to the markers.

``` R
 library(BGLR)
 data(wheat)

 nIter=12000
 burnIn=2000

 X=scale(wheat.X)/sqrt(ncol(wheat.X))
 y=wheat.Y[,i]

 fm1=BGLR( y=y,ETA=list(mrk=list(X=X,model='BRR')),
	   nIter=nIter,burnIn=burnIn,saveAt='brr_'
 	 )

 varE=scan('brr_varE.dat')
 varU=scan('brr_ETA_mrk_varB.dat')
 h2_1=varU/(varU+varE)
```

**(2) Providing the G-matrix**

BGLR Fits these Gaussian models using the eigenvalue decomposition og G. The eigenvalue decomposition is computed internally using 
`eigen()`.

```R
 G=tcrossprod(X)
 fm2=BGLR( y=y,ETA=list(G=list(K=G,model='RKHS')),
	   nIter=nIter,burnIn=burnIn,saveAt='eig_'
	 )
 varE=scan( 'eig_varE.dat')
 varU=scan('eig_ETA_mrk_varB.dat')
 h2_2=varB/(varB+varE)
```
**(3) Providing eigenvalues and eigenvectors**

This strategy can be used to avoid computing the eigen-decomposition internally. This can be useful if a model will be fitted several times (e.g., cross-validation).

```R
 EVD=eigen(G)
 
 fm2b=BGLR( y=y,ETA=list(G=list(V=EVD$vectors,d=EVD$values,model='RKHS')),
	    nIter=nIter,burnIn=burnIn,saveAt='eigb_')
 varE=scan( 'eigb_varE.dat')
 varU=scan('eigb_ETA_mrk_varB.dat')
 h2_3=varB/(varB+varE)
```

**(4) Providing scaled-eigenvectors and using `model='BRR'`**

```R
 PC=EVD$vectors
 for(i in 1:ncol(PC)){  PC[,i]=PC[,i]*sqrt(EVD$values[i]) }
 PC=PC[,EVD$values>1e-5]

 fm3=BGLR( y=y,ETA=list(pc=list(X=PC,model='BRR')),nIter=nIter,
	   burnIn=burnIn,saveAt='pc_')
			
 varE=scan( 'pc_varE.dat')
 varU=scan('pc_ETA_mrk_varB.dat')
 h2_3=varB/(varB+varE)
```