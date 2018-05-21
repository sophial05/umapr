
<!-- README.md is generated from README.Rmd. Please edit that file -->
umapr
=====

[![Travis-CI Build Status](https://travis-ci.org/ropenscilabs/umapr.svg?branch=master)](https://travis-ci.org/ropenscilabs/umapr)

Uniform Manifold Approximation and Projection (UMAP) is a non-linear dimensionality reduction algorithm. It is similar to t-SNE but computationally more efficient. UMAP was created by Leland McInnes and John Healy ([github](https://github.com/lmcinnes/umap), [arxiv](https://arxiv.org/abs/1802.03426)). `umapr` wraps the Python implementation of UMAP to make the algorithm accessible from within R.

Basic use
---------

Here is an example of running UMAP on the `iris` data set.

``` r
library(umapr)
# select only numeric columns
embedding <- as.data.frame(umap(as.matrix(iris[ , 1:4])))

# look at result
head(embedding)
#>           V1        V2
#> 1 -1.7324369 -5.463769
#> 2 -2.8849956 -3.761887
#> 3 -3.6021864 -4.281125
#> 4 -3.4141524 -4.057291
#> 5 -1.9330339 -5.352357
#> 6 -0.6892355 -4.836187

# plot result
suppressPackageStartupMessages(library(tidyverse))
ggplot(embedding, aes(x = V1, y = V2, color = iris$Species)) + 
  geom_point()
```

![](README-unnamed-chunk-2-1.png)

`umap` returns a data frame with two columns containing the UMAP embeddings of the original data and (optionally) the original data attached.

There are a few important parameters. These are fully described in the UMAP Python [documentation](https://github.com/lmcinnes/umap/blob/bf1c3e5c89ea393c9de10bd66c5e3d9bc30588ee/notebooks/UMAP%20usage%20and%20parameters.ipynb).

The `n_neighbor` argument can range from 2 to n-1 where n is the number of rows in the data.

``` r
neighbors <- c(4, 8, 16, 32, 64, 128)

f <- lapply(neighbors, function(neighbor) {
  iris_result <- umap(as.matrix(iris[,1:4]), n_neighbors = as.integer(neighbor))
  cbind(as.data.frame(iris_result), data.frame(Species = iris$Species))
})

names(f) <- neighbors

bind_rows(f, .id = "Neighbor") %>% 
  mutate(Neighbor = as.integer(Neighbor)) %>% 
  ggplot(aes(V1, V2, color = Species)) + geom_point() + 
  facet_wrap(~ Neighbor, scales = "free")
```

![](README-unnamed-chunk-3-1.png)

The `min_dist` argument can range from 0 to 1.

``` r
dists <- c(0.001, 0.01, 0.05, 0.1, 0.5, 0.99)

f <- lapply(dists, function(dist) {
  iris_result <- umap(as.matrix(iris[,1:4]), min_dist = dist)
  cbind(as.data.frame(iris_result), data.frame(Species = iris$Species))
})

names(f) <- dists

bind_rows(f, .id = "Distance") %>% 
  ggplot(aes(V1, V2, color = Species)) + geom_point() + 
  facet_wrap(~ Distance, scales = "free")
```

![](README-unnamed-chunk-4-1.png)

The `distance` argument can be a bunch of stuff.

``` r
dists <- c("euclidean", "manhattan", "canberra", "cosine", "hamming", "dice")

f <- lapply(dists, function(dist) {
  iris_result <- umap(as.matrix(iris[,1:4]), metric = dist)
  cbind(as.data.frame(iris_result), data.frame(Species = iris$Species))
})

names(f) <- dists

bind_rows(f, .id = "Metric") %>% 
  ggplot(aes(V1, V2, color = Species)) + geom_point() + 
  facet_wrap(~ Metric, scales = "free")
```

![](README-unnamed-chunk-5-1.png)
