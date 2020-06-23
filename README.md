# hTBG

## Hierarchical Time-Biased Gain (hTBG)

Hierarchical Time-Biased Gain (hTBG) implementation from the ACL2020 paper [A Prioritization Model for Suicidality Risk Assessment](https://www.aclweb.org/anthology/2020.acl-main.723.pdf) by Han-Chin Shing, Philip Resnik, and Doug Oard.

This repository contains implementation for hTBG and TBG only. The SuicideWatch dataset, due to privacy concern, is not included. Please see the [UMD SuicideWatch Dataset](http://legacydirs.umiacs.umd.edu/~resnik/umd_reddit_suicidality_dataset.html) page for how to obtain the data.

## Commend Line Usage

See below for expected input and output format.

```
Usage:
    hTBG.py --relevance=<json>
            --prediction=<json>
            [--t_half_lives=<arg>]...
            [options]
    hTBG.py --help

Options:
    -h --help                    show this help message and exit
    -t --tbg                     run TBG, the non-hierarchical version of hTBG, stopping probabilities set to zero
    -v --verbose                 be verbose, print out hyper-parameters
    --p_click_true=<prob>        probability of clicking based on the summary if the target is relevant [default: 0.64]
    --p_click_false=<prob>       probability of clicking based on the summary if the target is not relevant [default: 0.39]
    --p_save_true=<prob>         probability of saving based on detailed review if the target is relevant [default: 0.77]
    --p_save_false=<prob>        probability of saving based on detailed review if the target is not relevant [default: 0.27]
    --t_summary=<sec>            time (in second) to read a summary [default: 4.4]
    --t_alpha=<float>            scaling factor from word to second [default: 0.018]
    --t_beta=<float>             bias factor from word to second [default: 7.8]
    --t_half_lives=<arg>         time (in second) of the half-live parameter in the decay function [default: 224. 1800.]
```

For the detailed meaning of the hyperparameters:

`Smucker, Mark D., and Charles LA Clarke. "Time-based calibration of effectiveness measures." SIGIR 2012`

For the detailed description of hTBG:

`Shing, Han-Chin, Resnik, Philip, and Oard, Doug. "A Prioritization Model for Suicidality Risk Assessment." ACL 2020`

## Use as a library

```python
from hTBG import hTBG
from pprint import pprint

htbg_parameters = {
    "p_click_true": 0.64,
    "p_click_false": 0.39,
    "p_save_true": 0.77,
    "p_save_false": 0.27,
    "t_summary": 4.4,
    "t_alpha": 0.018,
    "t_beta": 7.8,
    "t_half_lives": [224., 1800.]
}

# relevance and prediction follows the same input format as specified below.
# with open(relevance_path, 'r') as input_file:
#     relevance = json.load(input_file)
# with open(prediction_path, 'r') as input_file:
#     prediction = json.load(input_file)

htbg = hTBG(relevance=relevance,
            prediction=prediction,
            hierarchical=True,
            verbose=False,
            **htbg_parameters)

print('evaluation results:')
pprint(htbg.evaluate())

print('best possible values:')
pprint(htbg.evaluate_best())
```

## Expected Input Format:

### The relevance (truth) structure:

```
{
    "query_name": {
        "higher_level_name": [
            true_score (0 or 1),
            {
                "lower_level_name": [stopping_probability, cost_of_lower_level],
                ...
            }
        ],
        ...
    },
    ...
}
```

For an example, see `./hTBG_test/truth.json`

### The prediction structure:

```
{
    "query_name": {
        "higher_level_name": [
            predicted_score (float),
            {
                "lower_level_name": predicted_lower_level_score (float),
                ...
            }
        ],
        ...
    },
    ...
}
```

For an example, see `./hTBG_test/prediction.json`

### Output format

```
{
    "query_name": {
        t_half_life: score,
        ...
    },
    ...
}
```

## Example

### A toy example for hTBG

```
python hTBG.py --relevance ./hTBG_test/truth.json --prediction ./hTBG_test/prediction.json \
    --t_half_lives=3 --t_half_lives=5 --t_half_lives=10
```

using `./hTBG_test/truth.json` as the truth, `./hTBG_test/prediction.json` as the prediction, and evaluate at t_half_lives=3, 5, and 10

#### Expected output:

```
evaluation results:
{'q_1': {3.0: 0.5248706964598764,
         5.0: 0.588460647126441,
         10.0: 0.7099210881142366},
 'q_2': {3.0: 0.06428104166337158,
         5.0: 0.17063498548694406,
         10.0: 0.3878882375890544}}
best possible values:
{'q_1': {3.0: 0.543081360426777,
         5.0: 0.6180888697456681,
         10.0: 0.7412800897670984},
 'q_2': {3.0: 0.5406284846869924,
         5.0: 0.6143850709821594,
         10.0: 0.7375797438106515}}
```

### A toy example for TBG

Just turn on the `-t` parameter (equivalent to setting the stopping probabilities to zero)

```
python hTBG.py --relevance ./hTBG_test/truth.json --prediction ./hTBG_test/prediction.json -t \
    --t_half_lives=3 --t_half_lives=5 --t_half_lives=10
```

#### Expected output:

```
evaluation results:
{'q_1': {3.0: 0.5217239318926233,
         5.0: 0.5827130392122296,
         10.0: 0.7032973769997782},
 'q_2': {3.0: 0.06407785194702847,
         5.0: 0.169888953131207,
         10.0: 0.3864369224412395}}
best possible values:
{'q_1': {3.0: 0.5364948631648881,
         5.0: 0.607966590522515,
         10.0: 0.731031181438315},
 'q_2': {3.0: 0.5364948631648881,
         5.0: 0.607966590522515,
         10.0: 0.731031181438315}}
```

## Citation

If you use this software, please cite

**Shing et al., A Prioritization Model for Suicidality Risk Assessment, ACL2020**

```
@inproceedings{shing-etal-2020-prioritization,
    title = "A Prioritization Model for Suicidality Risk Assessment",
    author = "Shing, Han-Chin  and
      Resnik, Philip  and
      Oard, Douglas",
    booktitle = "Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics",
    month = jul,
    year = "2020",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/2020.acl-main.723",
    pages = "8124--8137",
    abstract = "We reframe suicide risk assessment from social media as a ranking problem whose goal is maximizing detection of severely at-risk individuals given the time available. Building on measures developed for resource-bounded document retrieval, we introduce a well founded evaluation paradigm, and demonstrate using an expert-annotated test collection that meaningful improvements over plausible cascade model baselines can be achieved using an approach that jointly ranks individuals and their social media posts.",
}
```
