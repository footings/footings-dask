---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: footings-dask
  language: python
  name: footings-dask

execution:
  timeout: -1
---

# Example

As an example consider we have a simple model that calculates the present value (PV) of premium for a single policy holder. It has 3 parameters -

- policy_id - the unique identifier for the policy holder,
- premium - a list of premium payments, and
- interest_rate - the interest rate used for discounting.

The model returns a `pandas.DataFrame` with one record where one column holds the policy_id and the other the PV of premium.

```{code-cell} ipython3
import pandas as pd
from footings.model import (
    model,
    step,
    def_parameter,
    def_intermediate,
    def_return,
)

@model(steps=["_calc_pv"])
class PvPremium:
    policy_id = def_parameter()
    premium = def_parameter()
    interest_rate = def_parameter()
    ret = def_return()

    @step(uses=["premium", "interest_rate"], impacts=["ret"])
    def _calc_pv(self):
        def discount():
            return sum([
                p / (1 + self.interest_rate) ** i for i, p in enumerate(self.premium)
            ])

        self.ret = pd.DataFrame({
            "POLICY_ID": [self.policy_id],
            "PV_PREMIUM": [discount()]
        })

```

The code below show how this model can be used for a single policy holder.

```{code-cell} ipython3
PvPremium(policy_id="policy1", premium=[100, 100, 100], interest_rate=0.05).run()
```

In practice, actuarial models need to be applied against many policy holders not just one. Thus, the `footings-dask` library provides a function `create_dask_foreach_jig` which makes it easy to turn a model that works for one policyholder into a function that can run the model on many policyholders. This is shown in the code section below -

```{code-cell} ipython3
from footings_dask import create_dask_foreach_jig

foreach_model = create_dask_foreach_jig(
    model=PvPremium,
    iterator_name="records",
    iterator_keys=("policy_id",),
    pass_iterator_keys=("policy_id",),
    constant_params=("interest_rate",),
    success_wrap=pd.concat,
)
```

A note on the parameters -

- `model` - represents the model which we want to run repeatedly
- `iterator_name` - the name to assign the iterator to be passed
- `iterator_keys` - the keys that form the unique id for a each iterator
- `pass_iterator_keys` - the iterator_keys to pass into the model
- `constant_params` - any parameters that are constant (i.e., same across all iterations)
- `success_wrap` - the function to apply at the end to the iterations that have been successfully run


Some of the parameters make more sense if the signature is inspected as the code snippet below shows.

```{code-cell} ipython3
from inspect import signature
signature(foreach_model)
```

The function takes two parameters -

- `records` which was assigned by the parameter `iterator_name` and
- `interest_rate` which was assigned by the parameter `constant_params`.

`records` is the `iterator` that will be passed to the model which will contain the `policy_id` and their associated `premium`. `interest_rate` is a constant parameter used across all iterations.

An example of using the function is below. Note that a bad record (i.e., "policy4") was included to show that the foreach_model will catch and record errors that show during iteration and continue to process any remaining records.

```{code-cell} ipython3
records = [
    {"policy_id": "policy1", "premium": [100, 100, 100]},
    {"policy_id": "policy2", "premium": [200, 200, 200]},
    {"policy_id": "policy3", "premium": [300, 300, 300]},
    {"policy_id": "policy4", "premium": ["1", "1", "1"]},
]

success, failure = foreach_model(records=records, interest_rate=0.05)

success
```

```{code-cell} ipython3
failure
```
