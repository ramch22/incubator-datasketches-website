# DataSketches Directions

## Introduction

### Data Streaming

When analyzing massive data sets, generating exact answers to even very basic queries about the data can require huge compute resources (memory and compute time). Examples of such queries include identifying frequent items, unique count queries, quantiles and histogram queries, matrix analysis tasks such as Principle Components Analysis and Latent Semantic Analysis, and more complicated downstream machine learning tasks. This often leads to failure to scale, and precludes real-time analysis of data.

However, in many settings, approximate answers are acceptable, as long as the approximation error is carefully controlled. For example, if data are noisy, then any answer with less error than the noise that is already inherent in the data is just as useful as an exact algorithm. Even if data are noiseless, many high-level business decisions do not require exact knowledge of the data: knowing up to, say, 0.5% error how many unique users visited a website in a given time period is typically as effective as an exact answer.

When approximate answers are acceptable, systems designers have at their disposal a vast literature on streaming algorithms. These algorithms process a massive dataset in a single pass, and compute very small summaries (also called sketches) of the dataset, from which it is possible to derive accurate (though approximate) answers to queries. Many streaming algorithms use mere kilobytes of space even for datasets that are petabytes in size, and process each datum in constant time, thereby enabling real-time analyses.

### Mergeable Summaries

Ideally, streaming algorithms will produce summaries that are mergeable, meaning that one can process many different streams of data independently, and then the summaries computed from each stream can be quickly combined to obtain an accurate summary of various combinations of the datasets (union, intersection, etc.). Mergeable summaries enable massive datasets to be automatically processed in a fully distributed and parallel manner, by partitioning the data arbitrarily across many machines, summarizing each partition, and seamlessly combining the results. In addition to drastically reducing memory usage, compute time, and latency compared to exact methods, mergeable summaries also greatly simplify system architecture; they allow non-additive queries (such as unique count queries) to be treated as though they are additive, with the “sum” of two sketches being their merge. This means that data can be partitioned into fragments and each fragment sketched separately, with the sketches stored in a simple data-mart architecture and merged at query time.

A final important application of mergeable summaries is to power conservation in weak peripheral devices. For example, one of the key benefits of the Internet of Things (IoT) is that it enables the monitoring and aggregation of data from IoT devices such as sensors and appliances. Such devices are often power limited, so it is essential to minimize the amount of data that must be sent from each device to the aggregation center. Mergeable summaries enable this: each device can itself compute a summary of its own data, and send only the summary to the aggregation center, which merges all received summaries to obtain a global summary for all devices’ datasets.

### The Data Sketches Open Source Library

This library has been designed from the beginning to be high-performance and production-quality suitable for integration into large data processing systems that must deal with massive data.
The library is written in pure Java, and contains state of the art algorithms for a variety of basic query classes, including identifying frequent items, unique count queries, computing quantiles and histograms, and sampling. It will soon contain algorithms for matrix analytic tasks such as PCA as well. 
All algorithms in the library produce mergeable summaries, 
and come with formal guarantees on the accuracy of the answers returned.

Currently, the core contributors to the library are Lee Rhodes, Kevin Lang, Jon Malkin, and Alex Saydakov (all at Yahoo/Oath), Justin Thaler (Professor at Georgetown University, and Edo Liberty (Principal Scientist at Amazon Web Services and manager of the Algorithms group at Amazon AI).

The library has been adapted throughout industry and government. For example, at Yahoo, where it was conceived and created, the library is widely used internally to reduce processing time from days to seconds for many tasks. At SpliceMachine, it is used for database query planning and optimization. It is also deeply embedded into a low-latency open source data store called Druid, as well as an open source graph database called Gaffer that is maintained by the British intelligence agency GCHQ.

Beyond its utility in deployed systems, the process of developing of developing the Data Sketches library has led to interesting research. This has involved important contributions to both the theory of streaming algorithms as well in addressing issues that are crucial in real-world stream engines but are often ignored in the academic literature. These issues include mergeability, and dealing with weighted stream updates (i.e., data steams where each piece of data comes with an associated importance measure).

In particular, work on the Data Sketches library has led to novel algorithms achieving state of the art practical performance for identifying frequent items in data streams [ABL+17], and mergeable summaries for unique count queries [DLRT16]. On the theoretical side, work on the library has led to the resolution of the space complexity of streaming approximation algorithms for quantile queries, which was a longstanding open question [KLL16], as well as for the problem of identifying frequent itemsets [LMTU16].

There are major opportunities to incorporate algorithms for new and richer types of queries into the library, as well as to improve the efficiency of the algorithms that are already implemented.

## Improved Algorithms for Unique Counting

In a recent pre-print that grew out of work on the Data Sketches library [Lan17], Kevin Lang describes several streaming algorithms for estimating the number of distinct elements in a data stream. The al- gorithms have a better space/accuracy tradeoff than the previous state of the art algorithm, Hyperloglog (HLL) [FFGM07], which has been considered the gold standard in practical performance for this problem for nearly a decade. Specifically, Lang’s algorithms use up to 20% less space than the entropy of the HLL sketch, and hence 20% space than any possible implementation of HLL.

Regarding runtime, Lang’s pre-print shows that some of his algorithms have comparable speed to straightforward implementations of HLL, but are somewhat slower than heavily optimized HLL imple- mentations. Significant research remains to optimize the speed of the new algorithms, determine which variant algorithm performs best on real data and is most suitable for a production environment, and finally to produce a production-quality implementation of this algorithm.

## Algorithms For Anomaly Detection

A common goal when processing massive data streams is to identify anomalous events or data points. For example, an online advertiser or content provider may try to identify fraud in clickstream data, a network operator may try to quickly identify network intruders or DDoS attacks, or a data analyst may simply try to cleanse outliers from a dataset before running subsequent learning algorithms.

The Data Sketches library already contains several algorithms useful for anomaly detection in data streams. One example is the library’s algorithm for answering quantile queries. In the quantiles problem, the stream specifies a list of real numbers, and any stream update in (say) the top or bottom percentile is (by definition) an outlier or anomaly. As mentioned above, the Data Sketches team has resolved the asymptotic space complexity of streaming quantile computation [KLL16], and is currently completing a careful empirical study of various quantile algorithms that have been proposed in the literature. A second algorithm in the library that is useful for anomaly detection is its novel algorithm for identifying frequent items [ABL+17]: any item that makes up an unusually large fraction of the dataset is inherently anomalous.

Moving forward, the Data Sketches team will incorporate algorithms for additional query classes that are useful for anomaly detection. Key targets include entropy computation [CCM10, Tha07], identification of hierarchical heavy hitters [MST12], and identification of superspreaders [VSGB05] (all of which have been intensely studied in the context of detecting anomalies in network flows).

The best known streaming algorithms for all three problems make black-box use of algorithms for the simpler task of identifying frequent items. Owing to the library’s state of the art algorithm for the latter task, the team is positioned to develop highly efficient solutions for these more complicated problems.

While existing streaming algorithms for identifying hierarchical heavy hitters and superspreaders pro- duce mergeable summaries, the only known practical algorithm for entropy computation [CCM10] does not. A challenging problem that the Data Sketches team will try to solve is the development of a practical mergeable sketching algorithm for entropy computation.

## Matrix and Clustering Algorithms

Low-rank approximations to matrices are useful in many unsupervised learning tasks including PCA. Low- rank approximations effectively uncover latent structure in datasets, by identifying the “most informative directions” of a data matrix. They also speed up downstream learning tasks, since these tasks can be run on the low-rank approximation instead of on the original matrix. In [Lib13], Liberty presented a nearly optimal streaming algorithm for approximating a data matrix by a low-rank matrix. The algorithm assumes that the data matrix is streamed row-wise, meaning each stream update atomically specifies a new row of the matrix.

Computing a low-rank approximation to a matrix can be viewed as identifying “frequent directions” in a stream of vectors, and Liberty’s algorithm can be viewed as a direct generalization of an algorithm for identifying frequent items in a stream of items. Building on the frequent items algorithm described in [ABL+17] and already implemented in the Data Sketches library, the Data Sketches team is nearing completion of a production-quality implementation of Liberty’s algorithm. Once this implementation is complete, ongoing research will identify additional optimizations to further improve the algorithm’s speed and accuracy, and will perform a careful empirical comparison of its performance relative to alternative algorithms in the literature. The Data Sketches team will also work to develop algorithms that can handle more general types of updates to the data matrix (not just row-wise updates).

A related direction that the team will pursue in the immediate future is to develop production-quality implementations of streaming algorithms for k-means clustering. The low-rank matrix approximation and clustering problems are closely related (see, e.g., [CEM+15]), and we are confident that ideas from low- rank matrix approximation will prove useful in developing effective clustering algorithms.

## Graph Algorithms

Very large graphs are ubiquitous. They arise in settings as diverse as social network analysis and network flow logs, such as those captured by the Amazon Virtual Private Cloud. There is a rich theoretical literature on streaming algorithms for graph processing (see the survey of McGregor [McG14] for an overview), though there has been comparatively less applied work on such algorithms.

Developing production-quality algorithms for graph streams is a key direction that the Data Sketches team will pursue in the immediate future. One prime target problem is graph sparsification, in which one throws away most of the edges in a dense graph, but does so in a careful way so that the resulting sparse graph inherits many of the same properties as the original dense graph. The sparse graph can be viewed as an approximation to the original dense graph, and it can be used in place of the dense graphs downstream in tasks such as clustering and community detection. When run on the sparse graph, these downstream algorithms will require far less compute power, yet the results on the sparse graph will provably approximate the results on the original dense graph.

## Sliding Windows

In many applications, data eventually grows stale or out of date, and queries should accordingly be restricted to relatively recent data. Mergeable summaries enable a simple solution to this problem: break the data into relatively small chunks (with each chunk covering, say, a one-hour time period), sketch each chunk separately, and at query time merge the summaries of only the most recent chunks.

This solution suffices in some applications, but for other applications the chunks must be more fine grained (e.g., when detecting anomalies or phenomena that last for seconds or minutes rather than hours). In these settings, a naive approach based on mergeable summaries becomes prohibitively expensive in terms of both memory usage and latency. For such settings, the ideal solution is a streaming algorithm that au- tomatically “forgets” data when it becomes stale. This setting has been studied in the literature on sliding window streaming algorithms. There has been working studying sliding window algorithms for frequent items (e.g. [GDD+03]), unique counts (e.g. [GT02]), and quantiles (e.g. [AM04]). Just as work on the Data Sketches library has led to significant recent progress in developing efficient algorithms for each of these problems in the standard (non-sliding window) streaming setting, the we are confident that related ideas will lead to similar progress in the sliding window setting as well.

References

...
