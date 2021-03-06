---
title: "What's New in RNeo4j?"
comments: true
layout: post
---



# What's New in RNeo4j?

[RNeo4j](http://nicolewhite.github.io/RNeo4j/) is Neo4j's R driver - it allows you to quickly and easily interact with a Neo4j database from your R environment. Some recent updates to RNeo4j include:

* My contributions
    * Functionality for retrieiving and handling paths
    * Additional sample datasets
* Community contributions
    * Open the Neo4j browser in RStudio
    * Set custom HTTP options
    
## Paths

Several functions have been added for retrieving and manipulating paths. These include:

* `getPaths`
* `nodes`
* `rels`
* `shortestPath`
* `allShortestPaths`

To demonstrate these functions, I'll first add data to the graph with [createNode](http://nicolewhite.github.io/RNeo4j/docs/create-node.html) and [createRel](http://nicolewhite.github.io/RNeo4j/docs/create-rel.html):


```r
library(RNeo4j)
neo4j = startGraph("http://localhost:7474/db/data/")

alice = createNode(neo4j, "User", name = "Alice")
bob = createNode(neo4j, "User", name = "Bob")
charles = createNode(neo4j, "User", name = "Charles")
david = createNode(neo4j, "User", name = "David")
elaine = createNode(neo4j, "User", name = "Elaine")

r1 = createRel(alice, "LIKES", bob, weight = 1)
r2 = createRel(bob, "LIKES", charles, weight = 2)
r3 = createRel(bob, "LIKES", david, weight = 3)
r4 = createRel(charles, "LIKES", david, weight = 4)
r5 = createRel(alice, "LIKES", elaine, weight = 5)
r6 = createRel(elaine, "LIKES", david, weight = 6)
```

### getPaths

[getPaths](http://nicolewhite.github.io/RNeo4j/docs/get-paths.html) allows you to retrieve a list of path objects with a Cypher query. The following query will find all paths (to a maximum depth of four) that traverse the relationship type `LIKES` between Alice and David:


```r
query = "
MATCH p = (u1:User)-[:LIKES*..4]->(u2:User)
WHERE u1.name = 'Alice' AND u2.name = 'David'
RETURN p
"

p = getPaths(neo4j, query)
```

Again, this returns a list of path objects. You can find how many paths were found by returning the length of this list:


```r
length(p)
```

```
## [1] 3
```

### nodes

[nodes](http://nicolewhite.github.io/RNeo4j/docs/nodes.html) extracts the node objects from a path object. Because `paths` is a list of path objects, we need to `lapply` through it and apply the function `nodes` to each path in the list:


```r
n = lapply(p, nodes)
```

`n` is a list of lists. If we wanted to view the names of all the people on each path, for example, we can again use [the apply family of functions](https://nsaunders.wordpress.com/2010/08/20/a-brief-introduction-to-apply-in-r/). As we saw earlier when using `length`, three paths were found. The following displays the names of the nodes on each of the paths:


```r
lapply(n, function(x) sapply(x, `[[`, 'name'))
```

```
## [[1]]
## [1] "Alice"   "Bob"     "Charles" "David"  
## 
## [[2]]
## [1] "Alice" "Bob"   "David"
## 
## [[3]]
## [1] "Alice"  "Elaine" "David"
```

### rels

[rels](http://nicolewhite.github.io/RNeo4j/docs/rels.html), similarly to `nodes`, extracts the relationship objects from a path object. Recall that each relationship has a `weight` property.


```r
r = lapply(p, rels)
lapply(r, function(x) sapply(x, `[[`, 'weight'))
```

```
## [[1]]
## [1] 1 2 4
## 
## [[2]]
## [1] 1 3
## 
## [[3]]
## [1] 5 6
```

### shortestPath, allShortestPaths

[shortestPath](http://nicolewhite.github.io/RNeo4j/docs/shortest-path.html) and [allShortestPaths](http://nicolewhite.github.io/RNeo4j/docs/all-shortest-paths.html) find a single shortest path or all shortest paths between two node objects, respectively.


```r
p = shortestPath(alice, "LIKES", david, max_depth = 4)
sapply(nodes(p), `[[`, 'name')
```

```
## [1] "Alice" "Bob"   "David"
```


```r
p = allShortestPaths(alice, "LIKES", david, max_depth = 4)
n = lapply(p, nodes)
lapply(n, function(x) sapply(x, `[[`, 'name'))
```

```
## [[1]]
## [1] "Alice"  "Elaine" "David" 
## 
## [[2]]
## [1] "Alice" "Bob"   "David"
```

`allShortestPaths` found both of the these paths because they tie for the shortest path (they're both length-two paths). `shortestPath`, on the other hand, returned just one of these paths arbitrarily.

## Additional Sample Datasets

If you want to quickly begin exploring the capabilities of RNeo4j but don't have any datasets, you can import one of the sample datasets shipped with this package with [importSample](http://nicolewhite.github.io/RNeo4j/docs/import-sample.html). This will import the selected dataset into Neo4j. There are four datasets, ranging from travel to entertainment to social. These include:

* [Twitter](http://nicolewhite.github.io/RNeo4j/samples/#Tweets)
* [Dallas/Forth Worth Airport](http://nicolewhite.github.io/RNeo4j/samples/#DFW)
* [Caltrain](http://nicolewhite.github.io/RNeo4j/samples/#Caltrain)
* [Movies](http://nicolewhite.github.io/RNeo4j/samples/#Movies)

Each dataset can be imported with its respective keyword. To import the Twitter dataset, for example, use the keyword `"tweets"`:




```r
importSample(neo4j, "tweets")
```

When executing `importSample`, you will have to answer a prompt confirming it is okay to wipe your Neo4j database and import the selected dataset. You can get a decent overview of what's been imported into Neo4j with `summary`, which will show you what is related and how:


```r
summary(neo4j)
```

```
##      This       To   That
## 1    User    POSTS  Tweet
## 2 Hashtag     TAGS  Tweet
## 3   Tweet MENTIONS   User
## 4   Tweet REPLY_TO  Tweet
## 5   Tweet RETWEETS  Tweet
## 6   Tweet    USING Source
## 7   Tweet CONTAINS   Link
```

We can look at the top-mentioned users with [cypher](http://nicolewhite.github.io/RNeo4j/docs/cypher.html), which returns tabular Cypher results as a data.frame:


```r
query = "
MATCH (tweet:Tweet)-[:MENTIONS]->(user:User)
RETURN user.screen_name AS user, COUNT(tweet) AS mentions
ORDER BY mentions DESC
LIMIT 5
"

cypher(neo4j, query)
```

```
##              user mentions
## 1           neo4j       28
## 2        ikwattro        8
## 3 _nicolemargaret        7
## 4     rvanbruggen        7
## 5      Linkurious        6
```

## Open the Neo4j Browser in RStudio

A [pull request](https://github.com/nicolewhite/RNeo4j/pull/13) by [Kenneth Darrell](http://darrkj.github.io/blogs) makes it so you can open the Neo4j browser in [RStudio](http://www.rstudio.com/)'s viewer pane with [browse](http://nicolewhite.github.io/RNeo4j/docs/browse.html). I use this functionality all the time now, as I usually want to do a quick check to see if my data was imported correctly.

Recall that the Twitter dataset is currently in Neo4j, as we imported it earlier. Often, I'll open the Neo4j browser and run the query...

```
MATCH n RETURN n LIMIT 50
``` 

...so I can click around and make sure everthing looks as expected:


```r
browse(neo4j)
```

<a href="http://i.imgur.com/sGCrZ7h.png" target="_blank"><img src="http://i.imgur.com/sGCrZ7h.png" width="100%" height="100%"></a>

## Set Custom HTTP Options

A [pull request](https://github.com/nicolewhite/RNeo4j/pull/16) by [Mark Needham](http://www.markhneedham.com/blog/) makes it so you can set custom HTTP options. In particular, this is useful for setting the HTTP timeout. These options are set in [startGraph](http://nicolewhite.github.io/RNeo4j/docs/start-graph.html):


```r
neo4j = startGraph("http://localhost:7474/db/data/", 
                   opts = list(timeout=2))
```

Now, any terrible Cypher queries you write will time out after two seconds and you will regain access to your R console without the need to terminate R:


```r
query = "
MATCH p = ()-[*..5]-() RETURN LENGTH(p)
"
test = try(cypher(neo4j, query))
cat(test[1])
```

```
## Error in function (type, msg, asError = TRUE)  : 
##   Operation timed out after 2002 milliseconds with 0 bytes received
```

Note, however, that the query continues to run server-side. See [Mark's blog post](http://www.markhneedham.com/blog/2013/10/17/neo4j-setting-query-timeout/) on setting server timeouts.

Feel free to make your own contributions by [submitting pull requests to the RNeo4j GitHub repository](https://github.com/nicolewhite/RNeo4j/pulls).
