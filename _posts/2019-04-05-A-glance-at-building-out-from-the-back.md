---
title: "A glance at building out from the back"
layout: post
--- 

### Introduction 
Many years ago the football analytics pioneer, Charles Reep, came to conclusion that [short sequences of play are more successful in football.](https://fivethirtyeight.com/features/how-one-mans-bad-math-helped-ruin-decades-of-english-soccer/)  This mistaken interpretation coincided with similar approaches being adopted by many teams, including several based in England.

However, this trend seems to have reversed in recent years with more and more teams looking to build out from the back. This requires defenders and goalkeepers to have better passing skills and be heavily involved in the initial stage of the build-up. However, this comes with additional risks as every ball lost in defensive areas leaves a team vulnerable.

By identifying underlying patterns in how opponents look to build out from the back, coaches can obtain a quick reference that shows their preferred passing locations and style of play. These findings can back up tactical analysis, so that a team can devise strategies to stop an opponent’s attacks in the initial stage or, simply by team positioning, force an opponent to play the ball in an untrained or inefficient way.

Therefore, for my OptaPro Forum project I used event data from the 2017/18 Premier League season to prepare a framework that would allow me to discover any underlying patterns in how each team built out from the back. This was a two-stage analysis that employed cluster analysis in both parts.

Cluster analysis is a data mining approach which classifies observations in unsupervised settings, i.e. without any pre-known labels. Groups formed as a result of clustering should contain observations that are similar to each other, and the observations from separate groups should not be similar.

### Part 1 – Clustering Of Initiating Passes
For the first part of this analysis, these observations were initiating passes, which were defined as passes that:

– progressed the ball significantly higher up the pitch (passes with less than a 15 degree angle measured against the goal line were excluded);

– had a starting point within the defensive third;

– were made by goalkeepers or defenders;

– were not headed passes or goalkeepers’ throws.

It must be stressed that this definition has several limitations. Firstly, passes made by defensive midfielders, who often support centre backs during a build-up phase, were not taken into account. This is because detailed positions were absent in the data sample.

One potential solution would have been to try and estimate the average position of events relating to a player, but such an approach could be affected by drawbacks of the mean – as players sometimes change their positions during a game, their resulting average position could be misleading. Therefore, for the purpose of this analysis, passes made by all midfielders were excluded. Nevertheless, when analysing a particular team, an analyst could decide which players might be playing as a holding midfielder and include them in the input dataset.

Another limitation is the fact that this analysis is based on data for an entire season and therefore does not account for managerial or player changes, which may affect a team’s style of play. However having a full season of data allows analysis to be consistent for each team and ensures comparable sample sizes.

As the chosen algorithm, which will be discussed later, allows us to specify a minimum number of similar passes required to classify a pass into a cluster, this parameter could be reduced and analysis from significantly fewer games could be performed. This would allow opposition analysis based on more recent games or matches taking into account any changes.

The definition resulted in a dataset of pass spatial coordinates containing on average 1,527 passes per team, which constitutes the input data for stage one.

Before throwing the data into a clustering algorithm, it’s worth disclosing how they are scattered. Here, principal component analysis (PCA) usually helps. PCA is a dimensionality reduction technique and as such, it allows us to represent a pass by a single point in 2-dimensional space, preserving as much variability from its original coordinates (starting x, y and ending x, y coordinates) as possible.

An example output from PCA, presenting initiating passes made by Manchester United, can be seen below.

<img src="https://kubamichalczyk.github.io/images/PCA-output.svg" style="display: block; margin: auto;" />
***Figure 1. PCA output for Manchester United initiating passes. The lighter the colour, the more passes in the region.***

From this output we can observe seven dense regions – six smaller, circular regions on the exterior with one big area in the middle, all connected with variably numerous bridge points. These bridge points make the data hard to separate, and therefore, to cluster, especially with optimisation-based clustering methods like the widely known k-means algorithm.

However, my intuition behind clusters was different than k-means assumptions. As I wanted to detect repetitive patterns, my goal was to catch dense regions, possibly ignoring bridge points and any other form of noise. Therefore, a DBSCAN (density-based spatial clustering on applications with noise) algorithm was employed. Here, noise can be interpreted as passes not made in normal build-up settings, for example, made under pressure, out of position, etc. Depending on the team, 53-80% of passes were classified as noise, leaving us only with significant patterns of play.

Another essential choice that had to be made was a form of dissimilarity measure. As my primary objective was to establish direction of play, with the forward progress of passes being a secondary concern, I decided to put more weight on the y coordinate. The reason behind these chosen weights was that I cared more about pass direction with respect to pitch width instead of pitch length.

<img src="https://kubamichalczyk.github.io/images/medoids-plot.svg" style="display: block; margin: auto;" />
***Figure 2. Medoids (cluster representatives) for each Premier League team. Arrowhead size indicates number of passes within a cluster.***

Figure 2 shows medoid passes for each Premier League team. Medoids are cluster representatives that are the most similar to all other passes within a cluster.  We can see that some teams avoid passing into central midfield and prefer to distribute the ball wide (such as Bournemouth) or long (West Brom) while, perhaps unsurprisingly, Manchester City play a lot of short, central passes. Another nice example is Leicester City, with diagonal balls targeted towards the half-way line, a type of pass which did not show up with any other team.

One can also spot similarities between Huddersfield Town and Liverpool, although The Terriers did not play short, central passes and exhibited long-ball clusters instead.

Figure 2 can also be used to analyse build-up involvement by position. For example, Crystal Palace’s left back is probably much more involved in their build-up play than their right back. However, such a conclusion should be counterchecked with cluster homogeneity.

Now we will switch our attention to Arsenal. Looking at Figure 2, it may be tempting to conclude that Arsenal initiated their moves significantly more often down the right side than down the left.

The graphic below presents the complete structure of Arsenal’s clusters, with cluster medoids marked dark blue. The actual difference is not so apparent, as there are two homogeneous clusters on the left, while on the right only one cluster has been formed with a more heterogeneous structure. So the graphic helps us to take into consideration within-cluster variation, while establishing Arsenal’s preferred direction of build-up.
<img src="https://kubamichalczyk.github.io/images/example-clusters-plot.png" style="display: block; margin: auto;" />
***Figure 3. Initiating pass clusters for Arsenal. All passes within each cluster are plotted with medoids marked dark blue.***

### Part 2 – Modal Subsequent Actions

In the second part of my analysis I wanted to answer this question: How do the most frequent build-ups look, when they start in a particular initiating-pass cluster?

For this purpose, I took all sequences of play that began in a particular cluster and clustered them using affinity propagation combined with a similarity measure appropriate for time-series data – dynamic time warping (DTW). This measure allows identification of paths that are of a similar shape.

Before calculating the distance between two sequences, DTW tries to align one sequence to make it resemble the reference as much as possible, so any differences in speed and number of passes within a sequence are ignored, as long as the overall paths are similar.

One sequence, however, could contain a few initiating passes (as defined in part 1). Therefore, to prevent some parts of a sequence being considered twice, sequences were split into two subsequences if the ball re-entered the defensive third. As a result, a considerable proportion of subsequences were just two-pass exchanges, with the ball played from a defender and immediately back again, which are not informative enough. However, a two-pass subsequence may still be of interest if the ball was played long.

For this reason, all subsequences that did not cross the half-way line were removed. Because my main focus was on how teams build out from the back and not how they attack, subsequences were trimmed if the ball entered the final third. This was done to avoid matching subsequences using information that is irrelevant in this context.

Finally, all subsequences made up of unsuccessful initiating passes were removed, as these were already classified in stage one.

<img src="https://kubamichalczyk.github.io/images/subsequences-plot.svg" style="display: block; margin: auto;" />
***Figure 4. An example output from stage two – top three mode build-ups following initiating pass from cluster 1. A dashed line indicates ball being carried. A solid line indicates a pass. Colour indicates sequence order, starting from dark to light blue. Width is measured as maximum horizontal difference in metres within the sequence. Absolute width is measured as a maximum distance in metres from a central vertical line within the sequence. Directness is measured as the net difference in distance to the opponent’s goal line, divided by total distance the ball travelled during the sequence.***

Figure 4 presents Arsenal’s three most frequent build-ups for an initiating pass from cluster 1. Although these results should probably be taken with a pinch of salt because of the small sample size, the presented clusters may sometimes inform us, for example, which initiating passes tend to activate a direct attack on the flank, or a longer build-up play through the middle.

### Conclusions
The overall feedback I received at the Forum was overwhelmingly positive. Club analysts particularly appreciated the ability to pick out differences between teams and detect potential weak spots. Detecting these points of interest was made possible by focussing not on the league perspective, but rather analysing the data at team level.

One thing that could be developed further is a statistical measure that would allow the quality of clustering to be formally assessed. I spent quite a bit of time studying about different cluster-validation techniques and none of those seemed appropriate from a footballing perspective.

Therefore, all clusters were validated visually and DBSCAN parameters were chosen so that the clusters were stable across the neighbouring values. Having a cluster-validation statistic would not only reduce human bias, but also help to automate the process.

### The App
If you are interested in the results for any particular team, please take a look at the [ShinyApp](https://kubamichalczyk.shinyapps.io/a-glance-at-building-out-from-the-back/). In addition, please do not hesitate to share your thoughts or insights with me on [Twitter](https://twitter.com/KubaMichalczyk), either publicly or through direct messaging.

**Acknowledgments**: This work has been done during NYA Fellowship sponsored by [North Yard Analytics](http://www.northyardanalytics.com/) and earlier presented as a poster at OptaPro Forum 2019. The data was provided as a courtesy of [OptaPro](https://www.optasportspro.com/).
