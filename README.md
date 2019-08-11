Welcome to project Druidry!
=======================================

![build_status](https://api.travis-ci.org/zapr-oss/druidry.svg?branch=master) [![License: Apache License 2](https://img.shields.io/badge/license-Apache%202-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0.txt)

Druid is an extremely popular tool to perform OLAP queries on event data. Druid drives real-time dashboards in most of the organisations right now. We@Zapr love Druid! Therefore we want to contribute towards making Druid, even more, friendlier to the ever expanding community. 

We want to make the process of deep meaningful conversations with Druid little easier. What do we mean is that we don’t want developers to write big, scary JSON anymore but instead use a simple Java-based query generator to help with the querying. 

Creating JSON freely can cause tedious bugs such as date type mistakes or spelling mistakes and potentially code can get bigger and messier and less readable. So, in reality, we want to keep the main focus of querying to be the use-case, not the type-checks.

We are excited to know whether you liked it or loved it, so please reach out to us at opensource@zapr.in


Description
-----------

Druidry is an open-source Java based utility library which supports creating query to Druid automatically taking care of following,

* Type checking.
* Spelling Checks.
* Code reviewability and readability.

This library is still growing and does not support each and every constructs, however it supports the most common one used internally @Zapr.


Getting Started
---------------

Prerequisite
-----------

* Maven
* Java 8

Usage
-----

Add this in your pom.xml (assuming maven based project)

```xml
        <dependency>
            <groupId>in.zapr.druid</groupId>
            <artifactId>druidry</artifactId>
            <version>${LATEST_VERSION}</version>
        </dependency>
```

Replace ${LATEST_VERSION} with latest release version 

Examples
--------

Taking from Druid's example query

```json

{
     "queryType": "topN",
     "dataSource": "sample_data",
     "dimension": "sample_dim",
     "threshold": 5,
     "metric": "count",
     "granularity": "all",
     "filter": {
        "type": "and",
          "fields": [
            {
              "type": "selector",
              "dimension": "dim1",
              "value": "some_value"
            },
            {
              "type": "selector",
              "dimension": "dim2",
              "value": "some_other_val"
            }
          ]
     },
     "aggregations": [
        {
          "type": "longSum",
          "name": "count",
          "fieldName": "count"
        },
        {
          "type": "doubleSum",
          "name": "some_metric",
          "fieldName": "some_metric"
        }
     ],
     "postAggregations": [
         {
            "type": "arithmetic",
            "name": "sample_divide",
            "fn": "/",
            "fields": [
              {
                "type": "fieldAccess",
                "name": "some_metric",
                "fieldName": "some_metric"
              },
              {
                "type": "fieldAccess",
                "name": "count",
                "fieldName": "count"
              }
            ]
         }
      ],
      "intervals": [
        "2013-08-31T00:00:00.000/2013-09-03T00:00:00.000"
      ]
}
```


```java
SelectorFilter selectorFilter1 = new SelectorFilter("dim1", "some_value");
SelectorFilter selectorFilter2 = new SelectorFilter("dim2", "some_other_val");

AndFilter filter = new AndFilter(Arrays.asList(selectorFilter1, selectorFilter2));

DruidAggregator aggregator1 = new LongSumAggregator("count", "count");
DruidAggregator aggregator2 = new DoubleSumAggregator("some_metric", "some_metric");

FieldAccessPostAggregator fieldAccessPostAggregator1
        = new FieldAccessPostAggregator("some_metric", "some_metric");

FieldAccessPostAggregator fieldAccessPostAggregator2
        = new FieldAccessPostAggregator("count", "count");

DruidPostAggregator postAggregator = ArithmeticPostAggregator.builder()
        .name("sample_divide")
        .function(ArithmeticFunction.DIVIDE)
        .fields(Arrays.asList(fieldAccessPostAggregator1, fieldAccessPostAggregator2))
        .build();

DateTime startTime = new DateTime(2013, 8, 31, 0, 0, 0, DateTimeZone.UTC);
DateTime endTime = new DateTime(2013, 9, 3, 0, 0, 0, DateTimeZone.UTC);
Interval interval = new Interval(startTime, endTime);

Granularity granularity = new SimpleGranularity(PredefinedGranularity.ALL);
DruidDimension dimension = new SimpleDimension("sample_dim");
TopNMetric metric = new SimpleMetric("count");

DruidTopNQuery query = DruidTopNQuery.builder()
        .dataSource("sample_data")
        .dimension(dimension)
        .threshold(5)
        .topNMetric(metric)
        .granularity(granularity)
        .filter(filter)
        .aggregators(Arrays.asList(aggregator1, aggregator2))
        .postAggregators(Collections.singletonList(postAggregator))
        .intervals(Collections.singletonList(interval))
        .build();

ObjectMapper mapper = new ObjectMapper();
String requiredJson = mapper.writeValueAsString(query);
```

``` java
DruidConfiguration config =  DruidConfiguration
               .builder()
               .host("druid.io")
               .endpoint("druid/v2/")
               .build();

DruidClient client = new DruidJerseyClient(druidConfiguration);
client.connect();
List<DruidResponse> responses = client.query(query, DruidResponse.class);
client.close();
```

Supported Features
------------------

#### Queries
* Aggregation Queries
    * TopN
    * TimeSeries
    * GroupBy
* DruidScanQuery
* DruidSelectQuery


#### Aggregators

* Cardinality
* Count
* DoubleMax
* DoubleMin
* DoubleSum
* DoubleLast
* DoubleFirst
* FloatFirst
* FloatLast
* Filtered
* HyperUnique
* Javascript
* LongMax
* LongMin
* LongSum
* LongFirst
* LongLast
* DistinctCount
* Histogram
* [Data Sketches](https://druid.apache.org/docs/latest/development/extensions-core/datasketches-extension.html)
    * ThetaSketch
    * TupleSketch
    * QuantilesSketch
    * HllSketchBuild
    * HllSketchMerge

#### Filters

* And
* Bound
* In
* Interval (Without Extraction Function)
* Javascript
* Not
* Or
* Regex
* Search (Without Extraction Function)
* Selector

#### Post Aggregators
* Arithmetic
* Constant
* FieldAccess
* HyperUniqueCardinality
* Javascript
* [Data Sketches](https://druid.apache.org/docs/latest/development/extensions-core/datasketches-extension.html)
    * [Theta Sketch](https://druid.apache.org/docs/latest/development/extensions-core/datasketches-theta.html)
        * ThetaSketchEstimate
        * ThetaSketchSetOp
    * [Tuple Sketch](https://druid.apache.org/docs/latest/development/extensions-core/datasketches-tuple.html)
        * TupleSketchToEstimate
        * TupleSketchToEstimateAndBounds
        * TupleSketchToNumEntries
        * TupleSketchToMeans
        * TupleSketchToVariances
        * TupleSketchToQuantilesSketch
        * TupleSketchSetOp
        * TupleSketchTTest
        * TupleSketchToString
    * [Quantiles Sketch](https://druid.apache.org/docs/latest/development/extensions-core/datasketches-quantiles.html)
        * QuantilesSketchToQuantile
        * QuantilesSketchToQuantiles
        * QuantilesSketchToHistogram
        * QuantilesSketchToString
    * [HLL Sketch](https://druid.apache.org/docs/latest/development/extensions-core/datasketches-hll.html)
        * HllSketchEstimateWithBounds
        * HllSketchUnion
        * HllSketchToString

#### Virtual Columns
* Expression

#### Granularity
* Duration
* Period
* Predefined

Contact
------

For any features or bugs, please raise it in issues section

If anything else, get in touch with us at [opensource@zapr.in](opensource@zapr.in)

Story Time
------

It all began with this 4th or 5th PR that I raised to zapr-oss/druidry repository.
It showed the tag saying `first time contributor`. I had my suspicions then and there.
I raised mail to support asking about why it shows such tag even though its not my first commit/PR, how do then assign this tag, etc, etc.

Few days went by, 8 to be precise. No reply from github developer support.
Then today I was going through the commits of zapr-oss/druidry to find a particular commit and 
I observed all commits which were supposed to be by my personal user, was showing as done by another user.
Then I went to my personal fork and you guessed it right, there also it showed as done by another user.
How can that be ? How can another user commit to my fork ? That to without even PR, directly.

Then I discovered that I made a major f*ck up not setting repo local git user.email. 
Global git user.email is my work email and now all of my commits from my personal user is signed as committed by my work user.
All my contributions, shows that they are done by work account. :-(

So now testing if github allows you to set any valid user.email and if it assigns the commit to that user.
If so, this commit should seen as signed by @detel

Update : It did exactly that. What the f*ck !!

IMO this method of assigning commits to whatever email you provide in your git user.email config is wrong.
May be this could be purely representational thing, may be internally commit-id still somehow points to my user. 
But still it's wrong, it should not show it as committed by different user, even for representational purpose.
I can give any email and once that commit is assigned to the desired user, 
I can contribute to some repository and once my contributions are merged, 
in that repository's history that particular commit will be shown as done by my desired user who doesn't event know he did that contribution. 
They should assign commit to fork-owner/account/sshkey through which is comes from.
Or at least they should check that @detel user, whom I have not given write access to my repo/fork, how is he/she able to write ? And raise some error.