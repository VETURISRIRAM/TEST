# IR HW4

Name: Sriram Veturi

UIN: 651427659

## Description

PageRank to derive a ranking of words in a document based on their PageRank scores.

## How to run?

There are two command line inputs

1) `data_path`: Path the files of the data.

2) `w` : w parameter

### Example run:

```python main.py --data_path ./www/www/ --w 6```

### Sample Output for w=6

```
K = 1 --> MMR : 0.0
K = 2 --> MMR : 0.6421052631578947
K = 3 --> MMR : 0.6255639097744361
K = 4 --> MMR : 0.5319548872180455
K = 5 --> MMR : 0.4689223057644111
K = 6 --> MMR : 0.4561403508771925
K = 7 --> MMR : 0.4510651629072684
K = 8 --> MMR : 0.3342337987826705
K = 9 --> MMR : 0.32255639097744243
K = 10 --> MMR : 0.2771631459601365
```

`Note`: You can also find this dictionary in a JSON file (mmr_map.json) that gets created at every run. 

I faced problem in applying the algorithm to this problem. I asked Vinay Valluru to explain me the algorithm. I followed the algorithm explained by him in this code.
