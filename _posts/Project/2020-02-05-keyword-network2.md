---
layout: post
title:  "Summarizing AAG conference abstracts and keywords using keywords network analysis (2)"
date:   2020-02-05 11:40:00
author: Youngho Lee
categories: Project
cover:  "/assets/project/keyword-network/wordcloud.PNG"
---

본 포스팅은 과학기술정보통신부와 정보통신기획평가원이 주관하는 "2019년 글로벌 핵심인재 양성지원 사업"의 프로젝트 과정에서 사용한 코드들을 정리하기 위해 작성하였다. 프로젝트 주제는 "컨퍼런스 최신 연구 동향 분석 : 언어네트워크 분석과 빅데이터 분석을 적용한 Generic Platform 개발"로, Association for American Geographers(AAG, http://aag.org) 컨퍼런스에 1998~2019년간 투고된 논문들의 초록과 키워드들을 사례로 지리학의 연구경향을 파악하고자 하는 연구이다. AAG 컨퍼런스는 매년 5일간 약 7천여명의 연구자들이 등록하는 대규모 학술공동체이며, GIS·인문지리·환경지리·지형학·교통지리·원격탐사 등 지리학의 다양한 분야의 연구들이 매년 소개되고 있다. 이번 글은 이전 글과 마찬가지로 첫 번째 세부프로젝트인 "키워드 네트워크 분석을 활용한 AAG 컨퍼런스 요약"에 관한 글로, 전처리한 키워드 데이터를 바탕으로 네트워크를 구축하고 결과물로 저장하는 과정에 대해 정리할 것이다.

<img src = "/assets/project/keyword-network/post2/data_example1.PNG" title = "plot1" alt = "plot1" width = "1008" style = "display: block; margin: auto;" />

위의 데이터는 이전 글을 통해 키워드만 추출한 데이터에 세부적인 전처리 과정을 진행한 결과물이다. 모든 단어를 Title Case로 바꾸었고, 분석 과정에서의 사소한 오류를 예방하기 위해 모든 키워드에 Semicolons 추가, 영국식 단어를 미국식 단어로 변환 등의 전처리 과정이 이전 글의 결과물에 적용되었다. 우선 전체적인 함수의 코드를 살펴보면 다음과 같다 (설명의 편의를 위해 코드 앞에 숫자를 입력하였다).

{% highlight javascript %}
1  # Packages
2  library(bibliometrix)
3  library(igraph)
4
5  # Data import
6  data <- read.csv(file.choose())
7
8  # Data handling
9  names(data)[1] <- "ID"
10 data$ID <- as.character(data$ID)
11
12 # Make a biblionetwork from data
13 NetMatrix <- biblioNetwork(data, analysis = "co-occurrences",
14                            network = "keywords", sep = ";")
15
16 # Extract matrix from biblionetwork
17 keywords.mat <- as.matrix(NetMatrix); keywords.freq <- diag(keywords.mat)
18 keywords.mat[row(keywords.mat) == col(keywords.mat)] <- 0
19
20 # Assign 1 to any edge greater than 1
21 keywords.adj <- keywords.mat
22 keywords.adj[keywords.adj > 1] <- 1
23
24 # Make an igraph from adjacency matrix.
25 # With "Weighted = NULL", multiple edges are created between two nodes.
26 # Use "weighted = TRUE" to make a signgle-edge between two nodes.
27 k2019.igraph <- graph_from_adjacency_matrix(keywords.adj, mode = "undirected", 
28                                             weighted = NULL)
29
30 # Attach node attribute (nodeFrequency)
31 V(k2019.igraph)$nodeFrequency <- keywords.freq
32
33 # Attach node attribute (degreeCentrality, i.e. singleEdgeDegCentrality, 
34 # from the single-edge graph)
35 singleEdgeDegCentrality <- centr_degree(k2019.igraph, mode = "all")$res
36 V(k2019.igraph)$singleEdgeDegCentrality <- singleEdgeDegCentrality
37
38 # Attach node attribute (eigenVectorCentrality, i.e. 
39 # singleEdgeEigenvectorCentrality, from the single-edge graph)
40 singleEdgeEigenvectorCentrality <- eigen_centrality(k2019.igraph)$vector
41 V(k2019.igraph)$singleEdgeEigenvectorCentrality <- singleEdgeEigenvectorCentrality
42
43 # Attach edge attribute (frequency)
44 tmp.igraph <- graph_from_adjacency_matrix(keywords.mat, mode = "undirected", 
45                                           weighted = TRUE)
46 E(k2019.igraph)$frequency <- E(tmp.igraph)$weight
47
48 # Attach node attribute (cluster - louvain)
49 clusterID <- cluster_louvain(tmp.igraph, weights = E(tmp.igraph)$weight)$membership
50 V(k2019.igraph)$clusterID <- clusterID
51
52 # Convert to graphml
53 write.graph(k2019.igraph, "2019keywords.graphml", format = "graphml")
{% endhighlight %}

함수의 전체를 한 번에 설명하기보다는 사용한 패키지부터 차례대로 자세하게 설명하고자 한다.

{% highlight javascript %}
1  # Packages
2  library(bibliometrix)
3  library(igraph)
{% endhighlight %}

이 코드에서는 두 가지의 패키지를 사용하였다. 첫 번째는 `bibliometrix` 패키지로, 이 패키지는 연구 동향 분석과 같이 계량서지학 연구를 위해 개발되었다. 이번 코드에서는 키워드 데이터를 네트워크 구축을 위한 행렬 형태로 변환하기 위해 사용된다. 두 번째는 `igraph` 패키지로, R뿐만 아니라 다양한 프로그래밍 언어에서 네트워크 관련 분석을 위해 가장 많이 사용되고 있는 패키지이다. 이번 코드에서는 네트워크 구축, 중심성 분석, 네트워크 클러스터링 분석을 위해 사용된다.

{% highlight javascript %}
5  # Data import
6  data <- read.csv(file.choose())
7
8  # Data handling
9  names(data)[1] <- "ID"
10 data$ID <- as.character(data$ID)
11
12 # Make a biblionetwork from data
13 NetMatrix <- biblioNetwork(data, analysis = "co-occurrences",
14                            network = "keywords", sep = ";")
{% endhighlight %}

우선, 데이터를 불러오고 `bibliometrix` 패키지 내의 `biblioNetwork` 함수에서 작동할 수 있도록 column name을 "ID"로 변경하고, 불러온 데이터가 factor 형태이기 때문에 문자형을 바꾸어준다. 다음으로 데이터를 `biblioNetwork` 함수에 넣어주는데, 이 때 키워드 간의 co-occurrences를 살펴보는 것이 목적이라면, 인자를 위와 같이 설정하면 된다. analysis는 "co-citation", "coupling", "collaboration", "co-occurrences", 총 4개, network는 "authors", "titles", "keywords" 등 총 9개의 값을 선택할 수 있다. 구분자는 데이터에 맞게 사용하면 된다.

{% highlight javascript %}
16 # Extract matrix from biblionetwork
17 keywords.mat <- as.matrix(NetMatrix); keywords.freq <- diag(keywords.mat)
18 keywords.mat[row(keywords.mat) == col(keywords.mat)] <- 0
{% endhighlight %}

<img src = "/assets/project/keyword-network/post2/data_example1.PNG" title = "plot2" alt = "plot2" width = "1008" style = "display: block; margin: auto;" />

앞서 만든 `Netmatrix` 객체는 dgCMatrix 형태의 클래스로, 다양한 값들을 포함하고 있다. 이번 분석에서는 행렬 형태의 데이터만 활용할 것이기 때문에, `as.matrix` 함수를 통해 행렬 데이터만 얻었다. 위의 그림에 나와있는 데이터가 `as.matrix` 함수를 통해 얻은 행렬 데이터이다. 키워드별로 동시 발생 빈도를 확인할 수 있고, 대각선에 있는 값들을 통해 전체 네트워크에서 해당 키워드 얼마나 많이 사용되었는지를 나타내는 빈도도 파악할 수 있다. 대각선에 위치한 키워드의 빈도는 추후에 유용하게 사용되기 때문에, 따로 객체로 저장해놓고, 이 행렬에서는 사용되지 않을 것이므로 대각선에 "0"을 입력해준다.

{% highlight javascript %}
20 # Assign 1 to any edge greater than 1
21 keywords.adj <- keywords.mat
22 keywords.adj[keywords.adj > 1] <- 1
23
24 # Make an igraph from adjacency matrix.
25 # With "weighted = NULL", multiple edges are created between two nodes.
26 # Use "weighted = TRUE" to make a signgle-edge between two nodes.
27 k2019.igraph <- graph_from_adjacency_matrix(keywords.adj, mode = "undirected", 
28                                             weighted = NULL)
{% endhighlight %}

이 코드부터는 네트워크를 구축하는 과정이다. 네트워크 분석에서는 single edge를 사용할 것인지, multiple edges를 사용할 것인지에 따라 결과가 달라질 수 있다. 이번 분석에서는 single edge를 사용하는 것이 중심성 분석의 측면에서 네트워크의 구조를 더욱 잘 나타낼 수 있다고 판단하였다. 따라서 single edge 네트워크를 만들기 위해 `keywords.adj` 객체를 따로 만들어서 동시 발생 빈도에 모두 "1"의 값을 입력하였다. 다음으로 `graph_from_adjacency_matrix` 함수를 통해 행렬을 그래프 형태로 변경한다. 키워드 네트워크는 방향성을 가지고 있지 않기 때문에 mode는 "undirected"로 하고, weighted는 "NULL"로 설정하였다. multiple egdes 네트워크를 만들 계획이라면, weighted를 "TRUE"로 설정해야 한다.

{% highlight javascript %}
30 # Attach node attribute (nodeFrequency)
31 V(k2019.igraph)$nodeFrequency <- keywords.freq
32
33 # Attach node attribute (degreeCentrality, i.e. singleEdgeDegCentrality, 
34 # from the single-edge graph)
35 singleEdgeDegCentrality <- centr_degree(k2019.igraph, mode = "all")$res
36 V(k2019.igraph)$singleEdgeDegCentrality <- singleEdgeDegCentrality
37
38 # Attach node attribute (eigenVectorCentrality, i.e. 
39 # singleEdgeEigenvectorCentrality, from the single-edge graph)
40 singleEdgeEigenvectorCentrality <- eigen_centrality(k2019.igraph)$vector
41 V(k2019.igraph)$singleEdgeEigenvectorCentrality <- singleEdgeEigenvectorCentrality
{% endhighlight %}

다음은 노드에 속성을 부여하는 과정이다. 노드는 각 키워드를 나타내며, V("네트워크 객체") 함수를 통해 노드에 속성을 부여할 수 있다. 노드에 부여되는 속성은 키워드의 빈도, 연결정도 중심성 (degree centrality) 값, 고유벡터 중심성 (eigenvector centrality) 값, 군집 ID, 총 4개이다. 키워드의 빈도는 앞선 객체로 저장한 `keywords.freq`를 넣어주고, 각 중심성 분석은 `igraph`에 내장된 함수들을 바탕으로 진행되었다.

{% highlight javascript %}
43 # Attach edge attribute (frequency)
44 tmp.igraph <- graph_from_adjacency_matrix(keywords.mat, mode = "undirected", 
45                                           weighted = TRUE)
46 E(k2019.igraph)$frequency <- E(tmp.igraph)$weight
47
48 # Attach node attribute (cluster - louvain)
49 clusterID <- cluster_louvain(tmp.igraph, weights = E(tmp.igraph)$weight)$membership
50 V(k2019.igraph)$clusterID <- clusterID
51
52 # Convert to graphml
53 write.graph(k2019.igraph, "2019keywords.graphml", format = "graphml")
{% endhighlight %}

마지막 과정은 엣지에 속성 부여, 노드에 네트워크 클러스터링 분석 결과 속성 부여, 그래프 파일 저장에 관한 것이다. 우선 앞의 과정에서 사용한 single edge 네트워크인 `k2019.igraph`는 키워드들 간의 동시 발생 빈도 값을 가지고 있지 않다. 엣지에 부여하고자 하는 속성은 키워드들 간의 동시 발생 빈도이기 때문에, multiple edges 네트워크를 새로 만들어준다. 새로 만든 그래프의 edge의 weight가 동시 발생 빈도를 나타내므로, 이 값들을 `k2019.igraph` 그래프에 넣어준다. 다음은 네트워크 클러스터링 분석 결과를 노드 속성으로 부여하는 과정이다. 이 분석에서는 `cluster_louvain` 함수가 사용되었지만, 목적에 따라 다른 네트워크 클러스터링 분석 방법을 사용하면 된다. 네트워크 클러스터링 분석은 multiple edges 네트워크로 진행되었으므로 마지막에 진행되었다. 그래프 시각화는 R에서도 `igraph`를 통해 가능하지만, 각 노드들의 세부적인 위치 조정이 어려우므로 "Cytoscape"라는 오픈소스 프로그램에서 진행되었다. 이를 위해 네트워크 객체를 graphml 파일 형식으로 저장하였다. 이번 포스팅에서 사용한 함수들과 중심성 분석, 네트워크 클러스터링 분석 방법은 네트워크 분석에 있어서 작은 부분들만을 차지하고 있다. 따라서 다른 사용자들은 보다 다양한 방법들을 활용하는 것을 통해 새로운 결과물을 만들어 낼 수 있을 것이다.