# Recommending Tasks based on Search Queries and Missions

This repository provides resources developed within the article "Recommending Tasks based on Search Queries and Missions".


## Datasets

The datasets constructed in our work are placed under the directory `data/datasets/`.


### Corpus of Search Queries and Missions

This corpus, presented in Section 5.1 of the article, is in `data/datasets/corpus_of_missions.tsv`.

 - The format is the same as in [the original corpus](https://webis.de/data/webis-smc-12.html) on which ours is based. This is, each row in the file is in the format:

```
<UserID>	<Query>	<TimeStamp>	<ClickRank>	<ClickDomain>	<MissionID>	<Comment>
```


### Corpus of Procedural Search Missions

Our corpus of procedural missions, described in Section 5.2 of the article, is in the JSON file `data/datasets/corpus_of_procedural_missions.json`. The content is of the shape:

```
{
    <mission ID>: {
                      "all_queries": {<query ID>: <query>},
	              "best_queries": {<query ID>: <query>}
	              }
}
```

where:

  - `<mission ID>` is the mission ID (by construction, prefixed by the corresponding userID in the corpus of search queries and missions);
  - `"best_queries"` field holds only the best queries found per search mission; and
  - `"all_queries"` field contains all the queries in the search mission, including the best one(s).


### Test Collection

The test collection built in Section 5.4 of the article is placed under the directory `data/datasets/qrels/`. Each file is in the usual format of an input qrels file (i.e. the file with relevance judgements) for the `trec_eval` command.

- `qrels-QB.tsv` is the basic collection, which is used to evaluate query-based task recommendation (Section 6.2 of the article). Its format is:

```
<query ID>	Q0	<task ID>	<relevance>
```

- `qrels-MB.tsv` is used to evaluate a mission-based task recommendation runfile. The collection is obtained as explained in the last paragraph of Section 5.4 of the article. Its format is:

```
<mission ID>	Q0	<task ID>	<relevance>
```

- `qrels-QB-restricted.tsv` is used to evaluate the query-based task recommendation runfile in order to be comparable with a mission-based runfile. It's obtained after restricting the original collection to contain only a single best query per mission, randomly selected. Additionally, its query IDs are replaced by the respective mission IDs. Then, its format is:

```
<mission ID>	Q0	<task ID>	<relevance>
```


## Features for Learning to Recommend Tasks

The file `data/features/features-QB.json` contains the features for each query-task instance, used for learning to recommend tasks based on search queries.

Analogously, `data/features/features-MB.json` contains the features for each query-task instance, used for learning to recommend tasks based on search missions.

Each of these feature sets has this format:

```
{
  "QID:<query ID>|TASKID:<task ID>": {
    "features": {
      <feature>: <feature value>,
    },
    "properties": {
      "task_id": <task ID>,
      "query_id": <query ID>,
      "task_title": <task title>,
      "query": <query>
    },
    "target": <relevance>
  },
  ...
}
```


## Task Recommendation

The output task recommendation runfiles are placed under the directory `output/`. Each file is in the usual format of an input runfile for the `trec_eval` command. Its format is:

```
<query ID>	Q0	<task ID>	<rank position>	<score>	<run ID>
```

- `1-baselines/` contains the outputs from the text-based baselines (Section 6.1 of the article);
- `2-query_based/` contains the outputs from our query-based task recommendation approach (Section 6.2 of the article); and
- `3-mission_based/` contains the outputs from our mission-based task recommendation approach (Section 6.3 of the article).

[`trec_eval`](https://trec.nist.gov/trec_eval/) is [the standard tool used by the TREC community for
evaluating an retrieval method](https://www-nlpir.nist.gov/projects/trecvid/trecvid.tools/trec_eval_video/A.README). It calculates and prints various evaluation measures, evaluating the results file against the file of relevance judgements.

In order to evaluate a `<runfile>` against a `<qrels file>` in terms of the metrics used in the article, one can run the following:

```
$ path/to/trec_eval -c -m ndcg_cut.10 -m P.10 -m map <qrels file> <runfile>
```

For example, below it's shown how to call for the evaluation of the several methods and configurations to obtain the results from Tables 4, 5, and 6 in the article:

```
# Query-based: BM25-Title
path/to/trec_eval -c -m ndcg_cut.10 -m P.10 -m map data/datasets/qrels/qrels-QB.tsv output/1-baselines/BM25/results-BM25-title.trec.txt

# Query-based: configurations by some feature subsets
path/to/trec_eval -c -m ndcg_cut.10 -m P.10 -m map data/datasets/qrels/qrels-QB.tsv output/2-query_based/results-<LTR configuration>.trec.txt

# Query-based: restricted to be comparable with any mission-based runfile
path/to/trec_eval -c -m ndcg_cut.10 -m P.10 -m map data/datasets/qrels/qrels-QB-restricted.tsv output/2-query_based/results-LTR-restricted.trec.txt 

# Mission-based: configurations by method family and aggregator function
path/to/trec_eval -c -m ndcg_cut.10 -m P.10 -m map data/datasets/qrels/qrels-MB.tsv output/3-mission_based/by_<method family>/aggr_<aggregator function>/results.trec.txt
```

