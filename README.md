# Ease multi-version support for scikit-learn compatible library

[![SPEC 0 — Minimum Supported Dependencies](https://img.shields.io/badge/SPEC-0-green?labelColor=%23004811&color=%235CA038)](https://scientific-python.org/specs/spec-0000/)

`sklearn_compat` is a small Python package that allows you to support new
scikit-learn features with older versions of scikit-learn.

The aim is to support a range of scikit-learn versions as specified in the
[SPEC0](https://scientific-python.org/specs/spec-0000/). It means that you will find
utility to support the last 4 released version of scikit-learn.

# How to adapt your scikit-learn code

## Upgrading to scikit-learn 1.6

### `validate_data`

Your previous code could have looked like this:

```python
class MyEstimator(BaseEstimator):
    def fit(self, X, y=None):
        X = self._validate_data(X, force_all_finite=True)
        return self
```

There is two major changes in scikit-learn 1.6:

- `validate_data` has been moved to `sklearn.utils.validation`.
- `force_all_finite` is deprecated in favor of the `ensure_all_finite` parameter.

You can now use the following code for backward compatibility:

```python
from sklearn_compat.utils.validation import validate_data

class MyEstimator(BaseEstimator):
    def fit(self, X, y=None):
        X = validate_data(self, X=X, ensure_all_finite=True)
        return self
```

### `_check_n_features` and `_check_feature_names`

Similarly to `validate_data`, these two functions have been moved to
`sklearn.utils.validation` instead of being methods of the estimators. So the following
code:

```python
class MyEstimator(BaseEstimator):
    def fit(self, X, y=None):
        self._check_n_features(X, reset=True)
        self._check_feature_names(X, reset=True)
        return self
```

becomes:

```python
from sklearn_compat.utils.validation import _check_n_features, _check_feature_names

class MyEstimator(BaseEstimator):
    def fit(self, X, y=None):
        _check_n_features(self, X, reset=True)
        _check_feature_names(self, X, reset=True)
        return self
```

### `Tags`, `__sklearn_tags__` and estimator tags

The estimator tags infrastructure in scikit-learn 1.6 has changed. In order to be
compatible with multiple scikit-learn versions, your estimator should implement both
`_more_tags` and `__sklearn_tags__`:

```python
class MyEstimator(BaseEstimator):
    def _more_tags(self):
        return {"non_deterministic": True, "poor_score": True}

    def __sklearn_tags__(self):
        tags = super().__sklearn_tags__()
        tags.non_deterministic = True
        tags.regressor_tags.poor_score = True
        return tags
```

In order to get the tags of a given estimator, you can use the `get_tags` function:

```python
from sklearn_compat.utils import get_tags

tags = get_tags(MyEstimator())
```

Which uses `sklearn.utils.get_tags` under the hood from scikit-learn 1.6+.

## Upgrading to scikit-learn 1.5

In scikit-learn 1.5, many developer utilities have been moved to dedicated modules.
We provide a compatibility layer such that you don't have to check the version or try
to import the utilities from different modules.

In the future, when supporting scikit-learn 1.6+, you will have to change the import
from:

```python
from sklearn_compat.utils._indexing import _safe_indexing
```

to

```python
from sklearn.utils._indexing import _safe_indexing
```

Thus, the module path will already be correct. Now, we will go into details for
each module and function impacted.

### `extmath` module

The function `safe_sqr` and `_approximate_mode` have been moved from `sklearn.utils` to
`sklearn.utils.extmath`.

So some code looking like this:

```python
from sklearn.utils import safe_sqr, _approximate_mode

safe_sqr(np.array([1, 2, 3]))
_approximate_mode(class_counts=np.array([4, 2]), n_draws=3, rng=0)
```

becomes:

```python
from sklearn_compat.utils.extmath import safe_sqr, _approximate_mode

safe_sqr(np.array([1, 2, 3]))
_approximate_mode(class_counts=np.array([4, 2]), n_draws=3, rng=0)
```

### `fixes` module

The functions `_in_unstable_openblas_configuration`, `_IS_32BIT` and `_IS_WASM` have
been moved from `sklearn.utils` to `sklearn.utils.fixes`.

So the following code:

```python
from sklearn.utils import (
    _in_unstable_openblas_configuration,
    _IS_32BIT,
    _IS_WASM,
)

_in_unstable_openblas_configuration()
print(_IS_32BIT)
print(_IS_WASM)
```

becomes:

```python
from sklearn_compat.utils.fixes import (
    _in_unstable_openblas_configuration,
    _IS_32BIT,
    _IS_WASM,
)

_in_unstable_openblas_configuration()
print(_IS_32BIT)
print(_IS_WASM)
```

### `validation` module

The function `_to_object_array` has been moved from `sklearn.utils` to
`sklearn.utils.validation`.

So the following code:

```python
from sklearn.utils import _to_object_array

_to_object_array([np.array([0]), np.array([1])])
```

becomes:

```python
from sklearn_compat.utils.validation import _to_object_array

_to_object_array([np.array([0]), np.array([1])])
```

### `_chunking` module

The functions `gen_batches`, `gen_even_slices` and `get_chunk_n_rows` have been moved
from `sklearn.utils` to `sklearn.utils._chunking`. The function `chunk_generator` has
been moved to `sklearn.utils._chunking` as well but was renamed from `_chunk_generator`
to `chunk_generator`.

So the following code:

```python
from sklearn.utils import (
    _chunk_generator as chunk_generator,
    gen_batches,
    gen_even_slices,
    get_chunk_n_rows,
)

_chunk_generator(range(10), 3)
gen_batches(7, 3)
gen_even_slices(10, 1)
get_chunk_n_rows(10)
```

becomes:

```python
from sklearn_compat.utils._chunking import (
    chunk_generator, gen_batches, gen_even_slices, get_chunk_n_rows,
)

chunk_generator(range(10), 3)
gen_batches(7, 3)
gen_even_slices(10, 1)
get_chunk_n_rows(10)
```

### `_indexing` module

The utility functions `_determine_key_type`, `_safe_indexing`, `_safe_assign`,
`_get_column_indices`, `resample` and `shuffle` have been moved from `sklearn.utils` to
`sklearn.utils._indexing`.

So the following code:

```python
import numpy as np
import pandas as pd
from sklearn.utils import (
    _get_column_indices,
    _safe_indexing,
    _safe_assign,
    resample,
    shuffle,
)

_determine_key_type(np.arange(10))

df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
_get_column_indices(df, key="b")
_safe_indexing(df, 1, axis=1)
_safe_assign(df, 1, np.array([7, 8, 9]))

array = np.arange(10)
resample(array, n_samples=20, replace=True, random_state=0)
shuffle(array, random_state=0)
```

becomes:

```python
import numpy as np
import pandas as pd
from sklearn_compat.utils._indexing import (
    _determine_key_type,
    _safe_indexing,
    _safe_assign,
    _get_column_indices,
    resample,
    shuffle,
)

_determine_key_type(np.arange(10))

df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
_get_column_indices(df, key="b")
_safe_indexing(df, 1, axis=1)
_safe_assign(df, 1, np.array([7, 8, 9]))

array = np.arange(10)
resample(array, n_samples=20, replace=True, random_state=0)
shuffle(array, random_state=0)
```

### `_mask` module

The functions `safe_mask`, `axis0_safe_slice` and `indices_to_mask` have been moved from
`sklearn.utils` to `sklearn.utils._mask`.

So the following code:

```python
from sklearn.utils import safe_mask, axis0_safe_slice, indices_to_mask

safe_mask(data, condition)
axis0_safe_slice(X, mask, X.shape[0])
indices_to_mask(indices, 5)
```

becomes:

```python
from sklearn_compat.utils._mask import safe_mask, axis0_safe_slice, indices_to_mask

safe_mask(data, condition)
axis0_safe_slice(X, mask, X.shape[0])
indices_to_mask(indices, 5)
```

### `_missing` module

The functions `is_scalar_nan` have been moved from `sklearn.utils` to
`sklearn.utils._missing`. The function `_is_pandas_na` has been moved to
`sklearn.utils._missing` as well and renamed to `is_pandas_na`.

So the following code:

```python
from sklearn.utils import is_scalar_nan, _is_pandas_na

is_scalar_nan(float("nan"))
_is_pandas_na(float("nan"))
```

becomes:

```python
from sklearn_compat.utils._missing import is_scalar_nan, is_pandas_na

is_scalar_nan(float("nan"))
is_pandas_na(float("nan"))
```

### `_user_interface` module

The function `_print_elapsed_time` has been moved from `sklearn.utils` to
`sklearn.utils._user_interface`.

So the following code:

```python
from sklearn.utils import _print_elapsed_time

with _print_elapsed_time("sklearn_compat", "testing"):
    time.sleep(0.1)
```

becomes:

```python
from sklearn_compat.utils._user_interface import _print_elapsed_time

with _print_elapsed_time("sklearn_compat", "testing"):
    time.sleep(0.1)
```

### `_optional_dependencies` module

The functions `check_matplotlib_support` and `check_pandas_support` have been moved from
`sklearn.utils` to `sklearn.utils._optional_dependencies`.

So the following code:

```python
from sklearn.utils import check_matplotlib_support, check_pandas_support

check_matplotlib_support("sklearn_compat")
check_pandas_support("sklearn_compat")
```

becomes:

```python
from sklearn_compat.utils._optional_dependencies import (
    check_matplotlib_support, check_pandas_support
)

check_matplotlib_support("sklearn_compat")
check_pandas_support("sklearn_compat")
```

## Upgrading to scikit-learn 1.4

### `process_routing`

The signature of the `process_routing` function changed in scikit-learn 1.4. You don't
need to change the import but only the call to the function. Calling the function with
the new signature will be compatible with the previous signature as well. So a code
looking like this:

```python
class MetaEstimator(BaseEstimator):
    def fit(self, X, y, sample_weight=None, **fit_params):
        params = process_routing(self, "fit", fit_params, sample_weight=sample_weight)
        return self
```

becomes:

```python
class MetaEstimator(BaseEstimator):
    def fit(self, X, y, sample_weight=None, **fit_params):
        params = process_routing(self, "fit", sample_weight=sample_weight, **fit_params)
        return self
```

## Parameter validation

scikit-learn introduced a new way to validate parameters at `fit` time. The recommended
way to support this feature in scikit-learn 1.3+ is to inherit from
`sklearn.base.BaseEstimator` and decorate the `fit` method using the decorator
`sklearn.base._fit_context`. In this package, we provide a mixin class in case you
don't want to inherit from `sklearn.base.BaseEstimator`.

So a small example could have been in the past:

```python
class MyEstimator:
    def __init__(self, a=1):
        self.a = a

    def fit(self, X, y=None):
        if self.a < 0:
            raise ValueError("a must be positive")
        return self
```

becomes:

```python
from sklearn_compat.base import ParamsValidationMixin

class MyEstimator(ParamsValidationMixin):
    _parameter_constraints = {"a": [Interval(Integral, 0, None, closed="left")]}

    def __init__(self, a=1):
        self.a = a

    @_fit_context(prefer_skip_nested_validation=True)
    def fit(self, X, y=None):
        return self
```

The advantage is that the error raised will be more informative and consistent across
estimators. Also, we have the possibility to skip the validation of the parameters when
using this estimator as a meta-estimator.
