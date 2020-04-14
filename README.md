# hTBG

## Hierarchical Time-Biased Gain (hTBG)

A Hierarchical Time-Biased Gain (hTBG) implementation from the paper **Shing et al., A Prioritization Model for Suicidality Risk Assessment, ACL2020, To appear**

## Usage

```
Usage:
    hTBG.py --relevance=<json>
            --prediction=<json>
            [--t_half_lives=<arg>]...
            [options]
    hTBG.py --help

Options:
    -h --help                    show this help message and exit
    -t --tbg                     run with non-hierarchical version of hTBG, stopping probabilities set to zero
    -v --verbose                 be verbose, print out hyper-parameters
    --p_click_true=<prob>        probability of clicking based on the summary if the target is relevant [default: 0.64]
    --p_click_false=<prob>       probability of clicking based on the summary if the target is not relevant [default: 0.39]
    --p_save_true=<prob>         probability of saving based on detailed review if the target is relevant [default: 0.77]
    --p_save_false=<prob>        probability of saving based on detailed review if the target is not relevant [default: 0.27]
    --t_summary=<sec>            time (in second) to read a summary [default: 4.4]
    --t_alpha=<float>            scaling factor from word to second [default: 0.018]
    --t_beta=<float>             bias factor from word to second [default: 7.8]
    --t_half_lives=<arg>         time (in second) of the half-live parameter in the decay function [default: 224. 1800.]
    -o <path>, --output <path>   output path
Example:
    python hTBG.py --relevance ./hTBG_test/truth.json --prediction ./hTBG_test/prediction.json \
    --t_half_lives=3 --t_half_lives=5 --t_half_lives=10
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

## An example

```
python hTBG.py --relevance ./hTBG_test/truth.json --prediction ./hTBG_test/prediction.json \
    --t_half_lives=3 --t_half_lives=5 --t_half_lives=10
```

using `./hTBG_test/truth.json` as the truth, `./hTBG_test/prediction.json` as the prediction, and evaluate at t_half_lives=3, 5, and 10

### Expected output:

```
evaluation results:
{'q_1': {3: 0.5248706964598764, 5: 0.588460647126441, 10: 0.7099210881142366},
 'q_2': {3: 0.06428104166337158, 5: 0.17063498548694406, 10: 0.3878882375890544}}
best possible values:
{'q_1': {3: 0.543081360426777, 5: 0.6180888697456681, 10: 0.7412800897670984},
 'q_2': {3: 0.5406284846869924, 5: 0.6143850709821594, 10: 0.7375797438106515}}
```

## Citation

If you use this software, please cite

**Shing et al., A Prioritization Model for Suicidality Risk Assessment, ACL2020**
