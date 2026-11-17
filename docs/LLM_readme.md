# tslearn — LLM_readme

LLM-facing API documentation. Generated from the API surface, project profile, and distilled docs.

## API Test: `AutoVARIMA`

### Signature
```python
class AutoVARIMA(TimeSeriesMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/forecasting/_arima.py:466_

_Source doc:_ Automatically selects the best Vector Autoregressive Integrated Moving Average (VARIMA) model through Hyndman-Khandakar algorithm [1]_ [2]_. Parameters ---------- max_p : int (default: 5) Maximum order of the AutoRegressive (AR) component to consider. max_d : int (default: 2) Maximum order of differencing considered to achieve stationarity. max_q : int (default: 5) Maximum order of the Moving-Average (MA) component to consider. default_d_for_non_stationarity : int or None (default 0) Used as differentiation order if stationarity cannot be achieved within max_d. If None, an error is raised if stationarity cannot be achieved within max_d. seasonal_period: int or None (default: None) Naïve seasonal integration to apply to VARIMA models max_iter : int (default: 50) The maximum number of iterations to apply to VARIMA models fitting. verbose : int (default 0) When set to a positive integer, displays logs of the tested VARIMA models selection process. If set to 2 or more, also propagates verbosity to the VARIMA models. Attributes ---------- best_estimator_: VARIMA the fitted VARIMA model. Notes ----- This estimator supports variable length time-series See Also -------- VARIMA: Vector AutoRegressive Integrated Moving Average (VARIMA) estimator. References ---------- .. [1] R. J. Hyndman and G. Athanasopoulos, Forecasting: Principles and Practice. OTexts, 2014. https://otexts.com/fpp3/ .. [2] Hyndman, R. J., & Khandakar, Y. (2008). Automatic time series forecasting: The forecast package for R. Journal of Statistical Software, 27(1), 1–22.

### Goal
`AutoVARIMA` performs automated model selection for Vector Autoregressive Integrated Moving Average (VARIMA) processes by searching for the optimal order parameters (p, d, q) using the Hyndman-Khandakar algorithm.

### Parameters
*   **max_p** (int, default: 5): Maximum order of the AutoRegressive (AR) component.
*   **max_d** (int, default: 2): Maximum order of differencing to achieve stationarity.
*   **max_q** (int, default: 5): Maximum order of the Moving-Average (MA) component.
*   **default_d_for_non_stationarity** (int or None, default: 0): Differentiation order used if stationarity is not reached within `max_d`. If `None`, raises an error on failure.
*   **seasonal_period** (int or None, default: None): Period for naïve seasonal integration.
*   **max_iter** (int, default: 50): Maximum iterations for fitting individual VARIMA models.
*   **verbose** (int, default: 0): Controls logging output; values >= 2 propagate verbosity to internal models.

### Input
*   **Data:** 3D NumPy array of shape `(n_ts, sz, d)`.
*   **Preconditions:** Input must be formatted as a time-series dataset (use `tslearn.utils.to_time_series_dataset` if necessary). Supports variable-length time series.

### Output
Returns an instance of `AutoVARIMA` with a populated `best_estimator_` attribute, which is a fitted `VARIMA` model.

### Valid Call Patterns
```python
from tslearn.forecasting import AutoVARIMA
import numpy as np

# Initialize and fit
model = AutoVARIMA(max_p=1, max_q=1, max_d=1)
model.fit(X)

# Access the best model
best_model = model.best_estimator_
```

### LLM Instruction Prompt
- Always initialize `AutoVARIMA` with specific order constraints if the dataset is large to avoid excessive search times.
- Ensure input data is 3D; if using 1D or 2D arrays, wrap them in `tslearn.utils.to_time_series_dataset`.
- Use `verbose=1` to monitor the selection process if the model takes longer than expected to converge.

### Prompt Snippet
```text
from tslearn.forecasting import AutoVARIMA
model = AutoVARIMA(max_p=2, max_d=1, max_q=2)
model.fit(X_train)
```

### Common Failure Modes
- **ValueError:** Raised if `default_d_for_non_stationarity` is `None` and the data fails to reach stationarity within `max_d`.
- **ValueError:** Raised if the input time series length is insufficient for the specified model orders or seasonal period.
- **Convergence Issues:** If `max_iter` is too low, the internal `VARIMA` models may fail to converge, leading to suboptimal selection.

### Fix Code Hint
```python
# If fitting fails due to stationarity, increase max_d or check data scaling
model = AutoVARIMA(max_d=3, default_d_for_non_stationarity=1).fit(X)
```

## API Test: `Backend`

### Signature
```python
class Backend(object)
```
_Source: tslearn/tslearn/backend/backend.py:55_

_Source doc:_ Class for the  backend. Parameter --------- data : array-like or string or None Indicates the backend to choose. If data is a Numpy array or data equals 'numpy' or data is None, self.backend is set to NumpyBackend(). If data is a PyTorch array or data equals 'pytorch', self.backend is set to PytorchBackend(). Optional, default equals None.

### Goal
The `Backend` class manages the selection and instantiation of the underlying computation engine (NumPy or PyTorch) used for time-series metric calculations and alignment operations.

### Parameters
_None._

### Input
The constructor accepts an optional `data` argument:
*   **`data` (array-like, string, or None):** 
    *   If `None` or `'numpy'`, initializes the `NumPyBackend`.
    *   If `'pytorch'`, initializes the `PyTorchBackend`.
    *   If an array-like object is provided, the backend is inferred based on the array type (NumPy array vs. PyTorch tensor).

### Output
Returns an instance of the `Backend` class, which acts as a wrapper for the selected computation engine.

### Valid Call Patterns
```python
from tslearn.backend import Backend
from tslearn.backend import NumPyBackend, PyTorchBackend

# Initialize with default (NumPy)
be = Backend()

# Initialize with explicit backend string
be_torch = Backend("pytorch")

# Switch backend dynamically
be.set_backend("numpy")
```

### LLM Instruction Prompt
- When using `tslearn` metrics that support backend selection, use the `Backend` class to manage the engine state.
- Always verify if the `pytorch` package is installed before attempting to initialize `Backend("pytorch")`.
- Use `is_pytorch` or `is_numpy` properties to check the current state of the backend instance.

### Prompt Snippet
```text
from tslearn.backend import Backend
be = Backend("pytorch")
# Use be.get_backend() to access the underlying engine
```

### Common Failure Modes
- **Missing Dependency:** Initializing `Backend("pytorch")` will raise a `ValueError` or `ImportError` if the `pytorch` library is not installed in the environment.
- **Invalid String:** Passing an unsupported string (e.g., `Backend("tensorflow")`) will result in an error as only `'numpy'` and `'pytorch'` are supported.
- **State Mismatch:** Assuming a specific backend is active without checking `is_pytorch` or `is_numpy` can lead to type errors when passing tensors to functions expecting NumPy arrays.

### Fix Code Hint
```python
from tslearn.backend import Backend
try:
    be = Backend("pytorch")
except Exception:
    be = Backend("numpy")
    print("PyTorch not available, falling back to NumPy.")
```

## API Test: `BaseModelPackage`

### Signature
```python
class BaseModelPackage(object)
```
_Source: tslearn/tslearn/bases/bases.py:78_

### Goal
`BaseModelPackage` serves as a foundational base class for internal model packaging within the `tslearn` ecosystem, providing a common structure for estimators.

### Parameters
_None._

### Input
This class is intended for internal inheritance. It does not accept direct arguments upon instantiation.

### Output
Returns an instance of `BaseModelPackage`.

### Valid Call Patterns
```python
from tslearn.bases import BaseModelPackage

# Instantiation as a base class
model = BaseModelPackage()
print(f"Correctness witness: {isinstance(model, BaseModelPackage)}")
```

### LLM Instruction Prompt
- Use `BaseModelPackage` only as a parent class when defining custom estimators that need to integrate with `tslearn`'s internal architecture.
- Do not attempt to pass data directly to `BaseModelPackage` as it does not implement `fit` or `predict` methods.

### Prompt Snippet
```text
from tslearn.bases import BaseModelPackage
class MyCustomEstimator(BaseModelPackage):
    ...
```

### Common Failure Modes
- **Direct Usage:** Attempting to use `BaseModelPackage` as a functional estimator (e.g., calling `.fit()` on it) will result in an `AttributeError` as it lacks standard scikit-learn estimator methods.
- **Misunderstanding Inheritance:** Assuming `BaseModelPackage` provides automatic `scikit-learn` compatibility without implementing the required `fit` and `predict` interface methods.

### Fix Code Hint
```python
from tslearn.bases import BaseModelPackage
from sklearn.base import BaseEstimator

# Ensure custom estimators inherit from both the internal base and scikit-learn's BaseEstimator
class MyCustomEstimator(BaseModelPackage, BaseEstimator):
    def fit(self, X, y=None):
        # Implementation required
        return self
```

## API Test: `CachedDatasets`

### Signature
```python
class CachedDatasets
```
_Source: tslearn/tslearn/datasets/cached.py:5_

_Source doc:_ A convenience class to access cached time series datasets. Note, that these *cached datasets* are statically included into *tslearn* and are distinct from the ones in :class:`UCR_UEA_datasets`. When using the Trace dataset, please cite [1]_. See Also -------- UCR_UEA_datasets : Provides more datasets and supports caching. References ---------- .. [1] A. Bagnall, J. Lines, W. Vickers and E. Keogh, The UEA & UCR Time Series Classification Repository, www.timeseriesclassification.com

### Goal
Provides a mechanism to load and list time-series datasets that are statically bundled within the `tslearn` package for quick experimentation.

### Parameters
_None._

### Input
No input parameters are required to instantiate the class. The datasets are pre-cached within the library installation.

### Output
Returns an instance of `CachedDatasets`. The instance provides methods (such as `list_datasets()`) to inspect available data and load specific time-series benchmarks.

### Valid Call Patterns
```python
from tslearn.datasets import CachedDatasets

# Instantiate the loader
data_loader = CachedDatasets()

# List available datasets
available = data_loader.list_datasets()
print(f"Available datasets: {available}")
```

### LLM Instruction Prompt
- Use `CachedDatasets` when the user requests a quick, built-in time-series dataset for testing or demonstration purposes.
- Always instantiate the class before calling its methods.
- Note that these datasets are distinct from those provided by `UCR_UEA_datasets`.
- If the user specifically requests the "Trace" dataset, remind them to cite the reference provided in the class documentation.

### Prompt Snippet
```text
from tslearn.datasets import CachedDatasets
loader = CachedDatasets()
# Use loader.list_datasets() to see options or load_dataset(name) to retrieve data
```

### Common Failure Modes
- **AttributeError:** Attempting to call methods on the class itself rather than an instantiated object (e.g., `CachedDatasets.list_datasets()` instead of `CachedDatasets().list_datasets()`).
- **Dataset Not Found:** Passing a dataset name to a loading method that is not included in the static cache (verify availability via `list_datasets()` first).
- **Confusion with UCR_UEA_datasets:** Expecting `CachedDatasets` to support the full range of UCR/UEA repository features; it is limited to the statically included subset.

### Fix Code Hint
```python
# Ensure the object is instantiated
data_loader = CachedDatasets()
# Verify the dataset exists before loading
if "Trace" in data_loader.list_datasets():
    # Proceed with loading
    pass
```

## API Test: `EmptyClusterError`

### Signature
```python
class EmptyClusterError(Exception)
```
_Source: tslearn/tslearn/clustering/utils.py:17_

### Goal
`EmptyClusterError` is a custom exception raised by clustering algorithms when an iteration results in a cluster containing no assigned time-series samples.

### Parameters
_None._

### Input
This exception is raised internally by clustering estimators (e.g., `TimeSeriesKMeans`) during the `fit` process. It does not require direct user input to instantiate, but it is triggered when the input dataset provided to an estimator is insufficient to populate all initialized clusters.

### Output
Returns `unspecified` — This is an exception class; when raised, it halts execution of the clustering algorithm and provides a traceback indicating that the model configuration (e.g., number of clusters `n_clusters`) is incompatible with the provided data distribution.

### Valid Call Patterns
```python
from tslearn.clustering.utils import EmptyClusterError

# This exception is typically raised by the library, 
# but can be caught in user code:
try:
    # ... clustering logic ...
    raise EmptyClusterError("Cluster is empty")
except EmptyClusterError as e:
    print(f"Caught expected error: {e}")
```

### LLM Instruction Prompt
- When generating code that uses `tslearn` clustering estimators, ensure the user is aware that `EmptyClusterError` may be raised if `n_clusters` is set too high relative to the number of samples in the dataset.
- Do not attempt to instantiate this class as a return value; it is intended for error handling and control flow interruption.

### Prompt Snippet
```text
from tslearn.clustering.utils import EmptyClusterError
# Handle potential empty clusters during model fitting
try:
    model.fit(X)
except EmptyClusterError:
    # Handle the case where n_clusters is too large for the dataset
```

### Common Failure Modes
- **Over-clustering:** Setting `n_clusters` to a value greater than or equal to the number of time series in the dataset, which often leads to empty clusters.
- **Poor Initialization:** Using random initialization strategies on sparse or highly localized data that may result in empty Voronoi cells during the first iteration of K-Means.

### Fix Code Hint
```python
# If EmptyClusterError is raised, reduce the number of clusters:
model = TimeSeriesKMeans(n_clusters=min(n_clusters, len(X)))
model.fit(X)
```

## API Test: `GlobalArgminPooling1D`

### Signature
```python
class GlobalArgminPooling1D(Layer)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:97_

_Source doc:_ Global argmin pooling operation for temporal data. # Input shape 3D tensor with shape: `(batch_size, steps, features)`. # Output shape 2D tensor with shape: `(batch_size, features)`. Examples -------- >>> x = numpy.array([5.0, 6.8, numpy.inf]) >>> x = x.reshape([1, 3, 1]) >>> ops.convert_to_numpy(GlobalArgminPooling1D()(x)) array([[0.]], dtype=float32)

### Goal
Performs a global argmin pooling operation across the temporal dimension of a 3D time-series tensor to extract the index of the minimum value for each feature.

### Parameters
_None._

### Input
- **Data Format:** A 3D tensor (NumPy array or backend-specific tensor) with shape `(batch_size, steps, features)`.
- **Preconditions:** Input must be a 3D tensor. If data is 1D or 2D, it must be converted to the standard `(n_ts, max_sz, d)` format using `tslearn.utils.to_time_series_dataset` prior to processing.

### Output
Returns a 2D tensor of shape `(batch_size, features)` containing the indices (as floats) where the minimum value occurs along the temporal axis for each feature.

### Valid Call Patterns
```python
import numpy
from tslearn.shapelets import GlobalArgminPooling1D
from tslearn.backend import ops

x = numpy.array([5.0, 6.8, 2.0]).reshape([1, 3, 1])
pooling_layer = GlobalArgminPooling1D()
result = pooling_layer(x)
print(ops.convert_to_numpy(result))
```

### LLM Instruction Prompt
- Use `GlobalArgminPooling1D` when you need to reduce a 3D time-series tensor to a 2D representation by identifying the temporal index of the minimum value.
- Always ensure the input is a 3D tensor of shape `(batch_size, steps, features)`.
- The output is a 2D tensor; use `tslearn.backend.ops.convert_to_numpy` if you need to inspect the result as a standard NumPy array.

### Prompt Snippet
```text
from tslearn.shapelets import GlobalArgminPooling1D
# Assuming X is a 3D tensor (batch_size, steps, features)
pooled_data = GlobalArgminPooling1D()(X)
```

### Common Failure Modes
- **Shape Mismatch:** Passing a 2D array instead of a 3D tensor will raise a shape error. Ensure input is reshaped to `(n_ts, sz, d)`.
- **Backend Incompatibility:** If using custom backend tensors, ensure the input tensor is compatible with the `tslearn` backend operations.
- **Data Type:** Passing non-numeric data will result in a runtime error during the argmin calculation.

### Fix Code Hint
```python
# Ensure 3D shape for GlobalArgminPooling1D
if X.ndim == 2:
    X = X.reshape((X.shape[0], X.shape[1], 1))
result = GlobalArgminPooling1D()(X)
```

## API Test: `GlobalMinPooling1D`

### Signature
```python
class GlobalMinPooling1D(Layer)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:66_

_Source doc:_ Global min pooling operation for temporal data. # Input shape 3D tensor with shape: `(batch_size, steps, features)`. # Output shape 2D tensor with shape: `(batch_size, features)`. Examples -------- >>> x = numpy.array([5.0, 6.8, numpy.inf]) >>> x = x.reshape([1, 3, 1]) >>> ops.convert_to_numpy(GlobalMinPooling1D()(x)) array([[5.]], dtype=float32)

### Goal
Performs a global minimum pooling operation across the temporal dimension of a 3D time-series tensor to extract the minimum value per feature for each sample.

### Parameters
_None._

### Input
The input must be a 3D tensor (NumPy array or backend-specific tensor) with shape `(batch_size, steps, features)`. If the data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` prior to processing.

### Output
Returns a 2D tensor of shape `(batch_size, features)` containing the minimum value found along the temporal axis for each feature in each time series.

### Valid Call Patterns
```python
import numpy as np
from tslearn.shapelets import GlobalMinPooling1D
from tslearn import backend as ops

# Example usage based on source documentation
x = np.array([5.0, 6.8, 10.0]).reshape([1, 3, 1])
pooling_layer = GlobalMinPooling1D()
result = pooling_layer(x)
print(ops.convert_to_numpy(result))
# Expected output: [[5.]]
```

### LLM Instruction Prompt
- Always ensure the input is a 3D tensor of shape `(batch_size, steps, features)`.
- Use `tslearn.backend.instantiate_backend` if working with specific backend tensors (e.g., PyTorch) to ensure compatibility.
- The layer is callable; instantiate it first, then pass the data as an argument.

### Prompt Snippet
```text
# Ensure input is 3D: (n_ts, sz, d)
pooling = GlobalMinPooling1D()
output = pooling(input_data)
```

### Common Failure Modes
- **Shape Mismatch:** Passing a 2D array `(n_ts, sz)` instead of a 3D array `(n_ts, sz, 1)` will raise a dimension error. Use `reshape` or `to_time_series_dataset` to fix.
- **Backend Incompatibility:** Passing a PyTorch tensor when the layer expects a NumPy array (or vice versa) without proper backend handling.
- **Empty Input:** Passing an empty array or a tensor with zero steps may result in undefined behavior or errors depending on the backend.

### Fix Code Hint
```python
# Ensure 3D shape for GlobalMinPooling1D
if X.ndim == 2:
    X = X.reshape((X.shape[0], X.shape[1], 1))
result = GlobalMinPooling1D()(X)
```

## API Test: `KNeighborsTimeSeries`

### Signature
```python
class KNeighborsTimeSeries(KNeighborsTimeSeriesMixin, NearestNeighbors, BaseModelPackage)
```
_Source: tslearn/tslearn/neighbors/neighbors.py:236_

_Source doc:_ Unsupervised learner for implementing neighbor searches for Time Series. Parameters ---------- n_neighbors : int (default: 5) Number of nearest neighbors to be considered for the decision. metric : {'dtw', 'softdtw', 'ctw', 'euclidean', 'sqeuclidean', \ 'cityblock',  'sax'} (default: 'dtw') Metric to be used at the core of the nearest neighbor procedure. DTW and SAX are described in more detail in :mod:`tslearn.metrics`. When SAX is provided as a metric, the data is expected to be normalized such that each time series has zero mean and unit variance. Other metrics are described in `scipy.spatial.distance doc <https://docs.scipy.org/doc/scipy/reference/spatial.distance.html>`_. metric_params : dict or None (default: None) Dictionary of metric parameters. For metrics that accept parallelization of the cross-distance matrix computations, `n_jobs` and `verbose` keys passed in `metric_params` are overridden by the `n_jobs` and `verbose` arguments. For 'sax' metric, these are hyper-parameters to be passed at the creation of the `SymbolicAggregateApproximation` object. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel for cross-distance matrix computations. Ignored if the cross-distance matrix cannot be computed using parallelization. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details. Notes ----- The training data are saved to disk if this model is serialized and may result in a large model file if the training dataset is large. Examples -------- >>> time_series = to_time_series_dataset([[1, 2, 3, 4], ...                                       [3, 3, 2, 0], ...                                       [1, 2, 2, 4]]) >>> knn = KNeighborsTimeSeries(n_neighbors=1).fit(time_series) >>> dataset = to_time_series_dataset([[1, 1, 2, 2, 2, 3, 4]]) >>> dist, ind = knn.kneighbors(dataset, return_distance=True) >>> dist array([[0.]]) >>> print(ind) [[0]] >>> knn2 = KNeighborsTimeSeries(n_neighbors=10, ...                             metric="euclidean").fit(time_series) >>> print(knn2.kneighbors(return_distance=False)) [[2 1] [2 0] [0 1]]

### Goal
`KNeighborsTimeSeries` provides an unsupervised scikit-learn compatible estimator to perform k-nearest neighbor searches on time-series datasets using specialized alignment-based or spatial distance metrics.

### Parameters
- **n_neighbors**: `int` (default: 5). Number of neighbors to return.
- **metric**: `str` (default: 'dtw'). Distance metric; options include 'dtw', 'softdtw', 'ctw', 'euclidean', 'sqeuclidean', 'cityblock', or 'sax'.
- **metric_params**: `dict` or `None` (default: None). Additional parameters for the chosen metric.
- **n_jobs**: `int` or `None` (default: None). Parallelization factor for distance matrix computation.

### Input
- **Data Format**: Input must be a 3D NumPy array of shape `(n_ts, max_sz, d)`.
- **Preconditions**: Use `tslearn.utils.to_time_series_dataset` to convert raw sequences or variable-length lists into the required 3D format before calling `.fit()`.
- **Normalization**: If `metric='sax'`, input data must be pre-normalized to have zero mean and unit variance.

### Output
- **kneighbors()**: Returns a tuple `(distances, indices)` if `return_distance=True`, or just `indices` if `False`. `distances` is a 2D array of shape `(n_query, n_neighbors)`, and `indices` is a 2D array of shape `(n_query, n_neighbors)` containing indices of the training samples.

### Valid Call Patterns
```python
import numpy as np
from tslearn.neighbors import KNeighborsTimeSeries
from tslearn.utils import to_time_series_dataset

# Deterministic check
X = np.random.randn(15, 10, 3)
model = KNeighborsTimeSeries(n_neighbors=3, metric='dtw')
model.fit(X)
dist, ind = model.kneighbors(X[:1], return_distance=True)
print(f"Witness: Found {len(ind[0])} neighbors for query.")
```

### LLM Instruction Prompt
- Always ensure input data is converted to 3D format via `to_time_series_dataset` before fitting.
- When using 'sax' as a metric, verify the data is scaled (e.g., using `TimeSeriesScalerMeanVariance`).
- Use `kneighbors` to retrieve indices and distances after the model is fitted.

### Prompt Snippet
```text
knn = KNeighborsTimeSeries(n_neighbors=K, metric='METRIC_NAME')
knn.fit(X_train)
indices = knn.kneighbors(X_query, return_distance=False)
```

### Common Failure Modes
- **Shape Mismatch**: Passing 2D arrays directly without `to_time_series_dataset` will raise a `ValueError`.
- **Invalid Metric**: Passing a metric string not supported by the class (e.g., 'invalid') will raise a `ValueError` during `fit`.
- **Unfitted Model**: Calling `kneighbors` before `fit` will raise a `NotFittedError`.

### Fix Code Hint
```python
# Ensure 3D shape (n_samples, n_timesteps, n_features)
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X_raw)
knn = KNeighborsTimeSeries(metric='dtw').fit(X_formatted)
```

## API Test: `KNeighborsTimeSeriesClassifier`

### Signature
```python
class KNeighborsTimeSeriesClassifier(KNeighborsTimeSeriesMixin, KNeighborsClassifier, BaseModelPackage)
```
_Source: tslearn/tslearn/neighbors/neighbors.py:431_

_Source doc:_ Classifier implementing the k-nearest neighbors vote for Time Series. Parameters ---------- n_neighbors : int (default: 5) Number of nearest neighbors to be considered for the decision. weights : str or callable, optional (default: 'uniform') Weight function used in prediction. Possible values: - 'uniform' : uniform weights. All points in each neighborhood are weighted equally. - 'distance' : weight points by the inverse of their distance. in this case, closer neighbors of a query point will have a greater influence than neighbors which are further away. - [callable] : a user-defined function which accepts an array of distances, and returns an array of the same shape containing the weights. metric : one of the metrics allowed for :class:`.KNeighborsTimeSeries`\ class (default: 'dtw') Metric to be used at the core of the nearest neighbor procedure metric_params : dict or None (default: None) Dictionnary of metric parameters. For metrics that accept parallelization of the cross-distance matrix computations, `n_jobs` and `verbose` keys passed in `metric_params` are overridden by the `n_jobs` and `verbose` arguments. For 'sax' metric, these are hyper-parameters to be passed at the creation of the `SymbolicAggregateApproximation` object. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel for cross-distance matrix computations. Ignored if the cross-distance matrix cannot be computed using parallelization. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details. verbose : int, optional (default=0) The verbosity level: if non zero, progress messages are printed. Above 50, the output is sent to stdout. The frequency of the messages increases with the verbosity level. If it more than 10, all iterations are reported. `Glossary <https://joblib.readthedocs.io/en/latest/parallel.html#parallel-reference-documentation>`__ for more details. Notes ----- The training data are saved to disk if this model is serialized and may result in a large model file if the training dataset is large. Examples -------- >>> clf = KNeighborsTimeSeriesClassifier(n_neighbors=2, metric="dtw") >>> clf.fit([[1, 2, 3], [1, 1.2, 3.2], [3, 2, 1]], ...         y=[0, 0, 1]).predict([[1, 2.2, 3.5]]) array([0]) >>> clf = KNeighborsTimeSeriesClassifier(n_neighbors=2, ...                                      metric="dtw",

### Goal
Performs time-series classification by assigning a class label to a query time series based on the majority vote of its $k$ nearest neighbors using specialized alignment-based distance metrics.

### Parameters
_None._

### Input
- **X**: 3D `numpy` array of shape `(n_ts, max_sz, d)`. If input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` before fitting.
- **y**: Array-like of shape `(n_ts,)` containing the target class labels.
- **Preconditions**: The estimator follows the `scikit-learn` `fit`/`predict` pattern. Data should be preprocessed (e.g., scaled) if necessary.

### Output
- **Returns**: An array of shape `(n_query_ts,)` containing the predicted class labels for the input time series.

### Valid Call Patterns
```python
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
clf = KNeighborsTimeSeriesClassifier(n_neighbors=2, metric="dtw")
clf.fit(X_train, y_train)
predictions = clf.predict(X_test)
```

### LLM Instruction Prompt
- Always ensure input data `X` is a 3D array `(n_ts, sz, d)`. Use `tslearn.utils.to_time_series_dataset` if the input is a list of sequences or 2D array.
- When using `metric="dtw"` or other alignment metrics, ensure `metric_params` is provided as a dictionary if specific constraints (e.g., `sakoe_chiba_radius`) are required.
- The estimator is compatible with `scikit-learn` pipelines.

### Prompt Snippet
```text
# Example usage:
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
knn = KNeighborsTimeSeriesClassifier(n_neighbors=3, metric="dtw")
knn.fit(X, y)
y_pred = knn.predict(X_new)
```

### Common Failure Modes
- **Shape Mismatch**: Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError`.
- **Memory Usage**: Large training datasets are stored in memory and serialized; avoid using this estimator for extremely large datasets without dimensionality reduction or shapelet-based approaches.
- **Metric Parameters**: Passing invalid keys in `metric_params` for specific metrics (e.g., `sax`) will cause runtime errors during distance computation.

### Fix Code Hint
```python
import numpy as np
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
from tslearn.utils import to_time_series_dataset

# Ensure 3D shape
X = to_time_series_dataset([[1, 2, 3], [4, 5, 6]])
y = np.array([0, 1])

clf = KNeighborsTimeSeriesClassifier(n_neighbors=1, metric="dtw")
clf.fit(X, y)
print(f"Correctness witness: {clf.predict(X)}")
```

## API Test: `KNeighborsTimeSeriesMixin`

### Signature
```python
class KNeighborsTimeSeriesMixin(TimeSeriesMixin)
```
_Source: tslearn/tslearn/neighbors/neighbors.py:28_

_Source doc:_ Mixin for k-neighbors searches on Time Series.

### Goal
Provides a standardized interface for k-nearest neighbors search functionality within time-series estimators by inheriting from `TimeSeriesMixin`.

### Parameters
_None._

### Input
This class is a mixin intended for inheritance by estimator classes. It expects the inheriting class to implement methods compatible with `scikit-learn`'s neighbor search patterns, typically operating on 3D NumPy arrays of shape `(n_ts, max_sz, d)` or variable-length time series datasets prepared via `tslearn.utils.to_time_series_dataset`.

### Output
Returns `unspecified` — As a mixin class, it does not return a value upon instantiation; it provides methods (such as `kneighbors`) to the inheriting estimator.

### Valid Call Patterns
```python
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
# KNeighborsTimeSeriesClassifier inherits from KNeighborsTimeSeriesMixin
knn = KNeighborsTimeSeriesClassifier(n_neighbors=3)
# The mixin provides the interface for the following:
# knn.kneighbors(X, n_neighbors=None, return_distance=True)
```

### LLM Instruction Prompt
- When generating code involving `KNeighborsTimeSeriesMixin`, do not instantiate it directly. 
- Use it as a base class for custom estimators or assume its presence when working with `tslearn.neighbors` classes like `KNeighborsTimeSeriesClassifier` or `KNeighborsTimeSeriesRegressor`.
- Ensure input data is formatted as a 3D array `(n_ts, max_sz, d)` before calling methods provided by this mixin.

### Prompt Snippet
```text
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
# KNeighborsTimeSeriesMixin is inherited by:
model = KNeighborsTimeSeriesClassifier(n_neighbors=5)
model.fit(X_train, y_train)
distances, indices = model.kneighbors(X_test)
```

### Common Failure Modes
- **Direct Instantiation:** Attempting to instantiate `KNeighborsTimeSeriesMixin()` directly will not provide functional neighbor search capabilities as it lacks the required `fit` and `kneighbors` implementation details provided by concrete subclasses.
- **Data Shape Mismatch:** Passing 2D arrays directly without using `tslearn.utils.to_time_series_dataset` will cause shape errors in the underlying distance computations.
- **Missing Dependencies:** Relying on this mixin for custom estimators without ensuring the `scikit-learn` estimator contract is fully satisfied.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
from tslearn.neighbors import KNeighborsTimeSeriesClassifier

# Ensure data is 3D
X = to_time_series_dataset([[1, 2, 3], [1, 2, 3, 4]])
y = [0, 1]

# Use the concrete class that implements the mixin
clf = KNeighborsTimeSeriesClassifier(n_neighbors=1)
clf.fit(X, y)
```

## API Test: `KNeighborsTimeSeriesRegressor`

### Signature
```python
class KNeighborsTimeSeriesRegressor(KNeighborsTimeSeriesMixin, KNeighborsRegressor, BaseModelPackage)
```
_Source: tslearn/tslearn/neighbors/neighbors.py:614_

_Source doc:_ Regressor implementing the k-nearest neighbors vote for Time Series. Parameters ---------- n_neighbors : int (default: 5) Number of nearest neighbors to be considered for the decision. weights : str or callable, optional (default: 'uniform') Weight function used in prediction. Possible values: - 'uniform' : uniform weights. All points in each neighborhood are weighted equally. - 'distance' : weight points by the inverse of their distance. in this case, closer neighbors of a query point will have a greater influence than neighbors which are further away. - [callable] : a user-defined function which accepts an array of distances, and returns an array of the same shape containing the weights. metric : one of the metrics allowed for :class:`.KNeighborsTimeSeries`\ class (default: 'dtw') Metric to be used at the core of the nearest neighbor procedure metric_params : dict or None (default: None) Dictionnary of metric parameters. For metrics that accept parallelization of the cross-distance matrix computations, `n_jobs` and `verbose` keys passed in `metric_params` are overridden by the `n_jobs` and `verbose` arguments. For 'sax' metric, these are hyper-parameters to be passed at the creation of the `SymbolicAggregateApproximation` object. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel for cross-distance matrix computations. Ignored if the cross-distance matrix cannot be computed using parallelization. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details. verbose : int, optional (default=0) The verbosity level: if non zero, progress messages are printed. Above 50, the output is sent to stdout. The frequency of the messages increases with the verbosity level. If it more than 10, all iterations are reported. `Glossary <https://joblib.readthedocs.io/en/latest/parallel.html#parallel-reference-documentation>`__ for more details. Examples -------- >>> clf = KNeighborsTimeSeriesRegressor(n_neighbors=2, metric="dtw") >>> clf.fit([[1, 2, 3], [1, 1.2, 3.2], [3, 2, 1]], ...         y=[0.1, 0.1, 1.1]).predict([[1, 2.2, 3.5]]) array([0.1]) >>> clf = KNeighborsTimeSeriesRegressor(n_neighbors=2, ...                                     metric="dtw", ...                                     n_jobs=2) >>> clf.fit([[1, 2, 3], [1, 1.2, 3.2], [3, 2, 1]], ...         y=[0.1, 0.1, 1.1]).predict([[1, 2.2, 3.5]]) array([0.1]) >>> clf = KNeighborsTimeSeriesRegressor(n_neighbors=2, ...                                     metric="dtw",

### Goal
Performs time-series regression by predicting target values based on the average (weighted or uniform) of the `k` nearest neighbors in the training set, using time-series specific distance metrics like DTW.

### Parameters
*   **n_neighbors**: `int` (default: 5). Number of nearest neighbors to consider.
*   **weights**: `str` or `callable` (default: 'uniform'). Weighting strategy for neighbors ('uniform', 'distance', or custom function).
*   **metric**: `str` (default: 'dtw'). Distance metric used for neighbor search.
*   **metric_params**: `dict` or `None` (default: None). Additional parameters for the chosen metric.
*   **n_jobs**: `int` or `None` (default: None). Parallelization factor for distance matrix computation.
*   **verbose**: `int` (default: 0). Verbosity level for progress reporting.

### Input
*   **X**: 3D `numpy` array of shape `(n_ts, max_sz, d)`. If input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to fitting.
*   **y**: Array-like of target values for regression.

### Output
Returns an array of predicted target values corresponding to the input query time series.

### Valid Call Patterns
```python
from tslearn.neighbors import KNeighborsTimeSeriesRegressor
clf = KNeighborsTimeSeriesRegressor(n_neighbors=2, metric="dtw")
clf.fit([[1, 2, 3], [1, 1.2, 3.2], [3, 2, 1]], y=[0.1, 0.1, 1.1])
preds = clf.predict([[1, 2.2, 3.5]])
print(f"Witness: {preds}")
```

### LLM Instruction Prompt
- Always ensure input data `X` is a 3D array. Use `tslearn.utils.to_time_series_dataset` if the input is a list of sequences or a 2D array.
- The `metric` parameter must be a valid string supported by `tslearn` distance metrics (e.g., 'dtw', 'softdtw').
- The estimator follows the standard scikit-learn `fit(X, y).predict(X)` pattern.

### Prompt Snippet
```text
from tslearn.neighbors import KNeighborsTimeSeriesRegressor
# Ensure X is 3D: (n_samples, n_timesteps, n_features)
model = KNeighborsTimeSeriesRegressor(n_neighbors=k, metric=metric_name)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

### Common Failure Modes
- **Shape Mismatch**: Passing 2D arrays directly without using `to_time_series_dataset` will cause a dimension error.
- **Metric Errors**: Providing an unsupported metric string will raise a `ValueError` during the `fit` or `predict` phase.
- **Memory Exhaustion**: Using large datasets with computationally expensive metrics (like DTW) without setting `n_jobs` can lead to slow performance or memory issues.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset
from tslearn.neighbors import KNeighborsTimeSeriesRegressor

# Ensure data is 3D
X = to_time_series_dataset(np.random.rand(10, 5)) 
y = np.random.rand(10)
clf = KNeighborsTimeSeriesRegressor(n_neighbors=3, metric="dtw")
clf.fit(X, y)
```

## API Test: `KShape`

### Signature
```python
class KShape(TimeSeriesCentroidBasedClusteringMixin, ClusterMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/clustering/kshape.py:19_

_Source doc:_ KShape clustering for time series. KShape was originally presented in [1]_. Parameters ---------- n_clusters : int (default: 3) Number of clusters to form. max_iter : int (default: 100) Maximum number of iterations of the k-Shape algorithm. tol : float (default: 1e-6) Inertia variation threshold. If at some point, inertia varies less than this threshold between two consecutive iterations, the model is considered to have converged and the algorithm stops. n_init : int (default: 1) Number of time the k-Shape algorithm will be run with different centroid seeds. The final results will be the best output of n_init consecutive runs in terms of inertia. Ignored if initialization is not random. verbose : bool (default: False) Whether or not to print information about the inertia while learning the model. random_state : integer or numpy.RandomState, optional Generator used to initialize the centers. If an integer is given, it fixes the seed. Defaults to the global numpy random number generator. init : {'random' or ndarray} (default: 'random') Method for initialization. 'random': choose k observations (rows) at random from data for the initial centroids. If an ndarray is passed, it should be of shape (n_clusters, ts_size, d) and gives the initial centers. Attributes ---------- cluster_centers_ : numpy.ndarray of shape (sz, d). Centroids labels_ : numpy.ndarray of integers with shape (n_ts, ). Labels of each point inertia_ : float Sum of distances of samples to their closest cluster center. n_iter_ : int The number of iterations performed during fit. Notes ----- This method requires a dataset of equal-sized time series. Examples --------

### Goal
`KShape` performs partitional clustering of time series based on the shape-based distance (SBD) metric, which is invariant to shifts and scaling.

### Parameters
*   **n_clusters** (int, default=3): Number of clusters to form.
*   **max_iter** (int, default=100): Maximum number of iterations.
*   **tol** (float, default=1e-6): Inertia variation threshold for convergence.
*   **n_init** (int, default=1): Number of random initializations.
*   **verbose** (bool, default=False): Whether to print progress.
*   **random_state** (int or numpy.RandomState, optional): Seed for reproducibility.
*   **init** ({'random', ndarray}, default='random'): Initialization method or custom centroids.

### Input
*   **Data Format:** 3D `numpy` array of shape `(n_ts, sz, d)`.
*   **Preconditions:** Input time series must be of equal length (`sz`). Data should be preprocessed using `TimeSeriesScalerMeanVariance` as `KShape` relies on shape-based distance which assumes normalized series.

### Output
*   **Attributes:** `cluster_centers_` (centroids), `labels_` (cluster assignments), `inertia_` (sum of squared distances), and `n_iter_` (iterations performed).

### Valid Call Patterns
```python
from tslearn.clustering import KShape
from tslearn.preprocessing import TimeSeriesScalerMeanVariance

# Data must be scaled for KShape
X_scaled = TimeSeriesScalerMeanVariance().fit_transform(X)
ks = KShape(n_clusters=3, n_init=1, random_state=0).fit(X_scaled)
labels = ks.predict(X_scaled)
```

### LLM Instruction Prompt
- Always ensure input data is 3D and has been processed via `TimeSeriesScalerMeanVariance` before calling `fit`.
- `KShape` requires equal-length time series; do not pass variable-length datasets without prior resampling or padding.
- Use `random_state` for deterministic results in research workflows.

### Prompt Snippet
```text
from tslearn.clustering import KShape
ks = KShape(n_clusters=k, random_state=seed)
ks.fit(X_scaled)
```

### Common Failure Modes
- **ValueError:** Raised if `n_clusters` is greater than the number of samples.
- **ValueError:** Raised if `init` is an invalid string or an `ndarray` with incorrect dimensions.
- **Inaccurate Clustering:** Occurs if the input data is not scaled using `TimeSeriesScalerMeanVariance`, as SBD is sensitive to unnormalized series.

### Fix Code Hint
```python
# Ensure data is scaled to mean 0 and variance 1
from tslearn.preprocessing import TimeSeriesScalerMeanVariance
X_scaled = TimeSeriesScalerMeanVariance().fit_transform(X)
# Then fit the model
ks = KShape(n_clusters=3).fit(X_scaled)
```

## API Test: `KernelKMeans`

### Signature
```python
class KernelKMeans(TimeSeriesMixin, ClusterMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/clustering/kmeans.py:129_

_Source doc:_ Kernel K-means. Parameters ---------- n_clusters : int (default: 3) Number of clusters to form. kernel : string, or callable (default: "gak") The kernel should either be "gak", in which case the Global Alignment Kernel from [2]_ is used or a value that is accepted as a metric by `scikit-learn's pairwise_kernels <https://scikit-learn.org/stable/modules/generated/\ sklearn.metrics.pairwise.pairwise_kernels.html>`_ max_iter : int (default: 50) Maximum number of iterations of the k-means algorithm for a single run. tol : float (default: 1e-6) Inertia variation threshold. If at some point, inertia varies less than this threshold between two consecutive iterations, the model is considered to have converged and the algorithm stops. n_init : int (default: 1) Number of time the k-means algorithm will be run with different centroid seeds. The final results will be the best output of n_init consecutive runs in terms of inertia. kernel_params : dict or None (default: None) Kernel parameters to be passed to the kernel function. None means no kernel parameter is set. For Global Alignment Kernel, the only parameter of interest is `sigma`. If set to 'auto', it is computed based on a sampling of the training set (cf :ref:`tslearn.metrics.sigma_gak <fun-tslearn.metrics.sigma_gak>`). If no specific value is set for `sigma`, its defaults to 1. A `RuntimeError` is raised at fit time when computed or explicit value is close to 0 and therefore not compatible with 'gak' kernel. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel for GAK cross-similarity matrix computations. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`_ for more details. verbose : int (default: 0) If nonzero, joblib progress messages are printed. random_state : integer or numpy.RandomState, optional Generator used to initialize the centers. If an integer is given, it fixes the seed. Defaults to the global numpy random number generator. Attributes ---------- labels_ : numpy.ndarray Labels of each point

### Goal
Performs kernel-based clustering on time-series data, supporting the Global Alignment Kernel (GAK) to handle non-linear temporal alignments.

### Parameters
*   **n_clusters**: `int` (default: 3) Number of clusters to form.
*   **kernel**: `str` or `callable` (default: "gak") The kernel function to use.
*   **max_iter**: `int` (default: 50) Maximum iterations for the k-means algorithm.
*   **tol**: `float` (default: 1e-6) Inertia variation threshold for convergence.
*   **n_init**: `int` (default: 1) Number of runs with different centroid seeds.
*   **kernel_params**: `dict` or `None` (default: None) Parameters for the kernel (e.g., `{"sigma": "auto"}`).
*   **n_jobs**: `int` or `None` (default: None) Parallelization factor for GAK computations.
*   **verbose**: `int` (default: 0) Verbosity level.
*   **random_state**: `int` or `numpy.RandomState` (optional) Seed for reproducibility.

### Input
*   **Data**: 3D `numpy.ndarray` of shape `(n_ts, max_sz, d)`.
*   **Preconditions**: If input data is 1D or 2D, it must be converted to a 3D time-series dataset using `tslearn.utils.to_time_series_dataset` before fitting.

### Output
*   **labels_**: `numpy.ndarray` containing the cluster assignment for each input time series.

### Valid Call Patterns
```python
from tslearn.clustering import KernelKMeans
import numpy as np

n, sz, d = 15, 10, 3
X = np.random.randn(n, sz, d)

# Standard usage with GAK
gak_km = KernelKMeans(n_clusters=3, max_iter=5, kernel_params={"sigma": "auto"}, random_state=0)
gak_km.fit(X)
print(gak_km.labels_)
```

### LLM Instruction Prompt
- Always ensure input data is a 3D NumPy array.
- When using the "gak" kernel, `kernel_params` should typically include `{"sigma": "auto"}` to ensure valid kernel scaling.
- Use `fit(X)` to train the model and `predict(X)` to assign new data to existing clusters.

### Prompt Snippet
```text
from tslearn.clustering import KernelKMeans
model = KernelKMeans(n_clusters=K, kernel="gak", kernel_params={"sigma": "auto"})
model.fit(X)
```

### Common Failure Modes
- **RuntimeError**: Occurs if `sigma` is set to 0 or computed as 0 when using the "gak" kernel.
- **Shape Mismatch**: Passing 2D arrays directly without using `to_time_series_dataset` will cause dimension errors.
- **Convergence**: If `max_iter` is too low, the model may not reach the `tol` threshold, resulting in suboptimal clustering.

### Fix Code Hint
```python
# Ensure data is 3D
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
# Check sigma if RuntimeError occurs
model = KernelKMeans(kernel_params={"sigma": 1.0}) 
```

## API Test: `LearningShapelets`

### Signature
```python
class LearningShapelets(TimeSeriesMixin, ClassifierMixin, TransformerMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:290_

_Source doc:_ Learning Time-Series Shapelets model. Learning Time-Series Shapelets was originally presented in [1]_. From an input (possibly multidimensional) time series :math:`x` and a set of shapelets :math:`\{s_i\}_i`, the :math:`i`-th coordinate of the Shapelet transform is computed as: .. math:: ST(x, s_i) = \min_t \sum_{\delta_t} \left\|x(t+\delta_t) - s_i(\delta_t)\right\|_2^2 The Shapelet model consists in a logistic regression layer on top of this transform. Shapelet coefficients as well as logistic regression weights are optimized by gradient descent on a L2-penalized cross-entropy loss. Parameters ---------- n_shapelets_per_size: dict (default: None) Dictionary giving, for each shapelet size (key), the number of such shapelets to be trained (value). If None, :func:`grabocka_params_to_shapelet_size_dict` is used and the size used to compute is that of the shortest time series passed at fit time. max_iter: int (default: 10,000) Number of training epochs. .. versionchanged:: 0.3 default value for max_iter is set to 10,000 instead of 100 batch_size: int (default: 256) Batch size to be used. verbose: {0, 1, 2} (default: 0) `keras` verbose level. optimizer: str or keras.optimizers.Optimizer (default: "sgd") `keras` optimizer to use for training. weight_regularizer: float (default: 0.) Strength of the L2 regularizer to use for training the classification (softmax) layer. If 0, no regularization is performed. shapelet_length: float (default: 0.15) The length of the shapelets, expressed as a fraction of the time series length. Used only if `n_shapelets_per_size` is None. total_lengths: int (default: 3) The number of different shapelet lengths. Will extract shapelets of length i * shapelet_length for i in [1, total_lengths] Used only if `n_shapelets_per_size` is None. max_size: int or None (default: None) Maximum size for time series to be fed to the model. If None, it is set to the size (number of timestamps) of the training time series. scale: bool (default: False)

### Goal
`LearningShapelets` performs supervised time-series classification by simultaneously learning a set of discriminative sub-sequences (shapelets) and a logistic regression classifier that operates on the distances between these shapelets and the input time series.

### Parameters
_None._

### Input
- **Data:** 3D `numpy` array of shape `(n_ts, max_sz, d)`.
- **Preconditions:** Input data must be formatted as a 3D array; use `tslearn.utils.to_time_series_dataset` if the input is a list of variable-length sequences or 1D/2D arrays.
- **Labels:** A 1D array of target labels (integers or strings).

### Output
Returns a fitted `LearningShapelets` estimator instance. The model exposes `predict()` and `predict_proba()` methods for classification, and `shapelets_` / `shapelets_as_time_series_` attributes containing the learned features.

### Valid Call Patterns
```python
from tslearn.shapelets import LearningShapelets
from keras.optimizers import Adam

# Basic usage
clf = LearningShapelets(n_shapelets_per_size={2: 5}, max_iter=100)
clf.fit(X, y)
preds = clf.predict(X)

# Advanced usage with custom optimizer
clf = LearningShapelets(n_shapelets_per_size={3: 2, 4: 1}, 
                        optimizer=Adam(learning_rate=0.01),
                        max_iter=500)
clf.fit(X, y)
```

### LLM Instruction Prompt
- Always import `LearningShapelets` from `tslearn.shapelets`.
- Ensure input data is a 3D NumPy array.
- When using custom optimizers, import them from `keras.optimizers`.
- Note that `max_iter` defaults to 10,000; for testing or quick prototyping, set a lower value.
- The model is not serializable via HDF5; use standard pickle or JSON-based serialization for model persistence.

### Prompt Snippet
```text
from tslearn.shapelets import LearningShapelets
clf = LearningShapelets(n_shapelets_per_size={...}, max_iter=...)
clf.fit(X, y)
```

### Common Failure Modes
- **Input Shape:** Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError`.
- **Serialization:** Attempting to save the model using HDF5 will raise a `NotImplementedError`.
- **Convergence:** If `max_iter` is too low, the model may not converge, leading to poor classification performance.
- **Dependency:** Requires `keras` to be installed as it relies on the Keras backend for gradient descent optimization.

### Fix Code Hint
```python
import numpy as np
from tslearn.shapelets import LearningShapelets
from tslearn.utils import to_time_series_dataset

# Ensure data is 3D
X = to_time_series_dataset(raw_data) 
clf = LearningShapelets(n_shapelets_per_size={5: 10}, max_iter=100)
clf.fit(X, y)
print(f"Correctness witness: {clf.shapelets_.shape}")
```

## API Test: `LocalSquaredDistanceLayer`

### Signature
```python
class LocalSquaredDistanceLayer(Layer)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:167_

_Source doc:_ Pairwise (squared) distance computation between local patches and shapelets # Input shape 4D tensor with shape: `(batch_size, steps - shapelet_size, shapelet_length, features)`. # Output shape 3D tensor with shape: `(batch_size, steps, n_shapelets)`.

### Goal
Computes the pairwise squared Euclidean distances between local time-series patches and learned shapelets within a neural network architecture.

### Parameters
_None._

### Input
The layer expects a 4D tensor input with shape `(batch_size, steps - shapelet_size, shapelet_length, features)`. This input represents extracted local patches from the time-series data. The layer is designed for use within a `PyTorch` or `Keras` model context as part of a shapelet-based learning pipeline.

### Output
Returns a 3D tensor with shape `(batch_size, steps, n_shapelets)`, representing the squared distance between each patch and each shapelet.

### Valid Call Patterns
```python
# Inferred from signature: usage within a Keras/TensorFlow model definition
from tslearn.shapelets import LocalSquaredDistanceLayer
layer = LocalSquaredDistanceLayer()
# output = layer(input_tensor)
```

### LLM Instruction Prompt
- Use `LocalSquaredDistanceLayer` only when constructing custom neural network architectures for shapelet discovery.
- Ensure the input tensor is 4D and matches the expected dimensions `(batch_size, steps - shapelet_size, shapelet_length, features)`.
- This layer is intended for deep learning frameworks; verify that the environment supports the required backend (e.g., TensorFlow/Keras).

### Prompt Snippet
```text
# Example of integrating the layer in a model
model.add(LocalSquaredDistanceLayer())
```

### Common Failure Modes
- **Shape Mismatch:** Providing a 3D tensor instead of the required 4D tensor will trigger a dimension error.
- **Framework Incompatibility:** Attempting to use this layer outside of a compatible deep learning framework (e.g., Keras) will result in an `ImportError` or `TypeError`.
- **Incorrect Input Dimensions:** If the input `steps` or `shapelet_size` do not align with the expected 4D structure, the layer will fail to compute the pairwise distances.

### Fix Code Hint
```python
# Ensure input is reshaped to 4D before passing to the layer
# input_tensor: (batch_size, n_patches, patch_length, n_features)
output = LocalSquaredDistanceLayer()(input_tensor)
```

## API Test: `MatrixProfile`

### Signature
```python
class MatrixProfile(TimeSeriesMixin, TransformerMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/matrix_profile/matrix_profile.py:69_

_Source doc:_ Matrix Profile transformation. Matrix Profile was originally presented in [1]_. Parameters ---------- subsequence_length : int (default: 1) Length of the subseries (also called window size) to be used for subseries distance computations. implementation : str (default: "numpy") Matrix profile implementation to use. Defaults to "numpy" to use the pure numpy version. All the available implementations are ["numpy", "stump", "gpu_stump"]. "stump" and "gpu_stump" are both implementations from the stumpy python library, the latter requiring a GPU. Stumpy is a library for efficiently computing the matrix profile which is optimized for speed, performance and memory. See [2]_ for the documentation. "numpy" is the default pure numpy implementation and does not require stumpy to be installed. scale: bool (default: True) Whether input data should be scaled for each feature of each time series to have zero mean and unit variance. Default for this parameter is set to `True` to match the standard matrix profile setup. Examples -------- >>> time_series = [0., 1., 3., 2., 9., 1., 14., 15., 1., 2., 2., 10., 7.] >>> ds = [time_series] >>> mp = MatrixProfile(subsequence_length=4, scale=False) >>> mp.fit_transform(ds)[0, :, 0]  # doctest: +ELLIPSIS array([ 6.85...,  1.41...,  6.16...,  7.93..., 11.40..., 13.56..., 18.  ..., 13.96...,  1.41...,  6.16...]) References ---------- .. [1] C. M. Yeh, Y. Zhu, L. Ulanova, N.Begum et al. Matrix Profile I: All Pairs Similarity Joins for Time Series: A Unifying View that Includes Motifs, Discords and Shapelets. ICDM 2016. .. [2] STUMPY documentation https://stumpy.readthedocs.io/en/latest/

### Goal
Computes the Matrix Profile of a time series, a distance-based representation used to identify motifs, discords, and other structural patterns by calculating the distance to the nearest neighbor for every subsequence.

### Parameters
- `subsequence_length` (int, default: 1): The length of the sliding window used for subseries distance computations.
- `implementation` (str, default: "numpy"): The backend engine to use; options are "numpy", "stump", or "gpu_stump".
- `scale` (bool, default: True): Whether to normalize each feature of each time series to zero mean and unit variance before computation.

### Input
- **Format:** 3D `numpy` array of shape `(n_ts, max_sz, d)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted to the standard 3D `tslearn` format using `tslearn.utils.to_time_series_dataset` before fitting.

### Output
- Returns a 3D `numpy` array representing the Matrix Profile, where the values correspond to the distance to the nearest neighbor for each subsequence.

### Valid Call Patterns
```python
from tslearn.matrix_profile import MatrixProfile
import numpy as np

# Standard usage
mp = MatrixProfile(subsequence_length=10)
X = np.random.randn(1, 20, 1)
profile = mp.fit_transform(X)

# Using stumpy backend
mp_stumpy = MatrixProfile(subsequence_length=10, implementation="stump")
profile_stumpy = mp_stumpy.fit_transform(X)
```

### LLM Instruction Prompt
- Always ensure the input data is a 3D array.
- If using `implementation="stump"` or `"gpu_stump"`, verify that the `stumpy` library is installed in the environment.
- Set `scale=False` if the input data is already pre-normalized or if the specific analysis requires raw distance values.

### Prompt Snippet
```text
from tslearn.matrix_profile import MatrixProfile
mp = MatrixProfile(subsequence_length=m, implementation="numpy", scale=True)
X_transformed = mp.fit_transform(X_dataset)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 1D or 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError`.
- **Missing Dependency:** Attempting to use `implementation="stump"` without the `stumpy` package installed will result in an `ImportError`.
- **Invalid Window Size:** Setting `subsequence_length` larger than the time series length will cause computation errors.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset
from tslearn.matrix_profile import MatrixProfile

# Ensure 3D shape
X = to_time_series_dataset([my_series])
mp = MatrixProfile(subsequence_length=5)
# Witness: Verify output shape
print(f"Output shape: {mp.fit_transform(X).shape}")
```

## API Test: `NonMyopicEarlyClassifier`

### Signature
```python
class NonMyopicEarlyClassifier(TimeSeriesMixin, ClassifierMixin, BaseEstimator)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:18_

_Source doc:_ Early Classification modelling for time series using the model presented in [1]_. Parameters ---------- n_clusters : int Number of clusters to form. base_classifier : Estimator or None Estimator (instance) to be cloned and used for classifications. If None, the chosen classifier is a 1NN with Euclidean metric. min_t : int Earliest time at which a classification can be performed on a time series lamb : float Value of the hyper parameter lambda used during the computation of the cost function to evaluate the probability that a time series belongs to a cluster given the time series. cost_time_parameter : float Parameter of the cost function of time. This function is of the form : f(time) = time * cost_time_parameter random_state: int Random state of the base estimator Attributes ---------- classifiers_ : list A list containing all the classifiers trained for the model, that is, (maximum_time_stamp - min_t) elements. pyhatyck_ : array like of shape (maximum_time_stamp - min_t, n_cluster, __n_classes, __n_classes) Contains the probabilities of being classified as class y_hat given class y and cluster ck for a trained classifier. The penultimate dimension of the array is associated to the true class of the series and the last dimension to the predicted class. pyck_ : array like of shape (__n_classes, n_cluster) Contains the probabilities of being of true class y given a cluster ck X_fit_dims : tuple of the same shape as the training dataset Examples -------- >>> dataset = to_time_series_dataset([[1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [3, 2, 1, 1, 2, 3], ...                                   [3, 2, 1, 1, 2, 3]]) >>> y = [0, 0, 0, 1, 1, 1, 0, 0] >>> model = NonMyopicEarlyClassifier(n_clusters=3, lamb=1000.,

### Goal
`NonMyopicEarlyClassifier` performs early classification of time series by balancing classification accuracy and the cost of waiting for more data points, using a cluster-based probabilistic model.

### Parameters
*   **n_clusters** (int): Number of clusters to form.
*   **base_classifier** (Estimator or None): Estimator to be cloned and used for classifications; defaults to 1NN with Euclidean metric.
*   **min_t** (int): Earliest time index at which a classification can be performed.
*   **lamb** (float): Hyperparameter lambda for the cost function evaluating cluster membership probability.
*   **cost_time_parameter** (float): Parameter for the time-based cost function `f(time) = time * cost_time_parameter`.
*   **random_state** (int): Random state for the base estimator.

### Input
*   **Data Format:** Input must be a 3D NumPy array of shape `(n_ts, max_sz, d)`. Use `tslearn.utils.to_time_series_dataset` to convert sequences before fitting.
*   **Preconditions:** The dataset must contain enough time steps to satisfy `min_t`.

### Output
*   Returns an initialized `NonMyopicEarlyClassifier` estimator instance. After `fit`, `early_predict` returns a tuple of `(predictions, delays)`.

### Valid Call Patterns
```python
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
from tslearn.utils import to_time_series_dataset

model = NonMyopicEarlyClassifier(
    n_clusters=3,
    base_classifier=KNeighborsTimeSeriesClassifier(n_neighbors=1, metric="euclidean"),
    min_t=2,
    lamb=1000.0,
    cost_time_parameter=0.1,
    random_state=0
)
model.fit(dataset, y)
```

### LLM Instruction Prompt
- Always use `to_time_series_dataset` to format input data before passing it to `fit` or `early_predict`.
- Ensure `min_t` is less than the total number of time steps in the input data.
- Use `early_predict` to obtain both class predictions and the time delay (number of steps) for each series.

### Prompt Snippet
```text
model = NonMyopicEarlyClassifier(n_clusters=..., min_t=..., lamb=...)
model.fit(X, y)
pred, delays = model.early_predict(X_test)
```

### Common Failure Modes
- Passing 2D arrays directly instead of 3D arrays (use `to_time_series_dataset`).
- Setting `min_t` greater than the length of the input time series, resulting in `NaN` predictions.
- Providing an incompatible `base_classifier` that does not support `fit` or `predict` methods.

### Fix Code Hint
```python
import numpy as np
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.utils import to_time_series_dataset

# Ensure data is 3D
X = to_time_series_dataset([[1, 2, 3], [3, 2, 1]])
y = [0, 1]
clf = NonMyopicEarlyClassifier(n_clusters=2, min_t=1)
clf.fit(X, y)
print("Correctness witness: Model fitted successfully.")
```

## API Test: `NumPyBackend`

### Signature
```python
class NumPyBackend(object)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:22_

_Source doc:_ Class for the Numpy  backend.

### Goal
Provides a NumPy-based computation engine for time-series metrics and operations, ensuring compatibility with standard NumPy arrays within the `tslearn` ecosystem.

### Parameters
_None._

### Input
The backend is designed to operate on standard NumPy arrays. Input data should be formatted as 3D arrays of shape `(n_ts, max_sz, d)` or compatible structures. Precondition: Ensure data is converted to NumPy format using `tslearn.utils.to_time_series_dataset` if working with variable-length sequences or non-array inputs.

### Output
Returns an instance of `NumPyBackend`. This object acts as a provider for mathematical operations (e.g., distance computations) that are executed using NumPy's optimized routines.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import NumPyBackend

# Instantiate the backend directly
be = NumPyBackend()

# Verify the backend identity
print(f"Backend initialized: {be.backend_string}")
assert be.backend_string == "numpy"
```

### LLM Instruction Prompt
- Use `NumPyBackend` when performing time-series distance or alignment computations where automatic differentiation (PyTorch) is not required.
- Always prefer `tslearn.backend.instantiate_backend("numpy")` over direct instantiation if you want to follow the project's standard factory pattern.
- Ensure all input arrays are NumPy-compatible; do not pass PyTorch tensors to this backend.

### Prompt Snippet
```text
from tslearn.backend.numpy_backend import NumPyBackend
be = NumPyBackend()
# Use 'be' as the 'be' parameter in metrics like dtw(s1, s2, be=be)
```

### Common Failure Modes
- **Type Mismatch:** Passing PyTorch tensors to a `NumPyBackend` instance will result in `TypeError` or `AttributeError` during computation, as NumPy does not support tensor-specific operations.
- **Direct Instantiation vs. Factory:** While `NumPyBackend()` works, it bypasses the `instantiate_backend` factory, which is the recommended way to handle backend selection dynamically based on input types.
- **Missing Dependencies:** While `numpy` is a core dependency, ensure the environment is not expecting `torch` features (like `.backward()`) when using this backend, as they are not implemented.

### Fix Code Hint
```python
import numpy as np
from tslearn.backend import instantiate_backend

# Correct way to obtain the backend for NumPy arrays
be = instantiate_backend("numpy")

# Example usage in a metric
from tslearn.metrics import dtw
dist = dtw(np.array([[1, 2]]), np.array([[2, 3]]), be=be)
print(f"Distance: {dist}")
```

## API Test: `NumPyLinalg`

### Signature
```python
class NumPyLinalg
```
_Source: tslearn/tslearn/backend/numpy_backend.py:126_

### Goal
Provides a backend-specific implementation of linear algebra operations for the `NumPy` computation engine within `tslearn`.

### Parameters
_None._

### Input
This class is intended for internal use by the `tslearn` backend system. It does not require direct instantiation by the user; it is typically accessed via the `tslearn.backend.instantiate_backend` factory.

### Output
Returns `unspecified` — The class acts as a namespace or provider for linear algebra routines (e.g., matrix operations) compatible with `numpy` arrays.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import NumPyLinalg

# Accessing the backend helper
linalg_helper = NumPyLinalg()
```

### LLM Instruction Prompt
- Use `NumPyLinalg` only when working within the `tslearn` backend architecture.
- Do not instantiate this class manually for standard time-series analysis; prefer using `tslearn.backend.instantiate_backend("numpy")` to obtain a configured backend object.
- Ensure all input arrays are standard `numpy.ndarray` types when utilizing operations provided by this backend.

### Prompt Snippet
```text
# Example of backend usage
from tslearn.backend import instantiate_backend
be = instantiate_backend("numpy")
# be.linalg provides access to NumPyLinalg methods
```

### Common Failure Modes
- **Manual Instantiation:** Attempting to use `NumPyLinalg` as a standalone utility without the context of the `tslearn` backend system may lead to unexpected behavior or missing dependencies.
- **Type Mismatch:** Passing `torch.Tensor` objects to methods expecting `NumPyLinalg` inputs will cause errors; ensure the backend matches the data type (use `PyTorchBackend` for tensors).

### Fix Code Hint
```python
from tslearn.backend import instantiate_backend

# Correct way to access backend-specific linear algebra
be = instantiate_backend("numpy")
# Use be.linalg instead of direct NumPyLinalg instantiation
result = be.linalg.norm(my_array)
```

## API Test: `NumPyRandom`

### Signature
```python
class NumPyRandom
```
_Source: tslearn/tslearn/backend/numpy_backend.py:132_

### Goal
Provides a backend-specific random number generator interface for the `NumPy` computation engine within `tslearn`.

### Parameters
_None._

### Input
No specific input parameters are required to instantiate the class. It is intended for internal use by the `tslearn` backend system to maintain consistency in random state management across NumPy-based operations.

### Output
Returns an instance of `NumPyRandom` — an object that encapsulates random number generation utilities compatible with the `NumPy` backend.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import NumPyRandom

# Instantiate the random generator helper
rng = NumPyRandom()
```

### LLM Instruction Prompt
- Use `NumPyRandom` only when interacting with or extending the `NumPy` backend of `tslearn`.
- Do not attempt to pass arguments to the constructor, as it takes no parameters.
- This class is a low-level backend helper; prefer using high-level `tslearn` estimators or metric functions unless implementing custom backend-agnostic logic.

### Prompt Snippet
```text
# Example of initializing the NumPy backend random helper
from tslearn.backend.numpy_backend import NumPyRandom
rng_helper = NumPyRandom()
```

### Common Failure Modes
- Attempting to pass arguments to the constructor (e.g., `NumPyRandom(seed=42)`), which will raise a `TypeError` as the class does not accept parameters.
- Misinterpreting this as a general-purpose random number generator; it is specifically scoped to the `tslearn` NumPy backend architecture.

### Fix Code Hint
```python
# Correct usage: instantiate without arguments
from tslearn.backend.numpy_backend import NumPyRandom
rng = NumPyRandom()

# Verify the instance type
assert isinstance(rng, NumPyRandom)
print(f"Correctness witness: {type(rng).__name__} instantiated successfully.")
```

## API Test: `NumPyTesting`

### Signature
```python
class NumPyTesting
```
_Source: tslearn/tslearn/backend/numpy_backend.py:139_

### Goal
Provides a testing utility class for verifying operations and behaviors within the `NumPy` backend of `tslearn`.

### Parameters
_None._

### Input
No specific input parameters are required for instantiation. This class is intended for internal testing and validation of backend-specific logic.

### Output
Returns `unspecified` — The class instance provides methods for assertion and verification within the `NumPy` backend context.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import NumPyTesting

# Instantiate the testing utility
tester = NumPyTesting()
```

### LLM Instruction Prompt
- Use `NumPyTesting` only when writing unit tests or validation scripts for `tslearn` backend components.
- Do not use this class in production machine learning pipelines or data processing workflows.
- Ensure the `tslearn` environment is correctly configured before invoking testing utilities.

### Prompt Snippet
```text
from tslearn.backend.numpy_backend import NumPyTesting
tester = NumPyTesting()
# Use tester methods to validate NumPy backend operations
```

### Common Failure Modes
- **Import Errors:** Attempting to import `NumPyTesting` from the wrong module path (it resides in `tslearn.backend.numpy_backend`).
- **Misuse in Production:** Attempting to use this class for data transformation or model training, which will result in no functional output or errors as it is a testing helper.
- **Environment Mismatch:** Running tests in an environment where `tslearn` dependencies (NumPy) are not properly installed or version-compatible.

### Fix Code Hint
```python
# Ensure the import path is correct
from tslearn.backend.numpy_backend import NumPyTesting

# Verify the backend is active if necessary
from tslearn.backend import instantiate_backend
be = instantiate_backend("numpy")
```

## API Test: `OneD_SymbolicAggregateApproximation`

### Signature
```python
class OneD_SymbolicAggregateApproximation(SymbolicAggregateApproximation)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:530_

_Source doc:_ One-D Symbolic Aggregate approXimation (1d-SAX) transformation. 1d-SAX was originally presented in [1]_. Parameters ---------- n_segments : int (default: 1) Number of PAA segments to compute. alphabet_size_avg : int (default: 5) Number of SAX symbols to use to describe average values. alphabet_size_slope : int (default: 5) Number of SAX symbols to use to describe slopes. sigma_l : float or None (default: None) Scale parameter of the Gaussian distribution used to quantize slopes. If None, the formula given in [1]_ is used: :math:`\sigma_L = \sqrt{0.03 / L}` where :math:`L` is the length of each segment. scale: bool (default: False) Whether input data should be scaled for each feature of each time series to have zero mean and unit variance. Default for this parameter is set to `False` in version 0.4 to ensure backward compatibility, but is likely to change in a future version. Attributes ---------- breakpoints_avg_ : numpy.ndarray of shape (alphabet_size_avg - 1, ) List of breakpoints used to generate SAX symbols for average values. breakpoints_slope_ : numpy.ndarray of shape (alphabet_size_slope - 1, ) List of breakpoints used to generate SAX symbols for slopes. Notes ----- This method requires a dataset of equal-sized time series. Examples -------- >>> one_d_sax = OneD_SymbolicAggregateApproximation(n_segments=3, ...         alphabet_size_avg=2, alphabet_size_slope=2, sigma_l=1.) >>> data = [[-1., 2., 0.1, -1., 1., -1.], [1., 3.2, -1., -3., 1., -1.]] >>> one_d_sax_data = one_d_sax.fit_transform(data) >>> one_d_sax_data.shape (2, 3, 2) >>> one_d_sax_data array([[[1, 1], [0, 0], [1, 0]], <BLANKLINE> [[1, 1], [0, 0], [1, 0]]]) >>> one_d_sax.distance_sax(one_d_sax_data[0], one_d_sax_data[1]) 0.0 >>> one_d_sax.distance(data[0], data[1]) 0.0 >>> one_d_sax.inverse_transform(one_d_sax_data) array([[[ 0.33724488],

### Goal
`OneD_SymbolicAggregateApproximation` performs a symbolic representation of time series by transforming them into a sequence of discrete symbols based on both the average value and the slope of segments within the series.

### Parameters
*   **n_segments** (int, default=1): Number of PAA segments to compute.
*   **alphabet_size_avg** (int, default=5): Number of SAX symbols to use for average values.
*   **alphabet_size_slope** (int, default=5): Number of SAX symbols to use for slopes.
*   **sigma_l** (float or None, default=None): Scale parameter for slope quantization.
*   **scale** (bool, default=False): Whether to scale input data to zero mean and unit variance.

### Input
*   **Data:** A dataset of equal-sized time series, typically provided as a 3D NumPy array of shape `(n_ts, max_sz, d)`.
*   **Preconditions:** The input dataset must contain time series of equal length.

### Output
Returns a 3D NumPy array of shape `(n_ts, n_segments, 2)` containing the symbolic representation (average symbol, slope symbol) for each segment.

### Valid Call Patterns
```python
from tslearn.piecewise import OneD_SymbolicAggregateApproximation
import numpy as np

# Initialize
one_d_sax = OneD_SymbolicAggregateApproximation(n_segments=3, alphabet_size_avg=2, alphabet_size_slope=2)

# Fit and Transform
data = np.random.randn(2, 6, 1)
sax_data = one_d_sax.fit_transform(data)

# Distance computation
dist = one_d_sax.distance(data[0], data[1])
print(f"Correctness witness: distance is {dist}")
```

### LLM Instruction Prompt
- Always ensure the input data is a 3D array of equal-length time series.
- Use `fit_transform` to generate the symbolic representation.
- Use `distance` for raw time series or `distance_sax` for already transformed symbolic data.
- Remember that `scale=False` is the default for backward compatibility.

### Prompt Snippet
```text
from tslearn.piecewise import OneD_SymbolicAggregateApproximation
# ... initialize with n_segments, alphabet_size_avg, alphabet_size_slope ...
# ... fit_transform(X) ...
```

### Common Failure Modes
- **Unequal lengths:** Passing time series of different lengths will raise an error as this method requires equal-sized series.
- **NotFittedError:** Calling `distance` before `fit` or `fit_transform` will raise a `NotFittedError`.
- **Input shape:** Providing 1D or 2D arrays without converting them to the required 3D `(n_ts, sz, d)` format.

### Fix Code Hint
```python
# Ensure data is 3D
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(data)
# Ensure equal lengths
# If lengths differ, consider resampling or padding before using 1d-SAX
```

## API Test: `PatchingLayer`

### Signature
```python
class PatchingLayer(Layer)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:128_

_Source doc:_ Format data for processing with patches matching shapelets length and nans removal. # Input shape 3D tensor with shape: `(batch_size, steps, features)`. # Output shape 4D tensor with shape: `(batch_size, steps - shapelet_size, shapelet_length, features)`.

### Goal
`PatchingLayer` is a Keras-compatible layer designed to transform 3D time-series input tensors into 4D patch tensors, facilitating shapelet-based feature extraction by sliding windows of a specified length across the time dimension.

### Parameters
_None._

### Input
- **Data Format:** A 3D tensor (typically a `tf.Tensor` or `np.ndarray` compatible with Keras layers) with shape `(batch_size, steps, features)`.
- **Preconditions:** The input must be a 3D array. If the data is 1D or 2D, it must be converted to the standard `tslearn` 3D format `(n_ts, max_sz, d)` using `tslearn.utils.to_time_series_dataset` prior to layer ingestion.

### Output
Returns a 4D tensor with shape `(batch_size, steps - shapelet_size, shapelet_length, features)`. This represents the collection of all possible sub-sequences (patches) of length `shapelet_length` extracted from the input time series.

### Valid Call Patterns
```python
# Inferred from signature: usage within a Keras model architecture
from tslearn.shapelets import PatchingLayer
import tensorflow as tf

# Example: batch_size=1, steps=10, features=1, shapelet_size=3
layer = PatchingLayer()
# Note: PatchingLayer typically requires configuration of shapelet_size 
# via class instantiation or layer properties if not provided in signature.
```

### LLM Instruction Prompt
- When using `PatchingLayer`, ensure the input is a 3D tensor.
- The layer is intended for use within deep learning frameworks (Keras/TensorFlow) supported by `tslearn`.
- Always verify that the `steps` dimension of the input is greater than the `shapelet_size` to avoid negative dimension errors.

### Prompt Snippet
```text
# Example usage of PatchingLayer in a Keras model
model = tf.keras.Sequential([
    tf.keras.layers.Input(shape=(max_sz, d)),
    PatchingLayer(),
    ...
])
```

### Common Failure Modes
- **Shape Mismatch:** Providing a 2D array instead of a 3D tensor will cause a dimension error during the forward pass.
- **Insufficient Sequence Length:** If the input `steps` is smaller than the `shapelet_size`, the output dimension calculation `steps - shapelet_size` will result in an invalid shape.
- **Framework Incompatibility:** Attempting to use this layer outside of a Keras/TensorFlow environment, as it inherits from `Layer`.

### Fix Code Hint
```python
# Ensure input is 3D before passing to the layer
import numpy as np
from tslearn.utils import to_time_series_dataset

X = np.random.rand(10, 20, 1) # (n_ts, steps, features)
# If X were 2D, use: X = to_time_series_dataset(X)
```

## API Test: `PiecewiseAggregateApproximation`

### Signature
```python
class PiecewiseAggregateApproximation(TimeSeriesMixin, TransformerMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:61_

_Source doc:_ Piecewise Aggregate Approximation (PAA) transformation. PAA was originally presented in [1]_. Parameters ---------- n_segments : int (default: 1) Number of PAA segments to compute Notes ----- This method requires a dataset of equal-sized time series. Examples -------- >>> paa = PiecewiseAggregateApproximation(n_segments=3) >>> data = [[-1., 2., 0.1, -1., 1., -1.], [1., 3.2, -1., -3., 1., -1.]] >>> paa_data = paa.fit_transform(data) >>> paa_data.shape (2, 3, 1) >>> paa_data array([[[ 0.5 ], [-0.45], [ 0.  ]], <BLANKLINE> [[ 2.1 ], [-2.  ], [ 0.  ]]]) >>> float(paa.distance_paa(paa_data[0], paa_data[1]))  # doctest: +ELLIPSIS 3.15039... >>> float(paa.distance(data[0], data[1]))  # doctest: +ELLIPSIS 3.15039... >>> paa.inverse_transform(paa_data) array([[[ 0.5 ], [ 0.5 ], [-0.45], [-0.45], [ 0.  ], [ 0.  ]], <BLANKLINE> [[ 2.1 ], [ 2.1 ], [-2.  ], [-2.  ], [ 0.  ], [ 0.  ]]]) References ---------- .. [1] E. Keogh & M. Pazzani. Scaling up dynamic time warping for datamining applications. SIGKDD 2000, pp. 285--289.

### Goal
Reduces the dimensionality of time-series data by dividing each series into equal-sized segments and computing the mean value for each segment.

### Parameters
*   `n_segments` (int, default: 1): The number of segments to divide each time series into.

### Input
*   **Data Format:** Expects a 3D NumPy array of shape `(n_ts, max_sz, d)`.
*   **Preconditions:** The input dataset must consist of equal-sized time series. If data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to fitting.

### Output
*   Returns a 3D NumPy array of shape `(n_ts, n_segments, d)` containing the PAA-transformed time series.

### Valid Call Patterns
```python
from tslearn.piecewise import PiecewiseAggregateApproximation
import numpy as np

# Initialize and transform
paa = PiecewiseAggregateApproximation(n_segments=3)
data = np.array([[-1., 2., 0.1, -1., 1., -1.], [1., 3.2, -1., -3., 1., -1.]])
paa_data = paa.fit_transform(data)

# Compute distance on transformed data
dist = paa.distance_paa(paa_data[0], paa_data[1])
print(f"Correctness witness: {dist > 0}")
```

### LLM Instruction Prompt
- Always initialize `PiecewiseAggregateApproximation` with the `n_segments` parameter.
- Ensure input data is formatted as a 3D array before calling `fit` or `fit_transform`.
- Use `distance_paa` only after the estimator has been fitted.
- Remember that `inverse_transform` returns an approximation of the original series by repeating the segment means.

### Prompt Snippet
```text
from tslearn.piecewise import PiecewiseAggregateApproximation
paa = PiecewiseAggregateApproximation(n_segments=N)
X_transformed = paa.fit_transform(X)
```

### Common Failure Modes
- **NotFittedError:** Calling `distance` or `transform` before `fit`.
- **ValueError:** Providing time series of unequal lengths, which violates the PAA requirement for equal-sized series.
- **Shape Mismatch:** Passing 2D arrays directly without converting them to the required 3D `(n_ts, sz, d)` format.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure 3D shape (n_ts, sz, d)
X_3d = to_time_series_dataset(X)
paa = PiecewiseAggregateApproximation(n_segments=5)
X_paa = paa.fit_transform(X_3d)
```

## API Test: `PyTorchBackend`

### Signature
```python
class PyTorchBackend
class PyTorchBackend(object)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:29  (+1 more definition site/overload)_

### Goal
Provides a PyTorch-based computation engine for time-series metrics and operations, enabling automatic differentiation and GPU acceleration within `tslearn`.

### Parameters
_None._

### Input
- **Preconditions:** The `pytorch` package must be installed in the environment.
- **Usage:** This class is typically instantiated via `tslearn.backend.instantiate_backend("pytorch")` rather than direct instantiation. It is designed to handle 3D NumPy arrays or PyTorch tensors as inputs for time-series algorithms.

### Output
Returns an instance of `PyTorchBackend` — an object that exposes backend-specific implementations of distance metrics (e.g., `dtw`, `soft_dtw`) and mathematical operations compatible with PyTorch tensors.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend

# Standard way to obtain the backend instance
be = instantiate_backend("pytorch")
print(be.backend_string)
```

### LLM Instruction Prompt
- When using `PyTorchBackend`, ensure that input data is converted to PyTorch tensors if you intend to utilize automatic differentiation (`requires_grad=True`).
- Always prefer `instantiate_backend("pytorch")` over direct class instantiation to ensure proper integration with the `tslearn` backend registry.
- Use this backend specifically when the workflow requires gradient-based optimization or GPU-accelerated metric computation.

### Prompt Snippet
```text
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch")
# Use 'be' to perform operations that support PyTorch tensors
```

### Common Failure Modes
- **Missing Dependency:** Attempting to use the backend without `torch` installed will raise an `ImportError`.
- **Backend Mismatch:** Passing NumPy arrays to functions expecting PyTorch-specific gradient tracking without explicit conversion can lead to runtime errors or loss of differentiation capabilities.
- **Direct Instantiation:** Manually creating `PyTorchBackend()` instead of using `instantiate_backend` may bypass internal configuration checks required for seamless integration with `tslearn` estimators.

### Fix Code Hint
```python
import torch
from tslearn.backend import instantiate_backend

# Ensure the backend is correctly initialized
be = instantiate_backend("pytorch")

# Example: Verify backend identity
assert be.backend_string == "pytorch"
print(f"Correctness witness: Backend initialized as {be.backend_string}")
```

## API Test: `PyTorchLinalg`

### Signature
```python
class PyTorchLinalg
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:238_

### Goal
Provides a collection of linear algebra operations implemented using the PyTorch backend to support automatic differentiation in time-series metric computations.

### Parameters
_None._

### Input
This class is intended for internal use by the `PyTorchBackend`. It does not require direct instantiation by the user; it is typically accessed via the backend object returned by `tslearn.backend.instantiate_backend("pytorch")`.

### Output
Returns `unspecified` — The class acts as a namespace for PyTorch-based linear algebra routines (e.g., matrix operations, norms) used during the execution of distance metrics like DTW or Soft-DTW.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch")
# PyTorchLinalg is accessed as an attribute of the backend instance
linalg_ops = be.linalg
```

### LLM Instruction Prompt
- Use `PyTorchLinalg` only when working within the `PyTorchBackend` context.
- Do not instantiate this class directly; access it through the `linalg` attribute of a backend object initialized with `instantiate_backend("pytorch")`.
- Ensure that input tensors are on the appropriate device (CPU/GPU) before passing them to methods within this class.

### Prompt Snippet
```text
# Accessing PyTorch linear algebra helpers via the backend
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch")
# Use be.linalg for operations requiring automatic differentiation
```

### Common Failure Modes
- **Direct Instantiation:** Attempting to call `PyTorchLinalg()` directly instead of using the backend-provided instance will lead to unexpected behavior or attribute errors.
- **Backend Mismatch:** Attempting to use `PyTorchLinalg` methods with NumPy arrays instead of PyTorch tensors will cause type errors, as the backend expects `torch.Tensor` inputs for gradient tracking.
- **Missing Dependency:** Calling this in an environment where `pytorch` is not installed will result in an `ImportError` during backend initialization.

### Fix Code Hint
```python
from tslearn.backend import instantiate_backend
# Ensure pytorch is installed and available
be = instantiate_backend("pytorch")
# Use the backend's linalg property for safe, compatible operations
result = be.linalg.norm(tensor_data)
```

## API Test: `PyTorchRandom`

### Signature
```python
class PyTorchRandom
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:243_

### Goal
Provides a backend-specific random number generator interface for the `PyTorch` computation engine within `tslearn`.

### Parameters
_None._

### Input
No input parameters are required for instantiation. The caller must ensure that the `pytorch` package is installed and available in the environment, as this class is a component of the `PyTorch` backend.

### Output
Returns an instance of `PyTorchRandom` — an object that encapsulates random number generation utilities compatible with `torch.Tensor` operations.

### Valid Call Patterns
```python
from tslearn.backend.pytorch_backend import PyTorchRandom

# Instantiate the random generator helper
rng = PyTorchRandom()
print(f"Initialized: {type(rng)}")
```

### LLM Instruction Prompt
- Use `PyTorchRandom` only when working within the `PyTorch` backend context of `tslearn`.
- Do not pass arguments to the constructor.
- Ensure the environment has `torch` installed before invoking this class.

### Prompt Snippet
```text
# Example of initializing the PyTorch random helper
from tslearn.backend.pytorch_backend import PyTorchRandom
rng = PyTorchRandom()
```

### Common Failure Modes
- **ImportError:** Attempting to use `PyTorchRandom` in an environment where `torch` is not installed.
- **Backend Mismatch:** Attempting to use this class when the global `tslearn` backend is set to `NumPy`, leading to type inconsistencies if mixed with non-tensor operations.

### Fix Code Hint
```python
# Ensure PyTorch is available
import torch
from tslearn.backend.pytorch_backend import PyTorchRandom

try:
    rng = PyTorchRandom()
except ImportError:
    print("PyTorch is not installed; PyTorchRandom cannot be initialized.")
```

## API Test: `PyTorchTesting`

### Signature
```python
class PyTorchTesting
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:263_

### Goal
Provides a testing utility class for verifying the integrity and functionality of the `PyTorch` backend implementation within `tslearn`.

### Parameters
_None._

### Input
- No input parameters are required for instantiation.
- Precondition: The `pytorch` package must be installed in the environment to utilize the backend features this class tests.

### Output
Returns `unspecified` — The class acts as a test suite container; instances do not return values in a standard functional sense but are used to execute internal verification routines.

### Valid Call Patterns
```python
from tslearn.backend.pytorch_backend import PyTorchTesting

# Instantiate the testing utility
tester = PyTorchTesting()
```

### LLM Instruction Prompt
- Use `PyTorchTesting` only when writing unit tests or verification scripts for the `tslearn` PyTorch backend.
- Do not use this class in production machine learning pipelines or data processing workflows.
- Ensure the environment has `torch` available before invoking this class.

### Prompt Snippet
```text
from tslearn.backend.pytorch_backend import PyTorchTesting
tester = PyTorchTesting()
```

### Common Failure Modes
- **ImportError:** Attempting to use this class in an environment where `torch` is not installed.
- **Misuse:** Attempting to use this class as a functional component of a model (e.g., inside a `Pipeline` or `fit` method), which is not its intended purpose.

### Fix Code Hint
```python
# Ensure PyTorch is installed and available
import torch
from tslearn.backend.pytorch_backend import PyTorchTesting

try:
    tester = PyTorchTesting()
except ImportError:
    print("PyTorch backend testing requires the torch package.")
```

## API Test: `SoftDTW`

### Signature
```python
class SoftDTW
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:1068_

_Source doc:_ Soft Dynamic Time Warping

### Goal
`SoftDTW` provides a differentiable version of the Dynamic Time Warping (DTW) distance, enabling gradient-based optimization of time-series alignment parameters.

### Parameters
_None._

### Input
- **Data Format:** Expects time-series data as 3D NumPy arrays of shape `(n_ts, max_sz, d)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to use.
- **Backend:** Compatible with both `NumPy` and `PyTorch` backends. When using `PyTorch`, inputs should be tensors to enable automatic differentiation.

### Output
Returns `unspecified` — The output is a scalar distance value or a gradient tensor (if using the `PyTorch` backend) representing the soft-min alignment cost between time series.

### Valid Call Patterns
```python
from tslearn.metrics import SoftDTW
# Inferred from signature: SoftDTW is typically instantiated as an estimator or metric handler
sdtw = SoftDTW()
```

### LLM Instruction Prompt
- When using `SoftDTW`, ensure the input data is formatted as a 3D array `(n_ts, sz, d)`.
- If performing optimization, ensure the `PyTorch` backend is instantiated via `instantiate_backend("pytorch")` to support `backward()` calls on the resulting distance.
- Always verify that input arrays are preprocessed (e.g., scaled) if the application requires normalized distance metrics.

### Prompt Snippet
```text
from tslearn.metrics import SoftDTW
# Initialize SoftDTW and compute distance between time series X and Y
sdtw = SoftDTW()
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError` due to the expected 3D input shape.
- **Backend Incompatibility:** Attempting to use automatic differentiation features without the `pytorch` package installed or without initializing the `PyTorch` backend.
- **Non-Differentiable Inputs:** Providing non-tensor inputs when expecting gradient flow will result in a failure to compute derivatives.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset
from tslearn.metrics import SoftDTW

# Ensure 3D shape
X = to_time_series_dataset(np.random.rand(10, 5))
sdtw = SoftDTW()
# Proceed with distance computation
```

## API Test: `SoftDTWLossPyTorch`

### Signature
```python
class SoftDTWLossPyTorch
class SoftDTWLossPyTorch(torch.nn.Module)
```
_Source: tslearn/tslearn/metrics/soft_dtw_loss_pytorch.py:22  (+1 more definition site/overload)_

### Goal
Computes the Soft-DTW loss or divergence between batches of time series using PyTorch, enabling differentiable alignment-based optimization.

### Parameters
- **gamma** (float): Regularization parameter; strictly positive. Lower values result in less smoothing, approaching hard-DTW.
- **normalize** (bool): If True, computes the Soft-DTW divergence, which is positive and zero when inputs are identical. Default: False.
- **dist_func** (callable, optional): A custom distance function supporting PyTorch autograd. Must accept two inputs of shape (batch_size, ts_length, dim). Defaults to squared Euclidean distance.

### Input
- **Data Format**: Expects two 3D PyTorch tensors of shape `(batch_size, ts_length, dim)`.
- **Preconditions**: The `pytorch` package must be installed. Input tensors should be prepared as 3D arrays; if using 1D or 2D data, convert using `tslearn.utils.to_time_series_dataset` and cast to `torch.Tensor`.

### Output
Returns a `torch.Tensor` of shape `(batch_size,)` containing the computed Soft-DTW loss or divergence values for each pair in the batch.

### Valid Call Patterns
```python
import torch
import tslearn.metrics
from tslearn.metrics import SoftDTWLossPyTorch

# Initialize the loss module
soft_dtw = SoftDTWLossPyTorch(gamma=1.0, normalize=False)

# Define input batches (batch_size, ts_length, dim)
batch_ts_1 = torch.zeros((5, 10, 8), requires_grad=True)
batch_ts_2 = torch.ones((5, 12, 8), requires_grad=True)

# Compute loss
loss = soft_dtw.forward(batch_ts_1, batch_ts_2)
```

### LLM Instruction Prompt
- When using `SoftDTWLossPyTorch`, ensure inputs are `torch.Tensor` objects with 3 dimensions.
- Always instantiate the class before calling `.forward()`.
- If performing optimization, ensure `requires_grad=True` is set on the input tensors.
- Use `normalize=True` if you require a non-negative divergence metric that is zero for identical series.

### Prompt Snippet
```text
# Example usage of SoftDTWLossPyTorch for differentiable loss
criterion = SoftDTWLossPyTorch(gamma=0.1, normalize=True)
loss = criterion.forward(x_batch, y_batch)
loss.mean().backward()
```

### Common Failure Modes
- **Shape Mismatch**: Passing 2D arrays instead of 3D `(batch, length, dim)` tensors will raise a runtime error.
- **Missing PyTorch**: Attempting to use this class without `torch` installed or in an environment where `tslearn` cannot access the PyTorch backend.
- **Non-positive Gamma**: Providing a `gamma <= 0` will violate the mathematical definition of the soft-min operator and cause numerical instability or errors.
- **Device Mismatch**: Ensure both input tensors and the `SoftDTWLossPyTorch` module are on the same device (CPU/GPU).

### Fix Code Hint
```python
# Ensure inputs are 3D and on the correct device
x = x.view(x.shape[0], x.shape[1], -1) if x.ndim == 2 else x
y = y.view(y.shape[0], y.shape[1], -1) if y.ndim == 2 else y
loss_val = soft_dtw_module.forward(x, y)
```

## API Test: `SquaredEuclidean`

### Signature
```python
class SquaredEuclidean
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:1176_

_Source doc:_ Squared Euclidean distance.

### Goal
Provides a class-based implementation of the squared Euclidean distance metric for use in time-series alignment and comparison tasks.

### Parameters
_None._

### Input
The class is typically instantiated for use within estimators or metric frameworks that accept distance objects. Input data must be formatted as 3D NumPy arrays `(n_ts, max_sz, d)` or compatible tensors. If working with variable-length time series, ensure data is pre-processed or padded to a uniform shape as required by the specific estimator context.

### Output
Returns `unspecified` — The class instance acts as a callable or metric object; when invoked on time-series data, it returns the squared Euclidean distance (a scalar or array of distances depending on the context of the call).

### Valid Call Patterns
```python
from tslearn.metrics import SquaredEuclidean
import numpy as np

# Inferred usage as a metric object
metric = SquaredEuclidean()
# Example: calculating distance between two time series
# Note: Actual invocation depends on the specific estimator or backend integration
```

### LLM Instruction Prompt
- Use `SquaredEuclidean` when a squared Euclidean distance metric is required for time-series clustering or classification estimators.
- Ensure input data is converted to 3D arrays `(n_ts, max_sz, d)` using `tslearn.utils.to_time_series_dataset` before passing to any estimator utilizing this metric.
- Do not assume this class accepts parameters in its constructor based on the provided API facts.

### Prompt Snippet
```text
from tslearn.metrics import SquaredEuclidean
metric = SquaredEuclidean()
```

### Common Failure Modes
- Passing 1D or 2D arrays directly without converting them to the required 3D `(n_ts, max_sz, d)` format.
- Attempting to pass initialization arguments to the constructor, which is not supported by the current signature.
- Expecting this class to behave as a standalone function; it is designed as a class-based metric component.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
from tslearn.metrics import SquaredEuclidean

# Ensure data is 3D
X = to_time_series_dataset([[1, 2, 3], [1, 2, 4]])
# Use the metric object within compatible estimators
```

## API Test: `SymbolicAggregateApproximation`

### Signature
```python
class SymbolicAggregateApproximation(PiecewiseAggregateApproximation)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:276_

_Source doc:_ Symbolic Aggregate approXimation (SAX) transformation. SAX was originally presented in [1]_. Parameters ---------- n_segments : int (default: 1) Number of PAA segments to compute alphabet_size_avg : int (default: 5) Number of SAX symbols to use scale: bool (default: False) Whether input data should be scaled for each feature to have zero mean and unit variance across the dataset passed at fit time. Default for this parameter is set to `False` in version 0.4 to ensure backward compatibility, but is likely to change in a future version. Attributes ---------- breakpoints_avg_ : numpy.ndarray of shape (alphabet_size - 1, ) List of breakpoints used to generate SAX symbols Notes ----- This method requires a dataset of equal-sized time series. Examples -------- >>> sax = SymbolicAggregateApproximation(n_segments=3, alphabet_size_avg=2) >>> data = [[-1., 2., 0.1, -1., 1., -1.], [1., 3.2, -1., -3., 1., -1.]] >>> sax_data = sax.fit_transform(data) >>> sax_data.shape (2, 3, 1) >>> sax_data array([[[1], [0], [1]], <BLANKLINE> [[1], [0], [1]]]) >>> sax.distance_sax(sax_data[0], sax_data[1])  # doctest: +ELLIPSIS 0.0 >>> sax.distance(data[0], data[1])  # doctest: +ELLIPSIS 0.0 >>> sax.inverse_transform(sax_data) array([[[ 0.67448975], [ 0.67448975], [-0.67448975], [-0.67448975], [ 0.67448975], [ 0.67448975]], <BLANKLINE> [[ 0.67448975], [ 0.67448975], [-0.67448975], [-0.67448975], [ 0.67448975], [ 0.67448975]]])

### Goal
Performs Symbolic Aggregate approXimation (SAX) to transform time-series data into a symbolic representation by first reducing dimensionality via Piecewise Aggregate Approximation (PAA) and then discretizing the values based on Gaussian distribution breakpoints.

### Parameters
_None._ (Note: Configuration is handled via constructor arguments `n_segments`, `alphabet_size_avg`, and `scale`).

### Input
- **Data:** A dataset of equal-sized time series.
- **Format:** Must be a 3D NumPy array of shape `(n_ts, max_sz, d)`. If input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to fitting.
- **Preconditions:** The input time series must have equal lengths.

### Output
- **Transformation:** Returns a 3D NumPy array of shape `(n_ts, n_segments, d)` containing the symbolic indices.
- **Distance:** `distance()` and `distance_sax()` return a scalar float representing the distance between two time series.
- **Inverse:** `inverse_transform()` returns a 3D NumPy array of shape `(n_ts, max_sz, d)` representing the reconstructed time series from the symbolic approximation.

### Valid Call Patterns
```python
from tslearn.piecewise import SymbolicAggregateApproximation
import numpy as np

sax = SymbolicAggregateApproximation(n_segments=3, alphabet_size_avg=2)
data = np.array([[-1., 2., 0.1, -1., 1., -1.], [1., 3.2, -1., -3., 1., -1.]])
sax_data = sax.fit_transform(data)
dist = sax.distance(data[0], data[1])
```

### LLM Instruction Prompt
- Always ensure the input data is a 3D array of equal-length time series.
- Use `fit_transform` to generate the symbolic representation.
- Use `distance_sax` when comparing two already-transformed symbolic sequences, or `distance` for raw time series.
- Remember that `scale=True` performs normalization based on the dataset statistics at `fit` time.

### Prompt Snippet
```text
# Example of SAX transformation
sax = SymbolicAggregateApproximation(n_segments=N, alphabet_size_avg=A)
X_sax = sax.fit_transform(X)
```

### Common Failure Modes
- **Unequal Lengths:** Passing time series of different lengths will raise an error, as SAX requires equal-sized inputs.
- **NotFittedError:** Calling `distance()` or `inverse_transform()` before calling `fit()` or `fit_transform()`.
- **Dimensionality Mismatch:** Providing 1D or 2D arrays without converting them to the required 3D `(n_ts, sz, d)` format.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure 3D shape before passing to SAX
X_formatted = to_time_series_dataset(X)
sax = SymbolicAggregateApproximation(n_segments=3).fit(X_formatted)
```

## API Test: `TimeSeriesCentroidBasedClusteringMixin`

### Signature
```python
class TimeSeriesCentroidBasedClusteringMixin(TimeSeriesMixin)
```
_Source: tslearn/tslearn/clustering/utils.py:225_

_Source doc:_ Mixin class for centroid-based clustering of time series.

### Goal
Provides a standardized interface for centroid-based clustering estimators in `tslearn`, enabling compatibility with `scikit-learn` workflows for time-series data.

### Parameters
_None._

### Input
This is a mixin class intended for inheritance by clustering estimators. It expects the inheriting class to implement standard `scikit-learn` clustering methods (e.g., `fit`, `predict`) and operate on 3D NumPy arrays of shape `(n_ts, max_sz, d)`.

### Output
Returns `None` (as it is a mixin class definition).

### Valid Call Patterns
```python
from tslearn.clustering.utils import TimeSeriesCentroidBasedClusteringMixin

# This class is intended for inheritance, not direct instantiation.
class MyClusteringEstimator(TimeSeriesCentroidBasedClusteringMixin):
    def __init__(self):
        super().__init__()

# Verification of inheritance
instance = MyClusteringEstimator()
assert isinstance(instance, TimeSeriesCentroidBasedClusteringMixin)
print(f"Correctness witness: {type(instance).__mro__}")
```

### LLM Instruction Prompt
- Use `TimeSeriesCentroidBasedClusteringMixin` only as a base class for custom clustering estimators.
- Do not attempt to instantiate this class directly to perform clustering; use concrete implementations like `TimeSeriesKMeans` instead.
- Ensure that any class inheriting from this mixin also inherits from `TimeSeriesMixin` and implements the required `fit` and `predict` methods.

### Prompt Snippet
```text
from tslearn.clustering.utils import TimeSeriesCentroidBasedClusteringMixin
class CustomClustering(TimeSeriesCentroidBasedClusteringMixin, ...):
    ...
```

### Common Failure Modes
- **Direct Instantiation:** Attempting to use `TimeSeriesCentroidBasedClusteringMixin()` as a standalone clustering model will fail to provide any clustering functionality.
- **Missing Methods:** Inheriting from this mixin without implementing `fit` or `predict` will result in an estimator that is incompatible with `scikit-learn` pipelines.
- **Data Shape Mismatch:** Passing 2D arrays instead of the required 3D `(n_ts, max_sz, d)` format to the resulting estimator will trigger errors during the `fit` process.

### Fix Code Hint
```python
# Ensure the estimator inherits from the mixin and implements the required interface
from tslearn.clustering.utils import TimeSeriesCentroidBasedClusteringMixin
from sklearn.base import BaseEstimator

class MyEstimator(BaseEstimator, TimeSeriesCentroidBasedClusteringMixin):
    def fit(self, X, y=None):
        # Implementation logic here
        return self
```

## API Test: `TimeSeriesDBSCAN`

### Signature
```python
class TimeSeriesDBSCAN(TimeSeriesMixin, ClusterMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/clustering/dbscan.py:19_

_Source doc:_ DBSCAN clustering for time series. Parameters ---------- eps : float (default: 0.5) The maximum distance between two time series for one to be considered as in the neighborhood of the other. min_ts : int (default: 5) The number of time series (including itself) in a neighborhood for a time series to be considered as a core point. metric: {'dtw', 'ctw', 'frechet', 'euclidean', 'precomputed'} (default: 'dtw') Metric to be used for similarity measure between time series. metric_params : dict (default: None) Additional keyword arguments to pass to the metric function. For metrics that accept parallelization of the cross-distance matrix computations, `n_jobs` key passed in `metric_params` is overridden by the `n_jobs` argument. Parameters that do not match the metric computation function signature are ignored. n_jobs : int or None (default=None) The number of jobs to run in parallel for cross-distance matrix computations. Ignored if the cross-distance matrix cannot be computed using parallelization. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`_ for more details. Attributes ---------- core_ts_indices_ : numpy.ndarray of shape (n_core_ts). Indices of core time series. components_: numpy.ndarray of shape (n_core_ts, sz, d) Copy of each core time series found by training. labels_ : numpy.ndarray of integers with shape (n_ts). Labels of each time series. Noisy time series are given the label -1. n_features_in_ : int Number of features seen during training. Notes ----- If `metric` is set to `"euclidean"`, the algorithm expects a dataset of equal-sized time series. Examples -------- >>> from tslearn.generators import random_walk_blobs >>> from tslearn.preprocessing import TimeSeriesScalerMeanVariance >>> X, y = random_walk_blobs(n_ts_per_blob=20, sz=32, d=2, n_blobs=4, random_state=0) >>> X = TimeSeriesScalerMeanVariance(mu=0., std=1.).fit_transform(X) >>> db = TimeSeriesDBSCAN(eps=4, min_ts=3).fit(X) >>> np.unique(db.labels_) # Clusters and noise array([-1,  0,  1,  2,  3]) >>> list(db.labels_).count(-1) # Nb noisy elements 37

### Goal
Performs density-based spatial clustering of applications with noise (DBSCAN) on time-series datasets using specialized alignment-based distance metrics.

### Parameters
*   **eps** (float, default=0.5): Maximum distance between two time series to be considered neighbors.
*   **min_ts** (int, default=5): Minimum number of time series in a neighborhood to define a core point.
*   **metric** (str, default='dtw'): Distance metric to use; options are {'dtw', 'ctw', 'frechet', 'euclidean', 'precomputed'}.
*   **metric_params** (dict, default=None): Additional keyword arguments for the chosen metric.
*   **n_jobs** (int or None, default=None): Number of parallel jobs for distance matrix computation.

### Input
*   **Data Format:** A 3D NumPy array of shape `(n_ts, sz, d)`.
*   **Preconditions:** If `metric='euclidean'`, all time series must have the same length (`sz`). If `metric='precomputed'`, the input to `fit()` must be a precomputed distance matrix of shape `(n_ts, n_ts)`.
*   **Preprocessing:** It is highly recommended to scale data using `TimeSeriesScalerMeanVariance` or similar before clustering to ensure distance metrics are meaningful.

### Output
Returns the fitted `TimeSeriesDBSCAN` instance. The object populates `labels_` (cluster assignments, -1 for noise), `core_ts_indices_` (indices of core points), and `components_` (the core time series).

### Valid Call Patterns
```python
from tslearn.clustering import TimeSeriesDBSCAN
# Standard usage
db = TimeSeriesDBSCAN(eps=4.0, min_ts=3, metric='dtw')
db.fit(X)
# Precomputed distance matrix usage
db = TimeSeriesDBSCAN(metric='precomputed')
db.fit(distance_matrix)
```

### LLM Instruction Prompt
- Always ensure input data is a 3D NumPy array.
- If using `metric='euclidean'`, verify that all time series in the dataset have identical lengths.
- When using `metric='precomputed'`, ensure the input to `fit` is a square distance matrix.
- Use `fit(X)` to train the model; access `labels_` after fitting to retrieve cluster assignments.

### Prompt Snippet
```text
# Example of initializing and fitting TimeSeriesDBSCAN
from tslearn.clustering import TimeSeriesDBSCAN
db = TimeSeriesDBSCAN(eps=0.5, min_ts=5, metric='dtw')
db.fit(X_train)
labels = db.labels_
```

### Common Failure Modes
- **Dimension Mismatch:** Passing 2D arrays instead of 3D arrays (use `tslearn.utils.to_time_series_dataset` to fix).
- **Metric Incompatibility:** Using `metric='euclidean'` with variable-length time series, which will raise an error.
- **Invalid Metric:** Providing a metric string not in the allowed list (e.g., 'gak' is not supported by `TimeSeriesDBSCAN`).

### Fix Code Hint
```python
import numpy as np
from tslearn.clustering import TimeSeriesDBSCAN
from tslearn.utils import to_time_series_dataset

# Ensure 3D shape (n_ts, sz, d)
X = to_time_series_dataset(my_data) 
db = TimeSeriesDBSCAN(eps=0.5, min_ts=3, metric='dtw')
db.fit(X)
print(f"Labels: {db.labels_}")
```

## API Test: `TimeSeriesFeatureSynchronizer`

### Signature
```python
class TimeSeriesFeatureSynchronizer(TimeSeriesMixin, TransformerMixin, BaseEstimator)
```
_Source: tslearn/tslearn/preprocessing/_synchronizer.py:12_

_Source doc:_ Feature synchronizer for time series. Synchronizes features of each time series of a dataset to deal with: * acquisition at different sampling rates: linear interpolation is performed to match the sampling rate of the reference feature * desynchronized timestamps: linear interpolation is performed to match the temporal grid of the reference feature Parameters ---------- reference_feature_index : int (default: 0) The feature that is used as reference for synchronization among each time series. Examples -------- >>> data = [ ...    [[1, 2], [2, np.nan]], ...    [[1, 2], [np.nan, 3]], ... ] >>> TimeSeriesFeatureSynchronizer().fit_transform(data) array([[[ 1.,  2.], [ 2.,  2.]], <BLANKLINE> [[ 1.,  2.], [nan, nan]]]) >>> data = [[[1, 2], [2, 4] , [9, np.nan]]] >>> timestamps = np.array([ ...    [np.array(["2025-01-01", "2025-01-02"], dtype='datetime64'), ...     np.array(["2025-01-03", "2025-01-07"], dtype='datetime64'), ...     np.array(["2025-01-10", "nat"], dtype='datetime64')], ... ]) >>> TimeSeriesFeatureSynchronizer().fit_transform(data, timestamps=timestamps) array([[[1. , 2. ], [2. , 2.4], [9. , 4. ]]])

### Goal
`TimeSeriesFeatureSynchronizer` aligns multiple features within a time-series dataset by performing linear interpolation to match the sampling rate and temporal grid of a specified reference feature.

### Parameters
- `reference_feature_index` (int, default=0): The index of the feature within the time series to be used as the temporal reference for synchronization.

### Input
- `data`: A 3D array-like structure of shape `(n_ts, max_sz, d)` containing the time series data.
- `timestamps` (optional): An array of shape `(n_ts, max_sz)` containing `datetime64` objects corresponding to the temporal points of the input data.
- Preconditions: Input data must be 3D; if provided as 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` first. Timestamps must be strictly increasing for each time series.

### Output
- Returns a 3D NumPy array of shape `(n_ts, max_sz, d)` containing the synchronized time-series data.

### Valid Call Patterns
```python
from tslearn.preprocessing import TimeSeriesFeatureSynchronizer
import numpy as np

# Basic usage
synchronizer = TimeSeriesFeatureSynchronizer()
data = [[[1, 2], [2, np.nan]], [[1, 2], [np.nan, 3]]]
transformed = synchronizer.fit_transform(data)

# Usage with timestamps
data_ts = [[[1, 2], [2, 4] , [9, np.nan]]]
timestamps = np.array([
    [np.array(["2025-01-01", "2025-01-02"], dtype='datetime64'),
     np.array(["2025-01-03", "2025-01-07"], dtype='datetime64'),
     np.array(["2025-01-10", "nat"], dtype='datetime64')],
])
transformed_ts = synchronizer.fit_transform(data_ts, timestamps=timestamps)
```

### LLM Instruction Prompt
- Always ensure input data is in the 3D `(n_ts, sz, d)` format required by `tslearn`.
- If `timestamps` are provided, ensure they are formatted as `datetime64` and are strictly increasing per time series.
- Use `fit_transform` to process the data in a single step.

### Prompt Snippet
```text
from tslearn.preprocessing import TimeSeriesFeatureSynchronizer
synchronizer = TimeSeriesFeatureSynchronizer(reference_feature_index=0)
X_synced = synchronizer.fit_transform(X, timestamps=timestamps)
```

### Common Failure Modes
- `ValueError`: Raised if the provided `timestamps` are not strictly increasing.
- `ValueError`: Raised if input data is not 3D (e.g., passing a 2D array without conversion).
- `NaN` propagation: If the reference feature contains `NaN` values, interpolation may result in `NaN` values in the output.

### Fix Code Hint
```python
import numpy as np
from tslearn.preprocessing import TimeSeriesFeatureSynchronizer

# Ensure data is 3D and timestamps are valid
synchronizer = TimeSeriesFeatureSynchronizer(reference_feature_index=0)
try:
    X_synced = synchronizer.fit_transform(data, timestamps=timestamps)
    print("Synchronization successful.")
except ValueError as e:
    print(f"Synchronization failed: {e}")
```

## API Test: `TimeSeriesImputer`

### Signature
```python
class TimeSeriesImputer(TimeSeriesMixin, TransformerMixin, BaseEstimator)
```
_Source: tslearn/tslearn/preprocessing/preprocessing.py:422_

_Source doc:_ Missing value imputer for time series. Missing values (nans) are replaced according to the chosen imputation method. There might be cases where the computation of missing values is impossible, in which case they are left unchanged (ex: mean of all nans, ffill for the first value... ). The imputer can be configured so that trailing 'empty' samples (nans for all features) are unprocessed by setting the `keep_trailing_nans` parameter to `True`. This might be handy when dealing with variable length time series datasets formatted with :ref:`to_time_series_dataset <fun-tslearn.utils.to_time_series_dataset>`, where time series are padded with 'empty' samples to match the length of the longest time serie. This option aims at preserving the variable length nature of the input dataset. Time series are processed sequentially by the :func:`~transform` and :func:`~fit_transform` methods, and gathered using :ref:`to_time_series_dataset <fun-tslearn.utils.to_time_series_dataset>`, effectively padding if needed. Parameters ---------- method : {'mean', 'median', 'ffill', 'bfill', 'linear', 'constant', Callable}(default: 'mean') The method used to compute missing values. When using linear imputation, starting nans will be replaced with first non-null value and ending nans will be replaced with last non-null value ( except for 'empty' samples when `keep_trailing_nans` set to `True`). When using a Callable, the function should take an array-like representing a timeseries with missing values as input parameter and should return the transformed timeseries. value: float (default: nan) The value to replace missing values with. Only used when method is `constant`. keep_trailing_nans: bool (default: True) Whether trailing samples with nans on all dimensions should be considered padding for variable length time series and kept unprocessed. When set to `False` , trailing 'empty' samples  will be imputed. Notes ----- This method allows datasets of variable lenght time series. While most missing values should be replaced, there might still be nan values in the resulting dataset representing padding when used with variable length time series, or uncomputable data. Examples -------- >>> TimeSeriesImputer().fit_transform([[0, numpy.nan, 6]]) array([[[0.], [3.], [6.]]]) >>> # Dealing with variable length dataset >>> TimeSeriesImputer().fit_transform([[numpy.nan, 3, 6], [numpy.nan, 3]]) array([[[4.5], [3. ], [6. ]], <BLANKLINE>

### Goal
`TimeSeriesImputer` provides a scikit-learn compatible transformer to replace missing values (`NaN`s) in time-series datasets using various statistical or interpolation strategies while respecting variable-length padding.

### Parameters
*   **method**: `{'mean', 'median', 'ffill', 'bfill', 'linear', 'constant', Callable}`, default='mean'. The strategy for imputation.
*   **value**: `float`, default=`np.nan`. The constant value used for imputation if `method='constant'`.
*   **keep_trailing_nans**: `bool`, default=`True`. If `True`, trailing `NaN`s (padding) are ignored; if `False`, they are imputed.

### Input
*   **Data Format**: Accepts 3D NumPy arrays of shape `(n_ts, max_sz, d)`.
*   **Preconditions**: If input data is 1D or 2D, it must be converted to a 3D time-series dataset using `tslearn.utils.to_time_series_dataset` prior to imputation.

### Output
Returns a 3D NumPy array of shape `(n_ts, max_sz, d)` containing the imputed time-series data.

### Valid Call Patterns
```python
from tslearn.preprocessing import TimeSeriesImputer
import numpy as np

# Basic usage
imputer = TimeSeriesImputer(method="mean", keep_trailing_nans=True)
X_imputed = imputer.fit_transform(dataset)

# Variable length handling
X_transformed = TimeSeriesImputer(method="linear").fit_transform(variable_length_dataset)
```

### LLM Instruction Prompt
- Always ensure input data is a 3D array or converted via `to_time_series_dataset`.
- When dealing with variable-length datasets, keep `keep_trailing_nans=True` to avoid imputing padding values as actual data.
- Use `fit_transform` for a single-step workflow or `fit` followed by `transform` for pipeline integration.

### Prompt Snippet
```text
from tslearn.preprocessing import TimeSeriesImputer
imputer = TimeSeriesImputer(method='mean', keep_trailing_nans=True)
X_clean = imputer.fit_transform(X)
```

### Common Failure Modes
- **Shape Mismatch**: Passing 2D arrays directly without using `to_time_series_dataset` will cause dimension errors.
- **Uncomputable Imputation**: Methods like `ffill` or `bfill` may leave `NaN`s at the start or end of a series if no valid values exist to propagate; this is expected behavior.
- **Padding Imputation**: Setting `keep_trailing_nans=False` on variable-length datasets will incorrectly impute the padding `NaN`s, potentially biasing downstream models.

### Fix Code Hint
```python
# Ensure 3D shape for variable length data
from tslearn.utils import to_time_series_dataset
X_3d = to_time_series_dataset(my_list_of_arrays)
X_imputed = TimeSeriesImputer(method="mean").fit_transform(X_3d)
```

## API Test: `TimeSeriesKMeans`

### Signature
```python
class TimeSeriesKMeans(TimeSeriesCentroidBasedClusteringMixin, TransformerMixin, ClusterMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/clustering/kmeans.py:476_

_Source doc:_ K-means clustering for time-series data. Parameters ---------- n_clusters : int (default: 3) Number of clusters to form. max_iter : int (default: 50) Maximum number of iterations of the k-means algorithm for a single run. tol : float (default: 1e-6) Inertia variation threshold. If at some point, inertia varies less than this threshold between two consecutive iterations, the model is considered to have converged and the algorithm stops. n_init : int (default: 1) Number of time the k-means algorithm will be run with different centroid seeds. The final results will be the best output of n_init consecutive runs in terms of inertia. metric : {"euclidean", "dtw", "softdtw"} (default: "euclidean") Metric to be used for both cluster assignment and barycenter computation. If "dtw", DBA is used for barycenter computation. max_iter_barycenter : int (default: 100) Number of iterations for the barycenter computation process. Only used if `metric="dtw"` or `metric="softdtw"`. metric_params : dict or None (default: None) Parameter values for the chosen metric. For metrics that accept parallelization of the cross-distance matrix computations, `n_jobs` key passed in `metric_params` is overridden by the `n_jobs` argument. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel for cross-distance matrix computations. Ignored if the cross-distance matrix cannot be computed using parallelization. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`_ for more details. dtw_inertia: bool (default: False) Whether to compute DTW inertia even if DTW is not the chosen metric. verbose : int (default: 0) If nonzero, print information about the inertia while learning the model and joblib progress messages are printed. random_state : integer or numpy.RandomState, optional Generator used to initialize the centers. If an integer is given, it fixes the seed. Defaults to the global numpy random number generator. init : {'k-means++', 'random' or an ndarray} (default: 'k-means++') Method for initialization:

### Goal
Performs k-means clustering on time-series datasets using specialized distance metrics (Euclidean, DTW, or Soft-DTW) and barycenter computation methods to group similar temporal patterns.

### Parameters
_None._

### Input
- **Data:** A 3D NumPy array of shape `(n_ts, max_sz, d)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` before calling `fit`.

### Output
Returns a fitted `TimeSeriesKMeans` estimator instance. The object contains attributes such as `labels_` (cluster assignments for each input series) and `cluster_centers_` (the computed barycenters).

### Valid Call Patterns
```python
from tslearn.clustering import TimeSeriesKMeans
import numpy as np

# Example based on project test suite
n, sz, d = 15, 10, 3
rng = np.random.RandomState(0)
time_series = rng.randn(n, sz, d)

km = TimeSeriesKMeans(n_clusters=3, metric="euclidean", max_iter=5,
                      verbose=False, random_state=rng).fit(time_series)
```

### LLM Instruction Prompt
- Always ensure input data is a 3D NumPy array `(n_ts, sz, d)`.
- When using `metric="dtw"`, be aware that the algorithm uses DBA (Dynamic Time Warping Barycenter Averaging) for centroid computation, which is computationally more expensive than Euclidean.
- Use `random_state` to ensure reproducibility in clustering results.

### Prompt Snippet
```text
from tslearn.clustering import TimeSeriesKMeans
km = TimeSeriesKMeans(n_clusters=K, metric=METRIC, random_state=SEED)
km.fit(X)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError`.
- **Convergence Issues:** Setting `max_iter` too low for complex datasets may result in sub-optimal clustering.
- **Memory Errors:** Using `metric="dtw"` on very large datasets can be memory-intensive; consider `n_jobs` for parallelization.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure 3D shape
X_formatted = to_time_series_dataset(X)
km = TimeSeriesKMeans(n_clusters=3).fit(X_formatted)
```

## API Test: `TimeSeriesMLPClassifier`

### Signature
```python
class TimeSeriesMLPClassifier(TimeSeriesMixin, MLPClassifier)
```
_Source: tslearn/tslearn/neural_network/neural_network.py:11_

_Source doc:_ A Multi-Layer Perceptron classifier for time series. This class mainly reshapes data so that it can be fed to `scikit-learn`'s ``MLPClassifier``. It accepts the exact same hyper-parameters as ``MLPClassifier``, check `scikit-learn docs <https://scikit-learn.org/stable/modules/generated/sklearn.neural_network.MLPClassifier.html>`__ for a list of parameters and attributes. Notes ----- This method requires a dataset of equal-sized time series. Examples -------- >>> from tslearn.generators import random_walk_blobs >>> X, y = random_walk_blobs(n_ts_per_blob=30, sz=16, d=2, n_blobs=3, ...                          random_state=0) >>> mlp = TimeSeriesMLPClassifier(hidden_layer_sizes=(64, 64), ...                               random_state=0) >>> mlp.fit(X, y)  # doctest: +ELLIPSIS TimeSeriesMLPClassifier(...) >>> [c.shape for c in mlp.coefs_] [(32, 64), (64, 64), (64, 3)] >>> [c.shape for c in mlp.intercepts_] [(64,), (64,), (3,)]

### Goal
`TimeSeriesMLPClassifier` provides a Multi-Layer Perceptron classifier for time-series data by automatically flattening 3D input arrays into a 2D format compatible with `scikit-learn`'s `MLPClassifier`.

### Parameters
_None._

### Input
- **Data Format:** Requires a 3D NumPy array of shape `(n_ts, sz, d)`, where `n_ts` is the number of samples, `sz` is the time-series length, and `d` is the number of features.
- **Preconditions:** The dataset must consist of equal-sized time series. If data is variable-length, it must be resampled or padded to a uniform length before fitting.

### Output
Returns an instance of `TimeSeriesMLPClassifier` (a `scikit-learn` estimator). After calling `fit`, it exposes standard `MLPClassifier` attributes such as `coefs_` and `intercepts_`.

### Valid Call Patterns
```python
from tslearn.neural_network import TimeSeriesMLPClassifier
from tslearn.generators import random_walk_blobs

X, y = random_walk_blobs(n_ts_per_blob=30, sz=16, d=2, n_blobs=3, random_state=0)
mlp = TimeSeriesMLPClassifier(hidden_layer_sizes=(64, 64), random_state=0)
mlp.fit(X, y)
```

### LLM Instruction Prompt
- When using `TimeSeriesMLPClassifier`, ensure the input data is a 3D NumPy array of equal-sized time series.
- Pass any `scikit-learn` `MLPClassifier` hyper-parameters (e.g., `hidden_layer_sizes`, `activation`, `max_iter`) directly to the `TimeSeriesMLPClassifier` constructor.
- Do not attempt to pass variable-length time series directly; use `tslearn` preprocessing utilities to ensure uniform length first.

### Prompt Snippet
```text
from tslearn.neural_network import TimeSeriesMLPClassifier
clf = TimeSeriesMLPClassifier(hidden_layer_sizes=(100,), max_iter=200)
clf.fit(X_train, y_train)
```

### Common Failure Modes
- **Variable Length Error:** Passing a list of arrays with different lengths will cause a `ValueError` during `fit` because the underlying `MLPClassifier` requires a fixed-size input matrix.
- **Dimension Mismatch:** Providing 2D arrays instead of 3D arrays will result in shape errors; use `tslearn.utils.to_time_series_dataset` to ensure the input is 3D.
- **Convergence Issues:** Like standard MLPs, this model is sensitive to feature scaling; ensure data is normalized (e.g., using `TimeSeriesScalerMinMax`) before training.

### Fix Code Hint
```python
from tslearn.preprocessing import TimeSeriesScalerMinMax
from tslearn.utils import to_time_series_dataset

# Ensure 3D shape and uniform length
X_formatted = to_time_series_dataset(X_raw)
X_scaled = TimeSeriesScalerMinMax().fit_transform(X_formatted)
# Now safe to fit
mlp.fit(X_scaled, y)
```

## API Test: `TimeSeriesMLPRegressor`

### Signature
```python
class TimeSeriesMLPRegressor(TimeSeriesMixin, MLPRegressor)
```
_Source: tslearn/tslearn/neural_network/neural_network.py:133_

_Source doc:_ A Multi-Layer Perceptron regressor for time series. This class mainly reshapes data so that it can be fed to `scikit-learn`'s ``MLPRegressor``. It accepts the exact same hyper-parameters as ``MLPRegressor``, check `scikit-learn docs <https://scikit-learn.org/stable/modules/generated/sklearn.neural_network.MLPRegressor.html>`__ for a list of parameters and attributes. Notes ----- This method requires a dataset of equal-sized time series. Examples -------- >>> mlp = TimeSeriesMLPRegressor(hidden_layer_sizes=(64, 64), ...                               random_state=0) >>> mlp.fit(X=[[1, 2, 3], [1, 1.2, 3.2], [3, 2, 1]], ...         y=[0, 0, 1])  # doctest: +ELLIPSIS TimeSeriesMLPRegressor(...) >>> [c.shape for c in mlp.coefs_] [(3, 64), (64, 64), (64, 1)] >>> [c.shape for c in mlp.intercepts_] [(64,), (64,), (1,)]

### Goal
`TimeSeriesMLPRegressor` provides a wrapper around `scikit-learn`'s `MLPRegressor` to perform regression on time-series data by automatically flattening the temporal dimension for neural network processing.

### Parameters
_None._

### Input
- **X**: A 3D NumPy array of shape `(n_ts, max_sz, d)` representing the time-series dataset.
- **y**: A 1D array-like of shape `(n_ts,)` representing the target values for regression.
- **Preconditions**: The input dataset must consist of equal-sized time series. If data is variable-length, it must be resampled or padded to a uniform length before fitting.

### Output
Returns an instance of `TimeSeriesMLPRegressor`. After calling `fit`, the object exposes standard `scikit-learn` MLP attributes such as `coefs_` and `intercepts_`.

### Valid Call Patterns
```python
from tslearn.neural_network import TimeSeriesMLPRegressor

# Initialize with scikit-learn MLPRegressor hyperparameters
mlp = TimeSeriesMLPRegressor(hidden_layer_sizes=(64, 64), random_state=0)

# Fit on 3D time-series data
X = [[[1], [2], [3]], [[1], [1.2], [3.2]], [[3], [2], [1]]]
y = [0, 0, 1]
mlp.fit(X, y)
```

### LLM Instruction Prompt
- When using `TimeSeriesMLPRegressor`, ensure the input `X` is a 3D array.
- Pass any standard `MLPRegressor` hyperparameters (e.g., `hidden_layer_sizes`, `activation`, `learning_rate`) directly to the constructor.
- Remember that this estimator requires equal-length time series; verify input dimensions before calling `fit`.

### Prompt Snippet
```text
from tslearn.neural_network import TimeSeriesMLPRegressor
mlp = TimeSeriesMLPRegressor(hidden_layer_sizes=(64, 64))
mlp.fit(X_train, y_train)
```

### Common Failure Modes
- **Variable Length Input**: Passing a list of arrays with different lengths will cause a `ValueError` during the internal reshaping process.
- **Dimension Mismatch**: Providing 2D data instead of 3D `(n_ts, sz, d)` will result in shape errors.
- **Convergence Issues**: Like standard MLPs, this model may require scaling input features (e.g., using `TimeSeriesScalerMinMax`) to ensure stable convergence.

### Fix Code Hint
```python
from tslearn.preprocessing import TimeSeriesScalerMinMax
from tslearn.neural_network import TimeSeriesMLPRegressor

# Ensure data is scaled and uniform length
scaler = TimeSeriesScalerMinMax()
X_scaled = scaler.fit_transform(X)
mlp = TimeSeriesMLPRegressor(hidden_layer_sizes=(32,), max_iter=500)
mlp.fit(X_scaled, y)
print(f"Correctness witness: Model fitted with {len(mlp.coefs_)} layers.")
```

## API Test: `TimeSeriesMixin`

### Signature
```python
class TimeSeriesMixin(object)
```
_Source: tslearn/tslearn/bases/bases.py:53_

### Goal
Provides a base mixin class for `tslearn` estimators to ensure compatibility with `scikit-learn` workflows and standard time-series data handling conventions.

### Parameters
_None._

### Input
This class is intended to be used as a parent class for custom estimators. It does not accept direct input parameters. When used, it expects the inheriting class to follow `scikit-learn` estimator patterns, specifically regarding the handling of 3D time-series arrays of shape `(n_ts, max_sz, d)`.

### Output
Returns `None` (it is a class definition used for inheritance).

### Valid Call Patterns
```python
from tslearn.bases import TimeSeriesMixin

class MyCustomEstimator(TimeSeriesMixin):
    def __init__(self):
        pass

# Verification of inheritance
instance = MyCustomEstimator()
assert isinstance(instance, TimeSeriesMixin)
print(f"Correctness witness: {type(instance).__name__} inherits from TimeSeriesMixin")
```

### LLM Instruction Prompt
- Use `TimeSeriesMixin` only as a base class for custom estimators within the `tslearn` ecosystem.
- Do not attempt to instantiate `TimeSeriesMixin` directly for functional logic; it provides structural mixin functionality for `scikit-learn` compatibility.
- Ensure that any class inheriting from `TimeSeriesMixin` implements the standard `fit` and `predict` methods to maintain consistency with the `tslearn` API.

### Prompt Snippet
```text
from tslearn.bases import TimeSeriesMixin
class CustomModel(TimeSeriesMixin):
    ...
```

### Common Failure Modes
- **Direct Instantiation:** Attempting to use `TimeSeriesMixin()` as a standalone utility function will result in an empty object that provides no functional time-series processing capabilities.
- **Missing Scikit-Learn Methods:** Inheriting from `TimeSeriesMixin` without implementing `fit` or `predict` will break compatibility with `scikit-learn` pipelines and `tslearn` utilities.
- **Data Shape Mismatch:** Failing to ensure that the inheriting class processes 3D NumPy arrays `(n_ts, max_sz, d)` will lead to runtime errors when integrated into standard `tslearn` workflows.

### Fix Code Hint
```python
# Ensure your estimator inherits from the mixin and implements required methods
from tslearn.bases import TimeSeriesMixin
from sklearn.base import BaseEstimator

class MyEstimator(BaseEstimator, TimeSeriesMixin):
    def fit(self, X, y=None):
        # X must be (n_ts, sz, d)
        return self
```

## API Test: `TimeSeriesResampler`

### Signature
```python
class TimeSeriesResampler(TimeSeriesMixin, TransformerMixin, BaseEstimator)
```
_Source: tslearn/tslearn/preprocessing/preprocessing.py:23_

_Source doc:_ Resampler for time series. Resample time series so that they reach the target size. Parameters ---------- sz : int (default: -1) Size of the output time series. If not strictly positive, the size of the longuest timeseries in the dataset is used. Examples -------- >>> TimeSeriesResampler(sz=5).fit_transform([[0, 3, 6]]) array([[[0. ], [1.5], [3. ], [4.5], [6. ]]])

### Goal
Resamples time-series datasets to a uniform target length using linear interpolation, enabling compatibility with estimators that require fixed-length input arrays.

### Parameters
- **sz** (int, default=-1): The desired number of time steps for the output time series. If set to -1 or a non-positive value, the resampler defaults to the length of the longest time series in the input dataset.

### Input
- **Data Format**: Expects a 3D NumPy array of shape `(n_ts, sz, d)`. 
- **Preconditions**: If input data is 1D or 2D, it must be converted to a 3D time-series dataset using `tslearn.utils.to_time_series_dataset` prior to calling `fit` or `fit_transform`.
- **Variable Length**: Supports variable-length input sequences, which are internally padded or truncated via interpolation to the target `sz`.

### Output
Returns a 3D NumPy array of shape `(n_ts, sz, d)` containing the resampled time series.

### Valid Call Patterns
```python
from tslearn.preprocessing import TimeSeriesResampler
from tslearn.utils import to_time_series_dataset

# Standard usage with explicit target size
X = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3]])
resampler = TimeSeriesResampler(sz=5)
X_resampled = resampler.fit_transform(X)
```

### LLM Instruction Prompt
- Always ensure input data is formatted as a 3D NumPy array using `to_time_series_dataset` before passing it to `TimeSeriesResampler`.
- When using in a `scikit-learn` Pipeline, ensure `TimeSeriesResampler` is placed before estimators that do not support variable-length input.
- If `sz` is not provided, the resampler will automatically determine the target size based on the longest sequence in the training set.

### Prompt Snippet
```text
# Example of using TimeSeriesResampler in a pipeline
from tslearn.preprocessing import TimeSeriesResampler
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ("resampler", TimeSeriesResampler(sz=10)),
    ("model", SomeEstimator())
])
```

### Common Failure Modes
- **Shape Mismatch**: Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError` as the estimator expects 3D input.
- **Invalid `sz`**: Providing a non-integer value for `sz` will result in a `TypeError`.
- **Empty Input**: Passing an empty list or array will cause the resampler to fail during the `fit` process.

### Fix Code Hint
```python
import numpy as np
from tslearn.preprocessing import TimeSeriesResampler
from tslearn.utils import to_time_series_dataset

# Ensure data is 3D
X = to_time_series_dataset([np.array([1, 2]), np.array([1, 2, 3, 4])])
resampler = TimeSeriesResampler(sz=3)
X_transformed = resampler.fit_transform(X)

# Correctness witness
assert X_transformed.shape == (2, 3, 1)
print(f"Resampled shape: {X_transformed.shape}")
```

## API Test: `TimeSeriesSVC`

### Signature
```python
class TimeSeriesSVC(TimeSeriesSVMMixin, ClassifierMixin, BaseEstimator)
```
_Source: tslearn/tslearn/svm/svm.py:110_

_Source doc:_ Time-series specific Support Vector Classifier. Parameters ---------- C : float, optional (default=1.0) Penalty parameter C of the error term. kernel : string, optional (default='gak') Specifies the kernel type to be used in the algorithm. It must be one of 'gak' or a kernel accepted by ``sklearn.svm.SVC``. If none is given, 'gak' will be used. If a callable is given it is used to pre-compute the kernel matrix from data matrices; that matrix should be an array of shape ``(n_samples, n_samples)``. degree : int, optional (default=3) Degree of the polynomial kernel function ('poly'). Ignored by all other kernels. gamma : float, optional (default='auto') Kernel coefficient for 'gak', 'rbf', 'poly' and 'sigmoid'. For 'gak' kernel, a `RuntimeError` is raised at fit time when value is close to 0 and therefore not compatible with 'gak' kernel. If gamma is 'auto' then: - for 'gak' kernel, it is computed based on a sampling of the training set (cf :ref:`tslearn.metrics.gamma_soft_dtw <fun-tslearn.metrics.gamma_soft_dtw>`). A `RuntimeError` is raised at fit time when computed value is close to 0 and therefore not compatible with 'gak' kernel. - for other kernels (eg. 'rbf'), 1/n_features will be used. coef0 : float, optional (default=0.0) Independent term in kernel function. It is only significant in 'poly' and 'sigmoid'. shrinking : boolean, optional (default=True) Whether to use the shrinking heuristic. probability : boolean, optional (default=False) Whether to enable probability estimates. This must be enabled prior to calling `fit`, and will slow down that method. Also, probability estimates are not guaranteed to match predict output. See our :ref:`dedicated user guide section <kernels-ml>` for more details. tol : float, optional (default=1e-3) Tolerance for stopping criterion. cache_size : float, optional (default=200.0) Specify the size of the kernel cache (in MB). class_weight : {dict, 'balanced'}, optional Set the parameter C of class i to class_weight[i]*C for SVC. If not given, all classes are supposed to have weight one. The "balanced" mode uses the values of y to automatically adjust weights inversely proportional to class frequencies in the input data as ``n_samples / (n_classes * np.bincount(y))``

### Goal
`TimeSeriesSVC` performs time-series classification using Support Vector Machines, supporting specialized kernels like the Global Alignment Kernel (GAK) to handle temporal alignments and variable-length sequences.

### Parameters
_None._ (Note: Parameters are passed via the constructor as keyword arguments).

### Input
- **X**: 3D `numpy` array of shape `(n_ts, max_sz, d)`. If data is variable-length, it must be formatted as a 3D array where `max_sz` is the length of the longest sequence, with padding applied to shorter sequences. Use `tslearn.utils.to_time_series_dataset` to ensure correct formatting.
- **y**: 1D array-like of shape `(n_ts,)` containing the target class labels.

### Output
Returns an instance of `TimeSeriesSVC`. After calling `.fit(X, y)`, the `.predict(X)` method returns an array of predicted class labels of shape `(n_ts,)`.

### Valid Call Patterns
```python
from tslearn.svm import TimeSeriesSVC
from tslearn.utils import to_time_series_dataset
import numpy as np

X = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3]])
y = [0, 1]
clf = TimeSeriesSVC(kernel="gak")
clf.fit(X, y)
predictions = clf.predict(X)
```

### LLM Instruction Prompt
- Always use `tslearn.utils.to_time_series_dataset` to prepare input data before passing it to `fit`.
- When using the default `kernel="gak"`, ensure the `gamma` parameter is either set to `'auto'` or a positive float; a value too close to 0 will trigger a `RuntimeError`.
- `TimeSeriesSVC` is fully compatible with `scikit-learn` pipelines and cross-validation tools.

### Prompt Snippet
```text
from tslearn.svm import TimeSeriesSVC
clf = TimeSeriesSVC(kernel='gak', C=1.0)
clf.fit(X_train, y_train)
```

### Common Failure Modes
- **Shape Mismatch**: Passing 2D arrays directly without using `to_time_series_dataset` will cause dimension errors.
- **GAK Gamma Error**: Setting `gamma` to a value near 0 for the `gak` kernel will raise a `RuntimeError` during `fit`.
- **Memory Issues**: Large datasets with `kernel='gak'` can be memory-intensive; adjust `cache_size` if necessary.

### Fix Code Hint
```python
# Ensure data is 3D before fitting
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X_raw)
clf = TimeSeriesSVC(kernel="gak").fit(X_formatted, y)
```

## API Test: `TimeSeriesSVMMixin`

### Signature
```python
class TimeSeriesSVMMixin(TimeSeriesMixin)
```
_Source: tslearn/tslearn/svm/svm.py:20_

_Source doc:_ Time series mixin for SVM based estimators.

### Goal
Provides a specialized mixin for SVM-based estimators in `tslearn` to ensure compatibility with time-series data structures and `scikit-learn` estimator patterns.

### Parameters
_None._

### Input
This class is intended to be used as a base class for custom SVM estimators. It expects the inheriting class to handle 3D NumPy arrays of shape `(n_ts, max_sz, d)` as per standard `tslearn` estimator requirements.

### Output
Returns `unspecified` — This is a mixin class; it does not return a value upon instantiation but provides methods and attributes to the inheriting estimator class.

### Valid Call Patterns
```python
from tslearn.svm import TimeSeriesSVMMixin

class MyTSClassifier(TimeSeriesSVMMixin):
    def __init__(self):
        super().__init__()

# Verification of inheritance
instance = MyTSClassifier()
assert isinstance(instance, TimeSeriesSVMMixin)
print(f"Correctness witness: {isinstance(instance, TimeSeriesSVMMixin)}")
```

### LLM Instruction Prompt
- Use `TimeSeriesSVMMixin` only as a parent class for custom SVM-based time-series estimators.
- Ensure the inheriting class also inherits from `sklearn.base.BaseEstimator` and `tslearn.base.TimeSeriesMixin` to maintain full `scikit-learn` compatibility.
- Do not attempt to instantiate `TimeSeriesSVMMixin` directly for data processing; it is a structural mixin.

### Prompt Snippet
```text
from tslearn.svm import TimeSeriesSVMMixin
class CustomSVMEstimator(TimeSeriesSVMMixin, ...):
    ...
```

### Common Failure Modes
- **Direct Instantiation:** Attempting to use `TimeSeriesSVMMixin()` as a standalone object instead of a mixin for a class definition.
- **Missing Base Classes:** Failing to inherit from `TimeSeriesMixin` or `BaseEstimator` alongside this mixin, which breaks the expected `tslearn`/`scikit-learn` interface (e.g., `fit`/`predict` methods).
- **Data Shape Mismatch:** Passing 2D arrays directly to an estimator using this mixin without first converting them to 3D `(n_ts, max_sz, d)` using `tslearn.utils.to_time_series_dataset`.

### Fix Code Hint
```python
# Ensure proper inheritance hierarchy
from sklearn.base import BaseEstimator
from tslearn.base import TimeSeriesMixin
from tslearn.svm import TimeSeriesSVMMixin

class MyEstimator(BaseEstimator, TimeSeriesMixin, TimeSeriesSVMMixin):
    def fit(self, X, y):
        # X must be 3D: (n_ts, max_sz, d)
        return self
```

## API Test: `TimeSeriesSVR`

### Signature
```python
class TimeSeriesSVR(TimeSeriesSVMMixin, RegressorMixin, BaseEstimator)
```
_Source: tslearn/tslearn/svm/svm.py:413_

_Source doc:_ Time-series specific Support Vector Regressor. Parameters ---------- C : float, optional (default=1.0) Penalty parameter C of the error term. kernel : string, optional (default='gak') Specifies the kernel type to be used in the algorithm. It must be one of 'gak' or a kernel accepted by ``sklearn.svm.SVC``. If none is given, 'gak' will be used. If a callable is given it is used to pre-compute the kernel matrix from data matrices; that matrix should be an array of shape ``(n_samples, n_samples)``. degree : int, optional (default=3) Degree of the polynomial kernel function ('poly'). Ignored by all other kernels. gamma : float, optional (default='auto') Kernel coefficient for 'gak', 'rbf', 'poly' and 'sigmoid'. For 'gak' kernel, a `RuntimeError` is raised at fit time when value is close to 0 and therefore not compatible with 'gak' kernel. If gamma is 'auto' then: - for 'gak' kernel, it is computed based on a sampling of the training set (cf :ref:`tslearn.metrics.gamma_soft_dtw <fun-tslearn.metrics.gamma_soft_dtw>`). A `RuntimeError` is raised at fit time when computed value is close to 0 and therefore not compatible with 'gak' kernel. - for other kernels (eg. 'rbf'), 1/n_features will be used. coef0 : float, optional (default=0.0) Independent term in kernel function. It is only significant in 'poly' and 'sigmoid'. tol : float, optional (default=1e-3) Tolerance for stopping criterion. epsilon : float, optional (default=0.1) Epsilon in the epsilon-SVR model. It specifies the epsilon-tube within which no penalty is associated in the training loss function with points predicted within a distance epsilon from the actual value. shrinking : boolean, optional (default=True) Whether to use the shrinking heuristic. cache_size :  float, optional (default=200.0) Specify the size of the kernel cache (in MB). n_jobs : int or None, optional (default=None) The number of jobs to run in parallel for GAK cross-similarity matrix computations. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`_ for more details. verbose : int, default: 0

### Goal
`TimeSeriesSVR` performs support vector regression on time-series data, utilizing specialized kernels like the Global Alignment Kernel (GAK) to handle temporal alignments and variable-length sequences.

### Parameters
_None._ (Parameters are passed via the constructor as keyword arguments).

### Input
- **Data:** Input `X` must be a 3D NumPy array of shape `(n_ts, max_sz, d)`. 
- **Preconditions:** If data is variable-length or provided as a list of sequences, it must be converted using `tslearn.utils.to_time_series_dataset` before fitting.
- **Target:** `y` should be a 1D array-like of shape `(n_ts,)` containing the continuous target values.

### Output
Returns a trained `TimeSeriesSVR` estimator instance. The `predict(X)` method returns a 1D NumPy array of predicted continuous values of shape `(n_samples,)`.

### Valid Call Patterns
```python
from tslearn.svm import TimeSeriesSVR
from tslearn.utils import to_time_series_dataset

X = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3]])
y = [0.5, 1.2]

svr = TimeSeriesSVR(kernel="gak")
svr.fit(X, y)
predictions = svr.predict(X)
```

### LLM Instruction Prompt
- Always ensure input data is converted to the 3D `(n_ts, sz, d)` format using `to_time_series_dataset` before calling `fit`.
- When using the `gak` kernel, be aware that `gamma` is critical; if the model raises a `RuntimeError` during `fit`, the `gamma` parameter may be too small.
- This estimator is fully compatible with `scikit-learn` cross-validation utilities like `KFold`.

### Prompt Snippet
```text
from tslearn.svm import TimeSeriesSVR
# Initialize with kernel='gak' for time-series alignment
model = TimeSeriesSVR(kernel='gak', C=1.0)
model.fit(X_train, y_train)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset` will cause a `ValueError`.
- **Gamma Error:** Using `kernel='gak'` with a `gamma` value that results in a near-zero kernel coefficient will trigger a `RuntimeError` at `fit` time.
- **Memory Issues:** Large datasets with the `gak` kernel can consume significant memory; adjust `cache_size` if necessary.

### Fix Code Hint
```python
# Ensure data is 3D and properly formatted
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X_raw)
# If fit fails due to gamma, try setting it explicitly
model = TimeSeriesSVR(kernel='gak', gamma=0.1)
model.fit(X_formatted, y)
```

## API Test: `TimeSeriesScalerMeanVariance`

### Signature
```python
class TimeSeriesScalerMeanVariance(TimeSeriesMixin, TransformerMixin, BaseEstimator)
```
_Source: tslearn/tslearn/preprocessing/preprocessing.py:283_

_Source doc:_ Scaler for time series datasets. When `per_timeseries` is False, scales features based on computation led on the fitted data, so that their mean (resp. standard deviation) in given dimensions is mu (resp. std). The transformation is stateless otherwise, dealing with each timeseries individually. Parameters ---------- mu : float (default: 0.) Mean of the output time series. std : float (default: 1.) Standard deviation of the output time series. per_timeseries: bool (default: True) Whether the scaling should be performed per time series. per_feature: bool (default: True) Whether the scaling should be performed per feature. Meaningless for univariate timeseries. Notes ----- NaNs within a time series are ignored when calculating mu and std. Examples -------- >>> TimeSeriesScalerMeanVariance(mu=0., ...                              std=1.).fit_transform([[0, 3, 6]]) array([[[-1.22474487], [ 0.        ], [ 1.22474487]]]) >>> TimeSeriesScalerMeanVariance(mu=0., ...                              std=1.).fit_transform([[numpy.nan, 3, 6]]) array([[[nan], [-1.], [ 1.]]]) >>> TimeSeriesScalerMeanVariance(per_timeseries=False, ...                              per_feature=False ... ).fit_transform([[[1, 2], [2, 3]], [[3, 4], [4, 5]]]) array([[[-1.63299316, -0.81649658], [-0.81649658,  0.        ]], <BLANKLINE> [[ 0.        ,  0.81649658], [ 0.81649658,  1.63299316]]])

### Goal
Standardizes time-series datasets by scaling individual series or the entire dataset to a target mean and standard deviation, ensuring compatibility with distance-based algorithms.

### Parameters
- `mu` (float, default: 0.): The desired mean of the output time series.
- `std` (float, default: 1.): The desired standard deviation of the output time series.
- `per_timeseries` (bool, default: True): If True, scales each time series independently; if False, scales based on global statistics of the fitted data.
- `per_feature` (bool, default: True): If True, scales each feature dimension independently; if False, scales across all dimensions.

### Input
- **Data Format:** 3D NumPy array of shape `(n_ts, max_sz, d)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` before passing to the scaler.

### Output
- Returns a 3D NumPy array of the same shape as the input, containing the scaled time-series data.

### Valid Call Patterns
```python
from tslearn.preprocessing import TimeSeriesScalerMeanVariance
import numpy as np

# Standard usage
scaler = TimeSeriesScalerMeanVariance(mu=0., std=1.)
X_scaled = scaler.fit_transform(X)

# Global scaling usage
scaler_global = TimeSeriesScalerMeanVariance(per_timeseries=False, per_feature=False)
X_global_scaled = scaler_global.fit_transform(X)
```

### LLM Instruction Prompt
- Always ensure input data is a 3D NumPy array.
- Use `fit_transform` to perform the scaling operation in a single step.
- When working with pipelines, use this class as a transformer step before distance-based estimators like `KShape` or `TimeSeriesKMeans`.

### Prompt Snippet
```text
from tslearn.preprocessing import TimeSeriesScalerMeanVariance
scaler = TimeSeriesScalerMeanVariance(mu=0., std=1.)
X_scaled = scaler.fit_transform(X)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError`.
- **Constant Series:** If a time series has zero variance (all values are identical), the scaler may produce `NaN` values; check for constant series before scaling.
- **Pipeline Integration:** Forgetting to include the scaler in a `sklearn.pipeline.Pipeline` when using distance-based models that are sensitive to scale.

### Fix Code Hint
```python
import numpy as np
from tslearn.preprocessing import TimeSeriesScalerMeanVariance
from tslearn.utils import to_time_series_dataset

# Ensure 3D shape
X = to_time_series_dataset(np.array([[1, 1, 1]]))
scaler = TimeSeriesScalerMeanVariance()
X_scaled = scaler.fit_transform(X)
print(f"Correctness witness: {X_scaled.shape == (1, 3, 1)}")
```

## API Test: `TimeSeriesScalerMinMax`

### Signature
```python
class TimeSeriesScalerMinMax(TimeSeriesMixin, TransformerMixin, BaseEstimator)
```
_Source: tslearn/tslearn/preprocessing/preprocessing.py:141_

_Source doc:_ Scaler for time series datasets. When `per_timeseries` is False, scales features based on computation led on the fitted data, so that their span in given dimensions is between ``min`` and ``max`` where ``value_range=(min, max)``. The transformation is stateless otherwise, dealing with each timeseries individually. Parameters ---------- value_range : tuple (default: (0., 1.)) The minimum and maximum value for the output time series. per_timeseries: bool (default: True) Wether the scaling should be performed per time series. per_feature: bool (default: True) Wether the scaling should be performed per feature. Meaningless for univariate timeseries. Notes ----- NaNs within a time series are ignored when calculating min and max. Examples -------- >>> TimeSeriesScalerMinMax(value_range=(1., 2.)).fit_transform([[0, 3, 6]]) array([[[1. ], [1.5], [2. ]]]) >>> TimeSeriesScalerMinMax(value_range=(1., 2.)).fit_transform( ...     [[numpy.nan, 3, 6]] ... ) array([[[nan], [ 1.], [ 2.]]]) >>> TimeSeriesScalerMinMax(value_range=(1., 2.), per_timeseries=False, per_feature=False).fit_transform( ...    [[[1, 2], [2, 3]], ...    [[3, 4], [4, 5]]] ... ) array([[[1.  , 1.25], [1.25, 1.5 ]], <BLANKLINE> [[1.5 , 1.75], [1.75, 2.  ]]])

### Goal
Scales time-series datasets such that the values in each series (or across the dataset) are mapped to a specified range, preserving the relative structure of the time series while handling missing values (NaNs) by ignoring them during min/max calculation.

### Parameters
*   **value_range** (tuple, default=(0., 1.)): The target minimum and maximum values for the output time series.
*   **per_timeseries** (bool, default=True): If True, scales each time series independently. If False, scales based on the global min/max of the fitted dataset.
*   **per_feature** (bool, default=True): If True, scales each feature dimension independently. Meaningless for univariate time series.

### Input
*   **Data:** 3D NumPy array of shape `(n_ts, max_sz, d)`.
*   **Preconditions:** If input is 1D or 2D, it must be converted to a 3D time-series dataset using `tslearn.utils.to_time_series_dataset` before passing to the scaler.

### Output
Returns a 3D NumPy array of the same shape as the input, containing the scaled time-series data.

### Valid Call Patterns
```python
from tslearn.preprocessing import TimeSeriesScalerMinMax
import numpy as np

# Standard usage
scaler = TimeSeriesScalerMinMax(value_range=(1., 2.))
X_scaled = scaler.fit_transform(np.array([[[0], [3], [6]]]))

# Variable length / NaN handling
estimator = TimeSeriesScalerMinMax(per_timeseries=True)
transformed = estimator.fit_transform([[1, np.nan], [3, 4]])
```

### LLM Instruction Prompt
- Always ensure input data is a 3D array `(n_ts, sz, d)`. Use `tslearn.utils.to_time_series_dataset` if the input is a list of sequences or 2D array.
- When `per_timeseries=False`, ensure the scaler is fitted on the training set before transforming test data to maintain consistent scaling bounds.
- NaNs are handled automatically; do not attempt to impute them before scaling unless specific domain logic requires it.

### Prompt Snippet
```text
from tslearn.preprocessing import TimeSeriesScalerMinMax
scaler = TimeSeriesScalerMinMax(value_range=(min_val, max_val), per_timeseries=True)
X_scaled = scaler.fit_transform(X)
```

### Common Failure Modes
- **Shape Mismatch:** Passing a 2D array directly without using `to_time_series_dataset` will raise a `ValueError` or cause incorrect dimension interpretation.
- **Constant Series:** If a time series has a constant value (e.g., `[1, 1, 1]`), the min and max are equal, which may lead to division by zero or NaN outputs depending on the implementation; verify input data variance.
- **Statelessness:** Assuming `per_timeseries=True` uses global statistics; it does not, so it cannot be used to normalize test data based on training set bounds.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset
from tslearn.preprocessing import TimeSeriesScalerMinMax

# Ensure 3D shape
X = to_time_series_dataset([[1, 2, 3], [4, 5, 6]])
scaler = TimeSeriesScalerMinMax(value_range=(0, 1))
X_transformed = scaler.fit_transform(X)

# Witness correctness
assert X_transformed.shape == (2, 3, 1)
print(f"Min: {np.nanmin(X_transformed)}, Max: {np.nanmax(X_transformed)}")
```

## API Test: `TsLearnTags`

### Signature
```python
class TsLearnTags(Tags)
```
_Source: tslearn/tslearn/bases/bases.py:29_

### Goal
`TsLearnTags` defines the specific metadata and compatibility tags used by `tslearn` estimators to interface with `scikit-learn` utilities and internal validation mechanisms.

### Parameters
_None._

### Input
This class is intended for internal use by `tslearn` estimator classes. It does not require direct user input. It assumes the existence of the `scikit-learn` `Tags` base class.

### Output
Returns `unspecified` — This is a class definition that inherits from `Tags`, used to register estimator capabilities within the `tslearn` framework.

### Valid Call Patterns
```python
from tslearn.bases import TsLearnTags

# This class is typically used as a class attribute in estimators
class MyEstimator:
    def _get_tags(self):
        return TsLearnTags()
```

### LLM Instruction Prompt
- Use `TsLearnTags` only when defining custom estimators that need to inherit `tslearn` or `scikit-learn` compatibility tags.
- Do not instantiate this class for general data processing or metric computation tasks.
- Ensure the environment has `scikit-learn` installed, as `TsLearnTags` relies on the `Tags` base class from that library.

### Prompt Snippet
```text
from tslearn.bases import TsLearnTags
# Define estimator tags
tags = TsLearnTags()
```

### Common Failure Modes
- **ImportError:** Attempting to import `TsLearnTags` without `scikit-learn` installed in the environment.
- **Misuse:** Attempting to use `TsLearnTags` as a functional utility for time-series data (e.g., passing time-series arrays to it), which will result in a `TypeError` or logical error as it is a configuration class, not a processing function.

### Fix Code Hint
```python
# Ensure proper inheritance when creating custom estimators
from tslearn.bases import TsLearnTags
from sklearn.base import BaseEstimator

class CustomTimeSeriesEstimator(BaseEstimator):
    def _get_tags(self):
        return TsLearnTags()
```

## API Test: `UCR_UEA_datasets`

### Signature
```python
class UCR_UEA_datasets
```
_Source: tslearn/tslearn/datasets/ucr_uea.py:36_

_Source doc:_ A convenience class to access UCR/UEA time series datasets. When using one (or several) of these datasets in research projects, please cite [1]_. This class will attempt to recover from some known misnamed files, like the `StarLightCurves` dataset being provided in `StarlightCurves.zip` and alike. Parameters ---------- use_cache : bool (default: True) Whether a cached version of the dataset should be used in :meth:`~load_dataset`, if one is found. Datasets are always cached upon loading, and this parameter only determines whether the cached version shall be refreshed upon loading. root_dir : str or None (default: None) Directory to be used to cache downloaded datasets. If None, a default directory is used: - If the environment variable `XDG_DATA_HOME` is set, the default directory is `$XDG_DATA_HOME/tslearn/UCR_UEA`. - Otherwise, the default directory is `~/.tslearn/datasets/UCR_UEA`. Notes ----- Downloading dataset files can be time-consuming, it is recommended using `use_cache=True` (default) in order to only experience downloading time once per dataset and work on a cached version of the datasets afterward. See Also -------- CachedDatasets : Provides pre-selected datasets for offline use. References ---------- .. [1] A. Bagnall, J. Lines, W. Vickers and E. Keogh, The UEA & UCR Time Series Classification Repository, www.timeseriesclassification.com

### Goal
Provides a programmatic interface to download, cache, and load standard time-series classification datasets from the UCR/UEA repository for benchmarking and research.

### Parameters
*   **use_cache** : `bool` (default: `True`)
    Whether to use a previously cached version of the dataset if available.
*   **root_dir** : `str` or `None` (default: `None`)
    The local filesystem path where datasets are stored. Defaults to `~/.tslearn/datasets/UCR_UEA` or `$XDG_DATA_HOME/tslearn/UCR_UEA`.

### Input
The class requires an internet connection for the initial download of a dataset. The `load_dataset` method (called on an instance) requires a string identifier corresponding to a valid dataset name from the UCR/UEA repository.

### Output
The `load_dataset` method returns a tuple of four NumPy arrays: `(X_train, y_train, X_test, y_test)`, where `X` arrays are 3D time-series datasets of shape `(n_ts, max_sz, d)`.

### Valid Call Patterns
```python
from tslearn.datasets import UCR_UEA_datasets

# Standard usage
data_loader = UCR_UEA_datasets()
X_train, y_train, X_test, y_test = data_loader.load_dataset("Trace")

# Custom cache directory
data_loader = UCR_UEA_datasets(root_dir="/tmp/ts_data")
X_train, y_train, X_test, y_test = data_loader.load_dataset("Trace")
```

### LLM Instruction Prompt
- Always instantiate `UCR_UEA_datasets` before calling `load_dataset`.
- Use `load_dataset` with a valid string name from the UCR/UEA repository.
- Ensure the environment has write access to the `root_dir` for caching purposes.
- Always cite the UCR/UEA repository when using these datasets in research.

### Prompt Snippet
```text
from tslearn.datasets import UCR_UEA_datasets
loader = UCR_UEA_datasets()
X_train, y_train, X_test, y_test = loader.load_dataset("DatasetName")
```

### Common Failure Modes
- **Network Errors:** Failure to download datasets if the machine is offline or the repository is unreachable.
- **Invalid Dataset Name:** Passing a string that does not match a valid UCR/UEA dataset name will raise a `ValueError` or `FileNotFoundError`.
- **Permission Denied:** Attempting to cache data in a `root_dir` where the user lacks write permissions.

### Fix Code Hint
```python
# Ensure the dataset name is correct and the environment has internet access
try:
    data_loader = UCR_UEA_datasets(use_cache=True)
    X_train, y_train, X_test, y_test = data_loader.load_dataset("Trace")
except Exception as e:
    print(f"Failed to load dataset: {e}")
```

## API Test: `VARIMA`

### Signature
```python
class VARIMA(TimeSeriesMixin, BaseEstimator, BaseModelPackage)
```
_Source: tslearn/tslearn/forecasting/_arima.py:111_

_Source doc:_ Vector AutoRegressive Integrated Moving Average (VARIMA) estimator [1]_. Parameters ---------- p : int, (default: 1) AutoRegressive (AR) order of the model. d : int (default: 0) Differentiation order of the model. q : int (default: 0) Moving-Average (MA) order of the model. with_constant : bool (default: True) Whether the model should include an intercept term. seasonal_period: int or None (default: None) When set to a positive integer :math:`m`, the model includes a naïve seasonal integration step where :math:`x'_t = x_t - x_{t-m}`. max_iter : int (default: 50) The maximum number of iterations used while fitting the model. verbose : int (default 0) When set to a positive integer, displays logs of the iteration of the optimization. Not relevant if q=0. Attributes ---------- lle_ : float Loglikelihood of the fitted model intercept_ : array-like of shape=(n_features) Intercept term of the fitted model ar_coeffs_ : array-like of shape=(p, n_features, n_features) AR coefficients of the fitted model ma_coeffs_ : array-like of shape=(q, n_features, n_features) MA coefficients of the fitted model Notes ----- This estimator supports variable length time-series See Also -------- AutoVARIMA: Automatic order selection of a VARIMA model References ---------- .. [1] R. J. Hyndman and G. Athanasopoulos, Forecasting: Principles and Practice. OTexts, 2014. https://otexts.com/fpp3/

### Goal
`VARIMA` provides a multivariate time-series forecasting estimator that models dependencies using AutoRegressive, Integrated, and Moving-Average components.

### Parameters
*   **p** (int, default: 1): AutoRegressive (AR) order.
*   **d** (int, default: 0): Differentiation order.
*   **q** (int, default: 0): Moving-Average (MA) order.
*   **with_constant** (bool, default: True): Whether to include an intercept term.
*   **seasonal_period** (int or None, default: None): Period for naïve seasonal integration.
*   **max_iter** (int, default: 50): Maximum optimization iterations.
*   **verbose** (int, default: 0): Verbosity level for optimization logs.

### Input
Input data must be a 3D NumPy array of shape `(n_ts, max_sz, d)`. The estimator supports variable-length time series (represented by `NaN` padding). Data should be pre-processed if necessary using `tslearn` utilities.

### Output
Returns an instance of the `VARIMA` estimator. The `fit` method returns `self`, and `predict` returns a 3D array of forecasted values with shape `(n_ts, horizon, d)`.

### Valid Call Patterns
```python
from tslearn.forecasting import VARIMA
import numpy as np

# Initialize and fit
model = VARIMA(p=1, d=0, q=0, with_constant=True)
model.fit(X)

# Predict
forecast = model.predict(X, n=5)
```

### LLM Instruction Prompt
- Always initialize `VARIMA` with explicit integer orders `p`, `d`, and `q`.
- Ensure input data is a 3D NumPy array; use `tslearn.utils.to_time_series_dataset` if input is a list of sequences.
- When using `predict`, specify the `n` parameter to define the forecasting horizon.
- If `q > 0`, consider setting `verbose=1` to monitor optimization convergence.

### Prompt Snippet
```text
from tslearn.forecasting import VARIMA
model = VARIMA(p={p}, d={d}, q={q})
model.fit(X)
preds = model.predict(X, n={horizon})
```

### Common Failure Modes
- **Insufficient Data Length:** Providing time series that are too short for the specified `p`, `d`, or `q` orders will raise a `ValueError`.
- **Non-3D Input:** Passing 1D or 2D arrays without conversion via `to_time_series_dataset` will cause shape mismatch errors.
- **Convergence Issues:** If `max_iter` is too low for complex datasets, the model may not reach an optimal solution.

### Fix Code Hint
```python
# Ensure data is 3D and handle variable lengths with NaN
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X_list)
model = VARIMA(p=2, d=1, q=1).fit(X_formatted)
```

## API Test: `accumulated_matrix`

### Signature
```python
def accumulated_matrix(s1, s2, mask, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:106  (+2 more definition site/overload)_

_Source doc:_ Compute the DTW accumulated cost matrix score between two time series. It is not required that both time series share the same size, but they must be the same dimension. Parameters ---------- s1 : array-like, shape=(sz1,) or (sz1, d) First time series. s2 : array-like, shape=(sz2,) or (sz2, d) Second time series. mask : array-like, shape=(sz1, sz2) Mask used to constrain the region of computation. Unconsidered cells must have False values. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- mat : array-like, shape=(sz1, sz2) Accumulated cost matrix. Non computed cells due to masking have infinite value.

### Goal
Computes the Dynamic Time Warping (DTW) accumulated cost matrix, representing the cumulative minimum cost to align two time series under a specified mask constraint.

### Parameters
- `s1`: First time series, array-like of shape `(sz1,)` or `(sz1, d)`.
- `s2`: Second time series, array-like of shape `(sz2,)` or `(sz2, d)`.
- `mask`: Boolean array of shape `(sz1, sz2)` defining the valid computation region; cells set to `False` are excluded and result in infinite cost.
- `be`, default `None`: Backend engine to use for computation; accepts `None`, `"numpy"`, `"pytorch"`, or specific backend instances.

### Input
- **Data Format:** Input time series must be formatted as 1D or 2D arrays. If using 3D datasets, extract the individual series first.
- **Preconditions:** Both `s1` and `s2` must share the same dimension `d`. The `mask` must be pre-computed (e.g., via `tslearn.metrics.compute_mask`) to match the dimensions of `s1` and `s2`.

### Output
Returns an array-like object of shape `(sz1, sz2)` containing the accumulated cost matrix, where each cell `(i, j)` stores the minimum cost to reach that point in the alignment path.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

s1 = np.array([1, 2, 3])
s2 = np.array([1.0, 2.0, 2.0, 3.0])
mask = tslearn.metrics.compute_mask(s1, s2)
matrix = tslearn.metrics.dtw_variants.accumulated_matrix(s1, s2, mask)
print(matrix)
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` have the same feature dimension `d`.
- Use `tslearn.metrics.compute_mask` to generate the `mask` argument before calling this function.
- If using `PyTorch` tensors, explicitly set `be="pytorch"` or ensure the backend is correctly inferred.

### Prompt Snippet
```text
# Example of computing the accumulated cost matrix
mask = tslearn.metrics.compute_mask(s1, s2)
acc_mat = tslearn.metrics.dtw_variants.accumulated_matrix(s1, s2, mask)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing time series with different feature dimensions `d` will raise a `ValueError`.
- **Mask Mismatch:** Providing a `mask` that does not match the shape `(len(s1), len(s2))` will cause an index error or invalid computation.
- **Backend Incompatibility:** Mixing `PyTorch` tensors with the `NumPy` backend (or vice versa) without explicit `be` configuration can lead to type errors.

### Fix Code Hint
```python
# Ensure inputs are properly shaped and mask is valid
from tslearn.utils import to_time_series
s1_clean = to_time_series(s1)
s2_clean = to_time_series(s2)
mask = tslearn.metrics.compute_mask(s1_clean, s2_clean)
acc_mat = tslearn.metrics.dtw_variants.accumulated_matrix(s1_clean, s2_clean, mask)
```

## API Test: `accumulated_matrix_from_dist_matrix`

### Signature
```python
def accumulated_matrix_from_dist_matrix(dist_matrix, mask, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:456_

_Source doc:_ Compute the accumulated cost matrix score between two time series using a precomputed distance matrix. Parameters ---------- dist_matrix : array-like, shape=(sz1, sz2) Array containing the pairwise distances. mask : array-like, shape=(sz1, sz2) Mask. Unconsidered cells must have False values. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- mat : array-like, shape=(sz1, sz2) Accumulated cost matrix.

### Goal
Computes the accumulated cost matrix for time-series alignment by aggregating pairwise distances according to a provided mask, facilitating custom alignment constraints.

### Parameters
- `dist_matrix`: array-like, shape=(sz1, sz2). The precomputed pairwise distance matrix between two time series.
- `mask`: array-like, shape=(sz1, sz2). A boolean mask where `False` values indicate cells that should be excluded from the accumulation process.
- `be`, default `None`: Backend object, string ("numpy" or "pytorch"), or `None`. Determines the computational engine used for the accumulation.

### Input
- **Data Format:** `dist_matrix` and `mask` must be 2D array-like structures of identical shape `(sz1, sz2)`.
- **Preconditions:** The `mask` must explicitly define the valid path space for the alignment. If using the `PyTorch` backend, inputs must be `torch.Tensor` objects or compatible with the `PyTorchBackend`.

### Output
- Returns `mat`: array-like, shape=(sz1, sz2). The resulting accumulated cost matrix representing the minimum cost to reach each cell `(i, j)` given the constraints defined by the mask.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import accumulated_matrix_from_dist_matrix

# Example with NumPy backend
dist_matrix = np.array([[1.0, 2.0], [3.0, 4.0]])
mask = np.array([[True, True], [True, True]])
acc_mat = accumulated_matrix_from_dist_matrix(dist_matrix, mask, be="numpy")

print(f"Correctness witness: {acc_mat.shape == (2, 2)}")
```

### LLM Instruction Prompt
- Use `accumulated_matrix_from_dist_matrix` when you need to perform custom alignment path calculations that require a specific mask (e.g., Sakoe-Chiba band or Itakura parallelogram) on a precomputed distance matrix.
- Always ensure `dist_matrix` and `mask` have matching dimensions.
- Explicitly set `be` to "pytorch" if you require automatic differentiation support.

### Prompt Snippet
```text
from tslearn.metrics import accumulated_matrix_from_dist_matrix
# Ensure dist_matrix and mask are 2D arrays of shape (sz1, sz2)
acc_mat = accumulated_matrix_from_dist_matrix(dist_matrix, mask, be="numpy")
```

### Common Failure Modes
- **Shape Mismatch:** Passing `dist_matrix` and `mask` with different shapes will raise a `ValueError`.
- **Backend Mismatch:** Passing a `torch.Tensor` while specifying `be="numpy"` (or vice versa) may lead to type errors or performance degradation.
- **Invalid Masking:** Providing a mask that disconnects the start `(0,0)` or end `(sz1-1, sz2-1)` cells may result in `inf` values in the accumulated matrix.

### Fix Code Hint
```python
# Ensure inputs are properly shaped and aligned
dist_matrix = np.asarray(dist_matrix)
mask = np.asarray(mask, dtype=bool)
assert dist_matrix.shape == mask.shape, "Matrix and mask must have identical shapes"
```

## API Test: `all`

### Signature
```python
def all(x, axis=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:90_

### Goal
Tests whether all elements in a time-series array or tensor evaluate to `True` along a specified axis.

### Parameters
- `x`: The input array or tensor (e.g., a boolean mask or comparison result) to evaluate.
- `axis`, default `None`: The axis or axes along which to perform the logical AND reduction. If `None`, performs the reduction over the entire array.

### Input
The input `x` should be a NumPy array or PyTorch tensor. When working with time-series data, this is typically the result of a comparison operation (e.g., `labels == target`) performed on the output of a `tslearn` estimator.

### Output
Returns `unspecified` — A boolean or an array/tensor of booleans indicating if all elements are `True` along the specified axis.

### Valid Call Patterns
```python
import numpy as np
from tslearn.clustering import KShape
from tslearn.preprocessing import TimeSeriesScalerMeanVariance

# Example derived from tslearn/tests/test_clustering.py
rng = np.random.RandomState(0)
X = rng.randn(20, 10, 2)
X = TimeSeriesScalerMeanVariance().fit_transform(X)
kshape = KShape(n_clusters=2, n_init=5, random_state=rng).fit(X)

# Valid usage checking label consistency
assert all(kshape.labels_[0] == kshape.labels_[:10])
```

### LLM Instruction Prompt
- Use `all` to verify logical conditions across time-series labels or boolean masks generated by `tslearn` estimators.
- Ensure the input `x` is a boolean-compatible structure (NumPy array or PyTorch tensor).
- When checking equality across slices of labels, ensure the comparison is performed before passing to `all`.

### Prompt Snippet
```text
# Check if all labels in a cluster segment are identical
is_consistent = all(labels_array == expected_value)
```

### Common Failure Modes
- **Type Mismatch:** Passing non-boolean or non-numeric types that cannot be evaluated as truthy/falsy.
- **Axis Out of Bounds:** Providing an `axis` value that exceeds the dimensions of the input array (e.g., `axis=3` for a 3D time-series array).
- **Implicit Conversion:** Expecting `all` to handle list-of-lists directly without converting to a NumPy array or PyTorch tensor first.

### Fix Code Hint
```python
import numpy as np
# Ensure input is a numpy array before calling all()
bool_array = np.array(my_list)
result = all(bool_array)
```

## API Test: `array`

### Signature
```python
def array(self, val, dtype=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:106_

### Goal
Converts the input `val` into a backend-specific array format (e.g., a PyTorch tensor) compatible with the current `tslearn` backend instance.

### Parameters
- `self`: The backend instance (e.g., `PyTorchBackend`) that manages the array creation.
- `val`: The input data to be converted, typically a list, NumPy array, or existing tensor.
- `dtype`, default `None`: The desired data type for the resulting array. If `None`, the type is inferred from `val`.

### Input
- **Data:** Accepts array-like structures (lists, NumPy arrays).
- **Preconditions:** The caller must have an active backend instance (e.g., obtained via `instantiate_backend`). If using the `PyTorch` backend, the `pytorch` package must be installed.

### Output
Returns `unspecified` — A backend-specific array object (e.g., `torch.Tensor` when using the PyTorch backend) containing the data from `val`.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch")
# Inferred from signature:
arr = be.array([1.0, 2.0, 3.0])
```

### LLM Instruction Prompt
- Always use an instantiated backend object (e.g., `be`) to call `array`.
- Ensure the input `val` is compatible with the backend's expected numeric types.
- Do not assume the return type is a standard NumPy array; it will be a backend-specific tensor if the backend is set to `pytorch`.

### Prompt Snippet
```text
# Example of using the backend array helper
be = instantiate_backend("pytorch")
data_tensor = be.array(my_data_list)
```

### Common Failure Modes
- **Backend Mismatch:** Calling `array` on a backend instance that does not match the intended computation requirements (e.g., using `PyTorchBackend` without `torch` installed).
- **Type Incompatibility:** Passing `val` that cannot be cast to the requested `dtype` or the backend's default numeric type.
- **Missing Backend:** Attempting to call `array` without first initializing a backend via `instantiate_backend`.

### Fix Code Hint
```python
from tslearn.backend import instantiate_backend
# Ensure backend is initialized before calling array
be = instantiate_backend("pytorch")
try:
    result = be.array(data, dtype="float32")
except Exception as e:
    print(f"Backend array creation failed: {e}")
```

## API Test: `backward`

### Signature
```python
def backward(ctx, grad_output)
```
_Source: tslearn/tslearn/metrics/soft_dtw_loss_pytorch.py:62_

### Goal
Computes the gradient of the Soft-DTW loss function with respect to the input time-series tensors using the PyTorch autograd engine.

### Parameters
- `ctx`: A context object provided by the PyTorch autograd system to store information for the backward pass.
- `grad_output`: The gradient of the loss with respect to the output of the forward pass.

### Input
- **Preconditions:** Requires the `pytorch` backend to be active. The input time series must be provided as 3D `torch.Tensor` objects of shape `(n_ts, max_sz, d)`.
- **Format:** The function is typically invoked internally by the PyTorch autograd engine when `.backward()` is called on a loss tensor computed via `SoftDTWLossPyTorch`.

### Output
Returns `unspecified` — The gradients of the loss with respect to the input time-series tensors, propagated through the alignment path.

### Valid Call Patterns
```python
import torch
from tslearn.metrics import SoftDTWLossPyTorch

# The backward method is typically triggered via the PyTorch autograd graph
soft_dtw = SoftDTWLossPyTorch(gamma=1.0)
ts1 = torch.zeros((1, 10, 8), requires_grad=True)
ts2 = torch.ones((1, 10, 8), requires_grad=True)
loss = soft_dtw.forward(ts1, ts2)
loss.backward()

# Accessing the computed gradients
print(ts1.grad.shape)
```

### LLM Instruction Prompt
- When using `tslearn` with `PyTorch`, do not call `backward` directly. Instead, define your loss using `SoftDTWLossPyTorch`, perform a forward pass, and call `.backward()` on the resulting loss tensor. Ensure input tensors have `requires_grad=True` enabled.

### Prompt Snippet
```text
# Ensure PyTorch is installed and the backend is initialized
import torch
from tslearn.metrics import SoftDTWLossPyTorch

# Define loss and perform forward pass
loss_fn = SoftDTWLossPyTorch(gamma=1.0)
loss = loss_fn.forward(ts_tensor_a, ts_tensor_b)
loss.backward()
```

### Common Failure Modes
- **Missing PyTorch:** Attempting to use this functionality without `torch` installed or without the `PyTorch` backend will raise an `ImportError` or `RuntimeError`.
- **Non-differentiable inputs:** Passing tensors that do not have `requires_grad=True` will result in `None` gradients after the backward pass.
- **Direct Invocation:** Manually calling `backward(ctx, grad_output)` outside of the autograd context will fail as the `ctx` object is managed by the framework.

### Fix Code Hint
```python
# Ensure inputs are tensors with gradient tracking enabled
ts_input = torch.tensor(data, requires_grad=True)
# Use the standard PyTorch workflow
loss = soft_dtw_loss_pytorch.forward(ts_input, target)
loss.backward()
```

## API Test: `baseline_accuracy`

### Signature
```python
def baseline_accuracy(self, list_datasets=None, list_methods=None)
```
_Source: tslearn/tslearn/datasets/ucr_uea.py:121_

_Source doc:_ Report baseline performances as provided by UEA/UCR website (for univariate datasets only). Parameters ---------- list_datasets: list or None (default: None) A list of strings indicating for which datasets performance should be reported. If None, performance is reported for all datasets. list_methods: list or None (default: None) A list of baselines methods for which performance should be reported. If None, performance for all baseline methods is reported. Returns ------- dict A dictionary in which keys are dataset names and associated values are themselves dictionaries that provide accuracy scores for the requested methods. Examples -------- >>> uea_ucr = UCR_UEA_datasets() >>> dict_acc = uea_ucr.baseline_accuracy( ...         list_datasets=["Adiac", "ChlorineConcentration"], ...         list_methods=["C45"]) >>> len(dict_acc) 2 >>> dict_acc["Adiac"]  # doctest: +ELLIPSIS {'C45': 0.542199...} >>> all_dict_acc = uea_ucr.baseline_accuracy() >>> len(all_dict_acc) 85

### Goal
Retrieves and reports established baseline classification accuracy scores for univariate time-series datasets from the UEA/UCR archive.

### Parameters
- `self`: An instance of `tslearn.datasets.UCR_UEA_datasets`.
- `list_datasets`, default `None`: A list of strings specifying the names of datasets to query; if `None`, all available datasets are returned.
- `list_methods`, default `None`: A list of strings specifying the baseline algorithm names (e.g., "C45") to query; if `None`, all available baseline methods are returned.

### Input
The method requires an initialized `UCR_UEA_datasets` object. It is restricted to univariate datasets provided by the UEA/UCR repository.

### Output
Returns a `dict` where keys are dataset names (strings) and values are nested dictionaries mapping baseline method names (strings) to their corresponding accuracy scores (floats).

### Valid Call Patterns
```python
from tslearn.datasets import UCR_UEA_datasets
uea_ucr = UCR_UEA_datasets()

# Retrieve specific datasets and methods
dict_acc = uea_ucr.baseline_accuracy(
    list_datasets=["Adiac", "ChlorineConcentration"],
    list_methods=["C45"]
)

# Retrieve all available data
all_dict_acc = uea_ucr.baseline_accuracy()
```

### LLM Instruction Prompt
When calling `baseline_accuracy`, ensure the receiver is an instance of `tslearn.datasets.UCR_UEA_datasets`. Use this method only for univariate datasets, as it does not support multivariate data. If filtering by dataset or method, ensure the strings provided match the naming conventions used in the UEA/UCR archive.

### Prompt Snippet
```text
# Example of retrieving baseline accuracy for specific datasets
from tslearn.datasets import UCR_UEA_datasets
uea_ucr = UCR_UEA_datasets()
results = uea_ucr.baseline_accuracy(list_datasets=["Adiac"], list_methods=["C45"])
```

### Common Failure Modes
- **Multivariate Data:** Attempting to use this method for multivariate datasets will result in incomplete or inaccurate reporting, as the method is explicitly for univariate data.
- **Invalid Names:** Passing strings in `list_datasets` or `list_methods` that do not exist in the UEA/UCR repository will result in empty or missing keys in the returned dictionary.
- **Network/Environment:** While the method accesses internal data, ensure the `tslearn` environment is correctly installed to access the bundled dataset metadata.

### Fix Code Hint
```python
# Ensure the instance is correctly initialized before calling
from tslearn.datasets import UCR_UEA_datasets
loader = UCR_UEA_datasets()
# Verify dataset names if results are empty
print(loader.baseline_accuracy(list_datasets=["NonExistentDataset"]))
```

## API Test: `belongs_to_backend`

### Signature
```python
def belongs_to_backend(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:86  (+1 more definition site/overload)_

### Goal
Determines whether a given object (typically an array or tensor) is managed by the specific backend instance currently in use.

### Parameters
- `x`: The object (e.g., NumPy array or PyTorch tensor) to check for backend compatibility.

### Input
- **Data:** The input `x` should be an array-like structure or tensor.
- **Preconditions:** The caller must have an active backend instance (e.g., obtained via `instantiate_backend`) to invoke this method on the backend object.

### Output
Returns `bool` — `True` if the object `x` belongs to the backend instance, `False` otherwise.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend
import numpy as np

# Using the backend instance to verify an array
be = instantiate_backend("numpy")
arr = np.array([1, 2, 3])
assert be.belongs_to_backend(arr) is True
```

### LLM Instruction Prompt
- Always invoke `belongs_to_backend` as a method on a backend instance (e.g., `be.belongs_to_backend(x)`), not as a standalone function.
- Ensure the backend instance `be` has been initialized via `instantiate_backend` before calling this method.
- Use this method to verify that intermediate results (like distance matrices or paths) are compatible with the current computation engine before performing further operations.

### Prompt Snippet
```text
# Verify backend compatibility before processing
if not backend.belongs_to_backend(result):
    # Handle mismatch or convert data
```

### Common Failure Modes
- **Calling as a standalone function:** Attempting to call `belongs_to_backend(x)` directly without the backend instance qualifier will result in a `NameError` or `AttributeError`.
- **Type Mismatch:** Passing a PyTorch tensor to a `NumPyBackend` instance will return `False`, which may cause downstream errors if the code assumes the object is a NumPy array.
- **Uninitialized Backend:** Calling the method on a `None` or uninitialized backend object will raise an `AttributeError`.

### Fix Code Hint
```python
# Ensure the backend is instantiated correctly
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch") # or "numpy"

# Correct usage
is_compatible = be.belongs_to_backend(my_tensor)
```

## API Test: `build`

### Signature
```python
def build(self, input_shape)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:188_

### Goal
Initializes the internal architecture and weight tensors of a `LearningShapelets` estimator based on the dimensions of the input time-series dataset.

### Parameters
- `self`: An instance of the `LearningShapelets` estimator.
- `input_shape`: A tuple or list representing the shape of the input data, typically `(n_ts, max_sz, d)`.

### Input
- **Data Format:** The `input_shape` must reflect the 3D structure required by `tslearn` estimators: `(number of time series, maximum sequence length, number of features)`.
- **Preconditions:** This method is typically called internally during the `fit` process of a `LearningShapelets` model. If called manually, the estimator instance must be properly initialized with hyperparameters before invoking `build`.

### Output
Returns `None`. The method modifies the internal state of the estimator instance by initializing model parameters (such as shapelet weights) based on the provided `input_shape`.

### Valid Call Patterns
```python
from tslearn.shapelets import LearningShapelets
# Inferred from signature:
clf = LearningShapelets(n_shapelets_per_size={10: 5})
clf.build(input_shape=(100, 20, 1))
```

### LLM Instruction Prompt
- When using `LearningShapelets`, do not call `build` manually unless you are implementing a custom training loop or extending the estimator.
- Ensure the `input_shape` matches the 3D array format `(n_ts, max_sz, d)` expected by `tslearn`.
- If the input data is variable-length, ensure it has been preprocessed into a uniform 3D array using `tslearn.utils.to_time_series_dataset` before determining the `input_shape`.

### Prompt Snippet
```text
# Example of initializing shapelet model dimensions
clf.build(input_shape=(n_samples, n_timestamps, n_features))
```

### Common Failure Modes
- **Shape Mismatch:** Passing an incorrect number of dimensions (e.g., 2D instead of 3D) will cause downstream errors during weight initialization.
- **Premature Invocation:** Calling `build` on an uninitialized estimator that lacks necessary hyperparameter configurations (like `n_shapelets_per_size`).
- **Type Errors:** Passing non-tuple/non-list objects as `input_shape`.

### Fix Code Hint
```python
# Ensure input_shape is a tuple representing (n_ts, max_sz, d)
# If using variable length data, pad or resample first.
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(my_data)
clf.build(input_shape=X.shape)
```

## API Test: `cache_all`

### Signature
```python
def cache_all(self)
```
_Source: tslearn/tslearn/datasets/ucr_uea.py:376_

_Source doc:_ Cache all datasets from the UCR/UEA archive for later use.

### Goal
Downloads and caches the complete UCR/UEA time-series classification archive locally to accelerate subsequent data loading operations.

### Parameters
- `self`: An instance of the dataset manager class (typically `UCR_UEA_datasets`) that provides the interface to the archive.

### Input
- **Preconditions:** Requires an active internet connection for the initial download. The function assumes the environment has write permissions to the directory where `tslearn` stores cached datasets.

### Output
Returns `None`. The operation performs side effects by populating the local cache directory with dataset files.

### Valid Call Patterns
```python
from tslearn.datasets import UCR_UEA_datasets

# Inferred from signature: requires an instance of the dataset manager
loader = UCR_UEA_datasets()
loader.cache_all()
```

### LLM Instruction Prompt
- When using `cache_all`, ensure it is called on an instantiated `UCR_UEA_datasets` object. This method is intended for bulk preparation of the environment and should be executed before attempting to load specific datasets if offline access is required later.

### Prompt Snippet
```text
from tslearn.datasets import UCR_UEA_datasets
loader = UCR_UEA_datasets()
loader.cache_all()
```

### Common Failure Modes
- **Network Errors:** Fails if the machine is offline or if the UCR/UEA archive servers are unreachable during the download process.
- **Permission Denied:** Fails if the process lacks write access to the `tslearn` cache directory.
- **Disk Space:** May raise an `OSError` or `IOError` if the local storage is insufficient to hold the full archive.

### Fix Code Hint
```python
# Ensure network connectivity and sufficient disk space before calling
try:
    loader = UCR_UEA_datasets()
    loader.cache_all()
except Exception as e:
    print(f"Failed to cache UCR/UEA datasets: {e}")
```

## API Test: `call`

### Signature
```python
def call(self, inputs, mask=None)
def call(self, inputs)
def call(self, x, **kwargs)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:89  (+3 more definition site/overload)_

### Goal
Executes the forward pass of a `tslearn` layer or estimator component, typically used within deep learning-based time-series models like `LearningShapelets`.

### Parameters
- `self`: The instance of the estimator or layer (e.g., a `LearningShapelets` object).
- `inputs`: The input time-series data, typically a 3D NumPy array of shape `(n_ts, max_sz, d)` or a corresponding tensor.
- `mask`, default `None`: An optional mask to ignore specific time steps or padding in the input sequences.

### Input
- **Data Format:** Expects 3D arrays `(n_ts, max_sz, d)`. If data is 1D or 2D, it must be pre-processed using `tslearn.utils.to_time_series_dataset` before being passed to the model.
- **Preconditions:** The model must be initialized. If using the `PyTorch` backend, inputs should be compatible with `torch.Tensor` operations.

### Output
Returns `unspecified` — The output represents the transformed features or model predictions resulting from the forward pass, typically a tensor or array depending on the active backend.

### Valid Call Patterns
```python
# Inferred from signature: usage within a custom layer or model forward pass
# Example: model.call(X)
```

### LLM Instruction Prompt
- Use `call` only when implementing custom layers or extending `tslearn` estimators that inherit from base classes requiring a forward pass definition.
- Ensure the input `inputs` is a 3D array or tensor; if the input is variable-length, ensure it has been padded or formatted according to the specific estimator's requirements.

### Prompt Snippet
```text
# Example of calling the forward pass
output = model.call(X_input)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of the required 3D `(n_ts, max_sz, d)` format will trigger a dimension error.
- **Backend Incompatibility:** Calling `call` with `NumPy` arrays when the model is configured for the `PyTorch` backend (or vice versa) may lead to type errors during tensor operations.
- **Uninitialized State:** Calling `call` before the estimator has been properly initialized or fitted may result in undefined behavior or attribute errors.

### Fix Code Hint
```python
# Ensure data is 3D before calling
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
output = model.call(X_formatted)
```

## API Test: `cast`

### Signature
```python
def cast(x, dtype)
def cast(self, x, dtype)
def cast(data, array_type='numpy')
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:129  (+2 more definition site/overload)_

### Goal
Converts input time-series data into a specified backend format (NumPy, PyTorch, or list) to ensure compatibility with `tslearn` metric and estimator operations.

### Parameters
- `self`: The backend instance (when called as a method of a backend object).
- `x` / `data`: The input time-series data, which can be a list, NumPy array, or PyTorch tensor.
- `dtype` / `array_type`: The target format for the data. Supported values are `"numpy"`, `"pytorch"`, or `"list"`.

### Input
The input `data` should be a sequence or array-like structure. While `tslearn` estimators typically expect 3D arrays `(n_ts, max_sz, d)`, `cast` is a low-level utility used to normalize data types across different computation backends. Ensure the `pytorch` package is installed if casting to `"pytorch"`.

### Output
Returns `data_cast`: The input data converted to the requested `array_type`.

### Valid Call Patterns
```python
from tslearn.backend import cast

# Casting a list to a numpy array
data_np = cast([1, 2, 3], array_type='numpy')

# Casting within a backend-specific context (e.g., from test suite)
s1 = cast([1, 2, 3], 'numpy')
```

### LLM Instruction Prompt
- Always use `cast` when you need to ensure input data matches the expected backend format of a specific `tslearn` metric or estimator.
- When working with `PyTorch` backends, explicitly cast inputs to `'pytorch'` to ensure compatibility with automatic differentiation.
- Do not assume the input is already in the correct format; use `cast` to normalize inputs before passing them to backend-sensitive functions.

### Prompt Snippet
```text
# Ensure data is compatible with the chosen backend
data = cast(raw_data, array_type='numpy')
```

### Common Failure Modes
- **Missing Dependency:** Attempting to cast to `'pytorch'` without the `pytorch` package installed will raise an `ImportError` or `RuntimeError`.
- **Invalid Type:** Passing an unsupported string to `array_type` (e.g., `'tensorflow'`) will result in a `ValueError` or `TypeError` depending on the backend implementation.
- **Shape Mismatch:** `cast` does not reshape data; if the downstream estimator requires a 3D array, ensure the input is already formatted (e.g., via `to_time_series_dataset`) before or after casting.

### Fix Code Hint
```python
# If casting fails due to backend mismatch, verify the target type string
try:
    data = cast(data, array_type='numpy')
except Exception as e:
    print(f"Casting failed: {e}")
```

## API Test: `cdist`

### Signature
```python
def cdist(x, y, metric='euclidean', p=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:135_

### Goal
Computes the distance matrix between two collections of time-series observations using a specified metric.

### Parameters
- `x`: `numpy.ndarray` or `torch.Tensor` of shape `(n_samples_x, n_features)`.
- `y`: `numpy.ndarray` or `torch.Tensor` of shape `(n_samples_y, n_features)`.
- `metric`, default `'euclidean'`: The distance metric to use (e.g., 'euclidean', 'sqeuclidean').
- `p`, default `None`: The power parameter for the Minkowski metric, if applicable.

### Input
- **Data Format:** Expects 2D arrays/tensors where rows are samples and columns are features. If working with 3D time-series data `(n_ts, sz, d)`, the data must be flattened to 2D `(n_ts, sz * d)` before calling `cdist`.
- **Preconditions:** Ensure `x` and `y` are of the same backend type (NumPy or PyTorch) to avoid type mismatch errors during computation.

### Output
Returns `numpy.ndarray` or `torch.Tensor` of shape `(n_samples_x, n_samples_y)` containing the pairwise distances between elements of `x` and `y`.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import cdist

# Standard Euclidean distance matrix computation
n, sz, d = 15, 10, 3
X = np.random.randn(n, sz * d)
Y = np.random.randn(5, sz * d)
dists = cdist(X, Y, metric="euclidean")
print(f"Distance matrix shape: {dists.shape}")
```

### LLM Instruction Prompt
- Always ensure input arrays are 2D. If the input is 3D time-series data, use `.reshape((n_samples, -1))` before passing to `cdist`.
- Match the backend of `x` and `y` (do not mix NumPy arrays and PyTorch tensors).
- Use `cdist` for standard distance calculations; for alignment-based metrics (like DTW), use the specific `tslearn.metrics` functions instead.

### Prompt Snippet
```text
# Flatten 3D time series to 2D for cdist
X_flat = X.reshape((X.shape[0], -1))
Y_flat = Y.reshape((Y.shape[0], -1))
dists = cdist(X_flat, Y_flat, metric='euclidean')
```

### Common Failure Modes
- **Shape Mismatch:** Passing 3D arrays directly without flattening will result in incorrect distance calculations or dimension errors.
- **Backend Mismatch:** Passing a PyTorch tensor as `x` and a NumPy array as `y` will raise a `TypeError` or `RuntimeError`.
- **Metric Misuse:** Attempting to pass alignment-based metrics (like 'dtw') directly to `cdist` instead of using the dedicated `tslearn.metrics` functions.

### Fix Code Hint
```python
# Ensure inputs are 2D and same type
X = X.reshape(X.shape[0], -1)
Y = Y.reshape(Y.shape[0], -1)
# If using PyTorch, ensure both are tensors
# X = torch.tensor(X); Y = torch.tensor(Y)
dists = cdist(X, Y, metric='euclidean')
```

## API Test: `cdist_ctw`

### Signature
```python
def cdist_ctw(dataset1, dataset2=None, max_iter=100, n_components=None, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, n_jobs=None, verbose=0, be=None)
```
_Source: tslearn/tslearn/metrics/ctw.py:302_

_Source doc:_ Compute cross-similarity matrix using Canonical Time Warping (CTW) similarity measure. Canonical Time Warping is a method to align time series under rigid registration of the feature space. It should not be confused with Dynamic Time Warping (DTW), though CTW uses DTW. It is not required that both time series share the same size, nor the same dimension (CTW will find a subspace that best aligns feature spaces). CTW was originally presented in [1]_. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz1, d) or (n_ts1, sz1) or (sz1,) A dataset of time series. If shape is (n_ts1, sz1), the dataset is composed of univariate time series. If shape is (sz1,), the dataset is composed of a unique univariate time series. dataset2 : None or array-like, shape=(n_ts2, sz2, d) or (n_ts2, sz2) or (sz2,) (default: None) Another dataset of time series. If `None`, self-similarity of `dataset1` is returned. If shape is (n_ts2, sz2), the dataset is composed of univariate time series. If shape is (sz2,), the dataset is composed of a unique univariate time series. max_iter : int (default: 100) Number of iterations for the CTW algorithm. Each iteration n_components : int (default: None) Number of components to be used for Canonical Correlation Analysis. If None, the lower minimum number of features between seq1 and seq2 is used. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for DTW calls. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n-jobs>`__ for more details. verbose : int, optional (default=0) The verbosity level: if non zero, progress messages are printed. Above 50, the output is sent to stdout. The frequency of the messages increases with the verbosity level. If it more than 10, all iterations are reported.

### Goal
Computes a cross-similarity matrix between two datasets of time series using Canonical Time Warping (CTW), which aligns time series by finding a subspace that optimizes feature space registration.

### Parameters
- `dataset1`: Array-like dataset of time series with shape `(n_ts1, sz1, d)` or `(n_ts1, sz1)` or `(sz1,)`.
- `dataset2`, default `None`: Optional second dataset of time series. If `None`, computes the self-similarity matrix of `dataset1`.
- `max_iter`, default `100`: Number of iterations for the CTW algorithm.
- `n_components`, default `None`: Number of components for Canonical Correlation Analysis (CCA). Defaults to the minimum feature count between sequences.
- `global_constraint`, default `None`: Global constraint for DTW path restriction; options are `"itakura"` or `"sakoe_chiba"`.
- `sakoe_chiba_radius`, default `None`: Radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Maximum slope for the Itakura parallelogram constraint.
- `n_jobs`, default `None`: Number of parallel jobs to run.
- `verbose`, default `0`: Verbosity level for progress reporting.
- `be`, default `None`: Backend object for computation (e.g., NumPy or PyTorch).

### Input
Input data must be formatted as 3D arrays `(n_ts, sz, d)`. Use `tslearn.utils.to_time_series_dataset` to ensure input compatibility if working with 1D or 2D lists/arrays.

### Output
Returns a cross-similarity matrix of shape `(n_ts1, n_ts2)` containing the CTW distances between all pairs of time series.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import cdist_ctw
from tslearn.utils import to_time_series_dataset

X = to_time_series_dataset([[1, 2, 3], [1, 2, 3, 4]])
dist_matrix = cdist_ctw(X)
print(f"Correctness witness: {dist_matrix.shape == (2, 2)}")
```

### LLM Instruction Prompt
- Always ensure input datasets are 3D arrays using `to_time_series_dataset` before passing to `cdist_ctw`.
- If `dataset2` is provided, ensure it is also formatted as a 3D array.
- When using `global_constraint`, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is set appropriately.

### Prompt Snippet
```text
cdist_ctw(dataset1=X, dataset2=Y, global_constraint="sakoe_chiba", sakoe_chiba_radius=2)
```

### Common Failure Modes
- Passing 1D or 2D arrays directly without conversion, leading to shape mismatch errors.
- Setting both `sakoe_chiba_radius` and `itakura_max_slope` without a valid `global_constraint`, which may trigger `RuntimeWarning` or unexpected behavior.
- Assuming `dataset2` is required; if omitted, the function computes the self-similarity matrix of `dataset1`.

### Fix Code Hint
```python
# Ensure 3D shape for tslearn metrics
from tslearn.utils import to_time_series_dataset
X_3d = to_time_series_dataset(X)
dist = cdist_ctw(X_3d)
```

## API Test: `cdist_dtw`

### Signature
```python
def cdist_dtw(dataset1, dataset2=None, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, n_jobs=None, verbose=0, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1872  (+1 more definition site/overload)_

_Source doc:_ Compute cross-similarity matrix using Dynamic Time Warping (DTW) similarity measure. DTW is computed as the Euclidean distance between aligned time series, i.e., if :math:`\pi` is the alignment path: .. math:: DTW(X, Y) = \sqrt{\sum_{(i, j) \in \pi} \|X_{i} - Y_{j}\|^2} Note that this formula is still valid for the multivariate case. It is not required that time series share the same size, but they must be the same dimension. DTW was originally presented in [1]_ and is discussed in more details in our :ref:`dedicated user-guide page <dtw>`. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz1, d) or (n_ts1, sz1) or (sz1,) A dataset of time series. If shape is (n_ts1, sz1), the dataset is composed of univariate time series. If shape is (sz1,), the dataset is composed of a unique univariate time series. dataset2 : None or array-like, shape=(n_ts2, sz2, d) or (n_ts2, sz2) or (sz2,) (default: None) Another dataset of time series. If `None`, self-similarity of `dataset1` is returned. If shape is (n_ts2, sz2), the dataset is composed of univariate time series. If shape is (sz2,), the dataset is composed of a unique univariate time series. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for DTW. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns'

### Goal
Computes the cross-similarity matrix between two datasets of time series using the Dynamic Time Warping (DTW) distance metric.

### Parameters
- `dataset1`: Array-like dataset of time series. Must be 3D `(n_ts, sz, d)` or 2D `(n_ts, sz)` for univariate data.
- `dataset2`, default `None`: Optional second dataset. If `None`, computes the self-similarity matrix of `dataset1`.
- `global_constraint`, default `None`: String specifying the path constraint ("itakura" or "sakoe_chiba").
- `sakoe_chiba_radius`, default `None`: Integer radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float maximum slope for the Itakura parallelogram constraint.
- `n_jobs`, default `None`: Number of parallel jobs to run.
- `verbose`, default `0`: Verbosity level.
- `be`, default `None`: Backend object for computation (e.g., NumPy or PyTorch).

### Input
Input data must be formatted as 3D NumPy arrays `(n_ts, sz, d)`. If data is 1D or 2D, use `tslearn.utils.to_time_series_dataset` to ensure compatibility before passing to `cdist_dtw`. Time series in `dataset1` and `dataset2` must have the same dimensionality `d`.

### Output
Returns a 2D NumPy array of shape `(n_ts1, n_ts2)` containing the pairwise DTW distances between the time series in the provided datasets.

### Valid Call Patterns
```python
from tslearn.metrics import cdist_dtw
import numpy as np

# Self-similarity
X = np.random.rand(10, 16, 1)
dists = cdist_dtw(X)

# Cross-similarity
X2 = np.random.rand(5, 16, 1)
dists_cross = cdist_dtw(X, X2, global_constraint="sakoe_chiba", sakoe_chiba_radius=2)
```

### LLM Instruction Prompt
- Always ensure input datasets are 3D arrays `(n_ts, sz, d)` using `tslearn.utils.to_time_series_dataset` if necessary.
- When using `global_constraint`, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is provided or understood to default correctly.
- If performing large-scale computations, set `n_jobs=-1` to utilize all available CPU cores.

### Prompt Snippet
```text
from tslearn.metrics import cdist_dtw
# Ensure X is (n_ts, sz, d)
dist_matrix = cdist_dtw(X, dataset2=Y, global_constraint="sakoe_chiba", sakoe_chiba_radius=2)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays without converting them to 3D `(n_ts, sz, d)` format.
- **Dimensionality Mismatch:** Providing time series with different dimensions `d` between `dataset1` and `dataset2`.
- **Constraint Conflict:** Setting both `sakoe_chiba_radius` and `itakura_max_slope` without specifying a valid `global_constraint`, which may trigger a `RuntimeWarning`.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Convert 2D (n_ts, sz) to 3D (n_ts, sz, 1)
X_formatted = to_time_series_dataset(X)
dists = cdist_dtw(X_formatted)
```

## API Test: `cdist_frechet`

### Signature
```python
def cdist_frechet(dataset1, dataset2=None, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, n_jobs=None, verbose=0, be=None)
```
_Source: tslearn/tslearn/metrics/_frechet.py:647_

_Source doc:_ Compute cross-similarity matrix using Frechet similarity measure [1]_. Frechet is computed as the maximum distance between aligned time series, i.e., if :math:`\pi` is an optimal alignment path: .. math:: Frechet(X, Y) = \max_{(i, j) \in \pi} \|X_{i} - Y_{j}\| Note that this formula is still valid for the multivariate case. It is not required that time series share the same size, but they must be the same dimension. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz1, d) or (n_ts1, sz1) or (sz1,) A dataset of time series. If shape is (n_ts1, sz1), the dataset is composed of univariate time series. If shape is (sz1,), the dataset is composed of a unique univariate time series. dataset2 : None or array-like, shape=(n_ts2, sz2, d) or (n_ts2, sz2) or (sz2,) (default: None) Another dataset of time series. If `None`, self-similarity of `dataset1` is returned. If shape is (n_ts2, sz2), the dataset is composed of univariate time series. If shape is (sz2,), the dataset is composed of a unique univariate time series. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for Frechet. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [2]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details.

### Goal
Computes the cross-similarity matrix between two datasets of time series using the Frechet distance, which measures the maximum distance between points along an optimal alignment path.

### Parameters
- `dataset1`: Array-like dataset of time series with shape `(n_ts1, sz1, d)` or `(n_ts1, sz1)` or `(sz1,)`.
- `dataset2`, default `None`: Optional second dataset of time series. If `None`, computes the self-similarity matrix of `dataset1`.
- `global_constraint`, default `None`: String constraint type, either `"itakura"` or `"sakoe_chiba"`, to restrict the alignment path.
- `sakoe_chiba_radius`, default `None`: Integer radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float maximum slope for the Itakura parallelogram constraint.
- `n_jobs`, default `None`: Number of parallel jobs to run.
- `verbose`, default `0`: Verbosity level for logging.
- `be`, default `None`: Backend object for computation (e.g., NumPy or PyTorch).

### Input
Input data must be formatted as 3D arrays `(n_ts, sz, d)`. Use `tslearn.utils.to_time_series_dataset` to ensure input compatibility if data is provided as lists or 1D/2D arrays. Time series must share the same dimensionality `d`.

### Output
Returns a cross-similarity matrix of shape `(n_ts1, n_ts2)` containing the Frechet distances between all pairs of time series.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import cdist_frechet
from tslearn.utils import to_time_series_dataset

X = to_time_series_dataset([[1, 2, 3], [1, 2, 3, 4]])
dist_matrix = cdist_frechet(X)
```

### LLM Instruction Prompt
- Always ensure input datasets are 3D arrays using `to_time_series_dataset` before passing to `cdist_frechet`.
- If using custom backends, pass the backend object via the `be` parameter.
- When applying constraints, ensure `global_constraint` is explicitly set to match the provided `sakoe_chiba_radius` or `itakura_max_slope`.

### Prompt Snippet
```text
# Example of computing Frechet cross-similarity
from tslearn.metrics import cdist_frechet
dist_matrix = cdist_frechet(dataset1, dataset2=dataset2)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing time series with different dimensions `d` will raise a `ValueError`.
- **Invalid Shape:** Passing 1D or 2D arrays without converting them to the required 3D `(n_ts, sz, d)` format.
- **Constraint Conflict:** Providing both `sakoe_chiba_radius` and `itakura_max_slope` without a valid `global_constraint` may trigger warnings or unexpected behavior.

### Fix Code Hint
```python
# Ensure data is 3D before calling
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(my_data)
dist = cdist_frechet(X)
```

## API Test: `cdist_gak`

### Signature
```python
def cdist_gak(dataset1, dataset2=None, sigma=1.0, n_jobs=None, verbose=0, be=None)
```
_Source: tslearn/tslearn/metrics/_gak.py:289  (+1 more definition site/overload)_

_Source doc:_ Compute cross-similarity matrix using Global Alignment kernel (GAK). Note that GAK is a kernel, the larger GAK values mean more similar time series. GAK was originally presented in [1]_. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz1, d) or (n_ts1, sz1) or (sz1,) A dataset of time series. If shape is (n_ts1, sz1), the dataset is composed of univariate time series. If shape is (sz1,), the dataset is composed of a unique univariate time series. dataset2 : None or array-like, shape=(n_ts2, sz2, d) or (n_ts2, sz2) or (sz2,) (default: None) Another dataset of time series. If `None`, self-similarity of `dataset1` is returned. If shape is (n_ts2, sz2), the dataset is composed of univariate time series. If shape is (sz2,), the dataset is composed of a unique univariate time series. sigma : float (default 1.) Bandwidth of the internal gaussian kernel used for GAK n_jobs : int or None, optional (default=None) The number of jobs to run in parallel. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details. verbose : int, optional (default=0) The verbosity level: if non zero, progress messages are printed. Above 50, the output is sent to stdout. The frequency of the messages increases with the verbosity level. If it more than 10, all iterations are reported. `Glossary <https://joblib.readthedocs.io/en/latest/parallel.html#parallel-reference-documentation>`__ for more details. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- array-like, shape=(n_ts1, n_ts2) Cross-similarity matrix. Examples -------- >>> cdist_gak([[1, 2, 2, 3], [1., 2., 3., 4.]], sigma=2.) array([[1.        , 0.65629661], [0.65629661, 1.        ]]) >>> cdist_gak([[1, 2, 2], [1., 2., 3., 4.]], ...           [[1, 2, 2, 3], [1., 2., 3., 4.], [1, 2, 2, 3]], ...           sigma=2.) array([[0.71059484, 0.29722877, 0.71059484], [0.65629661, 1.        , 0.65629661]]) See Also -------- gak : Compute Global Alignment kernel References

### Goal
Computes the cross-similarity matrix between two datasets of time series using the Global Alignment Kernel (GAK), where higher values indicate greater similarity.

### Parameters
- `dataset1`: A dataset of time series with shape `(n_ts1, sz1, d)` or `(n_ts1, sz1)` or `(sz1,)`.
- `dataset2`, default `None`: A second dataset of time series with shape `(n_ts2, sz2, d)` or `(n_ts2, sz2)` or `(sz2,)`. If `None`, computes the self-similarity of `dataset1`.
- `sigma`, default `1.0`: The bandwidth parameter for the internal Gaussian kernel.
- `n_jobs`, default `None`: Number of parallel jobs to run.
- `verbose`, default `0`: Verbosity level for progress reporting.
- `be`, default `None`: Backend object or string ("numpy" or "pytorch") to control the computation engine.

### Input
Input data must be formatted as 3D arrays `(n_ts, max_sz, d)`. If providing 1D or 2D arrays, use `tslearn.utils.to_time_series_dataset` to ensure compatibility.

### Output
Returns an `array-like` object of shape `(n_ts1, n_ts2)` containing the pairwise GAK similarity scores.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Example 1: Self-similarity
X = np.array([[1, 2, 2, 3], [1., 2., 3., 4.]])
sim = tslearn.metrics.cdist_gak(X, sigma=2.0)

# Example 2: Cross-similarity
X1 = np.array([[1, 2, 2], [1., 2., 3., 4.]])
X2 = np.array([[1, 2, 2, 3], [1., 2., 3., 4.], [1, 2, 2, 3]])
sim = tslearn.metrics.cdist_gak(X1, X2, sigma=2.0)
```

### LLM Instruction Prompt
- Always ensure input datasets are 3D arrays. Use `tslearn.utils.to_time_series_dataset` if inputs are 1D or 2D.
- When using `sigma=0`, the function will raise a `ZeroDivisionError`.
- Specify the `be` parameter if you require specific backend behavior (e.g., PyTorch for autograd).

### Prompt Snippet
```text
# Ensure data is 3D before calling cdist_gak
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(my_data)
sim = tslearn.metrics.cdist_gak(X, sigma=2.0)
```

### Common Failure Modes
- `ZeroDivisionError`: Occurs if `sigma` is set to 0.
- `ValueError`: Occurs if input arrays are not properly formatted as 3D time-series datasets.
- Backend Mismatch: Passing arrays from different backends (e.g., mixing PyTorch tensors and NumPy arrays) without specifying the `be` parameter.

### Fix Code Hint
```python
# Ensure consistent backend and valid sigma
import tslearn.metrics
import numpy as np

# Use a positive sigma
sim = tslearn.metrics.cdist_gak(dataset1, dataset2, sigma=1.0)
```

## API Test: `cdist_normalized_cc`

### Signature
```python
def cdist_normalized_cc(dataset1, dataset2, norms1, norms2, self_similarity)
```
_Source: tslearn/tslearn/metrics/cycc.py:54_

_Source doc:_ Compute the distance matrix between two time series dataset. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz, d), dtype=float64 A dataset of time series. dataset2 : array-like, shape=(n_ts2, sz, d), dtype=float64 Another dataset of time series. norms1 : array-like, shape=(n_ts1,), dtype=float64 norms2 : array-like, shape=(n_ts2,), dtype=float64 self_similarity : bool Returns ------- dists : array-like, shape=(n_ts1, n_ts2), dtype=float64

### Goal
Computes a cross-distance matrix between two time-series datasets using normalized cross-correlation metrics.

### Parameters
- `dataset1`: A 3D array-like object of shape `(n_ts1, sz, d)` containing the first set of time series.
- `dataset2`: A 3D array-like object of shape `(n_ts2, sz, d)` containing the second set of time series.
- `norms1`: A 1D array-like object of shape `(n_ts1,)` representing the precomputed norms for `dataset1`.
- `norms2`: A 1D array-like object of shape `(n_ts2,)` representing the precomputed norms for `dataset2`.
- `self_similarity`: A boolean indicating whether the computation is a self-similarity matrix (used for optimization).

### Input
- **Data Format:** Must be 3D NumPy arrays of shape `(n_ts, sz, d)`. Use `tslearn.utils.to_time_series_dataset` to ensure input compatibility.
- **Preconditions:** The `norms` arrays must correspond to the L2 norms of the respective time series in `dataset1` and `dataset2`.

### Output
Returns a 2D NumPy array of shape `(n_ts1, n_ts2)` containing the computed distance matrix.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.cycc import cdist_normalized_cc

# Inferred usage pattern
n_ts1, n_ts2, sz, d = 2, 2, 4, 1
ds1 = np.random.rand(n_ts1, sz, d)
ds2 = np.random.rand(n_ts2, sz, d)
norms1 = np.linalg.norm(ds1, axis=(1, 2))
norms2 = np.linalg.norm(ds2, axis=(1, 2))

dists = cdist_normalized_cc(ds1, ds2, norms1, norms2, False)
print(f"Correctness witness: dists shape {dists.shape}")
```

### LLM Instruction Prompt
- Always ensure `dataset1` and `dataset2` are 3D arrays.
- Ensure `norms1` and `norms2` match the length of the first dimension of their respective datasets.
- Set `self_similarity` to `True` only if `dataset1` and `dataset2` are identical to optimize computation.

### Prompt Snippet
```text
from tslearn.metrics.cycc import cdist_normalized_cc
# Ensure inputs are 3D arrays and norms are precomputed
dists = cdist_normalized_cc(dataset1, dataset2, norms1, norms2, self_similarity=False)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will cause a dimension error.
- **Norm Mismatch:** Providing `norms` arrays that do not match the number of time series (`n_ts`) in the datasets will result in an index or broadcast error.
- **Dtype:** Inputs must be `float64` as per the implementation requirements.

### Fix Code Hint
```python
# Ensure data is 3D and norms are calculated correctly
from tslearn.utils import to_time_series_dataset
dataset = to_time_series_dataset(raw_data)
norms = np.linalg.norm(dataset, axis=(1, 2))
```

## API Test: `cdist_sax`

### Signature
```python
def cdist_sax(dataset1, breakpoints_avg, size_fitted, dataset2=None, n_jobs=None, verbose=0)
```
_Source: tslearn/tslearn/metrics/sax.py:10_

_Source doc:_ Calculates a matrix of distances (MINDIST) on SAX-transformed data, as presented in [1]_. It is important to note that this function expects the timeseries in dataset1 and dataset2 to be normalized to each have zero mean and unit variance. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz1, d) or (n_ts1, sz1) or (sz1,) A dataset of time series. If shape is (n_ts1, sz1), the dataset is composed of univariate time series. If shape is (sz1,), the dataset is composed of a unique univariate time series. breakpoints_avg : array-like, ndim=1 The breakpoints used to assign the alphabet symbols. size_fitted: int The original timesteps in the timeseries, before discretizing through SAX. dataset2 : None or array-like, shape=(n_ts2, sz2, d) or (n_ts2, sz2) or (sz2,) (default: None) Another dataset of time series. If `None`, self-similarity of `dataset1` is returned. If shape is (n_ts2, sz2), the dataset is composed of univariate time series. If shape is (sz2,), the dataset is composed of a unique univariate time series. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details. verbose : int, optional (default=0) The verbosity level: if non zero, progress messages are printed. Above 50, the output is sent to stdout. The frequency of the messages increases with the verbosity level. If it more than 10, all iterations are reported. `Glossary <https://joblib.readthedocs.io/en/latest/parallel.html#parallel-reference-documentation>`__ for more details. Returns ------- cdist : array-like, shape=(n_ts1, n_ts2) Cross-similarity matrix. References ---------- .. [1] Lin, Jessica, et al. "Experiencing SAX: a novel symbolic representation of time series." Data Mining and knowledge discovery 15.2 (2007): 107-144.

### Goal
Computes the MINDIST cross-similarity matrix between two datasets of time series using the Symbolic Aggregate approXimation (SAX) representation.

### Parameters
- `dataset1`: Array-like dataset of time series. Supports shapes `(n_ts1, sz1, d)`, `(n_ts1, sz1)`, or `(sz1,)`.
- `breakpoints_avg`: 1D array-like containing the breakpoints used for alphabet symbol assignment.
- `size_fitted`: Integer representing the original number of timesteps in the time series before SAX discretization.
- `dataset2`, default `None`: Optional second dataset of time series. If `None`, computes the self-similarity matrix of `dataset1`.
- `n_jobs`, default `None`: Number of parallel jobs to run. `None` defaults to 1; `-1` uses all available processors.
- `verbose`, default `0`: Verbosity level for progress reporting.

### Input
- **Precondition:** All time series in `dataset1` and `dataset2` must be normalized to have zero mean and unit variance prior to calling this function.
- **Format:** Input data should be provided as NumPy arrays or list-like structures compatible with `tslearn`'s internal time-series representation.

### Output
Returns a cross-similarity matrix of shape `(n_ts1, n_ts2)` containing the MINDIST values.

### Valid Call Patterns
```python
import numpy as np
import tslearn.metrics

# Self-similarity example
dists = tslearn.metrics.cdist_sax(
    [[-1, 0, 1], [1, 0, 1]],
    [-0.5, 0., 0.5],
    3,
    dataset2=[[-1, 0, 1], [1, 0, 1]]
)
print(f"Correctness witness: {dists.shape == (2, 2)}")
```

### LLM Instruction Prompt
- Always ensure input time series are normalized (zero mean, unit variance) before passing them to `cdist_sax`.
- Provide `size_fitted` as the original length of the time series before SAX transformation.
- Use `dataset2=None` if calculating the self-similarity matrix of a single dataset.

### Prompt Snippet
```text
tslearn.metrics.cdist_sax(dataset1, breakpoints_avg, size_fitted, dataset2=None, n_jobs=None, verbose=0)
```

### Common Failure Modes
- **Normalization Error:** Failing to normalize input time series to zero mean and unit variance will result in incorrect MINDIST calculations.
- **Shape Mismatch:** Providing `size_fitted` that does not match the original time series length can lead to invalid symbolic representation mapping.
- **Data Format:** Passing non-normalized or non-numeric data structures that cannot be interpreted as time series.

### Fix Code Hint
```python
from tslearn.preprocessing import TimeSeriesScalerMeanVariance
# Ensure data is normalized before SAX distance calculation
scaler = TimeSeriesScalerMeanVariance()
X_scaled = scaler.fit_transform(X)
dists = tslearn.metrics.cdist_sax(X_scaled, breakpoints, original_size)
```

## API Test: `cdist_soft_dtw`

### Signature
```python
def cdist_soft_dtw(dataset1, dataset2=None, gamma=1.0, be=None, compute_with_backend=False)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:765_

_Source doc:_ Compute cross-similarity matrix using Soft-DTW metric. Soft-DTW was originally presented in [1]_ and is discussed in more details in our :ref:`user-guide page on DTW and its variants<dtw>`. Soft-DTW is computed as: .. math:: \text{soft-DTW}_{\gamma}(X, Y) = \min_{\pi}{}^\gamma \sum_{(i, j) \in \pi} \|X_i, Y_j\|^2 where :math:`\min^\gamma` is the soft-min operator of parameter :math:`\gamma`. In the limit case :math:`\gamma = 0`, :math:`\min^\gamma` reduces to a hard-min operator and soft-DTW is defined as the square of the DTW similarity measure. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz1, d) or (n_ts1, sz1) or (sz1,) A dataset of time series. If shape is (n_ts1, sz1), the dataset is composed of univariate time series. If shape is (sz1,), the dataset is composed of a unique univariate time series. dataset2 : None or array-like, shape=(n_ts2, sz2, d) or (n_ts2, sz2) or (sz2,) (default: None) Another dataset of time series. If `None`, self-similarity of `dataset1` is returned. If shape is (n_ts2, sz2), the dataset is composed of univariate time series. If shape is (sz2,), the dataset is composed of a unique univariate time series. gamma : float (default 1.) Gamma parameter for Soft-DTW. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. compute_with_backend : bool, default=False This parameter has no influence when the NumPy backend is used. When a backend different from NumPy is used (cf parameter `be`): If `True`, the computation is done with the corresponding backend. If `False`, a conversion to the NumPy backend can be used to accelerate the computation. Returns ------- array-like, shape=(n_ts1, n_ts2) Cross-similarity matrix. Examples -------- >>> cdist_soft_dtw([[1, 2, 2, 3], [1., 2., 3., 4.]], gamma=.01) array([[-0.01098612,  1.        ], [ 1.        ,  0.        ]]) >>> cdist_soft_dtw([[1, 2, 2, 3], [1., 2., 3., 4.]], ...                [[1, 2, 2, 3], [1., 2., 3., 4.]], gamma=.01) array([[-0.01098612,  1.        ], [ 1.        ,  0.        ]])

### Goal
Computes the cross-similarity matrix between two datasets of time series using the differentiable Soft-DTW metric.

### Parameters
- `dataset1`: Array-like dataset of time series with shape `(n_ts1, sz1, d)` or `(n_ts1, sz1)` or `(sz1,)`.
- `dataset2`, default `None`: Optional second dataset of time series with shape `(n_ts2, sz2, d)` or `(n_ts2, sz2)` or `(sz2,)`. If `None`, computes self-similarity of `dataset1`.
- `gamma`, default `1.0`: Smoothing parameter for the soft-min operator; lower values approach the standard DTW squared distance.
- `be`, default `None`: Backend engine to use (`"numpy"` or `"pytorch"`). If `None`, inferred from input array types.
- `compute_with_backend`, default `False`: If `True` and using a non-NumPy backend, performs computation within that backend; otherwise, may convert to NumPy for acceleration.

### Input
Input data must be formatted as 3D arrays `(n_ts, sz, d)`. Use `tslearn.utils.to_time_series_dataset` to ensure compatibility if inputs are 1D or 2D lists/arrays.

### Output
Returns an `array-like` object of shape `(n_ts1, n_ts2)` containing the Soft-DTW similarity values.

### Valid Call Patterns
```python
from tslearn.metrics import cdist_soft_dtw
import numpy as np

# Basic usage with default gamma
data = np.random.rand(5, 10, 1)
sim_matrix = cdist_soft_dtw(data)

# Usage with explicit gamma and second dataset
data2 = np.random.rand(3, 10, 1)
sim_matrix = cdist_soft_dtw(data, data2, gamma=0.1)
```

### LLM Instruction Prompt
- Always ensure input datasets are 3D arrays. If providing lists, wrap them in `tslearn.utils.to_time_series_dataset`.
- When using `PyTorch` tensors, explicitly set `be="pytorch"` to enable automatic differentiation.
- The output is a square matrix if `dataset2` is `None`, or a rectangular matrix of shape `(n_ts1, n_ts2)`.

### Prompt Snippet
```text
# Example of computing Soft-DTW cross-similarity
from tslearn.metrics import cdist_soft_dtw
matrix = cdist_soft_dtw(dataset1, dataset2=dataset2, gamma=0.5)
```

### Common Failure Modes
- Passing 1D or 2D arrays directly without using `to_time_series_dataset` will cause shape mismatch errors.
- Setting `gamma=0` may lead to numerical instability or errors depending on the backend implementation.
- Mismatched feature dimensions `d` between `dataset1` and `dataset2` will raise a `ValueError`.

### Fix Code Hint
```python
# Ensure data is 3D before calling
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(my_data)
result = cdist_soft_dtw(X, gamma=0.1)
print(f"Correctness witness: Matrix shape is {result.shape}")
```

## API Test: `cdist_soft_dtw_normalized`

### Signature
```python
def cdist_soft_dtw_normalized(dataset1, dataset2=None, gamma=1.0, be=None, compute_with_backend=False)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:911_

_Source doc:_ Compute cross-similarity matrix using a normalized version of the Soft-DTW metric. Soft-DTW was originally presented in [1]_ and is discussed in more details in our :ref:`user-guide page on DTW and its variants<dtw>`. Soft-DTW is computed as: .. math:: \text{soft-DTW}_{\gamma}(X, Y) = \min_{\pi}{}^\gamma \sum_{(i, j) \in \pi} \|X_i, Y_j\|^2 where :math:`\min^\gamma` is the soft-min operator of parameter :math:`\gamma`. In the limit case :math:`\gamma = 0`, :math:`\min^\gamma` reduces to a hard-min operator and soft-DTW is defined as the square of the DTW similarity measure. This normalized version is defined as: .. math:: \text{norm-soft-DTW}_{\gamma}(X, Y) = \text{soft-DTW}_{\gamma}(X, Y) - \frac{1}{2} \left(\text{soft-DTW}_{\gamma}(X, X) + \text{soft-DTW}_{\gamma}(Y, Y)\right) and ensures that all returned values are positive and that :math:`\text{norm-soft-DTW}_{\gamma}(X, X) = 0`. Parameters ---------- dataset1 : array-like, shape=(n_ts1, sz1, d) or (n_ts1, sz1) or (sz1,) A dataset of time series. If shape is (n_ts1, sz1), the dataset is composed of univariate time series. If shape is (sz1,), the dataset is composed of a unique univariate time series. dataset2 : None or array-like, shape=(n_ts2, sz2, d) or (n_ts2, sz2) or (sz2,) (default: None) Another dataset of time series. If `None`, self-similarity of `dataset1` is returned. If shape is (n_ts2, sz2), the dataset is composed of univariate time series. If shape is (sz2,), the dataset is composed of a unique univariate time series. gamma : float (default 1.) Gamma parameter for Soft-DTW. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. compute_with_backend : bool, default=False This parameter has no influence when the NumPy backend is used. When a backend different from NumPy is used (cf parameter `be`): If `True`, the computation is done with the corresponding backend. If `False`, a conversion to the NumPy backend can be used to accelerate the computation. Returns

### Goal
Computes a cross-similarity matrix between two datasets of time series using the normalized Soft-DTW metric, ensuring non-negative values and zero self-similarity.

### Parameters
- `dataset1`: A dataset of time series with shape `(n_ts1, sz1, d)` or `(n_ts1, sz1)` or `(sz1,)`.
- `dataset2`, default `None`: A second dataset of time series with shape `(n_ts2, sz2, d)` or `(n_ts2, sz2)` or `(sz2,)`. If `None`, computes self-similarity of `dataset1`.
- `gamma`, default `1.0`: The smoothing parameter for the Soft-DTW operator.
- `be`, default `None`: The backend engine to use (`"numpy"` or `"pytorch"`). If `None`, inferred from input array types.
- `compute_with_backend`, default `False`: If `True`, forces computation within the specified non-NumPy backend; otherwise, may convert to NumPy for acceleration.

### Input
Input data must be 3D arrays `(n_ts, sz, d)`. If providing 1D or 2D arrays, use `tslearn.utils.to_time_series_dataset` to ensure compatibility with the expected 3D format.

### Output
Returns a 2D array of shape `(n_ts1, n_ts2)` containing the normalized Soft-DTW distances between all pairs of time series in the provided datasets.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np
dataset = np.random.rand(5, 10, 2)
dist_matrix = tslearn.metrics.cdist_soft_dtw_normalized(dataset, gamma=0.5)
print(f"Correctness witness: {dist_matrix.shape == (5, 5)}")
```

### LLM Instruction Prompt
- Always ensure input datasets are 3D arrays. Use `tslearn.utils.to_time_series_dataset` if inputs are 1D or 2D.
- When using `PyTorch` tensors, explicitly set `be="pytorch"` or ensure the backend is correctly instantiated.
- The output is a square matrix if `dataset2` is `None`, or a rectangular matrix of shape `(n_ts1, n_ts2)` otherwise.

### Prompt Snippet
```text
tslearn.metrics.cdist_soft_dtw_normalized(dataset1, dataset2=None, gamma=1.0, be=None)
```

### Common Failure Modes
- Passing 1D or 2D arrays directly without conversion to 3D `(n_ts, sz, d)` format.
- Mismatched dimensions between `dataset1` and `dataset2` in the feature dimension `d`.
- Attempting to use `PyTorch` backend features without the `pytorch` package installed.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure 3D shape
X = to_time_series_dataset(my_data) 
dist = tslearn.metrics.cdist_soft_dtw_normalized(X, gamma=1.0)
```

## API Test: `check_dataset`

### Signature
```python
def check_dataset(X, force_univariate=False, force_equal_length=False, force_single_time_series=False)
```
_Source: tslearn/tslearn/utils/utils.py:548_

_Source doc:_ Check if X is a valid tslearn dataset, with possibly additional extra constraints. Parameters ---------- X: array-like, shape=(n_ts, sz, d) Time series dataset. force_univariate: bool (default: False) If True, only univariate datasets are considered valid. force_equal_length: bool (default: False) If True, only equal-length datasets are considered valid. force_single_time_series: bool (default: False) If True, only datasets made of a single time series are considered valid. Returns ------- array-like, shape=(n_ts, sz, d) Formatted dataset, if it is valid Raises ------ ValueError Raised if X is not a valid dataset, or one of the constraints is not satisfied. Examples -------- >>> X = [[1, 2, 3], [1, 2, 3, 4]] >>> X_new = check_dataset(X) >>> X_new.shape (2, 4, 1) >>> check_dataset( ...     X, ...     force_equal_length=True ... )  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: All the time series in the array should be of equal lengths. >>> other_X = numpy.random.randn(3, 10, 2) >>> check_dataset( ...     other_X, ...     force_univariate=True ... )  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: Array should be univariate and is of shape: (3, 10, 2) >>> other_X = numpy.random.randn(3, 10, 2) >>> check_dataset( ...     other_X, ...     force_single_time_series=True ... )  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: Array should be made of a single time series (3 here)

### Goal
Validates that a given input array conforms to the `tslearn` 3D time-series dataset structure `(n_ts, sz, d)` and enforces specific structural constraints if requested.

### Parameters
- `X`: The input dataset to validate, expected as an array-like structure.
- `force_univariate`, default `False`: If `True`, raises a `ValueError` if the dataset contains more than one dimension (`d > 1`).
- `force_equal_length`, default `False`: If `True`, raises a `ValueError` if the time series in the dataset have varying lengths (`sz`).
- `force_single_time_series`, default `False`: If `True`, raises a `ValueError` if the dataset contains more than one time series (`n_ts > 1`).

### Input
The input `X` must be convertible to a 3D NumPy array of shape `(n_ts, sz, d)`. If the input is not already in this format, it should be pre-processed using `tslearn.utils.to_time_series_dataset` before validation.

### Output
Returns an `array-like` object of shape `(n_ts, sz, d)` representing the validated and formatted dataset.

### Valid Call Patterns
```python
from tslearn.utils import check_dataset
import numpy as np

# Basic validation
X = np.random.rand(5, 10, 1)
X_valid = check_dataset(X)

# Enforcing constraints
X_equal = check_dataset(X, force_equal_length=True)
```

### LLM Instruction Prompt
When calling `check_dataset`, ensure the input `X` is already in a 3D-compatible format. If the input is a list of lists or a 2D array, use `tslearn.utils.to_time_series_dataset` first. Use the `force_*` parameters to strictly validate data before passing it to `tslearn` estimators.

### Prompt Snippet
```text
from tslearn.utils import check_dataset
# Ensure X is 3D (n_ts, sz, d) before calling
X_checked = check_dataset(X, force_univariate=True)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing a 1D or 2D array without prior conversion via `to_time_series_dataset` will cause a `ValueError`.
- **Constraint Violation:** Passing a dataset with variable lengths when `force_equal_length=True` is set.
- **Multivariate Data:** Passing multivariate data (`d > 1`) when `force_univariate=True` is set.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import check_dataset

# Deterministic check
data = np.array([[[1], [2]], [[1], [2]]])
try:
    checked = check_dataset(data, force_equal_length=True)
    print(f"Correctness witness: {checked.shape == (2, 2, 1)}")
except ValueError as e:
    print(f"Validation failed: {e}")
```

## API Test: `check_dims`

### Signature
```python
def check_dims(X, X_fit_dims=None, extend=True, check_n_features_only=False)
```
_Source: tslearn/tslearn/utils/utils.py:65_

_Source doc:_ Reshapes X to a 3-dimensional array of X.shape[0] univariate timeseries of length X.shape[1] if X is 2-dimensional and extend is True. Then checks whether the provided X_fit_dims and the dimensions of X (except for the first one), match. Parameters ---------- X : array-like The first array to be compared. X_fit_dims : tuple (default: None) The dimensions of the data generated by fit, to compare with the dimensions of the provided array X. If None, then only perform reshaping of X, if necessary. extend : boolean (default: True) Whether to reshape X, if it is 2-dimensional. check_n_features_only: boolean (default: False) Returns ------- array Reshaped X array Examples -------- >>> X = numpy.empty((10, 3)) >>> check_dims(X).shape (10, 3, 1) >>> X = numpy.empty((10, 3, 1)) >>> check_dims(X).shape (10, 3, 1) >>> X_fit_dims = (5, 3, 1) >>> check_dims(X, X_fit_dims).shape (10, 3, 1) >>> X_fit_dims = (5, 3, 2) >>> check_dims(X, X_fit_dims)  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ValueError: Dimensions (except first) must match! ((5, 3, 2) and (10, 3, 1) are passed shapes) >>> X_fit_dims = (5, 5, 1) >>> check_dims(X, X_fit_dims, check_n_features_only=True).shape (10, 3, 1) >>> X_fit_dims = (5, 5, 2) >>> check_dims( ...     X, ...     X_fit_dims, ...     check_n_features_only=True ... )  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ValueError: Number of features of the provided timeseries must match! (last dimension) must match the one of the fitted data! ((5, 5, 2) and (10, 3, 1) are passed shapes) Raises ------ ValueError Will raise exception if X is None or (if X_fit_dims is provided) one of the dimensions of the provided data, except the first, does not match X_fit_dims.

### Goal
Ensures time-series data conforms to the expected 3D shape `(n_ts, sz, d)` required by `tslearn` estimators and validates dimensional consistency against previously fitted model parameters.

### Parameters
- `X`: The input time-series dataset (array-like).
- `X_fit_dims`, default `None`: A tuple representing the expected dimensions `(n_ts, sz, d)` of the data used during model fitting.
- `extend`, default `True`: If `True`, automatically reshapes 2D input arrays `(n_ts, sz)` into 3D `(n_ts, sz, 1)`.
- `check_n_features_only`, default `False`: If `True`, only validates that the number of features (the last dimension) matches `X_fit_dims`, ignoring time-series length mismatches.

### Input
- Expects `numpy` arrays or array-like structures.
- If `X` is 2D, `extend=True` is required to convert it to the standard 3D `tslearn` format.
- If `X_fit_dims` is provided, the input `X` must have matching dimensions (excluding the first dimension, which represents the number of samples).

### Output
Returns a 3D `numpy` array of shape `(n_ts, sz, d)` containing the reshaped time-series data.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import check_dims

# Standard 2D to 3D conversion
X = np.empty((10, 3))
X_3d = check_dims(X)
print(f"Witness: {X_3d.shape}") # (10, 3, 1)

# Validation against fit dimensions
X_fit = (5, 3, 1)
X_validated = check_dims(X, X_fit_dims=X_fit)
print(f"Witness: {X_validated.shape}") # (10, 3, 1)
```

### LLM Instruction Prompt
- Always use `tslearn.utils.check_dims` to sanitize input data before passing it to estimators if the input source is potentially 2D.
- When validating against a fitted model, pass the model's `input_dims_` (or equivalent) as `X_fit_dims`.
- Set `check_n_features_only=True` only when the time-series length is expected to vary but the feature dimensionality must remain constant.

### Prompt Snippet
```text
from tslearn.utils import check_dims
# Ensure X is 3D and matches model expectations
X_clean = check_dims(X, X_fit_dims=model.input_dims_)
```

### Common Failure Modes
- `ValueError`: Raised if `X` is `None`.
- `ValueError`: Raised if `X_fit_dims` is provided and the dimensions (excluding the first) do not match the input `X`.
- `ValueError`: Raised if `check_n_features_only=True` and the last dimension of `X` does not match `X_fit_dims`.

### Fix Code Hint
```python
# If you encounter a shape mismatch error, ensure your input is 3D:
# (n_samples, time_steps, n_features)
X = X.reshape(X.shape[0], X.shape[1], 1) if X.ndim == 2 else X
```

## API Test: `check_equal_size`

### Signature
```python
def check_equal_size(dataset, be=None)
```
_Source: tslearn/tslearn/utils/utils.py:425_

_Source doc:_ Check if all time series in the dataset have the same size. Parameters ---------- dataset: array-like The dataset to check. Returns ------- bool Whether all time series in the dataset have the same size. Examples -------- >>> check_equal_size([[1, 2, 3], [4, 5, 6], [5, 3, 2]]) True >>> check_equal_size([[1, 2, 3, 4], [4, 5, 6], [5, 3, 2]]) False >>> check_equal_size([]) True

### Goal
Determines whether all time series within a provided dataset share the same temporal length (number of time steps).

### Parameters
- `dataset`: array-like. The collection of time series to evaluate.
- `be`, default `None`: Backend object used for computation. If `None`, the default backend is used.

### Input
- **Data Format:** Accepts list-of-lists or array-like structures representing time series.
- **Preconditions:** While it accepts raw lists, it is recommended to pass data formatted as a 3D array `(n_ts, max_sz, d)` using `tslearn.utils.to_time_series_dataset` for consistency with other `tslearn` estimators.
- **Variable Length:** The function is specifically designed to detect if the dataset contains variable-length time series, which are supported by `tslearn` but require specific handling in downstream estimators.

### Output
Returns `bool`: `True` if all time series in the dataset have the same length, `False` otherwise. An empty dataset returns `True`.

### Valid Call Patterns
```python
import tslearn.utils
# Check a uniform dataset
assert tslearn.utils.check_equal_size([[0], [0]]) == True
# Check a variable-length dataset
assert tslearn.utils.check_equal_size([[0], [0, 0]]) == False
# Check an empty dataset
assert tslearn.utils.check_equal_size([]) == True
```

### LLM Instruction Prompt
- Use `tslearn.utils.check_equal_size` to validate dataset homogeneity before passing data to estimators that require fixed-length inputs.
- Always import the function from `tslearn.utils`.
- Do not assume the dataset is uniform; use this function to branch logic for variable-length handling.

### Prompt Snippet
```text
from tslearn.utils import check_equal_size
is_uniform = check_equal_size(X)
```

### Common Failure Modes
- **Input Type Mismatch:** Passing non-array-like objects (e.g., scalar values or nested dictionaries) will raise a `TypeError` or `ValueError` during internal iteration.
- **Backend Incompatibility:** Passing an unsupported backend object to `be` may result in an `AttributeError` if the backend does not implement the required size-checking logic.
- **Dimensionality Confusion:** Forgetting that `tslearn` expects 3D arrays; if the input is a 2D array `(n_ts, sz)`, the function checks the length of the second dimension, but it is safer to ensure the data is pre-processed via `to_time_series_dataset`.

### Fix Code Hint
```python
import tslearn.utils
# Ensure data is in the correct format before checking
X_formatted = tslearn.utils.to_time_series_dataset(my_data)
if not tslearn.utils.check_equal_size(X_formatted):
    # Handle variable length data (e.g., use estimators that support it)
    print("Dataset contains variable-length time series.")
```

## API Test: `check_keras_backend`

### Signature
```python
def check_keras_backend()
```
_Source: tslearn/tslearn/backend/__init__.py:12_

_Source doc:_ Select a backend based on installed packages when none is explicitly selected.

### Goal
Automatically detects and selects the appropriate deep learning backend (e.g., TensorFlow/Keras) based on the current environment's installed packages.

### Parameters
_None._

### Input
No input parameters are required. The function inspects the Python environment's installed packages to determine the available backend.

### Output
Returns `unspecified` — The function performs an internal configuration update to the `tslearn` backend state; it does not return a value to the caller.

### Valid Call Patterns
```python
from tslearn.backend import check_keras_backend

# Inferred from signature:
check_keras_backend()
```

### LLM Instruction Prompt
- Call `check_keras_backend()` when initializing deep learning-based components if no specific backend has been configured.
- Do not expect a return value; the function operates by side-effect on the internal `tslearn` configuration.

### Prompt Snippet
```text
from tslearn.backend import check_keras_backend
check_keras_backend()
```

### Common Failure Modes
- **Missing Dependencies:** If neither TensorFlow nor other supported Keras-compatible backends are installed, the function may raise an `ImportError` or `RuntimeError` depending on the internal implementation.
- **Environment Conflicts:** If multiple backend-compatible packages are installed, the function may select one based on internal priority logic that might not match the user's intent.

### Fix Code Hint
```python
# Ensure the required backend package (e.g., tensorflow) is installed
# in the environment before calling the check.
try:
    from tslearn.backend import check_keras_backend
    check_keras_backend()
    print("Backend check completed successfully.")
except Exception as e:
    print(f"Backend check failed: {e}")
```

## API Test: `check_variable_length_input`

### Signature
```python
def check_variable_length_input(X)
```
_Source: tslearn/tslearn/utils/utils.py:39_

_Source doc:_ Input validation on a variable length dataset. Parameters ---------- X : object Input object to check / convert. Returns ------- array_converted : object The converted and validated array.

### Goal
Validates and converts input datasets containing time series of varying lengths into a standardized format suitable for `tslearn` estimators.

### Parameters
- `X`: The input object (typically a list of arrays or a 3D array) representing a time-series dataset to be validated or converted.

### Input
- **Data Format:** Accepts lists of time series (where each series may have a different length) or 3D NumPy arrays.
- **Preconditions:** If the input is not already in a compatible 3D structure, it should ideally be processed via `tslearn.utils.to_time_series_dataset` prior to validation to ensure the internal representation matches `tslearn` expectations.

### Output
Returns `array_converted` — A validated and potentially converted object representing the time-series dataset, formatted for internal `tslearn` processing.

### Valid Call Patterns
```python
from tslearn.utils import check_variable_length_input, to_time_series_dataset
import numpy as np

# Example: Validating a list of variable-length sequences
data = [[1, 2, 3], [1, 2]]
X = to_time_series_dataset(data)
validated_X = check_variable_length_input(X)
print(f"Validated shape: {validated_X.shape}")
# Witness: Validated shape: (2, 3, 1)
```

### LLM Instruction Prompt
- Use `check_variable_length_input` when you need to ensure that a dataset containing variable-length time series is correctly formatted for `tslearn` algorithms.
- Always ensure the input has been passed through `to_time_series_dataset` if it originates from raw Python lists or non-standard array formats.

### Prompt Snippet
```text
# Ensure variable length data is validated before estimator fitting
X_validated = check_variable_length_input(X_dataset)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing a 1D or 2D array that has not been converted to the required 3D `(n_ts, max_sz, d)` format will cause validation errors.
- **Non-numeric Data:** Passing non-numeric sequences will result in conversion failures during the internal validation process.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset, check_variable_length_input

# If validation fails, ensure the data is converted to 3D first
X_formatted = to_time_series_dataset(raw_data)
X_validated = check_variable_length_input(X_formatted)
```

## API Test: `classes_`

### Signature
```python
def classes_(self)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:124_

### Goal
Retrieves the unique class labels identified during the fitting process of a time-series classification estimator.

### Parameters
- `self`: The fitted estimator instance (e.g., `NonMyopicEarlyClassifier`) that contains the learned class labels.

### Input
- **Preconditions:** The estimator must have been successfully trained using the `fit(X, y)` method.
- **Data:** The underlying `y` array provided during `fit` must contain the target labels.

### Output
Returns `unspecified` — Typically returns an array-like object containing the unique class labels found in the training data, or `None` if the estimator has not been fitted.

### Valid Call Patterns
```python
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.neighbors import KNeighborsTimeSeriesClassifier

# Initialize and fit the model
model = NonMyopicEarlyClassifier(
    n_clusters=3,
    base_classifier=KNeighborsTimeSeriesClassifier(n_neighbors=1)
)
# Before fit, classes_ is None
assert model.classes_ is None

# After fit, classes_ is populated
model.fit(dataset, y)
print(model.classes_)
```

### LLM Instruction Prompt
- Always check if the estimator has been fitted (e.g., `model.classes_ is not None`) before accessing the attribute to avoid unexpected `None` values.
- Use this attribute to inspect the label space learned by `tslearn` classifiers.

### Prompt Snippet
```text
# Accessing the learned classes from a fitted tslearn classifier
labels = model.classes_
```

### Common Failure Modes
- **Accessing before `fit`:** Calling `model.classes_` before the `fit` method has been executed will return `None`, which may cause downstream errors if the code expects an array of labels.
- **Incompatible Estimators:** This attribute is specific to estimators that perform classification; attempting to access it on non-classification estimators (like certain clustering models) may result in an `AttributeError`.

### Fix Code Hint
```python
# Ensure the model is fitted before accessing classes_
if model.classes_ is not None:
    print(f"Learned classes: {model.classes_}")
else:
    print("Model has not been fitted yet.")
```

## API Test: `coef_`

### Signature
```python
def coef_(self)
```
_Source: tslearn/tslearn/svm/svm.py:34_

### Goal
Retrieves the weights assigned to the features (coefficients) for linear models, specifically used in `TimeSeriesSVC` to inspect the importance of time-series features after fitting.

### Parameters
- `self`: The fitted `TimeSeriesSVC` estimator instance.

### Input
- **Preconditions:** The estimator must have been successfully trained using the `fit()` method on a 3D NumPy array of shape `(n_ts, max_sz, d)`.
- **Compatibility:** This attribute is only available when the `kernel` parameter of the `TimeSeriesSVC` is set to `'linear'`.

### Output
Returns `unspecified` — Typically returns a NumPy array of shape `(n_features,)` or `(n_classes, n_features)` representing the coefficients of the features in the decision function.

### Valid Call Patterns
```python
from tslearn.svm import TimeSeriesSVC
# Assuming X and y are prepared
clf = TimeSeriesSVC(kernel="linear")
clf.fit(X, y)
weights = clf.coef_
```

### LLM Instruction Prompt
- Only access `coef_` after ensuring the `TimeSeriesSVC` estimator has been fitted.
- Verify that the `kernel` parameter is set to `'linear'`, as `coef_` is not defined for non-linear kernels (e.g., `'rbf'`, `'poly'`).

### Prompt Snippet
```text
# Ensure the model is fitted and uses a linear kernel before accessing coef_
if hasattr(model, "coef_"):
    weights = model.coef_
```

### Common Failure Modes
- **AttributeError:** Occurs if `coef_` is accessed before calling `fit()`.
- **AttributeError:** Occurs if the `TimeSeriesSVC` is initialized with a non-linear kernel, as `coef_` is not computed for those kernels.
- **Shape Mismatch:** If the input data was not properly formatted as a 3D array via `to_time_series_dataset`, the underlying SVM may fail to fit, rendering `coef_` inaccessible.

### Fix Code Hint
```python
# Check kernel type and fit status
if clf.kernel == 'linear':
    try:
        print(clf.coef_)
    except AttributeError:
        print("Model must be fitted before accessing coef_.")
else:
    print("coef_ is only available for linear kernels.")
```

## API Test: `compute`

### Signature
```python
def compute(self)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:1111  (+1 more definition site/overload)_

_Source doc:_ Compute soft-DTW by dynamic programming. Returns ------- sdtw: float soft-DTW discrepancy.

### Goal
Computes the soft-DTW discrepancy between two time series using dynamic programming as implemented within a soft-DTW object instance.

### Parameters
- `self`: An instance of a soft-DTW estimator or computation class that has been initialized with the time series data and parameters (such as gamma) required for the discrepancy calculation.

### Input
- **Data:** The instance must be initialized with time series data, typically provided as 3D NumPy arrays of shape `(n_ts, max_sz, d)`.
- **Preconditions:** The object must be in a state where the internal time series buffers are populated. This method is typically called internally by the object's `__call__` or `fit` methods.

### Output
Returns `float` — The computed soft-DTW discrepancy value.

### Valid Call Patterns
```python
# Example inferred from signature and soft-DTW implementation patterns
from tslearn.metrics import SoftDTW
sdtw_obj = SoftDTW(dataset=X, gamma=1.0)
result = sdtw_obj.compute()
```

### LLM Instruction Prompt
- When using `compute`, ensure the parent object (e.g., `SoftDTW`) has been correctly instantiated with the target time-series data.
- Do not call `compute` directly if the higher-level `__call__` interface is available, as `compute` is a low-level dynamic programming routine.

### Prompt Snippet
```text
# Ensure the SoftDTW object is initialized before calling compute()
sdtw_instance = SoftDTW(dataset=my_data, gamma=0.1)
discrepancy = sdtw_instance.compute()
```

### Common Failure Modes
- **Uninitialized State:** Calling `compute` on an object that has not been provided with valid time-series data will result in an `AttributeError` or `ValueError` due to missing internal buffers.
- **Backend Mismatch:** If the object was initialized with a specific backend (e.g., `pytorch`), ensure the environment supports that backend, otherwise, the dynamic programming routine may fail during execution.

### Fix Code Hint
```python
# Ensure data is 3D and the object is properly initialized
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset([[1, 2, 3], [1, 2, 4]])
sdtw = SoftDTW(dataset=X, gamma=0.1)
# If compute() fails, verify the input shape and backend compatibility
val = sdtw.compute()
```

## API Test: `compute_mask`

### Signature
```python
def compute_mask(s1, s2, global_constraint=0, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
def compute_mask(self, inputs, mask=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1770  (+2 more definition site/overload)_

_Source doc:_ Compute the mask (region constraint). Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) or int A time series or integer. If shape is (sz1,), the time series is assumed to be univariate. If int, size sz1 used to dimension the mask. s2 : array-like, shape=(sz2, d) or (sz2,) or int Another time series or integer. If shape is (sz2,), the time series is assumed to be univariate. If int, size sz2 used to dimension the mask. global_constraint : {0, 1, 2} (default: 0) Global constraint to restrict admissible paths for DTW: - "itakura" if 1 - "sakoe_chiba" if 2 - no constraint otherwise sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to 2 (sakoe-chiba), a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to 1 (itakura), a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- mask : array-like, shape=(sz1, sz2) Constraint region.

### Goal
Computes a boolean mask representing the admissible alignment region for Dynamic Time Warping (DTW) between two time series, optionally applying Sakoe-Chiba or Itakura global constraints.

### Parameters
- `s1`: A time series (array-like, shape `(sz1, d)` or `(sz1,)`) or an integer representing the size `sz1` of the first time series.
- `s2`: A time series (array-like, shape `(sz2, d)` or `(sz2,)`) or an integer representing the size `sz2` of the second time series.
- `global_constraint`, default `0`: Integer flag for the constraint type: `0` (none), `1` (Itakura), or `2` (Sakoe-Chiba).
- `sakoe_chiba_radius`, default `None`: Integer radius for the Sakoe-Chiba band. Defaults to `1` if `global_constraint` is `2`.
- `itakura_max_slope`, default `None`: Float maximum slope for the Itakura parallelogram. Defaults to `2.0` if `global_constraint` is `1`.
- `be`, default `None`: Backend object or string (`"numpy"` or `"pytorch"`) to perform the computation.

### Input
- Accepts either raw time series arrays or integers representing sequence lengths.
- If using `PyTorch` backend, inputs should be `torch.Tensor` objects.
- If using `NumPy` backend, inputs should be `numpy.ndarray` objects.

### Output
Returns a boolean `array-like` of shape `(sz1, sz2)` where `True` indicates an admissible path cell and `False` indicates a restricted cell.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Basic usage with integer dimensions
mask = tslearn.metrics.compute_mask(4, 4)

# Usage with Sakoe-Chiba constraint
mask_sc = tslearn.metrics.compute_mask(4, 4, global_constraint=2, sakoe_chiba_radius=1)

# Usage with explicit backend
mask_np = tslearn.metrics.compute_mask([1, 2, 3], [1, 2, 2, 3], be="numpy")
```

### LLM Instruction Prompt
- Always specify the `global_constraint` if a specific alignment restriction is required.
- If `sakoe_chiba_radius` or `itakura_max_slope` are provided, ensure `global_constraint` is set to the corresponding integer (`2` or `1` respectively) to avoid `RuntimeWarning`.
- Use the `be` parameter to explicitly match the backend of your input tensors (e.g., `"pytorch"` for `torch.Tensor`).

### Prompt Snippet
```text
tslearn.metrics.compute_mask(s1, s2, global_constraint=2, sakoe_chiba_radius=1, be="numpy")
```

### Common Failure Modes
- Providing both `sakoe_chiba_radius` and `itakura_max_slope` while `global_constraint` is `0`, which triggers a `RuntimeWarning`.
- Mismatching the `be` (backend) parameter with the input array types (e.g., passing `torch.Tensor` to a `NumPyBackend`).
- Assuming the output is a distance matrix; it is a boolean mask for path restriction.

### Fix Code Hint
```python
# Ensure backend matches input type
import tslearn.metrics
import torch
s1, s2 = torch.tensor([1, 2]), torch.tensor([1, 2, 3])
mask = tslearn.metrics.compute_mask(s1, s2, be="pytorch")
assert mask.dtype == torch.bool
```

## API Test: `compute_output_shape`

### Signature
```python
def compute_output_shape(self, input_shape)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:86  (+3 more definition site/overload)_

### Goal
Determines the expected output shape of a layer or estimator based on the provided input dimensions, typically used within Keras-compatible model architectures in `tslearn`.

### Parameters
- `self`: The instance of the estimator or layer (e.g., `LearningShapelets`) that defines the transformation logic.
- `input_shape`: A tuple or list representing the dimensions of the input data (e.g., `(n_ts, max_sz, d)`).

### Input
The caller must provide a valid shape tuple corresponding to the input data format expected by the estimator. Precondition: The input must be compatible with the internal logic of the specific `tslearn` estimator (e.g., 3D arrays for time-series data).

### Output
Returns `unspecified` — The method returns a tuple representing the shape of the output tensor after the transformation or layer operation is applied.

### Valid Call Patterns
```python
# Inferred from signature: usage within a custom layer or estimator
# Example using a LearningShapelets instance
from tslearn.shapelets import LearningShapelets
ls = LearningShapelets(n_shapelets=10, max_len=5)
output_shape = ls.compute_output_shape((None, 100, 1))
print(output_shape)
```

### LLM Instruction Prompt
- Use `compute_output_shape` only when inspecting the architecture of `tslearn` estimators that support Keras-style shape inference.
- Always pass the `input_shape` as a tuple representing the dimensions of the time-series dataset.
- Do not assume the return type is a specific object; treat it as a shape tuple.

### Prompt Snippet
```text
# Example of checking output shape for a tslearn estimator
shape = estimator.compute_output_shape(input_shape=(None, n_timesteps, n_features))
```

### Common Failure Modes
- Passing an incorrect number of dimensions in `input_shape` (e.g., providing 2D when 3D is required).
- Calling the method on an uninitialized estimator that has not yet defined its internal layer structure.
- Assuming the return value is a concrete array rather than a shape tuple.

### Fix Code Hint
```python
# Ensure the input_shape matches the expected 3D format (n_ts, sz, d)
# If using variable length, ensure the first dimension is None
input_shape = (None, 50, 1)
output = estimator.compute_output_shape(input_shape)
```

## API Test: `compute_var`

### Signature
```python
def compute_var(p, X, with_constant=False)
```
_Source: tslearn/tslearn/forecasting/_arima.py:21_

_Source doc:_ Compute the VAR parameters associated with a given time series. Parameters ---------- p : int AR order X : array-like, shape (sz, d) time-series data, sz should be greater or equal than p with_constant : bool (default: False) whether to compute an intercept term. Returns ------- (intercept, ar_coeffs, residuals) tuple

### Goal
Computes the Vector Autoregression (VAR) model parameters for a given multivariate time series.

### Parameters
- `p`: `int`. The autoregressive (AR) order of the model.
- `X`: `array-like`, shape `(sz, d)`. The time-series data where `sz` is the number of time steps and `d` is the number of dimensions.
- `with_constant`, default `False`: `bool`. If `True`, an intercept term is included in the VAR model estimation.

### Input
- **Data Format:** `X` must be an array-like structure (typically a 2D NumPy array) with shape `(sz, d)`.
- **Preconditions:** The number of time steps `sz` must be greater than or equal to the AR order `p`.

### Output
Returns a `tuple` containing:
- `intercept`: The estimated intercept vector (if `with_constant` is `True`, else `None`).
- `ar_coeffs`: The estimated autoregressive coefficient matrices.
- `residuals`: The residuals of the fitted VAR model.

### Valid Call Patterns
```python
import numpy as np
from tslearn.forecasting import compute_var

# Example: VAR(p=2) on a 10x2 time series
X = np.random.rand(10, 2)
intercept, ar_coeffs, residuals = compute_var(p=2, X=X, with_constant=True)

print(f"Residuals shape: {residuals.shape}")
```

### LLM Instruction Prompt
- Use `compute_var` when performing linear forecasting analysis on multivariate time series.
- Ensure the input `X` is a 2D array of shape `(sz, d)` and that `sz >= p`.
- Always unpack the returned tuple into `(intercept, ar_coeffs, residuals)`.

### Prompt Snippet
```text
from tslearn.forecasting import compute_var
# intercept, ar_coeffs, residuals = compute_var(p=..., X=..., with_constant=...)
```

### Common Failure Modes
- **Shape Mismatch:** Passing an array where `sz < p` will result in an error during the least-squares estimation.
- **Dimensionality:** Passing a 3D array (common in `tslearn` estimators) directly to `compute_var` without selecting a single time series (2D slice) will cause a shape error.
- **Input Type:** Passing non-numeric data or missing values (NaNs) will lead to failure in the underlying linear algebra solver.

### Fix Code Hint
```python
# Ensure X is 2D and sz >= p
if X.ndim == 3:
    X = X[0]  # Select the first time series if input is a dataset
assert X.shape[0] >= p, "Time series length must be >= AR order p"
```

## API Test: `copy`

### Signature
```python
def copy(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:94  (+1 more definition site/overload)_

### Goal
Creates a deep copy of the provided time-series data structure or array, ensuring that subsequent modifications to the copy do not affect the original input.

### Parameters
- `x`: The input time-series data, typically a NumPy array or a similar array-like structure representing time-series sequences.

### Input
- **Format:** Expects a 3D NumPy array of shape `(n_ts, max_sz, d)` or a compatible array-like structure.
- **Preconditions:** The input must be a valid array-like object supported by the underlying backend (NumPy or PyTorch). If the input is not already in the required 3D format, it should be processed via `tslearn.utils.to_time_series_dataset` prior to calling this function.

### Output
Returns `unspecified` — A new object containing the same data as `x`, allocated in a new memory block.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import NumPyBackend
import numpy as np

# Inferred usage based on backend helper patterns
be = NumPyBackend()
X = np.array([[[1, 2], [3, 4]]])
X_copy = be.copy(X)

assert not (X_copy is X)
assert np.array_equal(X_copy, X)
print(f"Correctness witness: Copy created with shape {X_copy.shape}")
```

### LLM Instruction Prompt
- Use `copy` when you need to preserve the state of a time-series dataset before applying destructive transformations or preprocessing steps.
- Always ensure the input `x` is a valid array-like structure before passing it to the backend `copy` method.

### Prompt Snippet
```text
# Example of using copy to preserve original data
from tslearn.backend import instantiate_backend
be = instantiate_backend("numpy")
X_original = ... # 3D time-series array
X_working_copy = be.copy(X_original)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-array-like object (e.g., a list of lists that has not been converted to a 3D array) may result in backend-specific errors.
- **Backend Mismatch:** Attempting to use a `NumPyBackend` copy method on a `PyTorch` tensor may lead to unexpected behavior or errors; ensure the backend instance matches the data type.

### Fix Code Hint
```python
# Ensure data is in the correct 3D format before copying
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X_raw)
X_copy = be.copy(X_formatted)
```

## API Test: `ctw`

### Signature
```python
def ctw(s1, s2, max_iter=100, n_components=None, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, verbose=False, be=None)
```
_Source: tslearn/tslearn/metrics/ctw.py:200_

_Source doc:_ Compute Canonical Time Warping (CTW) similarity measure between (possibly multidimensional) time series and return the similarity. Canonical Time Warping is a method to align time series under rigid registration of the feature space. It should not be confused with Dynamic Time Warping (DTW), though CTW uses DTW. It is not required that both time series share the same size, nor the same dimension (CTW will find a subspace that best aligns feature spaces). CTW was originally presented in [1]_.

### Goal
Compute the Canonical Time Warping (CTW) similarity measure between two time series, which aligns them by finding a subspace that optimizes feature space registration while simultaneously performing temporal alignment.

### Parameters
- `s1`: A time series of shape `(sz1, d)` or `(sz1,)`.
- `s2`: Another time series of shape `(sz2, d)` or `(sz2,)`.
- `max_iter`, default `100`: Number of iterations for the CTW algorithm.
- `n_components`, default `None`: Number of components for Canonical Correlation Analysis (CCA). If `None`, defaults to the minimum number of features between `s1` and `s2`.
- `global_constraint`, default `None`: Global constraint for DTW, either `"itakura"` or `"sakoe_chiba"`.
- `sakoe_chiba_radius`, default `None`: Radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Maximum slope for the Itakura parallelogram constraint.
- `verbose`, default `False`: If `True`, prints scores at each iteration.
- `be`, default `None`: Backend object or string (`"numpy"` or `"pytorch"`) to handle computations.

### Input
- **Data:** Two time series arrays. If univariate, shape is `(sz,)`. If multivariate, shape is `(sz, d)`.
- **Preconditions:** Input arrays should be compatible with the selected backend. If using `PyTorch`, ensure inputs are tensors.

### Output
- Returns `float`: The similarity score between the two time series.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Basic usage with NumPy arrays
s1 = np.random.randn(15, 3)
s2 = np.random.randn(10, 1)
score = tslearn.metrics.ctw(s1, s2)
print(f"Similarity score: {score}")
```

### LLM Instruction Prompt
- Use `tslearn.metrics.ctw` to compute similarity when feature space alignment is required alongside temporal warping.
- Always ensure input arrays are 2D `(sz, d)` or 1D `(sz,)`.
- If using `PyTorch` for automatic differentiation, explicitly pass `be="pytorch"`.

### Prompt Snippet
```text
from tslearn.metrics import ctw
# Compute CTW similarity between s1 and s2
score = ctw(s1, s2, max_iter=50)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing arrays that are not 1D or 2D will cause shape errors.
- **Constraint Conflict:** Providing both `sakoe_chiba_radius` and `itakura_max_slope` without a valid `global_constraint` may trigger a `RuntimeWarning`.
- **Backend Mismatch:** Passing a PyTorch tensor while the default NumPy backend is active (or vice versa) without specifying `be`.

### Fix Code Hint
```python
# Ensure inputs are properly shaped and backend is consistent
import numpy as np
from tslearn.metrics import ctw

s1 = np.array([1, 2, 3])
s2 = np.array([1, 2, 2, 3])
# Use default backend
dist = ctw(s1, s2, max_iter=10)
print(f"Correctness witness: {dist >= 0}")
```

## API Test: `ctw_path`

### Signature
```python
def ctw_path(s1, s2, max_iter=100, n_components=None, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, verbose=False, be=None)
```
_Source: tslearn/tslearn/metrics/ctw.py:46_

_Source doc:_ Compute Canonical Time Warping (CTW) similarity measure between (possibly multidimensional) time series and return the alignment path, the canonical correlation analysis (sklearn) object and the similarity. Canonical Time Warping is a method to align time series under rigid registration of the feature space. It should not be confused with Dynamic Time Warping (DTW), though CTW uses DTW. It is not required that both time series share the same size, nor the same dimension (CTW will find a subspace that best aligns feature spaces). CTW was originally presented in [1]_. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. max_iter : int (default: 100) Number of iterations for the CTW algorithm. Each iteration n_components : int (default: None) Number of components to be used for Canonical Correlation Analysis. If None, the lower minimum number of features between s1 and s2 is used. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for DTW calls. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. verbose : bool (default: True) If True, scores are printed at each iteration of the algorithm. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- list of integer pairs Matching path represented as a list of index pairs. In each pair, the first index corresponds to s1 and the second one corresponds to s2

### Goal
Computes the Canonical Time Warping (CTW) alignment between two time series, returning the optimal path, the CCA object, and the similarity score.

### Parameters
- `s1`: First time series, shape `(sz1, d)` or `(sz1,)`.
- `s2`: Second time series, shape `(sz2, d)` or `(sz2,)`.
- `max_iter`, default `100`: Number of iterations for the CTW algorithm.
- `n_components`, default `None`: Number of components for Canonical Correlation Analysis.
- `global_constraint`, default `None`: Constraint type for DTW ("itakura" or "sakoe_chiba").
- `sakoe_chiba_radius`, default `None`: Radius for Sakoe-Chiba band.
- `itakura_max_slope`, default `None`: Maximum slope for Itakura parallelogram.
- `verbose`, default `False`: If True, prints scores per iteration.
- `be`, default `None`: Backend engine ("numpy" or "pytorch").

### Input
Requires two time series as array-like objects. If univariate, shape `(sz,)` is accepted; otherwise, `(sz, d)` is expected.

### Output
Returns a tuple: `(path, cca, dist)`, where `path` is a list of index pairs `(i, j)`, `cca` is the fitted `sklearn` CCA object, and `dist` is the similarity score.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np
path, cca, dist = tslearn.metrics.ctw_path([1, 2, 3], [1.0, 2.0, 2.0, 3.0])
print(f"Path: {path}, Dist: {dist}")
```

### LLM Instruction Prompt
- Always import `tslearn.metrics` to access `ctw_path`.
- Ensure inputs are array-like; use `numpy` arrays for standard CPU execution.
- If using `PyTorch` tensors, explicitly set `be="pytorch"` or ensure the backend is correctly inferred.

### Prompt Snippet
```text
from tslearn.metrics import ctw_path
path, cca, dist = ctw_path(s1, s2)
```

### Common Failure Modes
- Mismatched dimensions or incompatible array types when not using the correct backend.
- Providing `global_constraint` without setting the corresponding `sakoe_chiba_radius` or `itakura_max_slope` correctly, leading to `RuntimeWarning`.

### Fix Code Hint
```python
import tslearn.metrics
import numpy as np
# Ensure inputs are compatible
s1 = np.array([1, 2, 3])
s2 = np.array([1, 2, 2, 3])
path, cca, dist = tslearn.metrics.ctw_path(s1, s2)
print(f"Correctness Witness: {dist == 0.0}")
```

## API Test: `cydist_1d_sax`

### Signature
```python
def cydist_1d_sax(sax1, sax2, breakpoints_avg_middle_, breakpoints_slope_middle_, original_size)
```
_Source: tslearn/tslearn/metrics/cysax.py:135_

_Source doc:_ Compute distance between 1d-SAX representations as defined in [1]_. Parameters ---------- sax1 : array-like, shape=(sz, 2 * d), dtype=float64 (Linux and MacOS) or float32 (Windows) 1d-SAX representation of a time series. sax2 : array-like, shape=(sz, 2 * d), dtype=float64 (Linux and MacOS) or float32 (Windows) 1d-SAX representation of another time series. breakpoints_avg_middle_ : array-like, ndim=1, dtype=float64 breakpoints_slope_middle_ : array-like, ndim=1, dtype=float64 original_size : int64 (Linux and MacOS) or int32 (Windows) Length of the original time series. Returns ------- dist_1d_sax : float64 1d-SAX distance. Notes ----- Unlike SAX distance, 1d-SAX distance does not lower bound Euclidean distance between original time series. References ---------- .. [1] S. Malinowski, T. Guyet, R. Quiniou, R. Tavenard. 1d-SAX: a Novel Symbolic Representation for Time Series. IDA 2013.

### Goal
Computes the 1d-SAX distance between two symbolic representations of time series as defined by Malinowski et al. (2013).

### Parameters
- `sax1`: 1d-SAX representation of a time series, expected as an array-like object with shape `(sz, 2 * d)`.
- `sax2`: 1d-SAX representation of another time series, expected as an array-like object with shape `(sz, 2 * d)`.
- `breakpoints_avg_middle_`: 1D array of breakpoints for the average component of the 1d-SAX representation.
- `breakpoints_slope_middle_`: 1D array of breakpoints for the slope component of the 1d-SAX representation.
- `original_size`: The length of the original time series before symbolic transformation.

### Input
The function requires pre-computed 1d-SAX representations. Inputs must be numeric arrays. Note that `dtype` requirements are platform-dependent: `float64` for Linux/macOS and `float32` for Windows. `original_size` must be an integer (`int64` on Linux/macOS, `int32` on Windows).

### Output
Returns a `float64` representing the 1d-SAX distance between the two provided representations.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.cysax import cydist_1d_sax

# Inferred usage based on signature
dist = cydist_1d_sax(
    sax1=np.array([[0.1, 0.2]], dtype=np.float64),
    sax2=np.array([[0.3, 0.4]], dtype=np.float64),
    breakpoints_avg_middle_=np.array([0.0], dtype=np.float64),
    breakpoints_slope_middle_=np.array([0.0], dtype=np.float64),
    original_size=10
)
print(f"Distance: {dist}")
```

### LLM Instruction Prompt
- When calling `cydist_1d_sax`, ensure all array inputs are cast to the appropriate `dtype` for the host operating system (float64 for Linux/macOS, float32 for Windows).
- Ensure `sax1` and `sax2` have identical shapes `(sz, 2 * d)`.
- Do not assume this distance provides a lower bound for Euclidean distance.

### Prompt Snippet
```text
from tslearn.metrics.cysax import cydist_1d_sax
# Ensure inputs are numpy arrays of correct dtype for the platform
dist = cydist_1d_sax(sax1, sax2, avg_bp, slope_bp, orig_size)
```

### Common Failure Modes
- **Dtype Mismatch:** Providing `float32` on Linux/macOS or `float64` on Windows can lead to runtime errors or unexpected behavior in the underlying Cython implementation.
- **Shape Mismatch:** Passing `sax1` and `sax2` with different dimensions `(sz, 2 * d)` will cause the function to fail.
- **Incorrect Breakpoints:** Providing breakpoints that do not match the symbolic alphabet used to generate the SAX representations will result in mathematically invalid distances.

### Fix Code Hint
```python
import numpy as np
import sys

# Ensure correct dtype based on platform
dtype = np.float64 if sys.platform != "win32" else np.float32
sax1 = np.array(sax1, dtype=dtype)
sax2 = np.array(sax2, dtype=dtype)
```

## API Test: `cydist_sax`

### Signature
```python
def cydist_sax(sax1, sax2, breakpoints, original_size)
```
_Source: tslearn/tslearn/metrics/cysax.py:38_

_Source doc:_ Compute distance between SAX representations as defined in [1]_. Parameters ---------- sax1 : array-like, shape=(sz, d) SAX representation of a time series. sax2 : array-like, shape=(sz, d) SAX representation of another time series. breakpoints : array-like, ndim=1, dtype=float64 The breakpoints used to assign the alphabet symbols. original_size : int64 (Linux and MacOS) or int32 (Windows) Length of the original time series. Returns ------- dist_sax : float64 SAX distance. References ---------- .. [1] J. Lin, E. Keogh, L. Wei, et al. Experiencing SAX: a novel symbolic representation of time series. Data Mining and Knowledge Discovery, 2007. vol. 15(107)

### Goal
Computes the distance between two Symbolic Aggregate Approximation (SAX) representations of time series as defined by Lin et al. (2007).

### Parameters
- `sax1`: array-like, shape=(sz, d); the SAX representation of the first time series.
- `sax2`: array-like, shape=(sz, d); the SAX representation of the second time series.
- `breakpoints`: array-like, ndim=1, dtype=float64; the numerical breakpoints used to define the alphabet symbols.
- `original_size`: int64 (Linux/macOS) or int32 (Windows); the length of the original time series before SAX transformation.

### Input
The inputs must be pre-computed SAX representations. `sax1` and `sax2` must have identical shapes `(sz, d)`. `breakpoints` must be a 1D array of floats. `original_size` must match the length of the time series from which the SAX representations were derived.

### Output
Returns `float64`: The calculated SAX distance between the two representations.

### Valid Call Patterns
```python
from tslearn.metrics import cydist_sax
import numpy as np

# Inferred usage based on signature
sax1 = np.array([[0], [1], [2]], dtype=np.float64)
sax2 = np.array([[0], [2], [1]], dtype=np.float64)
breakpoints = np.array([-1.0, 0.0, 1.0], dtype=np.float64)
original_size = 10
dist = cydist_sax(sax1, sax2, breakpoints, original_size)
print(f"Correctness witness: {dist}")
```

### LLM Instruction Prompt
- Use `cydist_sax` only when comparing pre-computed SAX representations.
- Ensure `sax1` and `sax2` have the same dimensions.
- Ensure `original_size` is provided as an integer matching the original time series length.
- Import `cydist_sax` from `tslearn.metrics`.

### Prompt Snippet
```text
from tslearn.metrics import cydist_sax
dist = cydist_sax(sax1, sax2, breakpoints, original_size)
```

### Common Failure Modes
- Passing arrays of mismatched shapes for `sax1` and `sax2`.
- Providing `original_size` as a float instead of an integer, which may cause type errors in the underlying C implementation.
- Providing `breakpoints` that do not match the alphabet mapping used to generate the SAX representations.

### Fix Code Hint
```python
# Ensure inputs are correctly typed for the C-backend
import numpy as np
sax1 = np.ascontiguousarray(sax1, dtype=np.float64)
sax2 = np.ascontiguousarray(sax2, dtype=np.float64)
breakpoints = np.ascontiguousarray(breakpoints, dtype=np.float64)
original_size = int(original_size)
```

## API Test: `cyslopes`

### Signature
```python
def cyslopes(dataset, t0)
```
_Source: tslearn/tslearn/metrics/cysax.py:107_

_Source doc:_ Compute slopes. Parameters ---------- dataset : array-like, shape=(n_ts, sz, d), dtype=float64 t0 : int32 Returns ------- dataset_out : array-like, shape=(n_ts, d), dtype=float64

### Goal
Computes the slopes of time-series data at a specified time index `t0` to facilitate symbolic representation or feature extraction.

### Parameters
- `dataset`: array-like, shape=(n_ts, sz, d), dtype=float64. The input time-series dataset where `n_ts` is the number of series, `sz` is the length of each series, and `d` is the number of dimensions.
- `t0`: int32. The time index at which the slope calculation is performed.

### Input
The caller must provide a 3D NumPy array of shape `(n_ts, sz, d)` with `float64` precision. If the input is not in this format, use `tslearn.utils.to_time_series_dataset` to ensure compatibility. The index `t0` must be a valid integer within the bounds of the time-series length `sz`.

### Output
Returns `dataset_out`: array-like, shape=(n_ts, d), dtype=float64. This represents the computed slopes for each time series in the dataset across all dimensions at the specified time index.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.cysax import cyslopes

# Inferred usage: dataset must be (n_ts, sz, d)
X = np.random.rand(10, 20, 1).astype(np.float64)
t0 = 5
slopes = cyslopes(X, t0)
print(f"Computed slopes shape: {slopes.shape}")
```

### LLM Instruction Prompt
- When calling `cyslopes`, ensure the input `dataset` is a 3D NumPy array of type `float64`.
- Verify that `t0` is an integer index strictly less than the time-series length `sz`.
- The function is a low-level helper; ensure the input data is pre-processed into the standard `tslearn` 3D format before invocation.

### Prompt Snippet
```text
from tslearn.metrics.cysax import cyslopes
# Ensure X is (n_ts, sz, d) and float64
slopes = cyslopes(X, t0=index)
```

### Common Failure Modes
- **Shape Mismatch:** Passing a 2D array instead of the required 3D `(n_ts, sz, d)` array will cause a runtime error.
- **Dtype Error:** Passing integer arrays or non-float64 types may lead to unexpected behavior or errors in the underlying C implementation.
- **Index Out of Bounds:** Providing a `t0` value that exceeds the time-series length `sz` will result in an index error.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset

# Ensure input is 3D float64
X_formatted = to_time_series_dataset(X).astype(np.float64)
# Ensure t0 is within bounds
t0 = min(t0, X_formatted.shape[1] - 1)
slopes = cyslopes(X_formatted, t0)
```

## API Test: `decision_function`

### Signature
```python
def decision_function(self, X)
```
_Source: tslearn/tslearn/svm/svm.py:352_

_Source doc:_ Evaluates the decision function for the samples in X. Parameters ---------- X : array-like of shape=(n_ts, sz, d) Time series dataset. Returns ------- ndarray of shape (n_samples, n_classes * (n_classes-1) / 2) Returns the decision function of the sample for each class in the model. If decision_function_shape='ovr', the shape is (n_samples, n_classes).

### Goal
Computes the signed distance of time-series samples to the separating hyperplane(s) defined by a trained time-series SVM model.

### Parameters
- `self`: The trained `TimeSeriesSVC` estimator instance.
- `X`: array-like of shape=(n_ts, sz, d) representing the time-series dataset to evaluate.

### Input
- **Data Format:** Must be a 3D NumPy array of shape `(n_ts, sz, d)`. If the input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this method.
- **Preconditions:** The estimator must be fitted (i.e., `fit` has been called) before invoking `decision_function`.

### Output
Returns `ndarray` of shape `(n_samples, n_classes * (n_classes-1) / 2)` by default (one-vs-one). If the estimator was configured with `decision_function_shape='ovr'`, the output shape is `(n_samples, n_classes)`.

### Valid Call Patterns
```python
# Inferred from signature and scikit-learn compatibility
from tslearn.svm import TimeSeriesSVC
clf = TimeSeriesSVC(kernel="gak").fit(X_train, y_train)
scores = clf.decision_function(X_test)
print(f"Decision function shape: {scores.shape}")
```

### LLM Instruction Prompt
- When calling `decision_function`, ensure the input `X` is a 3D array.
- Always verify that the estimator has been trained via `fit` before calling this method.
- Note that the output dimensions depend on the `decision_function_shape` parameter set during estimator initialization.

### Prompt Snippet
```text
# Ensure X is 3D before calling decision_function
X_formatted = to_time_series_dataset(X)
scores = model.decision_function(X_formatted)
```

### Common Failure Modes
- **Shape Mismatch:** Passing a 2D array instead of a 3D array will raise a `ValueError` or `IndexError` during internal processing.
- **Unfitted Estimator:** Calling `decision_function` before `fit` will raise a `sklearn.exceptions.NotFittedError`.
- **Incompatible Backend:** Attempting to pass data that cannot be converted to the expected 3D NumPy format.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset

# Ensure input is 3D: (n_ts, sz, d)
if X.ndim == 2:
    X = to_time_series_dataset(X)
    
# Verify model is fitted
try:
    scores = model.decision_function(X)
except Exception as e:
    print(f"Ensure model is fitted and input is 3D: {e}")
```

## API Test: `distance`

### Signature
```python
def distance(self, ts1, ts2)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:219  (+2 more definition site/overload)_

_Source doc:_ Compute distance between PAA representations as defined in [1]_. Parameters ---------- ts1 : array-like A time series ts2 : array-like Another time series Returns ------- float PAA distance References ---------- .. [1] E. Keogh & M. Pazzani. Scaling up dynamic time warping for datamining applications. SIGKDD 2000, pp. 285--289.

### Goal
Computes the distance between two time series based on their Piecewise Aggregate Approximation (PAA) or Symbolic Aggregate Approximation (SAX) representations.

### Parameters
- `self`: An instance of a piecewise transformation estimator (e.g., `PiecewiseAggregateApproximation` or `SymbolicAggregateApproximation`).
- `ts1`: array-like; The first time series to compare.
- `ts2`: array-like; The second time series to compare.

### Input
- **Data Format:** Expects array-like structures representing time series.
- **Preconditions:** The estimator instance must be fitted (i.e., `fit` or `fit_transform` must have been called) before invoking `distance`. If the estimator is not fitted, a `NotFittedError` will be raised.

### Output
Returns `float` — The computed distance between the PAA or SAX representations of the two input time series.

### Valid Call Patterns
```python
from tslearn.piecewise import PiecewiseAggregateApproximation
import numpy as np

# Example using PiecewiseAggregateApproximation
paa = PiecewiseAggregateApproximation(n_segments=3)
X = np.random.randn(2, 10, 3)
paa.fit(X)
dist = paa.distance(X[0], X[1])
print(f"Correctness witness: {dist}")
```

### LLM Instruction Prompt
- Always ensure the estimator instance (e.g., `PiecewiseAggregateApproximation` or `SymbolicAggregateApproximation`) has been fitted using `fit()` or `fit_transform()` on a dataset before calling `distance()`.
- The input `ts1` and `ts2` should be compatible with the dimensions expected by the fitted estimator.

### Prompt Snippet
```text
# Ensure the estimator is fitted
estimator.fit(X_train)
# Compute distance
d = estimator.distance(ts1, ts2)
```

### Common Failure Modes
- **NotFittedError:** Calling `distance` before the estimator has been fitted to data.
- **Dimension Mismatch:** Providing time series that do not match the expected input dimensions or segment counts defined during estimator initialization.
- **Input Type:** Passing non-array-like objects that cannot be processed by the underlying NumPy-based backend.

### Fix Code Hint
```python
# Ensure the estimator is fitted before calling distance
if not hasattr(estimator, "n_segments"): # Simple check for fitted state
    estimator.fit(X)
distance_val = estimator.distance(ts1, ts2)
```

## API Test: `distance_1d_sax`

### Signature
```python
def distance_1d_sax(self, sax1, sax2)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:726_

_Source doc:_ Compute distance between 1d-SAX representations as defined in [1]_. Parameters ---------- sax1 : array-like 1d-SAX representation of a time series sax2 : array-like 1d-SAX representation of another time series Returns ------- float 1d-SAX distance Notes ----- Unlike SAX distance, 1d-SAX distance does not lower bound Euclidean distance between original time series. References ---------- .. [1] S. Malinowski, T. Guyet, R. Quiniou, R. Tavenard. 1d-SAX: a Novel Symbolic Representation for Time Series. IDA 2013.

### Goal
Computes the distance between two 1d-SAX (Symbolic Aggregate Approximation) representations of time series.

### Parameters
- `self`: An instance of `OneD_SymbolicAggregateApproximation` that has been configured with the parameters used to generate the SAX representations.
- `sax1`: array-like; the 1d-SAX representation of the first time series.
- `sax2`: array-like; the 1d-SAX representation of the second time series.

### Input
The inputs `sax1` and `sax2` must be the output of a `OneD_SymbolicAggregateApproximation.transform` call. The estimator `self` must be the same instance used to generate these representations to ensure the alphabet and segment parameters match.

### Output
Returns `float` — The calculated 1d-SAX distance between the two symbolic representations.

### Valid Call Patterns
```python
from tslearn.piecewise import OneD_SymbolicAggregateApproximation
import numpy as np

# Setup and fit
sax1d_est = OneD_SymbolicAggregateApproximation(n_segments=3, alphabet_size_avg=2, alphabet_size_slope=2)
X = np.random.randn(2, 10, 3)
sax1d = sax1d_est.fit_transform(X)

# Compute distance
dist = sax1d_est.distance_1d_sax(sax1d[0], sax1d[1])
print(f"Distance: {dist}")
```

### LLM Instruction Prompt
- Use `distance_1d_sax` only when working with `OneD_SymbolicAggregateApproximation` objects.
- Ensure the input arrays are the result of the `transform` method of the same estimator instance.
- Do not assume this distance lower-bounds Euclidean distance.

### Prompt Snippet
```text
# Calculate 1d-SAX distance using a fitted estimator
dist = estimator.distance_1d_sax(sax_rep1, sax_rep2)
```

### Common Failure Modes
- **NotFittedError**: Calling `distance_1d_sax` on an estimator that has not been fitted via `fit` or `fit_transform`.
- **Dimension Mismatch**: Passing representations generated by different estimators or with different `n_segments`/`alphabet_size` parameters.
- **Input Type**: Passing raw time-series data instead of the symbolic representations generated by the `OneD_SymbolicAggregateApproximation` transformer.

### Fix Code Hint
```python
# Ensure the estimator is fitted before calling distance_1d_sax
if not hasattr(estimator, "n_segments"):
    estimator.fit(X)
dist = estimator.distance_1d_sax(sax1, sax2)
```

## API Test: `distance_paa`

### Signature
```python
def distance_paa(self, paa1, paa2)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:195_

_Source doc:_ Compute distance between PAA representations as defined in [1]_. Parameters ---------- paa1 : array-like PAA representation of a time series paa2 : array-like PAA representation of another time series Returns ------- float PAA distance References ---------- .. [1] E. Keogh & M. Pazzani. Scaling up dynamic time warping for datamining applications. SIGKDD 2000, pp. 285--289.

### Goal
Computes the Euclidean distance between two Piecewise Aggregate Approximation (PAA) representations of time series.

### Parameters
- `self`: An instance of `tslearn.piecewise.PiecewiseAggregateApproximation`.
- `paa1`: array-like; The PAA representation of the first time series.
- `paa2`: array-like; The PAA representation of the second time series.

### Input
- **Format:** `paa1` and `paa2` must be arrays of shape `(n_segments, d)` where `n_segments` is the number of segments defined during the initialization of the `PiecewiseAggregateApproximation` estimator.
- **Preconditions:** The `PiecewiseAggregateApproximation` estimator must be initialized with a valid `n_segments` parameter. While the method can be called on an unfitted estimator, the inputs must be valid PAA-transformed arrays.

### Output
Returns `float` — The PAA distance between the two provided representations.

### Valid Call Patterns
```python
from tslearn.piecewise import PiecewiseAggregateApproximation
import numpy as np

# Initialize and fit
paa = PiecewiseAggregateApproximation(n_segments=3)
X = np.random.randn(2, 10, 1)
paa_repr = paa.fit_transform(X)

# Compute distance
dist = paa.distance_paa(paa_repr[0], paa_repr[1])
print(f"PAA Distance: {dist}")
```

### LLM Instruction Prompt
- Use `distance_paa` only when working with `PiecewiseAggregateApproximation` instances.
- Ensure inputs `paa1` and `paa2` are already transformed PAA representations (not raw time series).
- The method is an instance method; always call it on a configured `PiecewiseAggregateApproximation` object.

### Prompt Snippet
```text
# Example of calling distance_paa
dist = paa_estimator.distance_paa(paa_data_a, paa_data_b)
```

### Common Failure Modes
- **Shape Mismatch:** Passing arrays that do not match the `n_segments` configuration of the estimator will result in calculation errors.
- **Raw Data Input:** Passing raw time series instead of PAA-transformed representations will lead to incorrect distance values.
- **Unfitted Estimator:** While the method may execute, it is intended for use with representations generated by the same estimator configuration.

### Fix Code Hint
```python
# Ensure inputs are transformed first
paa_est = PiecewiseAggregateApproximation(n_segments=5)
paa_data = paa_est.fit_transform(X)
# Correct usage
dist = paa_est.distance_paa(paa_data[0], paa_data[1])
```

## API Test: `distance_sax`

### Signature
```python
def distance_sax(self, sax1, sax2)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:444_

_Source doc:_ Compute distance between SAX representations as defined in [1]_. Parameters ---------- sax1 : array-like SAX representation of a time series sax2 : array-like SAX representation of another time series Returns ------- float SAX distance References ---------- .. [1] J. Lin, E. Keogh, L. Wei, et al. Experiencing SAX: a novel symbolic representation of time series. Data Mining and Knowledge Discovery, 2007. vol. 15(107)

### Goal
Computes the distance between two Symbolic Aggregate Approximation (SAX) representations of time series as defined by Lin et al. (2007).

### Parameters
- `self`: An instance of `SymbolicAggregateApproximation` that has been fitted to data.
- `sax1`: array-like; The SAX representation of the first time series.
- `sax2`: array-like; The SAX representation of the second time series.

### Input
The inputs `sax1` and `sax2` must be the symbolic representations generated by a fitted `SymbolicAggregateApproximation` estimator. The estimator must be in a "fitted" state (i.e., `fit` or `fit_transform` has been called) before this method can be invoked.

### Output
Returns `float` — The calculated SAX distance between the two symbolic representations.

### Valid Call Patterns
```python
from tslearn.piecewise import SymbolicAggregateApproximation
import numpy as np

# Setup and fit
sax_est = SymbolicAggregateApproximation(n_segments=3, alphabet_size_avg=2)
X = np.random.randn(2, 10, 3)
sax_repr = sax_est.fit_transform(X)

# Valid call
dist = sax_est.distance_sax(sax_repr[0], sax_repr[1])
print(f"Correctness witness: {dist}")
```

### LLM Instruction Prompt
- Always ensure the `SymbolicAggregateApproximation` instance is fitted before calling `distance_sax`.
- Pass the symbolic representations (the output of `fit_transform`) as the `sax1` and `sax2` arguments.
- Do not attempt to pass raw time-series data directly to `distance_sax`; use the estimator's `distance` method for raw data or `fit_transform` to obtain the SAX representations first.

### Prompt Snippet
```text
# Ensure the estimator is fitted
sax_est.fit(X)
# Use distance_sax with the transformed representations
distance = sax_est.distance_sax(sax_repr[0], sax_repr[1])
```

### Common Failure Modes
- `NotFittedError`: Occurs if `distance_sax` is called on an instance of `SymbolicAggregateApproximation` that has not yet been fitted to data.
- `ValueError`: Occurs if the input arrays are not valid SAX representations or have mismatched dimensions relative to the estimator's configuration.

### Fix Code Hint
```python
# Ensure the estimator is fitted before calling distance_sax
if not hasattr(sax_est, "breakpoints_"):
    sax_est.fit(X)
dist = sax_est.distance_sax(sax_repr[0], sax_repr[1])
```

## API Test: `dtw`

### Signature
```python
def dtw(s1, s2, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:688  (+1 more definition site/overload)_

_Source doc:_ Compute Dynamic Time Warping (DTW) similarity measure between (possibly multidimensional) time series and return it. DTW is computed as the Euclidean distance between aligned time series, i.e., if :math:`\pi` is the optimal alignment path: .. math:: DTW(X, Y) = \sqrt{\sum_{(i, j) \in \pi} \|X_{i} - Y_{j}\|^2} Note that this formula is still valid for the multivariate case. It is not required that both time series share the same size, but they must be the same dimension. DTW was originally presented in [1]_ and is discussed in more details in our :ref:`dedicated user-guide page <dtw>`. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for DTW. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns -------

### Goal
Computes the Dynamic Time Warping (DTW) distance, a non-linear alignment-based similarity measure, between two time series of potentially different lengths.

### Parameters
- `s1`: The first time series; array-like of shape `(sz1, d)` or `(sz1,)`.
- `s2`: The second time series; array-like of shape `(sz2, d)` or `(sz2,)`.
- `global_constraint`, default `None`: The path constraint type, either `"itakura"` or `"sakoe_chiba"`.
- `sakoe_chiba_radius`, default `None`: Integer radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float maximum slope for the Itakura parallelogram constraint.
- `be`, default `None`: Backend engine; accepts `"numpy"`, `"pytorch"`, or a backend instance.

### Input
- **Data:** Two time series arrays. If univariate, shape is `(sz,)`. If multivariate, shape is `(sz, d)`.
- **Preconditions:** Both time series must have the same dimension `d`.
- **Backend:** If using `PyTorch`, ensure the `pytorch` package is installed.

### Output
Returns a scalar float representing the DTW distance (or a tensor if using the `PyTorch` backend).

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Standard usage with NumPy
s1 = np.array([1, 2, 3])
s2 = np.array([1, 2, 2, 3])
dist = tslearn.metrics.dtw(s1, s2)
print(f"Correctness Witness: {dist}")
```

### LLM Instruction Prompt
- Always import `tslearn.metrics` to access `dtw`.
- If performing automatic differentiation, explicitly set `be="pytorch"` and ensure inputs are `torch.Tensor` objects with `requires_grad=True`.
- Do not pass 3D datasets `(n_ts, sz, d)` directly; `dtw` expects individual time series `(sz, d)` or `(sz,)`.

### Prompt Snippet
```text
from tslearn.metrics import dtw
# Compute DTW between two series
distance = dtw(s1, s2, global_constraint="sakoe_chiba", sakoe_chiba_radius=2)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing time series with different dimensions `d` will raise a `ValueError`.
- **Empty Input:** Passing empty arrays will raise a `ValueError`.
- **Backend Mismatch:** Attempting to use `be="pytorch"` without the `torch` library installed or with incompatible input types.

### Fix Code Hint
```python
# Ensure inputs are 2D if multivariate
if s1.ndim == 1:
    s1 = s1.reshape(-1, 1)
if s2.ndim == 1:
    s2 = s2.reshape(-1, 1)
dist = tslearn.metrics.dtw(s1, s2)
```

## API Test: `dtw_barycenter_averaging`

### Signature
```python
def dtw_barycenter_averaging(X, barycenter_size=None, init_barycenter=None, max_iter=30, tol=1e-05, weights=None, metric_params=None, verbose=False, n_init=1, n_jobs=None)
```
_Source: tslearn/tslearn/barycenters/dba.py:485_

_Source doc:_ DTW Barycenter Averaging (DBA) method estimated through Expectation-Maximization algorithm. DBA was originally presented in [1]_. This implementation is based on an idea from [2]_ (Majorize-Minimize Mean Algorithm). Parameters ---------- X : array-like, shape=(n_ts, sz, d) Time series dataset. barycenter_size : int or None (default: None) Size of the barycenter to generate. If None, the size of the barycenter is that of the data provided at fit time or that of the initial barycenter if specified. init_barycenter : array or None (default: None) Initial barycenter to start from for the optimization process. max_iter : int (default: 30) Number of iterations of the Expectation-Maximization optimization procedure. tol : float (default: 1e-5) Tolerance to use for early stopping: if the decrease in cost is lower than this value, the Expectation-Maximization procedure stops. weights: None or array Weights of each X[i]. Must be the same size as len(X). If None, uniform weights are used. metric_params: dict or None (default: None) DTW constraint parameters to be used. See :ref:`tslearn.metrics.dtw_path <fun-tslearn.metrics.dtw_path>` for a list of accepted parameters If None, no constraint is used for DTW computations. verbose : boolean (default: False) Whether to print information about the cost at each iteration or not. n_init : int (default: 1) Number of different initializations to be tried (useful only is init_barycenter is set to None, otherwise, all trials will reach the same performance) n_jobs : int or None, optional (default=None) The number of jobs to run in parallel. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details. Returns ------- numpy.array of shape (barycenter_size, d) or (sz, d) if barycenter_size \ is None DBA barycenter of the provided time series dataset.

### Goal
Computes the DTW Barycenter Averaging (DBA) of a time-series dataset using an Expectation-Maximization algorithm to find a representative sequence that minimizes the sum of squared DTW distances to the input series.

### Parameters
- `X`: Time series dataset of shape `(n_ts, sz, d)`.
- `barycenter_size`, default `None`: The length of the resulting barycenter. If `None`, defaults to the length of the input time series.
- `init_barycenter`, default `None`: Initial sequence to start the optimization. If `None`, a random initialization is used.
- `max_iter`, default `30`: Maximum number of EM iterations.
- `tol`, default `1e-05`: Tolerance for early stopping based on the decrease in cost.
- `weights`, default `None`: Array of weights for each time series in `X`. Must match `len(X)`.
- `metric_params`, default `None`: Dictionary of parameters passed to the DTW metric (e.g., `sakoe_chiba_radius`).
- `verbose`, default `False`: If `True`, prints cost information during optimization.
- `n_init`, default `1`: Number of random initializations to perform.
- `n_jobs`, default `None`: Number of parallel jobs to run. `-1` uses all available processors.

### Input
The input `X` must be a 3D array of shape `(n_ts, sz, d)`. If the data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this function.

### Output
Returns a `numpy.array` of shape `(barycenter_size, d)` representing the computed DBA barycenter.

### Valid Call Patterns
```python
import numpy as np
from tslearn.barycenters import dtw_barycenter_averaging

# Example using inferred signature usage
X = np.random.randn(10, 20, 1)
barycenter = dtw_barycenter_averaging(X, max_iter=10, verbose=True)
print(f"Barycenter shape: {barycenter.shape}")
```

### LLM Instruction Prompt
- Always ensure `X` is a 3D NumPy array.
- Use `metric_params` to pass constraints like `{"sakoe_chiba_radius": 2}` if needed.
- When `n_jobs` is set to `-1`, ensure the environment supports `joblib` parallelization.

### Prompt Snippet
```text
dtw_barycenter_averaging(X, barycenter_size=None, init_barycenter=None, max_iter=30, tol=1e-05, weights=None, metric_params=None, verbose=False, n_init=1, n_jobs=None)
```

### Common Failure Modes
- Passing 2D arrays directly without using `to_time_series_dataset` will raise a shape error.
- Setting `max_iter` too low may result in a poor barycenter approximation.
- Mismatched `weights` length relative to `X` will cause an index or broadcast error.

### Fix Code Hint
```python
# Ensure 3D shape
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
barycenter = dtw_barycenter_averaging(X_formatted, max_iter=50)
```

## API Test: `dtw_barycenter_averaging_one_init`

### Signature
```python
def dtw_barycenter_averaging_one_init(X, barycenter_size=None, init_barycenter=None, max_iter=30, tol=1e-05, weights=None, metric_params=None, verbose=False, n_jobs=None)
```
_Source: tslearn/tslearn/barycenters/dba.py:621_

### Goal
Computes the DTW Barycenter Averaging (DBA) of a time-series dataset using an Expectation-Maximization algorithm to find a representative sequence that minimizes the sum of squared DTW distances.

### Parameters
- `X`: Time series dataset, array-like of shape `(n_ts, sz, d)`.
- `barycenter_size`, default `None`: The length of the resulting barycenter. If `None`, defaults to the length of the input time series or the initial barycenter.
- `init_barycenter`, default `None`: Initial sequence to start the optimization.
- `max_iter`, default `30`: Maximum number of EM iterations.
- `tol`, default `1e-05`: Tolerance for early stopping based on the decrease in cost.
- `weights`, default `None`: Array of weights for each time series in `X`. Must match `len(X)`.
- `metric_params`, default `None`: Dictionary of DTW constraints (e.g., `global_constraint`, `sakoe_chiba_radius`).
- `verbose`, default `False`: Whether to print iteration progress.
- `n_jobs`, default `None`: Number of parallel jobs to run.

### Input
The input `X` must be a 3D NumPy array of shape `(n_ts, sz, d)`. If the data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this function.

### Output
Returns a tuple containing:
1. `numpy.array`: The computed DBA barycenter of shape `(barycenter_size, d)`.
2. `float`: The associated inertia (sum of squared DTW distances).

### Valid Call Patterns
```python
import numpy as np
from tslearn.barycenters import dtw_barycenter_averaging_one_init

# Example with synthetic data
X = np.random.rand(10, 5, 1)
barycenter, inertia = dtw_barycenter_averaging_one_init(X, max_iter=10)
print(f"Barycenter shape: {barycenter.shape}, Inertia: {inertia}")
```

### LLM Instruction Prompt
- Always ensure `X` is a 3D array.
- If `metric_params` are provided, ensure they are compatible with `tslearn.metrics.dtw_path`.
- The function returns two values (barycenter and inertia); ensure the caller unpacks both.

### Prompt Snippet
```text
from tslearn.barycenters import dtw_barycenter_averaging_one_init
# X: (n_ts, sz, d)
barycenter, inertia = dtw_barycenter_averaging_one_init(X, ...)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will raise a `ValueError`.
- **Weight Mismatch:** Providing a `weights` array that does not match the number of time series in `X` will cause an index error during computation.
- **Convergence:** If `max_iter` is too low, the barycenter may not reach a stable local optimum.

### Fix Code Hint
```python
# Ensure 3D shape for input
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
barycenter, inertia = dtw_barycenter_averaging_one_init(X_formatted)
```

## API Test: `dtw_barycenter_averaging_petitjean`

### Signature
```python
def dtw_barycenter_averaging_petitjean(X, barycenter_size=None, init_barycenter=None, max_iter=30, tol=1e-05, weights=None, metric_params=None, verbose=False, n_jobs=None)
```
_Source: tslearn/tslearn/barycenters/dba.py:92_

_Source doc:_ DTW Barycenter Averaging (DBA) method. DBA was originally presented in [1]_. This implementation is not the one documented in the API, but is kept in the codebase to check the documented one for non-regression. Parameters ---------- X : array-like, shape=(n_ts, sz, d) Time series dataset. barycenter_size : int or None (default: None) Size of the barycenter to generate. If None, the size of the barycenter is that of the data provided at fit time or that of the initial barycenter if specified. init_barycenter : array or None (default: None) Initial barycenter to start from for the optimization process. max_iter : int (default: 30) Number of iterations of the Expectation-Maximization optimization procedure. tol : float (default: 1e-5) Tolerance to use for early stopping: if the decrease in cost is lower than this value, the Expectation-Maximization procedure stops. weights: None or array Weights of each X[i]. Must be the same size as len(X). If None, uniform weights are used. metric_params: dict or None (default: None) DTW constraint parameters to be used. See :ref:`tslearn.metrics.dtw_path <fun-tslearn.metrics.dtw_path>` for a list of accepted parameters If None, no constraint is used for DTW computations. verbose : boolean (default: False) Whether to print information about the cost at each iteration or not. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`__ for more details. Returns ------- numpy.array of shape (barycenter_size, d) or (sz, d) if barycenter_size \ is None DBA barycenter of the provided time series dataset. Examples -------- >>> time_series = [[1, 2, 3, 4], [1, 2, 4, 5]] >>> dtw_barycenter_averaging_petitjean(time_series, max_iter=5) array([[1. ], [2. ],

### Goal
Computes the DTW Barycenter Averaging (DBA) of a time-series dataset, providing a representative sequence that minimizes the sum of squared DTW distances to the input series.

### Parameters
- `X`: Time series dataset, array-like of shape `(n_ts, sz, d)`.
- `barycenter_size`, default `None`: The desired length of the resulting barycenter. If `None`, defaults to the length of the input series.
- `init_barycenter`, default `None`: Initial sequence to start the optimization.
- `max_iter`, default `30`: Maximum number of iterations for the Expectation-Maximization procedure.
- `tol`, default `1e-05`: Tolerance for early stopping based on the decrease in cost.
- `weights`, default `None`: Weights for each time series in `X`. Must match `len(X)`.
- `metric_params`, default `None`: Dictionary of DTW constraints (e.g., `sakoe_chiba_radius`).
- `verbose`, default `False`: Whether to print optimization progress.
- `n_jobs`, default `None`: Number of parallel jobs to run.

### Input
Input must be a 3D array of shape `(n_ts, sz, d)`. If the data is not in this format, use `tslearn.utils.to_time_series_dataset` to convert it before passing to the function.

### Output
Returns a `numpy.array` of shape `(barycenter_size, d)` representing the computed DBA barycenter.

### Valid Call Patterns
```python
import numpy as np
from tslearn.barycenters import dtw_barycenter_averaging_petitjean

# Example from test suite
n, sz, d = 15, 10, 3
time_series = np.random.randn(n, sz, d)
dba_bar = dtw_barycenter_averaging_petitjean(time_series, max_iter=5)
```

### LLM Instruction Prompt
- Always ensure input `X` is a 3D array.
- Use `max_iter=0` if you require the Euclidean barycenter as a baseline.
- When using `n_jobs=-1`, ensure the environment supports parallel execution via `joblib`.

### Prompt Snippet
```text
# Example usage:
from tslearn.barycenters import dtw_barycenter_averaging_petitjean
result = dtw_barycenter_averaging_petitjean(X, max_iter=10)
```

### Common Failure Modes
- Passing 2D arrays directly without using `to_time_series_dataset` will cause shape mismatch errors.
- Setting `max_iter` too low may result in a barycenter that has not converged to a local minimum.
- Providing `weights` that do not match the number of time series in `X` will raise a `ValueError`.

### Fix Code Hint
```python
# Ensure 3D shape for input
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
result = dtw_barycenter_averaging_petitjean(X_formatted, max_iter=30)
```

## API Test: `dtw_barycenter_averaging_subgradient`

### Signature
```python
def dtw_barycenter_averaging_subgradient(X, barycenter_size=None, init_barycenter=None, max_iter=30, initial_step_size=0.05, final_step_size=0.005, tol=1e-05, random_state=None, weights=None, metric_params=None, verbose=False)
```
_Source: tslearn/tslearn/barycenters/dba.py:740_

_Source doc:_ DTW Barycenter Averaging (DBA) method estimated through subgradient descent algorithm. DBA was originally presented in [1]_. This implementation is based on a idea from [2]_ (Stochastic Subgradient Mean Algorithm). Parameters ---------- X : array-like, shape=(n_ts, sz, d) Time series dataset. barycenter_size : int or None (default: None) Size of the barycenter to generate. If None, the size of the barycenter is that of the data provided at fit time or that of the initial barycenter if specified. init_barycenter : array or None (default: None) Initial barycenter to start from for the optimization process. max_iter : int (default: 30) Number of iterations of the Expectation-Maximization optimization procedure. initial_step_size : float (default: 0.05) Initial step size for the subgradient descent algorithm. Default value is the one suggested in [2]_. final_step_size : float (default: 0.005) Final step size for the subgradient descent algorithm. Default value is the one suggested in [2]_. tol : float (default: 1e-5) Tolerance to use for early stopping: if the decrease in cost is lower than this value, the Expectation-Maximization procedure stops. random_state : int, RandomState instance or None, optional (default=None) If int, random_state is the seed used by the random number generator; If RandomState instance, random_state is the random number generator; If None, the random number generator is the RandomState instance used by `np.random`. weights: None or array Weights of each X[i]. Must be the same size as len(X). If None, uniform weights are used. metric_params: dict or None (default: None) DTW constraint parameters to be used. See :ref:`tslearn.metrics.dtw_path <fun-tslearn.metrics.dtw_path>` for a list of accepted parameters If None, no constraint is used for DTW computations. verbose : boolean (default: False) Whether to print information about the cost at each iteration or not. Returns ------- numpy.array of shape (barycenter_size, d) or (sz, d) if barycenter_size \ is None

### Goal
Computes the DTW Barycenter Averaging (DBA) of a set of time series using a stochastic subgradient descent optimization approach.

### Parameters
- `X`: Time series dataset, array-like of shape `(n_ts, sz, d)`.
- `barycenter_size`, default `None`: Target length of the resulting barycenter. If `None`, defaults to the length of the input time series.
- `init_barycenter`, default `None`: Initial seed for the barycenter optimization.
- `max_iter`, default `30`: Maximum number of iterations for the subgradient descent.
- `initial_step_size`, default `0.05`: Starting step size for the subgradient descent.
- `final_step_size`, default `0.005`: Final step size for the subgradient descent.
- `tol`, default `1e-05`: Tolerance for early stopping based on cost decrease.
- `random_state`, default `None`: Seed or `RandomState` object for reproducibility.
- `weights`, default `None`: Array of weights for each time series in `X`. Must match `len(X)`.
- `metric_params`, default `None`: Dictionary of parameters passed to the DTW metric (e.g., `global_constraint`, `sakoe_chiba_radius`).
- `verbose`, default `False`: If `True`, prints optimization progress.

### Input
Requires a 3D NumPy array of shape `(n_ts, sz, d)`. If data is variable-length or 1D/2D, it must be pre-processed using `tslearn.utils.to_time_series_dataset` to ensure the 3D structure is maintained.

### Output
Returns a `numpy.array` of shape `(barycenter_size, d)` representing the computed barycenter.

### Valid Call Patterns
```python
import numpy as np
from tslearn.barycenters import dtw_barycenter_averaging_subgradient

# Inferred from signature:
X = np.random.rand(10, 20, 1)
barycenter = dtw_barycenter_averaging_subgradient(X, max_iter=10, verbose=True)
print(f"Barycenter shape: {barycenter.shape}")
```

### LLM Instruction Prompt
- Always ensure `X` is a 3D array.
- Use `metric_params` to pass constraints like `{"global_constraint": "sakoe_chiba", "sakoe_chiba_radius": 2}`.
- If reproducibility is required, always provide an integer to `random_state`.

### Prompt Snippet
```text
from tslearn.barycenters import dtw_barycenter_averaging_subgradient
# X: (n_ts, sz, d)
# barycenter = dtw_barycenter_averaging_subgradient(X, ...)
```

### Common Failure Modes
- Passing 2D arrays directly without using `to_time_series_dataset`.
- Mismatch between `weights` length and `n_ts`.
- Setting `max_iter` too low, resulting in poor convergence.

### Fix Code Hint
```python
# Ensure input is 3D
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
barycenter = dtw_barycenter_averaging_subgradient(X_formatted)
```

## API Test: `dtw_limited_warping_length`

### Signature
```python
def dtw_limited_warping_length(s1, s2, max_length, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:922_

_Source doc:_ Compute Dynamic Time Warping (DTW) similarity measure between (possibly multidimensional) time series under an upper bound constraint on the resulting path length and return the similarity cost. DTW is computed as the Euclidean distance between aligned time series, i.e., if :math:`\pi` is the optimal alignment path: .. math:: DTW(X, Y) = \sqrt{\sum_{(i, j) \in \pi} \|X_{i} - Y_{j}\|^2} Note that this formula is still valid for the multivariate case. It is not required that both time series share the same size, but they must be the same dimension. DTW was originally presented in [1]_. This constrained-length variant was introduced in [2]_. Both bariants are discussed in more details in our :ref:`dedicated user-guide page <dtw>` Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. max_length : int Maximum allowed warping path length. If greater than len(s1) + len(s2), then it is equivalent to unconstrained DTW. If lower than max(len(s1), len(s2)), no path can be found and a ValueError is raised. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- float Similarity score Examples -------- >>> float(dtw_limited_warping_length([1, 2, 3], [1., 2., 2., 3.], 5)) 0.0 >>> float(dtw_limited_warping_length([1, 2, 3], [1., 2., 2., 3., 4.], 5)) 1.0 See Also -------- dtw : Get the similarity score for DTW dtw_path_limited_warping_length : Get both the warping path and the similarity score for DTW with limited warping path length References ---------- .. [1] H. Sakoe, S. Chiba, "Dynamic programming algorithm optimization for

### Goal
Computes the DTW similarity score between two time series subject to a hard constraint on the maximum number of steps allowed in the optimal alignment path.

### Parameters
- `s1`: array-like, shape=(sz1, d) or (sz1,). The first time series.
- `s2`: array-like, shape=(sz2, d) or (sz2,). The second time series.
- `max_length`: int. The maximum allowed length of the warping path.
- `be`, default `None`: Backend object, string ("numpy" or "pytorch"), or None. Determines the computation engine.

### Input
- **Data:** Two time series of shape `(sz, d)`. If univariate, `(sz,)` is accepted.
- **Preconditions:** Both time series must have the same dimension `d`.
- **Backend:** If using `PyTorch`, ensure the `pytorch` package is installed.

### Output
Returns `float`: The DTW similarity score (Euclidean distance of the optimal path under the length constraint).

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

x = np.random.randn(15, 3)
y = np.random.randn(10, 3)
# Standard call with NumPy backend
score = tslearn.metrics.dtw_limited_warping_length(x, y, max_length=20)
print(f"Correctness witness: {score >= 0.0}")
```

### LLM Instruction Prompt
- Always verify that `max_length` is at least `max(len(s1), len(s2))`.
- If the backend is not specified, the function will infer it from the input array types (NumPy vs PyTorch).
- Use this function when you need to restrict the "flexibility" of the alignment path to prevent excessive warping.

### Prompt Snippet
```text
tslearn.metrics.dtw_limited_warping_length(s1, s2, max_length, be=None)
```

### Common Failure Modes
- `ValueError`: Raised if `max_length` is less than `max(len(s1), len(s2))`, as no valid path can exist.
- `ValueError`: Raised if `s1` and `s2` have different dimensions `d`.
- `TypeError`: Occurs if inputs are not array-like or if the backend is incompatible with the input types.

### Fix Code Hint
```python
# Ensure max_length is sufficient
min_required = max(len(s1), len(s2))
if max_length < min_required:
    max_length = min_required
score = tslearn.metrics.dtw_limited_warping_length(s1, s2, max_length)
```

## API Test: `dtw_path`

### Signature
```python
def dtw_path(s1, s2, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:299  (+1 more definition site/overload)_

_Source doc:_ Compute Dynamic Time Warping (DTW) similarity measure between (possibly multidimensional) time series and return both the path and the similarity. DTW is computed as the Euclidean distance between aligned time series, i.e., if :math:`\pi` is the alignment path: .. math:: DTW(X, Y) = \sqrt{\sum_{(i, j) \in \pi} (X_{i} - Y_{j})^2} It is not required that both time series share the same size, but they must be the same dimension. DTW was originally presented in [1]_ and is discussed in more details in our :ref:`dedicated user-guide page <dtw>`. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for DTW. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- list of integer pairs Matching path represented as a list of index pairs. In each pair, the first index corresponds to s1 and the second one corresponds to s2. float Similarity score

### Goal
Computes the optimal alignment path and the corresponding DTW similarity score between two time series, supporting both NumPy and PyTorch backends.

### Parameters
- `s1`: Array-like, shape `(sz1, d)` or `(sz1,)`. The first time series.
- `s2`: Array-like, shape `(sz2, d)` or `(sz2,)`. The second time series. Must have the same dimension `d` as `s1`.
- `global_constraint`, default `None`: String, either `"itakura"` or `"sakoe_chiba"`, to restrict the warping path.
- `sakoe_chiba_radius`, default `None`: Integer. The radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float. The maximum slope for the Itakura parallelogram constraint.
- `be`, default `None`: Backend object or string (`"numpy"` or `"pytorch"`). Determines the computation engine.

### Input
Input time series must be array-like. While `tslearn` estimators typically require 3D arrays `(n_ts, sz, d)`, `dtw_path` accepts individual time series as 1D or 2D arrays. If using the `PyTorch` backend, inputs should be `torch.Tensor` objects.

### Output
Returns a tuple: `(path, similarity)`, where `path` is a list of `(i, j)` index pairs representing the alignment, and `similarity` is a float representing the DTW distance.

### Valid Call Patterns
```python
import tslearn.metrics
path, dist = tslearn.metrics.dtw_path([1, 2, 3], [1.0, 2.0, 2.0, 3.0])
# Output: ([(0, 0), (1, 1), (1, 2), (2, 3)], 0.0)
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` have the same dimensionality `d`.
- If using `PyTorch` tensors, explicitly set `be="pytorch"` or ensure the backend is correctly inferred.
- When applying constraints, ensure `global_constraint` matches the provided `sakoe_chiba_radius` or `itakura_max_slope` to avoid `RuntimeWarning`.

### Prompt Snippet
```text
path, dist = tslearn.metrics.dtw_path(s1, s2, global_constraint="sakoe_chiba", sakoe_chiba_radius=2)
```

### Common Failure Modes
- `ValueError`: Raised if inputs are empty or have mismatched dimensions.
- `RuntimeWarning`: Raised if both `sakoe_chiba_radius` and `itakura_max_slope` are provided without a valid `global_constraint`.
- `TypeError`: Occurs if the backend `be` is incompatible with the input array types (e.g., passing NumPy arrays to a PyTorch-only operation).

### Fix Code Hint
```python
import numpy as np
from tslearn.metrics import dtw_path

# Ensure inputs are valid arrays
s1 = np.array([1, 2, 3])
s2 = np.array([1, 2, 2, 3])
path, dist = dtw_path(s1, s2)
print(f"Path: {path}, Distance: {dist}")
```

## API Test: `dtw_path_from_metric`

### Signature
```python
def dtw_path_from_metric(s1, s2=None, metric='euclidean', global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None, **kwds)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:495  (+1 more definition site/overload)_

_Source doc:_ Compute Dynamic Time Warping (DTW) similarity measure between (possibly multidimensional) time series using a distance metric defined by the user and return both the path and the similarity. Similarity is computed as the cumulative cost along the aligned time series. It is not required that both time series share the same size, but they must be the same dimension. DTW was originally presented in [1]_. Valid values for metric are the same as for scikit-learn `pairwise_distances`_ function i.e. a string (e.g. "euclidean", "sqeuclidean", "hamming") or a function that is used to compute the pairwise distances. See `scikit`_ and `scipy`_ documentations for more information about the available metrics. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) if metric!="precomputed", (sz1, sz2) otherwise A time series or an array of pairwise distances between samples. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,), optional (default: None) A second time series, only allowed if metric != "precomputed". If shape is (sz2,), the time series is assumed to be univariate. metric : string or callable (default: "euclidean") Function used to compute the pairwise distances between each points of `s1` and `s2`. If metric is "precomputed", `s1` is assumed to be a distance matrix. If metric is an other string, it must be one of the options compatible with sklearn.metrics.pairwise_distances. Alternatively, if metric is a callable function, it is called on pairs of rows of `s1` and `s2`. The callable should take two 1 dimensional arrays as input and return a value indicating the distance between them. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for DTW. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used.

### Goal
Computes the optimal alignment path and cumulative similarity score between two time series using a user-defined distance metric, supporting global constraints like Sakoe-Chiba or Itakura.

### Parameters
- `s1`: The first time series (shape `(sz1, d)`) or a precomputed distance matrix (shape `(sz1, sz2)` if `metric="precomputed"`).
- `s2`, default `None`: The second time series (shape `(sz2, d)`). Required unless `metric="precomputed"`.
- `metric`, default `'euclidean'`: Distance metric string (compatible with `sklearn.metrics.pairwise_distances`) or a callable function accepting two 1D arrays.
- `global_constraint`, default `None`: Constraint type to restrict the warping path; options are `"itakura"` or `"sakoe_chiba"`.
- `sakoe_chiba_radius`, default `None`: Integer radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float slope value for the Itakura parallelogram constraint.
- `be`, default `None`: Backend object (e.g., `NumPyBackend` or `PyTorchBackend`) used for computation.
- `**kwds`: Additional keyword arguments passed to the distance metric function.

### Input
Requires 1D or 2D array-like structures. If data is 3D (dataset), it must be indexed to provide individual time series. Ensure `s1` and `s2` have the same dimensionality `d`.

### Output
Returns a tuple `(path, similarity)`, where `path` is a list of `(index1, index2)` tuples representing the alignment, and `similarity` is the cumulative cost (float).

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np
s1 = np.random.rand(10, 2)
s2 = np.random.rand(15, 2)
path, dist = tslearn.metrics.dtw_path_from_metric(s1, s2, metric="sqeuclidean")
print(f"Path length: {len(path)}, Distance: {dist}")
```

### LLM Instruction Prompt
- Always specify the `metric` if not using the default Euclidean distance.
- If using `global_constraint`, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is provided or understood to default.
- When using `PyTorch` tensors, pass the appropriate backend via `be` to ensure compatibility.

### Prompt Snippet
```text
# Example of using dtw_path_from_metric with a custom metric
path, dist = tslearn.metrics.dtw_path_from_metric(s1, s2, metric="cityblock")
```

### Common Failure Modes
- Mismatched dimensions: `s1` and `s2` must have the same number of features `d`.
- Invalid metric: Providing a string not supported by `sklearn.metrics.pairwise_distances`.
- Backend mismatch: Passing `PyTorch` tensors without specifying a `PyTorch` backend.

### Fix Code Hint
```python
# Ensure inputs are 2D (n_timesteps, n_features)
s1 = s1.reshape(-1, 1) if s1.ndim == 1 else s1
s2 = s2.reshape(-1, 1) if s2.ndim == 1 else s2
path, dist = tslearn.metrics.dtw_path_from_metric(s1, s2, metric="euclidean")
```

## API Test: `dtw_path_limited_warping_length`

### Signature
```python
def dtw_path_limited_warping_length(s1, s2, max_length, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1063_

_Source doc:_ Compute Dynamic Time Warping (DTW) similarity measure between (possibly multidimensional) time series under an upper bound constraint on the resulting path length and return the path as well as the similarity cost. DTW is computed as the Euclidean distance between aligned time series, i.e., if :math:`\pi` is the optimal alignment path: .. math:: DTW(X, Y) = \sqrt{\sum_{(i, j) \in \pi} \|X_{i} - Y_{j}\|^2} Note that this formula is still valid for the multivariate case. It is not required that both time series share the same size, but they must be the same dimension. DTW was originally presented in [1]_. This constrained-length variant was introduced in [2]_. Both variants are discussed in more details in our :ref:`dedicated user-guide page <dtw>` Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. max_length : int Maximum allowed warping path length. If greater than len(s1) + len(s2), then it is equivalent to unconstrained DTW. If lower than max(len(s1), len(s2)), no path can be found and a ValueError is raised. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- list of integer pairs Optimal path float Similarity score Examples -------- >>> path, cost = dtw_path_limited_warping_length([1, 2, 3], ...                                              [1., 2., 2., 3.], 5) >>> float(cost) 0.0 >>> path [(0, 0), (1, 1), (1, 2), (2, 3)] >>> path, cost = dtw_path_limited_warping_length([1, 2, 3], ...                                              [1., 2., 2., 3., 4.], 5) >>> float(cost) 1.0 >>> path

### Goal
Computes the Dynamic Time Warping (DTW) similarity and the corresponding optimal alignment path between two time series, subject to a strict upper bound on the total number of steps in the warping path.

### Parameters
- `s1`: Array-like, shape=(sz1, d) or (sz1,). The first time series.
- `s2`: Array-like, shape=(sz2, d) or (sz2,). The second time series.
- `max_length`: Integer. The maximum allowed length of the warping path.
- `be`, default `None`: Backend object or string ("numpy" or "pytorch"). Determines the computation engine.

### Input
- **Data:** Time series must be provided as array-like structures. If univariate, shape is `(sz,)`; if multivariate, shape is `(sz, d)`.
- **Preconditions:** Both `s1` and `s2` must have the same dimension `d`.
- **Constraint:** `max_length` must be at least `max(len(s1), len(s2))`. If `max_length` is less than this, a `ValueError` is raised.

### Output
Returns a tuple containing:
- `path`: A list of integer pairs `(i, j)` representing the optimal alignment path.
- `cost`: A float representing the DTW similarity score.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

s1 = [1, 2, 3]
s2 = [1., 2., 2., 3.]
path, cost = tslearn.metrics.dtw_path_limited_warping_length(s1, s2, max_length=5)
print(f"Cost: {cost}, Path: {path}")
```

### LLM Instruction Prompt
- Always ensure `max_length` is an integer greater than or equal to the maximum length of the two input series.
- If using PyTorch tensors, explicitly set `be="pytorch"` or ensure the backend is correctly inferred.
- The output is a tuple: index 0 is the path (list of tuples), index 1 is the cost (float).

### Prompt Snippet
```text
path, cost = tslearn.metrics.dtw_path_limited_warping_length(s1, s2, max_length=N)
```

### Common Failure Modes
- `ValueError`: Raised if `max_length` is smaller than `max(len(s1), len(s2))`.
- `ValueError`: Raised if the dimensions `d` of `s1` and `s2` do not match.
- `TypeError`: Occurs if inputs are not array-like or if `max_length` is not an integer.

### Fix Code Hint
```python
# Ensure max_length is valid
max_len = max(len(s1), len(s2))
if max_length < max_len:
    max_length = max_len
path, cost = tslearn.metrics.dtw_path_limited_warping_length(s1, s2, max_length)
```

## API Test: `dtw_subsequence_path`

### Signature
```python
def dtw_subsequence_path(subseq, longseq, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1417_

_Source doc:_ Compute sub-sequence Dynamic Time Warping (DTW) similarity measure between a (possibly multidimensional) query and a long time series and return both the path and the similarity. DTW is computed as the Euclidean distance between aligned time series, i.e., if :math:`\pi` is the alignment path: .. math:: DTW(X, Y) = \sqrt{\sum_{(i, j) \in \pi} \|X_{i} - Y_{j}\|^2} Compared to traditional DTW, here, border constraints on admissible paths :math:`\pi` are relaxed such that :math:`\pi_0 = (0, ?)` and :math:`\pi_L = (N-1, ?)` where :math:`L` is the length of the considered path and :math:`N` is the length of the subsequence time series. It is not required that both time series share the same size, but they must be the same dimension. This implementation finds the best matching starting and ending positions for `subseq` inside `longseq`. Parameters ---------- subseq : array-like, shape=(sz1, d) or (sz1,) A query time series. If shape is (sz1,), the time series is assumed to be univariate. longseq : array-like, shape=(sz2, d) or (sz2,) A reference (supposed to be longer than `subseq`) time series. If shape is (sz2,), the time series is assumed to be univariate. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- list of integer pairs Matching path represented as a list of index pairs. In each pair, the first index corresponds to `subseq` and the second one corresponds to `longseq`. float Similarity score Examples -------- >>> path, dist = dtw_subsequence_path([2., 3.], [1., 2., 2., 3., 4.]) >>> path [(0, 2), (1, 3)] >>> float(dist) 0.0 See Also -------- dtw : Get the similarity score for DTW subsequence_cost_matrix: Calculate the required cost matrix subsequence_path: Calculate a matching path manually

### Goal
Finds the optimal subsequence alignment of a query time series within a longer reference time series by relaxing the start and end boundary constraints of standard DTW.

### Parameters
- `subseq`: The query time series (shape `(sz1, d)` or `(sz1,)`).
- `longseq`: The reference time series (shape `(sz2, d)` or `(sz2,)`), typically longer than `subseq`.
- `be`, default `None`: The computation backend; accepts `None`, `"numpy"`, `"pytorch"`, or specific backend instances.

### Input
- **Data Format:** Accepts array-like structures (NumPy arrays or PyTorch tensors).
- **Dimensionality:** Both `subseq` and `longseq` must share the same number of dimensions `d`.
- **Preconditions:** If using the `PyTorch` backend, ensure the `pytorch` package is installed.

### Output
Returns a tuple containing:
- `path`: A list of `(i, j)` integer pairs representing the optimal alignment path.
- `dist`: A float representing the DTW similarity score (Euclidean distance of the alignment).

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Standard usage with NumPy
path, dist = tslearn.metrics.dtw_subsequence_path([2., 3.], [1., 2., 2., 3., 4.])
print(f"Path: {path}, Distance: {dist}")
# Witness: Path: [(0, 2), (1, 3)], Distance: 0.0
```

### LLM Instruction Prompt
- Always import `tslearn.metrics` to access the function.
- Ensure `subseq` and `longseq` have matching dimensions.
- If performing differentiable operations, explicitly set `be="pytorch"`.

### Prompt Snippet
```text
from tslearn.metrics import dtw_subsequence_path
path, dist = dtw_subsequence_path(subseq_data, longseq_data)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing sequences with different feature dimensions `d` will raise a `ValueError`.
- **Backend Incompatibility:** Passing a PyTorch tensor while the default NumPy backend is active (or vice versa) without specifying `be` may lead to type errors.
- **Length Constraint:** While `longseq` is expected to be longer, providing a `subseq` longer than `longseq` may result in unexpected alignment behavior or errors depending on the implementation.

### Fix Code Hint
```python
# Ensure inputs are properly shaped for the backend
import numpy as np
subseq = np.array([2., 3.]).reshape(-1, 1)
longseq = np.array([1., 2., 2., 3., 4.]).reshape(-1, 1)
path, dist = tslearn.metrics.dtw_subsequence_path(subseq, longseq)
```

## API Test: `dual_coef_`

### Signature
```python
def dual_coef_(self)
```
_Source: tslearn/tslearn/svm/svm.py:29_

### Goal
Retrieves the dual coefficients of the support vectors in a fitted time-series SVM model.

### Parameters
- `self`: An instance of a fitted `tslearn` SVM estimator (e.g., `TimeSeriesSVC`).

### Input
- **Preconditions:** The estimator must have been successfully trained using the `fit()` method on a 3D NumPy array of shape `(n_ts, max_sz, d)`.
- **Data Format:** The model must be in a state where the optimization problem has been solved, populating the internal dual coefficients.

### Output
Returns `unspecified` — The dual coefficients (Lagrange multipliers) associated with the support vectors, typically returned as a NumPy array.

### Valid Call Patterns
```python
# Inferred from signature: requires a fitted estimator instance
# Example:
# from tslearn.svm import TimeSeriesSVC
# clf = TimeSeriesSVC().fit(X, y)
# dual_coeffs = clf.dual_coef_
```

### LLM Instruction Prompt
- Only call `dual_coef_` on an estimator instance that has already been fitted via `fit()`.
- Do not attempt to call this as a standalone function; it is an attribute/property accessor on `tslearn` SVM estimators.
- Ensure the input data used for `fit` was a 3D NumPy array as per `tslearn` requirements.

### Prompt Snippet
```text
# Accessing dual coefficients after training
dual_coeffs = estimator.dual_coef_
```

### Common Failure Modes
- **NotFittedError:** Calling `dual_coef_` before the `fit()` method has been executed on the estimator.
- **AttributeError:** Attempting to access this property on a `tslearn` estimator that does not implement SVM-based dual optimization (e.g., calling it on a `TimeSeriesKMeans` object).
- **Shape Mismatch:** Assuming a specific dimensionality for the returned array without checking the number of support vectors identified during training.

### Fix Code Hint
```python
# Ensure the model is fitted before accessing dual coefficients
if hasattr(clf, "dual_coef_"):
    print(clf.dual_coef_)
else:
    print("Model has not been fitted or does not support dual coefficients.")
```

## API Test: `early_classification_cost`

### Signature
```python
def early_classification_cost(self, X, y)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:517_

_Source doc:_ Compute early classification score. The score is computed as: .. math:: 1 - acc + \alpha \frac{1}{n} \sum_i t_i where :math:`\alpha` is the trade-off parameter (`self.cost_time_parameter`) and :math:`t_i` are prediction timestamps. Parameters ---------- X : array-like of shape (n_series, n_timestamps, n_features) Vector to be scored, where `n_series` is the number of time series, `n_timestamps` is the number of timestamps in the series and `n_features` is the number of features recorded at each timestamp. y : array-like, shape = (n_samples) or (n_samples, n_outputs) True labels for X. Returns ------- float Early classification cost (a positive number, the lower the better) Examples -------- >>> dataset = to_time_series_dataset([[1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [3, 2, 1, 1, 2, 3], ...                                   [3, 2, 1, 1, 2, 3]]) >>> y = [0, 0, 0, 1, 1, 1, 0, 0] >>> model = NonMyopicEarlyClassifier(n_clusters=3, lamb=1000., ...                                  cost_time_parameter=.1, ...                                  random_state=0) >>> model.fit(dataset, y)  # doctest: +ELLIPSIS NonMyopicEarlyClassifier(...) >>> preds, pred_times = model.predict_class_and_earliness(dataset) >>> preds array([0, 0, 0, 1, 1, 1, 0, 0]) >>> pred_times array([4, 4, 4, 4, 4, 4, 1, 1]) >>> float(model.early_classification_cost(dataset, y)) 0.325

### Goal
Computes a scalar cost metric for an early classification model that balances classification accuracy against the earliness of predictions.

### Parameters
- `self`: The fitted `NonMyopicEarlyClassifier` instance.
- `X`: Array-like of shape `(n_series, n_timestamps, n_features)` containing the time-series data to be evaluated.
- `y`: Array-like of shape `(n_samples)` or `(n_samples, n_outputs)` containing the true labels for the input series.

### Input
- **Data Format:** Must be a 3D NumPy array. If input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this method.
- **Preconditions:** The model must be fitted using the `fit` method before calling `early_classification_cost`.

### Output
- Returns a `float` representing the early classification cost. This is a positive value where lower scores indicate a better balance between accuracy and prediction speed.

### Valid Call Patterns
```python
import numpy as np
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.utils import to_time_series_dataset

# Assuming dataset and y are defined as per the source doc
model = NonMyopicEarlyClassifier(cost_time_parameter=0.1)
model.fit(dataset, y)
cost = model.early_classification_cost(dataset, y)
print(f"Cost: {cost}")
```

### LLM Instruction Prompt
- Always ensure the model is fitted before calling `early_classification_cost`.
- Ensure the input `X` is a 3D array formatted for `tslearn` estimators.
- Use the `cost_time_parameter` attribute of the model to understand the weight assigned to the earliness component of the cost.

### Prompt Snippet
```text
# Calculate the early classification cost for a fitted model
cost = model.early_classification_cost(X_test, y_test)
```

### Common Failure Modes
- **Not Fitted:** Calling the method before `fit()` will result in an `AttributeError` or `NotFittedError` depending on the internal state.
- **Shape Mismatch:** Passing a 2D array instead of a 3D array will cause a dimension error; ensure `to_time_series_dataset` is used.
- **Label Mismatch:** Providing `y` with a different number of samples than `X` will raise a `ValueError`.

### Fix Code Hint
```python
# Ensure data is 3D and model is fitted
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
model.fit(X_formatted, y)
cost = model.early_classification_cost(X_formatted, y)
```

## API Test: `early_predict`

### Signature
```python
def early_predict(self, X)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:574_

_Source doc:_ Provides predicted classes as well as estimated delays before prediction timestamps for a dataset of incomplete time series. Prediction timestamps are timestamps at which a prediction is made in early classification setting. Parameters ---------- X : array-like of shape (n_series, t, n_features) A dataset of incomplete time series observed up to time t Returns ------- array-like, shape (n_series,) Predicted classes. array-like, shape (n_series,) Estimated delays before prediction timestamps.

### Goal
Performs early classification on incomplete time-series data to predict class labels and estimate the remaining time delay required before a confident prediction can be made.

### Parameters
- `self`: An instance of an early classification estimator (e.g., `NonMyopicEarlyClassifier`) that has already been fitted to a training dataset.
- `X`: array-like of shape `(n_series, t, n_features)`. The dataset of incomplete time series observed up to time `t`.

### Input
- **Data Format:** Must be a 3D NumPy array of shape `(n_series, t, n_features)`.
- **Preconditions:** The estimator must be fitted using `fit(X, y)` prior to calling `early_predict`.
- **Compatibility:** If input data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` before passing to the method.

### Output
Returns a tuple containing:
- `predicted_classes`: array-like of shape `(n_series,)` containing the predicted class labels.
- `estimated_delays`: array-like of shape `(n_series,)` containing the estimated number of time steps remaining before the prediction timestamp.

### Valid Call Patterns
```python
# Example using NonMyopicEarlyClassifier
pred, delays = model.early_predict(dataset[:, :3])
```

### LLM Instruction Prompt
- When calling `early_predict`, ensure the input `X` is a 3D array representing incomplete time series.
- Always handle the returned tuple as `(predictions, delays)`.
- If the time series length `t` is less than the `min_t` parameter configured during estimator initialization, the method may return `np.nan` for predictions and delays.

### Prompt Snippet
```text
# Ensure model is fitted before calling
model.fit(X_train, y_train)
# Predict on incomplete data
predictions, delays = model.early_predict(X_incomplete)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will raise a `ValueError`.
- **Unfitted Estimator:** Calling `early_predict` before `fit` will result in an `AttributeError` or `NotFittedError`.
- **Insufficient Length:** Providing time series shorter than the `min_t` threshold defined in the estimator will result in `np.nan` outputs for those specific series.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset

# Ensure 3D shape: (n_series, time_steps, n_features)
X_formatted = to_time_series_dataset(X_raw)
# Check if model is fitted
if model.classes_ is not None:
    pred, delays = model.early_predict(X_formatted)
```

## API Test: `early_predict_proba`

### Signature
```python
def early_predict_proba(self, X)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:600_

_Source doc:_ Provides probability estimates as well as estimated delays before prediction timestamps for a dataset of incomplete time series. Prediction timestamps are timestamps at which a prediction is made in early classification setting. Parameters ---------- X : array-like of shape (n_series, t, n_features) A dataset of incomplete time series observed up to time t Returns ------- array-like, shape (n_series, n_classes) Probabilities for each class in the model, where classes are ordered as they are in ``self.classes_``. array-like of shape (n_series,) Estimated delays before prediction timestamps.

### Goal
Computes class probability estimates and the associated decision delays for incomplete time-series data within an early classification framework.

### Parameters
- `self`: An instance of an early classification estimator (e.g., `NonMyopicEarlyClassifier`) that has already been fitted to training data.
- `X`: array-like of shape `(n_series, t, n_features)`. A dataset of incomplete time series observed up to time `t`.

### Input
- **Data Format:** Must be a 3D NumPy array of shape `(n_series, t, n_features)`.
- **Preconditions:** The estimator must be fitted using `fit(X, y)` prior to calling this method. Input data should be preprocessed (e.g., scaled) consistently with the training data.

### Output
Returns a tuple containing:
- `probabilities`: array-like of shape `(n_series, n_classes)`, representing the probability of each class.
- `delays`: array-like of shape `(n_series,)`, representing the estimated time delay before the prediction timestamp.

### Valid Call Patterns
```python
import numpy as np
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
from tslearn.utils import to_time_series_dataset

# Setup and fit
dataset = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3, 3]])
y = [0, 1]
model = NonMyopicEarlyClassifier(n_clusters=2, base_classifier=KNeighborsTimeSeriesClassifier())
model.fit(dataset, y)

# Call
probs, delays = model.early_predict_proba(dataset[:, :3])
print(f"Probs shape: {probs.shape}, Delays shape: {delays.shape}")
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array. If the input is 1D or 2D, use `tslearn.utils.to_time_series_dataset` first.
- Ensure the estimator is fitted before calling `early_predict_proba`.
- The method returns two values: a probability matrix and a delay vector.

### Prompt Snippet
```text
# Example of calling early_predict_proba
probs, delays = estimator.early_predict_proba(X_incomplete)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will raise a `ValueError`.
- **Unfitted Estimator:** Calling the method before `fit()` will result in an `AttributeError` or `NotFittedError` depending on the underlying scikit-learn implementation.
- **Inconsistent Timestamps:** Providing `X` with a time dimension `t` smaller than the `min_t` parameter configured during estimator initialization may return `NaN` values.

### Fix Code Hint
```python
# Ensure 3D shape for incomplete series
X_3d = to_time_series_dataset(X_incomplete)
# Check if model is fitted
if hasattr(model, "classes_"):
    probs, delays = model.early_predict_proba(X_3d)
```

## API Test: `euclidean_barycenter`

### Signature
```python
def euclidean_barycenter(X, weights=None)
```
_Source: tslearn/tslearn/barycenters/euclidean.py:9_

_Source doc:_ Standard Euclidean barycenter computed from a set of time series. Parameters ---------- X : array-like, shape=(n_ts, sz, d) Time series dataset. weights: None or array Weights of each X[i]. Must be the same size as len(X). If None, uniform weights are used. Returns ------- numpy.array of shape (sz, d) Barycenter of the provided time series dataset. Notes ----- This method requires a dataset of equal-sized time series Examples -------- >>> time_series = [[1, 2, 3, 4], [1, 2, 4, 5]] >>> bar = euclidean_barycenter(time_series) >>> bar.shape (4, 1) >>> bar array([[1. ], [2. ], [3.5], [4.5]])

### Goal
Computes the arithmetic mean (barycenter) of a set of time series under the Euclidean distance metric.

### Parameters
- `X`: array-like, shape=(n_ts, sz, d). The time series dataset to average.
- `weights`, default `None`: array-like, shape=(n_ts,). Weights for each time series in `X`. If `None`, uniform weights (1/n_ts) are applied.

### Input
- **Format:** 3D NumPy array of shape `(n_ts, sz, d)`.
- **Preconditions:** All time series in `X` must have the same length (`sz`). If data is 1D or 2D, it must be converted to the standard `tslearn` 3D format using `tslearn.utils.to_time_series_dataset` before calling this function.

### Output
Returns `numpy.array` of shape `(sz, d)` representing the computed barycenter.

### Valid Call Patterns
```python
import numpy as np
import tslearn.barycenters
from tslearn.utils import to_time_series_dataset

# Standard usage with uniform weights
X = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 4, 5]])
bar = tslearn.barycenters.euclidean_barycenter(X)
print(f"Barycenter shape: {bar.shape}")

# Usage with custom weights
weights = np.array([0.2, 0.8])
bar_weighted = tslearn.barycenters.euclidean_barycenter(X, weights=weights)
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array of shape `(n_ts, sz, d)`.
- If `weights` are provided, ensure `len(weights) == X.shape[0]`.
- Do not use this function for variable-length time series; it requires equal-sized sequences.

### Prompt Snippet
```text
tslearn.barycenters.euclidean_barycenter(X, weights=None)
```

### Common Failure Modes
- **Shape Mismatch:** Passing time series of different lengths will result in a `ValueError` or incorrect array broadcasting.
- **Dimension Mismatch:** Passing 2D arrays without converting them to 3D `(n_ts, sz, d)` format via `to_time_series_dataset`.
- **Weight Mismatch:** Providing a `weights` array that does not match the number of time series (`n_ts`) in `X`.

### Fix Code Hint
```python
# Ensure 3D shape and equal lengths
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
# If lengths differ, resample or truncate before calling
bar = tslearn.barycenters.euclidean_barycenter(X_formatted)
```

## API Test: `extract_from_zip_url`

### Signature
```python
def extract_from_zip_url(url, target_dir=None, verbose=False)
```
_Source: tslearn/tslearn/datasets/datasets.py:16_

_Source doc:_ Download a zip file from its URL and unzip it. A `RuntimeWarning` is printed on failure. Parameters ---------- url : string URL from which to download. target_dir : str or None (default: None) Directory to be used to extract unzipped downloaded files. verbose : bool (default: False) Whether to print information about the process (cached files used, ...) Returns ------- str or None Directory in which the zip file has been extracted if the process was successful, None otherwise

### Goal
Automates the retrieval and local extraction of remote time-series datasets provided as compressed zip archives.

### Parameters
- `url`: The string URL pointing to the zip file to be downloaded.
- `target_dir`, default `None`: The local filesystem path where the contents should be extracted. If `None`, a default location is used.
- `verbose`, default `False`: A boolean flag to toggle the printing of process information, such as cache usage or extraction status.

### Input
The function requires a valid, reachable URL string. The target environment must have network access to perform the download and write permissions to the specified `target_dir`.

### Output
Returns `str` or `None`: The path to the directory where the zip file contents were extracted if successful; returns `None` if the operation fails.

### Valid Call Patterns
```python
from tslearn.datasets import extract_from_zip_url

# Inferred usage based on signature
path = extract_from_zip_url(url="https://example.com/data.zip", target_dir="./data", verbose=True)
print(f"Extracted to: {path}")
```

### LLM Instruction Prompt
- Use `extract_from_zip_url` when needing to fetch external datasets for time-series analysis.
- Always check the return value; if it is `None`, the download or extraction failed, and a `RuntimeWarning` will have been issued.
- Ensure the `target_dir` exists or is a valid path for the current user environment.

### Prompt Snippet
```text
from tslearn.datasets import extract_from_zip_url
# Example:
# extract_from_zip_url(url="...", target_dir="...")
```

### Common Failure Modes
- **Network Errors:** If the URL is unreachable or the server returns an error, the function will fail and print a `RuntimeWarning`.
- **Permission Denied:** If the process lacks write access to the `target_dir`, the extraction will fail.
- **Invalid URL:** Providing a malformed URL or a link that does not point to a valid zip archive will result in failure.

### Fix Code Hint
```python
# Ensure the URL is valid and the directory is writable
import os
from tslearn.datasets import extract_from_zip_url

url = "https://example.com/dataset.zip"
target = "./my_data"
if not os.path.exists(target):
    os.makedirs(target)

result = extract_from_zip_url(url, target_dir=target, verbose=True)
if result is None:
    print("Failed to extract dataset.")
```

## API Test: `fit`

### Signature
```python
def fit(self, X, y=None)
def fit(self, X, y=None, **kwargs)
def fit(self, X, y)
def fit(self, X, y, sample_weight=None)
def fit(self, X, y=None, sample_weight=None)
```
_Source: tslearn/tslearn/preprocessing/preprocessing.py:48  (+23 more definition site/overload)_

_Source doc:_ Compute kernel k-means clustering. Parameters ---------- X : array-like of shape=(n_ts, sz, d) Time series dataset. y Ignored sample_weight : array-like of shape=(n_ts, ) or None (default: None) Weights to be given to time series in the learning process. By default, all time series weights are equal.

### Goal
Trains a `tslearn` estimator or transformer on a provided time-series dataset to learn internal parameters, cluster centers, or transformation mappings.

### Parameters
- `self`: The estimator instance (e.g., `TimeSeriesKMeans`, `KNeighborsTimeSeriesClassifier`).
- `X`: array-like of shape `(n_ts, sz, d)`. The time-series dataset used for training.
- `y`, default `None`: Target values (labels) for supervised estimators; ignored for unsupervised estimators.
- `sample_weight`, default `None`: array-like of shape `(n_ts, )`. Weights assigned to individual time series during the learning process.
- `**kwargs`: Additional keyword arguments passed to specific estimator implementations.

### Input
- **Data Format:** Must be a 3D NumPy array of shape `(n_ts, sz, d)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling `fit`.
- **Compatibility:** Estimators are fully compatible with `scikit-learn` pipelines.

### Output
Returns `self`: The fitted estimator instance, allowing for method chaining.

### Valid Call Patterns
```python
# Standard estimator training
from tslearn.neighbors import KNeighborsTimeSeriesClassifier
knn = KNeighborsTimeSeriesClassifier(n_neighbors=1)
knn.fit(X_scaled, y)

# Clustering with sample weights
from tslearn.clustering import KernelKMeans
gak_km = KernelKMeans(n_clusters=3).fit(time_series, sample_weight=weights)
```

### LLM Instruction Prompt
- Always ensure `X` is a 3D array `(n_ts, sz, d)`. If the input is a list of sequences, wrap it in `tslearn.utils.to_time_series_dataset` first.
- For supervised estimators, ensure `y` is provided. For unsupervised estimators, `y` is typically `None`.
- When using `KernelKMeans` or similar, ensure `kernel_params` are correctly configured if not using defaults.

### Prompt Snippet
```text
# Ensure X is 3D before fitting
X_formatted = to_time_series_dataset(X_raw)
estimator.fit(X_formatted, y)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset`.
- **Missing Dependencies:** Attempting to use `PyTorch` backend features without `pytorch` installed.
- **Invalid Kernel Params:** Passing incompatible `kernel_params` (e.g., `sigma=0`) which triggers a `RuntimeError`.
- **Inconsistent Lengths:** While `tslearn` supports variable-length series, some specific metrics or estimators may require uniform lengths; check estimator-specific documentation.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset
from tslearn.clustering import TimeSeriesKMeans

# Ensure 3D shape (n_ts, sz, d)
X = to_time_series_dataset([[1, 2, 3], [1, 2, 3, 4]])
km = TimeSeriesKMeans(n_clusters=2)
km.fit(X)
print(f"Witness: Fitted {len(km.cluster_centers_)} centers.")
```

## API Test: `fit_predict`

### Signature
```python
def fit_predict(self, X, y=None, n=1)
def fit_predict(self, X, y=None)
```
_Source: tslearn/tslearn/forecasting/_arima.py:431  (+5 more definition site/overload)_

_Source doc:_ Computes VARIMA model and forecasts n timestamps for the given data. Parameters ---------- X: array-like, shape (n_ts, sz, d) Time-series dataset. y : Ignored n : int (default: 1) The number of timestamps to forecast, a.k.a. the horizon. Returns ------- array, shape = (n_ts, n, d) Array of forecasted timestamps

### Goal
Computes a VARIMA model on the provided time-series dataset and returns a forecast for a specified number of future timestamps.

### Parameters
- `self`: The `VARIMA` estimator instance.
- `X`: array-like, shape (n_ts, sz, d). The time-series dataset used to fit the model.
- `y`, default `None`: Ignored; included for scikit-learn compatibility.
- `n`, default `1`: The number of future timestamps to forecast (the horizon).

### Input
- **Data Format:** Must be a 3D array-like structure of shape `(n_ts, sz, d)`.
- **Preconditions:** If the input is 1D or 2D, it must be converted to a 3D time-series dataset using `tslearn.utils.to_time_series_dataset` prior to calling `fit_predict`.
- **Variable Length:** Supports variable-length time series (represented by `np.nan` padding).

### Output
Returns an `array` of shape `(n_ts, n, d)` containing the forecasted values for each time series in the input dataset.

### Valid Call Patterns
```python
from tslearn.forecasting import VARIMA
import numpy as np

# Multivariate, variable length data example
data = np.random.rand(10, 50, 3)
model = VARIMA(p=2, d=1, q=2, with_constant=False)
predicted = model.fit_predict(data, n=5)
print(f"Forecast shape: {predicted.shape}")
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array-like object.
- Use `fit_predict` when you need to both train the model and generate a forecast in a single step.
- If the input data contains variable lengths, ensure the padding is handled via `np.nan` as per `tslearn` conventions.

### Prompt Snippet
```text
# Example of using fit_predict for forecasting
from tslearn.forecasting import VARIMA
model = VARIMA(p=1, d=0, q=0)
forecast = model.fit_predict(X_train, n=10)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays without converting them to 3D `(n_ts, sz, d)` format will raise a `ValueError`.
- **Minimum Size Violation:** If the input time series are too short for the specified `p`, `d`, or `q` parameters of the `VARIMA` model, a `ValueError` will be raised.
- **Missing Dependencies:** Ensure `scipy` and `numpy` are installed, as `VARIMA` relies on these for underlying statistical computations.

### Fix Code Hint
```python
# Ensure data is 3D before passing to fit_predict
from tslearn.utils import to_time_series_dataset
X_3d = to_time_series_dataset(X_raw)
model = VARIMA(p=1, d=1, q=1)
forecast = model.fit_predict(X_3d, n=5)
```

## API Test: `fit_transform`

### Signature
```python
def fit_transform(self, X, y=None, **fit_params)
def fit_transform(self, X, y=None, **kwargs)
def fit_transform(self, X, y=None, **transform_params)
```
_Source: tslearn/tslearn/matrix_profile/matrix_profile.py_

_Source doc:_ Transform a dataset of time series into its Matrix Profile representation. Parameters ---------- X : array-like of shape (n_ts, sz, d) Time series dataset Returns ------- numpy.ndarray of shape (n_ts, output_size, 1) Matrix-Profile-Transformed dataset. `ouput_size` is equal to `sz - subsequence_length + 1`

### Goal
`fit_transform` performs a joint fit and transformation of time-series data, commonly used in preprocessing pipelines or feature extraction (such as Matrix Profile generation) to convert raw input sequences into a processed representation.

### Parameters
- `self`: The estimator instance (e.g., `TimeSeriesScalerMinMax` or `MatrixProfile`).
- `X`: array-like of shape `(n_ts, sz, d)`. The time-series dataset to be transformed.
- `y`, default `None`: Target values for supervised tasks; ignored in most preprocessing transformers.
- `**fit_params`: Additional parameters passed to the `fit` method of the estimator.

### Input
- **Format:** 3D `numpy` arrays with shape `(n_ts, max_sz, d)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` before calling `fit_transform`.
- **Compatibility:** Fully compatible with `scikit-learn` pipelines.

### Output
Returns `numpy.ndarray` of shape `(n_ts, output_size, 1)`. The specific `output_size` depends on the transformer (e.g., `sz - subsequence_length + 1` for Matrix Profile).

### Valid Call Patterns
```python
from tslearn.preprocessing import TimeSeriesScalerMinMax
from tslearn.matrix_profile import MatrixProfile
import numpy as np

X = np.random.rand(10, 20, 1)

# Preprocessing pattern
X_scaled = TimeSeriesScalerMinMax().fit_transform(X)

# Matrix Profile pattern
mp = MatrixProfile(subsequence_length=10)
X_tr = mp.fit_transform(X)
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array. If the input is a list of sequences, wrap it in `tslearn.utils.to_time_series_dataset` first.
- When using `MatrixProfile`, ensure `subsequence_length` is defined during object initialization.
- Use `fit_transform` instead of calling `fit` followed by `transform` to ensure efficient execution in `scikit-learn` pipelines.

### Prompt Snippet
```text
# Ensure X is 3D before calling fit_transform
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
X_transformed = estimator.fit_transform(X_formatted)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError`.
- **Subsequence Length:** In `MatrixProfile`, if `subsequence_length` is greater than the time series length `sz`, the transformation will fail.
- **Missing Dependencies:** Using specific implementations (like `stumpy` for `MatrixProfile`) requires the optional dependency to be installed; otherwise, a `ModuleNotFoundError` or `ImportError` occurs.

### Fix Code Hint
```python
# If you encounter a shape error, verify dimensions:
assert X.ndim == 3, "Input must be (n_ts, sz, d)"
# If using MatrixProfile, check length:
assert subsequence_length <= X.shape[1], "subsequence_length cannot exceed time series length"
```

## API Test: `fix_force_all_finite_warning`

### Signature
```python
def fix_force_all_finite_warning(func)
```
_Source: tslearn/tslearn/utils/utils.py:24_

_Source doc:_ Make sure to use sklearn >=1.6 ensure_finite_attribute if available rather than deprecated force_all_attribute

### Goal
Provides a compatibility wrapper to ensure that input validation for finite values uses the modern `scikit-learn` `ensure_finite_attribute` API instead of the deprecated `force_all_finite` attribute.

### Parameters
- `func`: A callable (typically a function or method) that performs input validation or data processing requiring finite value checks.

### Input
- The caller must provide a function object that is intended to be decorated or wrapped to handle finite value checking in a `scikit-learn` compatible manner.
- This utility is intended for internal library use to maintain compatibility across different `scikit-learn` versions (specifically handling the transition to version 1.6+).

### Output
Returns `unspecified` — The function returns the wrapped callable or the result of the compatibility logic applied to the input function.

### Valid Call Patterns
```python
from tslearn.utils import fix_force_all_finite_warning

# Inferred usage: wrapping a validation function
@fix_force_all_finite_warning
def my_validation_func(X):
    pass
```

### LLM Instruction Prompt
- Use `fix_force_all_finite_warning` only when defining internal validation logic that needs to remain compatible with both older and newer versions of `scikit-learn`.
- Do not use this function for general data preprocessing; it is a backend utility for library developers.

### Prompt Snippet
```text
from tslearn.utils import fix_force_all_finite_warning
# Wrap your validation logic to ensure sklearn 1.6+ compatibility
@fix_force_all_finite_warning
def validate_data(X):
    ...
```

### Common Failure Modes
- Passing an object that is not a callable to `func`.
- Attempting to use this in environments where `scikit-learn` is not installed or is significantly outdated, potentially leading to unexpected behavior if the internal `ensure_finite_attribute` logic is missing.

### Fix Code Hint
```python
# Ensure the input is a function or method before applying the decorator
if callable(func):
    decorated_func = fix_force_all_finite_warning(func)
```

## API Test: `forward`

### Signature
```python
def forward(ctx, D, gamma)
def forward(self, x, y)
```
_Source: tslearn/tslearn/metrics/soft_dtw_loss_pytorch.py:34  (+1 more definition site/overload)_

_Source doc:_ Parameters ---------- ctx : context D : Tensor, shape=[b, m, n] Matrix of pairwise distances. gamma : float Regularization parameter. Lower is less smoothed (closer to true DTW). Returns ------- loss : Tensor, shape=[batch_size,] The loss values.

### Goal
Computes the Soft-DTW loss for a batch of time series, providing a differentiable approximation of the Dynamic Time Warping distance suitable for gradient-based optimization.

### Parameters
- `ctx`: The context object (typically used in PyTorch autograd functions).
- `D`: A 3D Tensor of shape `[batch_size, m, n]` representing the matrix of pairwise distances between time series elements.
- `gamma`: A float representing the regularization parameter; lower values result in less smoothing, approaching the standard DTW distance.
- `x`: A batch of time series (used in the `SoftDTWLossPyTorch` class implementation).
- `y`: A batch of time series (used in the `SoftDTWLossPyTorch` class implementation).

### Input
- **Data Format:** Requires PyTorch tensors. Input time series must be 3D arrays of shape `(n_ts, max_sz, d)`.
- **Preconditions:** The `pytorch` package must be installed. When using the `SoftDTWLossPyTorch` class, ensure the backend is initialized for PyTorch.

### Output
Returns `Tensor` — A 1D tensor of shape `[batch_size,]` containing the computed loss values for each time series pair in the batch.

### Valid Call Patterns
```python
import torch
from tslearn.metrics.soft_dtw_loss_pytorch import SoftDTWLossPyTorch

# Using the class-based forward method
soft_dtw = SoftDTWLossPyTorch(gamma=1.0)
batch_ts_1 = torch.zeros((5, 10, 8), requires_grad=True)
batch_ts_2 = torch.ones((5, 12, 8), requires_grad=True)
loss = soft_dtw.forward(batch_ts_1, batch_ts_2)
print(f"Loss shape: {loss.shape}")
```

### LLM Instruction Prompt
- Use `forward` only when working within a PyTorch-based workflow.
- Always ensure inputs are PyTorch tensors; if using NumPy arrays, convert them using `torch.from_numpy()` first.
- When using the class-based `forward(self, x, y)`, ensure the `SoftDTWLossPyTorch` instance is initialized with the desired `gamma` and `normalize` parameters.

### Prompt Snippet
```text
# Example of calling forward via SoftDTWLossPyTorch instance
loss = soft_dtw_loss_pytorch_instance.forward(x_tensor, y_tensor)
```

### Common Failure Modes
- **Backend Mismatch:** Passing NumPy arrays instead of PyTorch tensors will raise a `TypeError` or attribute error.
- **Shape Mismatch:** Providing tensors that do not conform to the `(batch_size, time_steps, features)` structure will cause internal dimension errors during distance matrix computation.
- **Missing Dependency:** Calling this function without `torch` installed will result in an `ImportError`.

### Fix Code Hint
```python
# Ensure inputs are tensors and on the correct device
import torch
x = torch.tensor(x_numpy, dtype=torch.float32)
y = torch.tensor(y_numpy, dtype=torch.float32)
loss = soft_dtw_instance.forward(x, y)
```

## API Test: `frechet`

### Signature
```python
def frechet(s1, s2, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
```
_Source: tslearn/tslearn/metrics/_frechet.py:30_

_Source doc:_ Compute Frechet similarity [1]_ measure between (possibly multidimensional) time series and return it. Frechet similarity score is computed as the maximum distance between aligned time series, i.e., if :math:`\pi` is an optimal alignment path: .. math:: Frechet(X, Y) = \max_{(i, j) \in \pi} \|X_{i} - Y_{j}\| Note that this formula is still valid for the multivariate case. It is not required that both time series share the same size, but they must be the same dimension. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for Frechet distance. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [2]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- float

### Goal
Computes the Frechet distance between two time series, defined as the maximum distance between points along an optimal alignment path.

### Parameters
- `s1`: The first time series, shape `(sz1, d)` or `(sz1,)`.
- `s2`: The second time series, shape `(sz2, d)` or `(sz2,)`.
- `global_constraint`, default `None`: Constraint type to restrict the alignment path; options are `"itakura"` or `"sakoe_chiba"`.
- `sakoe_chiba_radius`, default `None`: Integer radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float maximum slope for the Itakura parallelogram constraint.
- `be`, default `None`: Backend engine to use for computation; either `"numpy"`, `"pytorch"`, or a backend instance.

### Input
- **Data:** Two time series arrays. They must have the same dimensionality `d`.
- **Preconditions:** If inputs are 1D, they are treated as univariate. If using the `PyTorch` backend, ensure `pytorch` is installed.

### Output
Returns `float`: The computed Frechet distance between the two time series.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

s1 = np.random.randn(10, 4)
s2 = np.random.randn(10, 4)

# Standard call
dist = tslearn.metrics.frechet(s1, s2)
print(f"Witness: {dist}")
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` have the same number of dimensions `d`.
- If specifying a `global_constraint`, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is provided or understood to use the default.
- Use `be="pytorch"` only when automatic differentiation is required and `torch` is available.

### Prompt Snippet
```text
tslearn.metrics.frechet(s1, s2, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing time series with different dimensions `d` will raise a `ValueError`.
- **Empty Input:** Passing empty lists or arrays will raise a `ValueError`.
- **Conflicting Constraints:** Providing both `sakoe_chiba_radius` and `itakura_max_slope` without a valid `global_constraint` may trigger a `RuntimeWarning`.

### Fix Code Hint
```python
# Ensure inputs are numpy arrays of consistent dimension
import numpy as np
s1 = np.array(s1)
s2 = np.array(s2)
if s1.shape[-1] != s2.shape[-1]:
    raise ValueError("Time series must have the same dimension.")
```

## API Test: `frechet_path`

### Signature
```python
def frechet_path(s1, s2, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
```
_Source: tslearn/tslearn/metrics/_frechet.py:171_

_Source doc:_ Compute Frechet similarity measure [1]_ between (possibly multidimensional) time series and an optimal alignment path. Frechet distance is computed as the maximium distance between aligned time series, i.e., if :math:`\pi` is the optimal alignment path: .. math:: Frechet(X, Y) = \max_{(i, j) \in \pi} \|X_{i} - Y_{j}\| It is not required that both time series share the same size, but they must be the same dimension. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for Frechet distance. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [2]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- list of integer pairs Matching path represented as a list of index pairs. In each pair, the first index corresponds to s1 and the second one corresponds to s2. float Similarity score Examples

### Goal
Compute the Frechet distance (the maximum distance between any two points in an optimal alignment) and the corresponding alignment path between two time series.

### Parameters
- `s1`: Array-like, shape=(sz1, d) or (sz1,). The first time series.
- `s2`: Array-like, shape=(sz2, d) or (sz2,). The second time series.
- `global_constraint`, default `None`: String ("itakura" or "sakoe_chiba") or None. Restricts the search space for the alignment path.
- `sakoe_chiba_radius`, default `None`: Integer or None. The radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float or None. The maximum slope for the Itakura parallelogram constraint.
- `be`, default `None`: Backend object, string ("numpy" or "pytorch"), or None. Determines the computation engine.

### Input
- Time series must have the same dimension `d`.
- Inputs can be univariate (shape `(sz,)`) or multivariate (shape `(sz, d)`).
- If using `PyTorch` backend, inputs should be `torch.Tensor` objects.

### Output
Returns a tuple containing:
- `path`: A list of integer index pairs `(i, j)` representing the optimal alignment.
- `dist`: A float representing the Frechet similarity score.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

s1 = np.array([1., 2., 3.])
s2 = np.array([1.0, 2.0, 2.0, 3.0])
path, dist = tslearn.metrics.frechet_path(s1, s2)
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` have the same dimensionality.
- If `global_constraint` is provided, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is set appropriately or left as default.
- Use `be="pytorch"` if the input arrays are `torch.Tensor` and require gradient tracking.

### Prompt Snippet
```text
path, dist = tslearn.metrics.frechet_path(s1, s2, global_constraint="sakoe_chiba", sakoe_chiba_radius=2)
```

### Common Failure Modes
- `ValueError`: Raised if `s1` and `s2` have different dimensions or are empty.
- `RuntimeWarning`: Raised if both `sakoe_chiba_radius` and `itakura_max_slope` are provided but `global_constraint` is `None`.
- Backend mismatch: Passing a `PyTorch` tensor while using the default `NumPy` backend (or vice versa) may cause errors; explicitly set `be` if unsure.

### Fix Code Hint
```python
# Ensure inputs are valid before calling
if s1.shape[-1] != s2.shape[-1]:
    raise ValueError("Time series must have the same dimension.")
path, dist = tslearn.metrics.frechet_path(s1, s2)
```

## API Test: `frechet_path_from_metric`

### Signature
```python
def frechet_path_from_metric(s1, s2=None, metric='precomputed', global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None, **kwds)
```
_Source: tslearn/tslearn/metrics/_frechet.py:457_

_Source doc:_ Compute Frechet similarity measure and an optimal alignment path [1]_ between (possibly multidimensional) time series using a distance metric defined by the user. It is not required that both time series share the same size, but they must be the same dimension. When using Pytorch backend only "precomputed", "euclidean", "sqeuclidean" and callable metrics are available. Otherwise, valid values for metric are the same as for scikit-learn `pairwise_distances`_ function i.e. a string (e.g. "euclidean", "sqeuclidean", "hamming") or a function that is used to compute the pairwise distances. See `scikit`_ and `scipy`_ documentations for more information about the available metrics.

### Goal
Computes the Frechet similarity measure and the corresponding optimal alignment path between two time series using a user-defined distance metric.

### Parameters
- `s1`: Time series (shape `(sz1, d)`) or a precomputed distance matrix (shape `(sz1, sz2)` if `metric="precomputed"`). Univariate series can be shape `(sz1,)`.
- `s2`, default `None`: Second time series (shape `(sz2, d)` or `(sz2,)`). Required unless `metric="precomputed"`.
- `metric`, default `'precomputed'`: Distance metric to use. Can be a string compatible with `sklearn.metrics.pairwise_distances` or a callable function.
- `global_constraint`, default `None`: Global constraint to restrict the alignment path; options are `"itakura"` or `"sakoe_chiba"`.
- `sakoe_chiba_radius`, default `None`: Integer radius for the Sakoe-Chiba band constraint. Defaults to 1 if `global_constraint="sakoe_chiba"`.
- `itakura_max_slope`, default `None`: Float maximum slope for the Itakura parallelogram constraint. Defaults to 2.0 if `global_constraint="itakura"`.
- `be`, default `None`: Backend object (e.g., `NumPyBackend` or `PyTorchBackend`) used for computation.
- `**kwds`: Additional keyword arguments passed to the distance metric function.

### Input
- **Data:** Time series must be provided as array-like structures.
- **Preconditions:** If `metric != "precomputed"`, `s1` and `s2` must have the same dimension `d`. If using the `PyTorch` backend, only `"precomputed"`, `"euclidean"`, `"sqeuclidean"`, and callable metrics are supported.

### Output
Returns a tuple containing the optimal alignment path (list of index pairs) and the Frechet distance (scalar).

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Example using Euclidean distance
s1 = np.array([[1.0], [2.0], [3.0]])
s2 = np.array([[1.0], [2.0], [2.0], [3.0]])
path, dist = tslearn.metrics.frechet_path_from_metric(s1, s2, metric="euclidean")
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` have matching dimensions if `metric` is not `"precomputed"`.
- If using `PyTorch` tensors, ensure the `be` parameter is set to a compatible backend or that the input tensors trigger the correct backend inference.
- When using global constraints, ensure `global_constraint` is explicitly set to `"itakura"` or `"sakoe_chiba"` to avoid warnings or ignored parameters.

### Prompt Snippet
```text
from tslearn.metrics import frechet_path_from_metric
# Compute path between s1 and s2 using a specific metric
path, dist = frechet_path_from_metric(s1, s2, metric="euclidean")
```

### Common Failure Modes
- **Dimension Mismatch:** Providing time series with different dimensions `d` when `metric` is not `"precomputed"`.
- **Backend Incompatibility:** Attempting to use complex string metrics (e.g., `"hamming"`) with the `PyTorch` backend, which only supports a limited subset.
- **Constraint Conflict:** Setting both `sakoe_chiba_radius` and `itakura_max_slope` without correctly specifying the `global_constraint`, leading to `RuntimeWarning` or ignored constraints.

### Fix Code Hint
```python
# Ensure inputs are 2D (n_samples, n_features) for consistency
s1 = s1.reshape(-1, 1) if s1.ndim == 1 else s1
s2 = s2.reshape(-1, 1) if s2.ndim == 1 else s2
path, dist = tslearn.metrics.frechet_path_from_metric(s1, s2, metric="euclidean")
```

## API Test: `from_cesium_dataset`

### Signature
```python
def from_cesium_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:713_

_Source doc:_ Transform a cesium-compatible dataset into a tslearn dataset. Parameters ---------- X: list of cesium TimeSeries cesium-formatted dataset (cf. `link <http://cesium-ml.org/docs/api/cesium.time_series.html#cesium.time_series.TimeSeries>`_) Returns ------- array, shape=(n_ts, sz, d) tslearn-formatted dataset. Examples -------- >>> from cesium.time_series import TimeSeries >>> cesium_ds = [TimeSeries(m=numpy.array([1, 2, 3, 4]))] >>> tslearn_arr = from_cesium_dataset(cesium_ds) >>> tslearn_arr.shape (1, 4, 2) Notes ----- Conversion from/to cesium format requires cesium to be installed.

### Goal
Converts a collection of `cesium.time_series.TimeSeries` objects into a standard `tslearn` 3D NumPy array format suitable for machine learning estimators.

### Parameters
- `X`: A list of `cesium.time_series.TimeSeries` objects representing the time-series dataset to be converted.

### Input
- **Data Format:** Requires the `cesium` library to be installed in the environment.
- **Preconditions:** The input `X` must be a list of valid `cesium` `TimeSeries` objects. The internal data within these objects must be consistent in dimensions to form a valid 3D array `(n_ts, sz, d)`.

### Output
Returns a 3D `numpy.ndarray` of shape `(n_ts, sz, d)`, where `n_ts` is the number of time series, `sz` is the number of time steps, and `d` is the number of features.

### Valid Call Patterns
```python
import numpy as np
import tslearn.utils
from cesium.time_series import TimeSeries

# Example using a list of cesium TimeSeries objects
cesium_ds = [TimeSeries(m=np.array([1, 2, 3, 4]))]
tslearn_arr = tslearn.utils.from_cesium_dataset(cesium_ds)
print(tslearn_arr.shape) # (1, 4, 1)
```

### LLM Instruction Prompt
- Always verify that the `cesium` library is available before suggesting this function.
- Ensure the input is a list of `cesium.time_series.TimeSeries` objects.
- Use `tslearn.utils.from_cesium_dataset` as the explicit import path.

### Prompt Snippet
```text
from tslearn.utils import from_cesium_dataset
# Ensure cesium is installed and X is a list of cesium.time_series.TimeSeries
X_tslearn = from_cesium_dataset(X_cesium)
```

### Common Failure Modes
- **Missing Dependency:** Calling the function without the `cesium` package installed will raise an `ImportError`.
- **Inconsistent Dimensions:** If the `cesium` `TimeSeries` objects have varying lengths or feature counts that cannot be broadcast into a single 3D array, the conversion may fail or produce unexpected shapes.
- **Incorrect Input Type:** Passing a raw NumPy array or a list of lists instead of a list of `cesium.time_series.TimeSeries` objects will result in an attribute error.

### Fix Code Hint
```python
import pytest
import numpy as np
import tslearn.utils
from numpy.testing import assert_allclose

# Ensure cesium is present for the conversion test
pytest.importorskip('cesium')

# Deterministic check: Round-trip conversion
n, sz, d = 5, 10, 2
data = np.random.rand(n, sz, d)
cesium_data = tslearn.utils.to_cesium_dataset(data)
converted = tslearn.utils.from_cesium_dataset(cesium_data)

assert_allclose(data, converted)
print(f"Correctness witness: {converted.shape} matches input shape.")
```

## API Test: `from_hdf5`

### Signature
```python
def from_hdf5(cls, path)
```
_Source: tslearn/tslearn/bases/bases.py:232_

_Source doc:_ Load model from a HDF5 file. Requires ``h5py`` http://docs.h5py.org/ Parameters ---------- path : str Full path to file. Returns ------- Model instance

### Goal
Deserializes a previously saved `tslearn` estimator from a HDF5 file format to restore its state and parameters.

### Parameters
- `cls`: The class reference of the estimator being loaded (e.g., `TimeSeriesKMeans`).
- `path`: A string representing the full file system path to the HDF5 file containing the serialized model.

### Input
- **Format:** A valid HDF5 file generated by a `tslearn` estimator's `to_hdf5` method.
- **Preconditions:** The `h5py` library must be installed in the environment. The file at `path` must exist and contain a valid `tslearn` model structure.

### Output
Returns an instance of the estimator class specified by `cls`, populated with the parameters and state loaded from the HDF5 file.

### Valid Call Patterns
```python
from tslearn.clustering import TimeSeriesKMeans
# Inferred from signature: loading a model from a file
model = TimeSeriesKMeans.from_hdf5("my_model.h5")
```

### LLM Instruction Prompt
- Always call `from_hdf5` as a class method on the specific `tslearn` estimator class (e.g., `TimeSeriesKMeans.from_hdf5(...)`).
- Ensure the `path` argument is a valid string pointing to an existing `.h5` file.
- Verify that the `h5py` dependency is available in the execution environment before attempting to load.

### Prompt Snippet
```text
# Load a saved model
from tslearn.clustering import TimeSeriesKMeans
model = TimeSeriesKMeans.from_hdf5("path/to/model.h5")
```

### Common Failure Modes
- **Missing Dependency:** Calling the function without `h5py` installed will raise an `ImportError`.
- **File Not Found:** Providing a `path` that does not exist or is inaccessible will raise an `OSError` or `FileNotFoundError`.
- **Incompatible Class:** Attempting to load a file saved from a different estimator type using the wrong `cls` may result in attribute errors or corrupted state.
- **Corrupted File:** If the HDF5 file was not created by `tslearn` or is truncated, the internal structure will fail to parse.

### Fix Code Hint
```python
import h5py
import os

# Ensure the file exists before loading
if os.path.exists("model.h5"):
    model = TimeSeriesKMeans.from_hdf5("model.h5")
else:
    print("File not found.")
```

## API Test: `from_json`

### Signature
```python
def from_json(cls, path)
```
_Source: tslearn/tslearn/bases/bases.py:273_

_Source doc:_ Load model from a JSON file. Parameters ---------- path : str Full path to file. Returns ------- Model instance

### Goal
Load a serialized `tslearn` estimator instance from a JSON file to restore a previously trained model state.

### Parameters
- `cls`: The class of the estimator to be instantiated (e.g., `TimeSeriesKMeans`).
- `path`: A string representing the full path to the JSON file containing the model configuration and parameters.

### Input
The caller must provide a valid file path to a JSON file that was previously generated by a `tslearn` estimator's `to_json` method. The file must be accessible in the current environment.

### Output
Returns `unspecified` — An instance of the estimator class (`cls`) populated with the parameters and state loaded from the JSON file.

### Valid Call Patterns
```python
# Inferred from signature: loading a model from a JSON file
from tslearn.clustering import TimeSeriesKMeans
model = TimeSeriesKMeans.from_json("model_config.json")
print(f"Model loaded: {type(model)}")
```

### LLM Instruction Prompt
- Use `from_json` as a class method on the specific `tslearn` estimator class (e.g., `TimeSeriesKMeans.from_json(...)`).
- Ensure the `path` argument is a valid string pointing to a file that exists and contains valid JSON-serialized `tslearn` model data.

### Prompt Snippet
```text
# Example of loading a model
from tslearn.clustering import TimeSeriesKMeans
model = TimeSeriesKMeans.from_json(path="path/to/model.json")
```

### Common Failure Modes
- `FileNotFoundError`: Occurs if the provided `path` does not point to an existing file.
- `JSONDecodeError`: Occurs if the file at `path` is not a valid JSON format or does not contain the expected `tslearn` model structure.
- `AttributeError`: Occurs if `from_json` is called on an object instance instead of the class, or if the class does not support JSON serialization.

### Fix Code Hint
```python
import os
from tslearn.clustering import TimeSeriesKMeans

# Ensure the file exists before attempting to load
file_path = "model.json"
if os.path.exists(file_path):
    model = TimeSeriesKMeans.from_json(file_path)
else:
    print(f"Error: {file_path} not found.")
```

## API Test: `from_numpy`

### Signature
```python
def from_numpy(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:98  (+1 more definition site/overload)_

### Goal
Converts a NumPy array into the internal representation used by the `tslearn` NumPy backend.

### Parameters
- `x`: The input NumPy array to be processed by the backend.

### Input
- **Data Format:** Expects a `numpy.ndarray`.
- **Preconditions:** The input should be a valid NumPy array compatible with the expected time-series dimensions (typically 3D: `n_ts, max_sz, d`).

### Output
Returns `unspecified` — The input array `x` as handled by the `NumPyBackend`.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import NumPyBackend
be = NumPyBackend()
x = [[1, 2, 3], [4, 5, 6]]
result = be.from_numpy(x)
```

### LLM Instruction Prompt
- Use `from_numpy` when you need to explicitly wrap or ensure a NumPy array is compatible with the `NumPyBackend` operations.
- Always ensure the input `x` is a `numpy.ndarray` before passing it to this method.

### Prompt Snippet
```text
# Ensure data is in NumPy format for the backend
from tslearn.backend.numpy_backend import NumPyBackend
be = NumPyBackend()
data_backend = be.from_numpy(my_numpy_array)
```

### Common Failure Modes
- **Type Mismatch:** Passing a PyTorch tensor or a list of lists directly without converting to a `numpy.ndarray` first will cause errors if the backend expects specific NumPy array attributes.
- **Dimension Mismatch:** Passing data that does not conform to the expected 3D shape `(n_ts, max_sz, d)` required by `tslearn` estimators.

### Fix Code Hint
```python
import numpy as np
# Ensure input is a numpy array before calling from_numpy
x_np = np.array(x)
result = be.from_numpy(x_np)
```

## API Test: `from_pickle`

### Signature
```python
def from_pickle(cls, path)
```
_Source: tslearn/tslearn/bases/bases.py:321_

_Source doc:_ Load model from a pickle file. Parameters ---------- path : str Full path to file. Returns ------- Model instance

### Goal
Load a serialized `tslearn` estimator from a pickle file to restore a previously trained time-series model.

### Parameters
- `cls`: The class of the estimator to be instantiated (e.g., `TimeSeriesKMeans`).
- `path`: A string representing the full file system path to the pickle file.

### Input
The caller must provide a valid file path to a file previously created by a `tslearn` estimator's `to_pickle` method. The file must be accessible in the current environment.

### Output
Returns an instance of the estimator class specified by `cls`, populated with the state loaded from the pickle file.

### Valid Call Patterns
```python
from tslearn.clustering import TimeSeriesKMeans
# Inferred from signature:
model = TimeSeriesKMeans.from_pickle("model.pkl")
```

### LLM Instruction Prompt
- Always call `from_pickle` as a class method on the specific `tslearn` estimator class (e.g., `TimeSeriesKMeans.from_pickle(...)`).
- Ensure the `path` argument is a valid string pointing to a file generated by `tslearn`.
- Do not attempt to load pickle files from untrusted sources.

### Prompt Snippet
```text
# Load a saved model
from tslearn.clustering import TimeSeriesKMeans
model = TimeSeriesKMeans.from_pickle("path/to/model.pkl")
```

### Common Failure Modes
- **FileNotFoundError**: Occurs if the provided `path` does not exist or is inaccessible.
- **UnpicklingError**: Occurs if the file at `path` is corrupted or was not created by a compatible `tslearn` estimator.
- **AttributeError**: Occurs if `from_pickle` is called on an object instance instead of the class, or if the class does not support serialization.

### Fix Code Hint
```python
import os
from tslearn.clustering import TimeSeriesKMeans

# Ensure the file exists before attempting to load
path = "model.pkl"
if os.path.exists(path):
    model = TimeSeriesKMeans.from_pickle(path)
else:
    print(f"File not found: {path}")
```

## API Test: `from_pyflux_dataset`

### Signature
```python
def from_pyflux_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:460_

_Source doc:_ Transform a pyflux-compatible dataset into a tslearn dataset. Parameters ---------- X: pandas data-frame pyflux-formatted dataset Returns ------- array, shape=(n_ts, sz, d), where n_ts=1 tslearn-formatted dataset. Column order is kept the same as in the original data frame. Examples -------- >>> import pandas as pd >>> pyflux_df = pd.DataFrame() >>> pyflux_df["dim_0"] = numpy.random.rand(10) >>> tslearn_arr = from_pyflux_dataset(pyflux_df) >>> tslearn_arr.shape (1, 10, 1) >>> pyflux_df = pd.DataFrame() >>> pyflux_df["dim_0"] = numpy.random.rand(10) >>> pyflux_df["dim_1"] = numpy.random.rand(10) >>> pyflux_df["dim_2"] = numpy.random.rand(10) >>> tslearn_arr = from_pyflux_dataset(pyflux_df) >>> tslearn_arr.shape (1, 10, 3) >>> pyflux_arr = numpy.random.randn(10, 1, 16) >>> from_pyflux_dataset( ...     pyflux_arr ... )  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: X is not a valid input pyflux array. Notes ----- Conversion from/to pyflux format requires pandas to be installed.

### Goal
Converts a PyFlux-formatted pandas DataFrame into a standard `tslearn` 3D time-series array of shape `(1, sz, d)`.

### Parameters
- `X`: A `pandas.DataFrame` representing a time-series dataset where columns correspond to dimensions and rows correspond to time steps.

### Input
- **Format:** Must be a `pandas.DataFrame`.
- **Preconditions:** The `pandas` library must be installed in the environment.
- **Constraint:** The input is strictly expected to be a DataFrame; passing other types (like NumPy arrays) will raise a `ValueError`.

### Output
Returns `numpy.ndarray` of shape `(1, sz, d)`, where `n_ts=1`, `sz` is the number of rows in the DataFrame, and `d` is the number of columns.

### Valid Call Patterns
```python
import numpy as np
import pandas as pd
from tslearn.utils import from_pyflux_dataset

# Create a valid PyFlux-compatible DataFrame
df = pd.DataFrame({"dim_0": np.random.rand(10), "dim_1": np.random.rand(10)})
tslearn_arr = from_pyflux_dataset(df)

print(f"Correctness witness: {tslearn_arr.shape == (1, 10, 2)}")
```

### LLM Instruction Prompt
- Always verify that the input `X` is a `pandas.DataFrame` before calling `from_pyflux_dataset`.
- Ensure the `pandas` dependency is available in the execution context.
- Note that the output will always have a batch dimension of 1 (`n_ts=1`).

### Prompt Snippet
```text
from tslearn.utils import from_pyflux_dataset
# Ensure input is a pandas DataFrame
# X = ...
ts_data = from_pyflux_dataset(X)
```

### Common Failure Modes
- **Missing Dependency:** Calling the function without `pandas` installed will result in an `ImportError`.
- **Invalid Input Type:** Passing a NumPy array or list instead of a `pandas.DataFrame` will trigger a `ValueError: X is not a valid input pyflux array`.
- **Dimension Mismatch:** Expecting the output to have multiple time series (`n_ts > 1`) when the input is a single DataFrame; the function only supports single-series conversion.

### Fix Code Hint
```python
import pandas as pd
# If input is a numpy array, convert to DataFrame first
if not isinstance(X, pd.DataFrame):
    X = pd.DataFrame(X)
ts_data = from_pyflux_dataset(X)
```

## API Test: `from_pyts_dataset`

### Signature
```python
def from_pyts_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:100_

_Source doc:_ Transform a pyts-compatible dataset into a tslearn dataset. Parameters ---------- X: array, shape = (n_ts, sz) or (n_ts, d, sz) pyts-formatted dataset Returns ------- array, shape=(n_ts, sz, d) tslearn-formatted dataset Examples -------- >>> pyts_arr = numpy.random.randn(10, 16) >>> tslearn_arr = from_pyts_dataset(pyts_arr) >>> tslearn_arr.shape (10, 16, 1) >>> pyts_arr = numpy.random.randn(10, 2, 16) >>> tslearn_arr = from_pyts_dataset(pyts_arr) >>> tslearn_arr.shape (10, 16, 2) >>> pyts_arr = numpy.random.randn(10) >>> from_pyts_dataset(pyts_arr)  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: X is not a valid input pyts array.

### Goal
Converts time-series datasets formatted for the `pyts` library into the standard `tslearn` 3D array format `(n_ts, sz, d)`.

### Parameters
- `X`: A `pyts`-formatted dataset, expected as a 2D array `(n_ts, sz)` or a 3D array `(n_ts, d, sz)`.

### Input
- **Data Format:** Must be a NumPy array.
- **Preconditions:** The input must strictly follow `pyts` dimensionality conventions. If the input is 1D, the function will raise a `ValueError`.

### Output
Returns a 3D NumPy array with shape `(n_ts, sz, d)`, which is the standard input format for `tslearn` estimators and metrics.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import from_pyts_dataset

# Example: Converting a 2D pyts array (n_ts, sz)
pyts_2d = np.random.randn(10, 16)
tslearn_arr = from_pyts_dataset(pyts_2d)
print(f"Output shape: {tslearn_arr.shape}") # (10, 16, 1)

# Example: Converting a 3D pyts array (n_ts, d, sz)
pyts_3d = np.random.randn(10, 2, 16)
tslearn_arr_3d = from_pyts_dataset(pyts_3d)
print(f"Output shape: {tslearn_arr_3d.shape}") # (10, 16, 2)
```

### LLM Instruction Prompt
- Use `from_pyts_dataset` only when the input data originates from the `pyts` ecosystem.
- Ensure the input `X` is at least 2-dimensional; 1D arrays are invalid and will trigger a `ValueError`.
- Always verify the resulting shape is `(n_ts, sz, d)` before passing the data to `tslearn` estimators.

### Prompt Snippet
```text
from tslearn.utils import from_pyts_dataset
# X is a pyts-formatted array
tslearn_data = from_pyts_dataset(X)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing a 1D array (e.g., a single time series) will raise a `ValueError: X is not a valid input pyts array.`
- **Non-NumPy Inputs:** Passing lists or other non-array types may cause internal indexing errors. Ensure inputs are cast to `numpy.ndarray` first.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import from_pyts_dataset

# Ensure input is at least 2D before conversion
data = np.array(my_pyts_data)
if data.ndim < 2:
    data = data.reshape(1, -1)
tslearn_data = from_pyts_dataset(data)
```

## API Test: `from_seglearn_dataset`

### Signature
```python
def from_seglearn_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:182_

_Source doc:_ Transform a seglearn-compatible dataset into a tslearn dataset. Parameters ---------- X: list of arrays, or array of arrays, shape = (n_ts, ) seglearn-formatted dataset. i-th sub-array in the list has shape (sz_i, d) Returns ------- array, shape=(n_ts, sz, d), where sz is the maximum of all array lengths tslearn-formatted dataset Examples -------- >>> seglearn_arr = [numpy.random.randn(10, 1), numpy.random.randn(10, 1)] >>> tslearn_arr = from_seglearn_dataset(seglearn_arr) >>> tslearn_arr.shape (2, 10, 1) >>> seglearn_arr = [numpy.random.randn(10, 1), numpy.random.randn(5, 1)] >>> tslearn_arr = from_seglearn_dataset(seglearn_arr) >>> tslearn_arr.shape (2, 10, 1) >>> seglearn_arr = numpy.random.randn(2, 10, 1) >>> tslearn_arr = from_seglearn_dataset(seglearn_arr) >>> tslearn_arr.shape (2, 10, 1)

### Goal
Converts time-series data formatted for the `seglearn` library into the standard 3D `(n_ts, max_sz, d)` NumPy array format required by `tslearn`.

### Parameters
- `X`: A `seglearn`-formatted dataset, provided as a list of arrays or an array of arrays, where each sub-array has shape `(sz_i, d)`.

### Input
- **Data Format:** Accepts a list of 2D arrays or a 3D NumPy array.
- **Preconditions:** The input must represent a collection of time series where each series has `d` features. If series have variable lengths (`sz_i`), the function will pad them to the maximum length found in the dataset.

### Output
Returns a 3D NumPy array of shape `(n_ts, sz, d)`, where `sz` is the maximum length among all input time series.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import from_seglearn_dataset

# Example with variable length series
seglearn_data = [np.random.randn(10, 1), np.random.randn(5, 1)]
tslearn_data = from_seglearn_dataset(seglearn_data)
print(f"Output shape: {tslearn_data.shape}") # (2, 10, 1)
```

### LLM Instruction Prompt
- Use `from_seglearn_dataset` when integrating data pipelines from `seglearn` into `tslearn`.
- Ensure the input `X` is a collection of 2D arrays (time steps, features).
- Note that the output will be zero-padded to the length of the longest time series in the input.

### Prompt Snippet
```text
from tslearn.utils import from_seglearn_dataset
# X is a list of arrays of shape (sz_i, d)
tslearn_X = from_seglearn_dataset(X)
```

### Common Failure Modes
- **Inconsistent Dimensions:** Passing arrays with different feature counts `d` across the list will raise a `ValueError` during array construction.
- **Non-Array Inputs:** Passing non-numeric types or nested structures that cannot be coerced into NumPy arrays will result in a `TypeError`.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import from_seglearn_dataset

# Ensure all sub-arrays have the same number of features (d)
# If d varies, you must preprocess or truncate before calling this function.
try:
    tslearn_data = from_seglearn_dataset(seglearn_data)
except ValueError as e:
    print(f"Check that all sub-arrays have the same feature dimension: {e}")
```

## API Test: `from_sktime_dataset`

### Signature
```python
def from_sktime_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:335_

_Source doc:_ Transform a sktime-compatible dataset into a tslearn dataset. Parameters ---------- X: pandas data-frame sktime-formatted dataset Returns ------- array, shape=(n_ts, sz, d) tslearn-formatted dataset Examples -------- >>> import pandas as pd >>> sktime_df = pd.DataFrame() >>> sktime_df["dim_0"] = [pd.Series([1, 2, 3]), pd.Series([4, 5, 6])] >>> tslearn_arr = from_sktime_dataset(sktime_df) >>> tslearn_arr.shape (2, 3, 1) >>> sktime_df = pd.DataFrame() >>> sktime_df["dim_0"] = [pd.Series([1, 2, 3]), ...                       pd.Series([4, 5, 6, 7])] >>> sktime_df["dim_1"] = [pd.Series([8, 9, 10]), ...                       pd.Series([11, 12, 13, 14])] >>> tslearn_arr = from_sktime_dataset(sktime_df) >>> tslearn_arr.shape (2, 4, 2) >>> sktime_arr = numpy.random.randn(10, 1, 16) >>> from_sktime_dataset( ...     sktime_arr ... )  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: X is not a valid input sktime array. Notes ----- Conversion from/to sktime format requires pandas to be installed.

### Goal
Converts a `sktime`-formatted pandas DataFrame (where columns represent dimensions and rows contain `pd.Series` objects) into a standard `tslearn` 3D NumPy array of shape `(n_ts, sz, d)`.

### Parameters
- `X`: A `pandas.DataFrame` structured as an `sktime` dataset, where each column corresponds to a time-series dimension and each row contains a `pd.Series` of varying or fixed lengths.

### Input
- **Format:** Requires `pandas` to be installed.
- **Preconditions:** The input must be a `pandas.DataFrame` compatible with `sktime`'s nested format. If the input is a standard NumPy array, the function will raise a `ValueError`.

### Output
- Returns a 3D `numpy.ndarray` with shape `(n_ts, sz, d)`, where `n_ts` is the number of time series, `sz` is the maximum length of the series (padded with `NaN` if lengths vary), and `d` is the number of dimensions.

### Valid Call Patterns
```python
import pandas as pd
from tslearn.utils import from_sktime_dataset

# Example with nested pandas series
sktime_df = pd.DataFrame()
sktime_df["dim_0"] = [pd.Series([1, 2, 3]), pd.Series([4, 5, 6])]
tslearn_arr = from_sktime_dataset(sktime_df)
print(f"Witness: {tslearn_arr.shape}")
```

### LLM Instruction Prompt
- Always ensure `pandas` is available in the environment before calling this function.
- Only pass `pandas.DataFrame` objects formatted for `sktime`.
- Do not pass raw NumPy arrays; use `tslearn.utils.to_time_series_dataset` for standard NumPy inputs instead.

### Prompt Snippet
```text
from tslearn.utils import from_sktime_dataset
# Ensure input is a pandas DataFrame in sktime format
tslearn_data = from_sktime_dataset(sktime_df)
```

### Common Failure Modes
- **Missing Dependency:** Calling the function without `pandas` installed will result in an `ImportError`.
- **Invalid Input Type:** Passing a 3D NumPy array directly will trigger a `ValueError: X is not a valid input sktime array`.
- **Inconsistent Dimensions:** If the DataFrame columns do not contain `pd.Series` objects, the conversion logic may fail or produce unexpected shapes.

### Fix Code Hint
```python
import pandas as pd
import numpy as np
from tslearn.utils import from_sktime_dataset

# Ensure input is a DataFrame with pd.Series columns
def safe_convert(df):
    try:
        return from_sktime_dataset(df)
    except ValueError as e:
        print(f"Conversion failed: {e}")
        return None
```

## API Test: `from_stumpy_dataset`

### Signature
```python
def from_stumpy_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:256_

_Source doc:_ Transform a stumpy-compatible dataset into a tslearn dataset. Parameters ---------- X: list of arrays of shapes (d, sz_i) if d > 1 or (sz_i, ) otherwise stumpy-formatted dataset. Returns ------- array, shape=(n_ts, sz, d), where sz is the maximum of all array lengths tslearn-formatted dataset Examples -------- >>> stumpy_arr = [numpy.random.randn(10), numpy.random.randn(10)] >>> tslearn_arr = from_stumpy_dataset(stumpy_arr) >>> tslearn_arr.shape (2, 10, 1) >>> stumpy_arr = [numpy.random.randn(3, 10), numpy.random.randn(3, 5)] >>> tslearn_arr = from_stumpy_dataset(stumpy_arr) >>> tslearn_arr.shape (2, 10, 3)

### Goal
Converts time-series data formatted for the `stumpy` library into the standard 3D `(n_ts, sz, d)` NumPy array format required by `tslearn` estimators.

### Parameters
- `X`: A list of NumPy arrays representing the dataset. Each array in the list must have a shape of `(d, sz_i)` for multivariate data (where `d > 1`) or `(sz_i, )` for univariate data.

### Input
- **Format:** A list of arrays where each array represents a single time series.
- **Preconditions:** The input must be compatible with `stumpy`'s expected data structure. If the input is not a list of arrays, the function may raise a `ValueError` or `TypeError` during array concatenation.

### Output
- **Format:** A 3D NumPy array of shape `(n_ts, sz, d)`.
- **Description:** The output is a standardized `tslearn` dataset where `n_ts` is the number of time series, `sz` is the maximum length among all input series (padded if necessary), and `d` is the number of dimensions.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import from_stumpy_dataset

# Univariate example
stumpy_arr = [np.random.randn(10), np.random.randn(10)]
tslearn_arr = from_stumpy_dataset(stumpy_arr)
print(f"Shape: {tslearn_arr.shape}") # (2, 10, 1)

# Multivariate example
stumpy_arr_multi = [np.random.randn(3, 10), np.random.randn(3, 5)]
tslearn_arr_multi = from_stumpy_dataset(stumpy_arr_multi)
print(f"Shape: {tslearn_arr_multi.shape}") # (2, 10, 3)
```

### LLM Instruction Prompt
- Use `from_stumpy_dataset` only when converting data originating from the `stumpy` library.
- Ensure the input is a list of NumPy arrays.
- Always verify the resulting shape is `(n_ts, sz, d)` before passing the data to `tslearn` estimators.

### Prompt Snippet
```text
from tslearn.utils import from_stumpy_dataset
# X is a list of stumpy-formatted arrays
dataset = from_stumpy_dataset(X)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing arrays with inconsistent dimensions `d` across the list will cause a failure during the internal stacking process.
- **Non-List Input:** Passing a single array instead of a list of arrays will result in an error, as the function expects an iterable of time series.
- **Empty Input:** Passing an empty list will result in an empty array or an error depending on the NumPy version and internal handling.

### Fix Code Hint
```python
# Ensure input is a list of arrays even if only one time series exists
if isinstance(X, np.ndarray):
    X = [X]
tslearn_data = from_stumpy_dataset(X)
```

## API Test: `from_tsfresh_dataset`

### Signature
```python
def from_tsfresh_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:578_

_Source doc:_ Transform a tsfresh-compatible dataset into a tslearn dataset. Parameters ---------- X: pandas data-frame tsfresh-formatted dataset ("flat" data frame, as described `there <https://tsfresh.readthedocs.io/en/latest/text/data_formats.html#input-option-1-flat-dataframe-or-wide-dataframe>`_) Returns ------- array, shape=(n_ts, sz, d) tslearn-formatted dataset. Column order is kept the same as in the original data frame. Examples -------- >>> import pandas as pd >>> tsfresh_df = pd.DataFrame(columns=["id", "time", "a", "b"]) >>> tsfresh_df["id"] = [0, 0, 0] >>> tsfresh_df["time"] = [0, 1, 2] >>> tsfresh_df["a"] = [-1, 4, 7] >>> tsfresh_df["b"] = [8, -3, 2] >>> tslearn_arr = from_tsfresh_dataset(tsfresh_df) >>> tslearn_arr.shape (1, 3, 2) >>> tsfresh_df = pd.DataFrame(columns=["id", "time", "a"]) >>> tsfresh_df["id"] = [0, 0, 0, 1, 1] >>> tsfresh_df["time"] = [0, 1, 2, 0, 1] >>> tsfresh_df["a"] = [-1, 4, 7, 9, 1] >>> tslearn_arr = from_tsfresh_dataset(tsfresh_df) >>> tslearn_arr.shape (2, 3, 1) >>> tsfresh_df = numpy.random.randn(10, 1, 16) >>> from_tsfresh_dataset( ...     tsfresh_df ... )  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: X is not a valid input tsfresh array. Notes ----- Conversion from/to tsfresh format requires pandas to be installed.

### Goal
Converts a "flat" pandas DataFrame formatted for `tsfresh` into a 3D NumPy array compatible with `tslearn` estimators.

### Parameters
- `X`: A pandas DataFrame containing at least an "id" column and a "time" column, representing time-series data in a long-format structure.

### Input
- **Format:** Must be a pandas DataFrame.
- **Preconditions:** The `pandas` library must be installed. The DataFrame must follow the `tsfresh` flat format (containing "id" and "time" columns).
- **Constraint:** The input must be a valid `tsfresh` DataFrame; passing raw NumPy arrays or other non-DataFrame types will raise a `ValueError`.

### Output
- Returns a 3D NumPy array of shape `(n_ts, sz, d)`, where `n_ts` is the number of unique time series, `sz` is the number of time steps, and `d` is the number of features (columns other than "id" and "time").

### Valid Call Patterns
```python
import pandas as pd
from tslearn.utils import from_tsfresh_dataset

# Example: Converting a flat DataFrame to tslearn format
df = pd.DataFrame({"id": [0, 0, 1, 1], "time": [0, 1, 0, 1], "val": [1, 2, 3, 4]})
tslearn_data = from_tsfresh_dataset(df)
print(tslearn_data.shape)  # (2, 2, 1)
```

### LLM Instruction Prompt
- Always ensure the input is a pandas DataFrame before calling `from_tsfresh_dataset`.
- Verify that the DataFrame contains the required "id" and "time" columns.
- Use this function when integrating data pipelines originating from `tsfresh` into `tslearn` models.

### Prompt Snippet
```text
from tslearn.utils import from_tsfresh_dataset
# Ensure X is a pandas DataFrame with 'id' and 'time' columns
X_tslearn = from_tsfresh_dataset(X_pandas)
```

### Common Failure Modes
- **Missing Dependencies:** Calling the function without `pandas` installed will result in an `ImportError`.
- **Invalid Input Type:** Passing a NumPy array or list instead of a pandas DataFrame will raise a `ValueError`.
- **Missing Columns:** If the DataFrame lacks the "id" or "time" columns required by the `tsfresh` specification, the function will fail to parse the structure correctly.

### Fix Code Hint
```python
import pandas as pd
# Ensure the input is a DataFrame and has the necessary columns
if not isinstance(X, pd.DataFrame):
    X = pd.DataFrame(X)
if "id" not in X.columns or "time" not in X.columns:
    raise ValueError("Input must be a tsfresh-formatted DataFrame with 'id' and 'time' columns.")
```

## API Test: `gak`

### Signature
```python
def gak(s1, s2, sigma=1.0, be=None)
```
_Source: tslearn/tslearn/metrics/_gak.py:99  (+1 more definition site/overload)_

_Source doc:_ Compute Global Alignment Kernel (GAK) between (possibly multidimensional) time series and return it. .. math:: \text{gak}(\mathbf{x}, \mathbf{y}) = \frac{k(\mathbf{x}, \mathbf{y})} {\sqrt{k(\mathbf{x}, \mathbf{x})k(\mathbf{y}, \mathbf{y})}} where .. math:: k(\mathbf{x}, \mathbf{y}) = \sum_{\pi \in \mathcal{A}(\mathbf{x}, \mathbf{y})} \prod_{i=1}^{ | \pi | } \exp \left( - \frac{ \left\| x_{\pi_1(i)} - y_{\pi_2{j}} \right\|^2}{2 \sigma^2} \right) It is not required that both time series share the same size, but they must be the same dimension. GAK was originally presented in [1]_. This is a normalized version that ensures that :math:`gak(x,x)=1` for all :math:`x` and :math:`gak(x,y) \in [0, 1]` for all :math:`x, y`. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. sigma : float (default 1.) Bandwidth of the internal gaussian kernel used for GAK. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- float Kernel value Examples -------- >>> float(gak([1, 2, 3], [1., 2., 2., 3.], sigma=2.))  # doctest: +ELLIPSIS 0.839... >>> float(gak([1, 2, 3], [1., 2., 2., 3., 4.]))  # doctest: +ELLIPSIS 0.273... See Also -------- cdist_gak : Compute cross-similarity matrix using Global Alignment kernel References ---------- .. [1] M. Cuturi, "Fast global alignment kernels,"

### Goal
Computes the normalized Global Alignment Kernel (GAK) similarity between two time series, providing a kernel-based alignment measure that maps time-series pairs to the range [0, 1].

### Parameters
- `s1`: The first time series, provided as an array-like object of shape `(sz1, d)` or `(sz1,)`.
- `s2`: The second time series, provided as an array-like object of shape `(sz2, d)` or `(sz2,)`.
- `sigma`, default `1.0`: The bandwidth parameter of the internal Gaussian kernel; controls the sensitivity of the alignment.
- `be`, default `None`: The backend engine to use for computation. Can be a `NumPyBackend` instance, `PyTorchBackend` instance, or the strings `"numpy"` or `"pytorch"`.

### Input
- **Data Format:** Time series must be provided as array-like structures. If univariate, shape `(sz,)` is accepted; if multivariate, shape `(sz, d)` is required.
- **Preconditions:** Both `s1` and `s2` must have the same dimensionality `d`.
- **Backend:** If using `PyTorch` tensors, ensure the `pytorch` package is installed.

### Output
Returns a `float` representing the normalized kernel similarity value between the two input time series.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Standard usage with NumPy
s1 = np.array([1, 2, 3])
s2 = np.array([1, 2, 2, 3])
result = tslearn.metrics.gak(s1, s2, sigma=2.0)
print(f"Witness: {result:.3f}")
```

### LLM Instruction Prompt
- Always import `tslearn.metrics` to access `gak`.
- Ensure `s1` and `s2` have matching dimensions.
- If performing differentiable programming, explicitly set `be="pytorch"` and ensure inputs are `torch.Tensor` objects.
- Use `sigma` to tune the kernel bandwidth; a value of 0 will raise a `ZeroDivisionError`.

### Prompt Snippet
```text
from tslearn.metrics import gak
# Compute GAK between two series
score = gak(s1, s2, sigma=1.0)
```

### Common Failure Modes
- **ZeroDivisionError:** Occurs if `sigma` is set to 0.
- **Dimension Mismatch:** Passing time series with different dimensions `d` will result in an error.
- **Backend Mismatch:** Attempting to use `PyTorch` features without the `pytorch` package installed or without passing the correct `be` parameter.

### Fix Code Hint
```python
# Ensure sigma > 0 and dimensions match
try:
    score = tslearn.metrics.gak(s1, s2, sigma=1.0)
except ZeroDivisionError:
    print("Sigma must be greater than 0.")
except ValueError:
    print("Time series must have the same dimension.")
```

## API Test: `gamma_soft_dtw`

### Signature
```python
def gamma_soft_dtw(dataset, n_samples=100, random_state=None, be=None)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:474_

_Source doc:_ Compute gamma value to be used for GAK/Soft-DTW. This method was originally presented in [1]_. Parameters ---------- dataset : array-like, shape=(n_ts, sz, d) or (n_ts, sz1) or (sz,) A dataset of time series. If shape is (n_ts, sz), the dataset is composed of univariate time series. If shape is (sz,), the dataset is composed of a unique univariate time series. n_samples : int (default: 100) Number of samples on which median distance should be estimated. random_state : integer or numpy.RandomState or None (default: None) The generator used to draw the samples. If an integer is given, it fixes the seed. Defaults to the global numpy random number generator. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- float Suggested :math:`\gamma` parameter for the Soft-DTW. Examples -------- >>> dataset = [[1, 2, 2, 3], [1., 2., 3., 4.]] >>> float(gamma_soft_dtw(dataset=dataset, ...                      n_samples=200, ...                      random_state=0))  # doctest: +ELLIPSIS 8.0... See Also -------- sigma_gak : Compute sigma parameter for Global Alignment kernel References ---------- .. [1] M. Cuturi, "Fast global alignment kernels," ICML 2011.

### Goal
Computes the optimal $\gamma$ (gamma) smoothing parameter for Soft-DTW or Global Alignment Kernel (GAK) by estimating the median distance between time series in the provided dataset.

### Parameters
- `dataset`: The input time-series data. Can be a 3D array `(n_ts, sz, d)`, a 2D array `(n_ts, sz)`, or a 1D array `(sz,)`.
- `n_samples`, default `100`: The number of random pairs to sample from the dataset to estimate the median distance.
- `random_state`, default `None`: Seed or random state object to ensure reproducibility of the sampling process.
- `be`, default `None`: The computation backend. Accepts `NumPyBackend`, `PyTorchBackend`, or strings `"numpy"`/`"pytorch"`. If `None`, the backend is inferred from the input `dataset`.

### Input
The `dataset` must be formatted as a time-series dataset. If the input is not already in a standard 3D format, use `tslearn.utils.to_time_series_dataset` to ensure compatibility. The function expects numerical data (NumPy arrays or PyTorch tensors).

### Output
Returns a `float` representing the suggested $\gamma$ parameter value for use in Soft-DTW or GAK algorithms.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

dataset = [[1, 2, 2, 3], [1.0, 2.0, 3.0, 4.0]]
gamma = tslearn.metrics.gamma_soft_dtw(
    dataset=dataset, 
    n_samples=200, 
    random_state=0
)
print(f"Suggested gamma: {gamma}")
```

### LLM Instruction Prompt
- Always use `tslearn.metrics.gamma_soft_dtw` to access this function.
- Ensure the `dataset` is passed as a list or array-like structure compatible with `tslearn`'s internal conversion.
- If using `PyTorch` tensors, explicitly set `be="pytorch"` or ensure the backend is correctly instantiated to avoid type mismatch errors.

### Prompt Snippet
```text
# Example of calculating gamma for Soft-DTW
gamma = tslearn.metrics.gamma_soft_dtw(dataset=X, n_samples=100, random_state=42)
```

### Common Failure Modes
- **Shape Mismatch:** Passing data that is not a valid time-series format (e.g., missing the time dimension). Use `to_time_series_dataset` first.
- **Backend Incompatibility:** Passing a PyTorch tensor while the default NumPy backend is active without specifying `be="pytorch"`.
- **Insufficient Samples:** Providing a dataset with fewer samples than required for the `n_samples` parameter, which may lead to inaccurate median distance estimation.

### Fix Code Hint
```python
# Ensure data is 3D before passing to gamma_soft_dtw
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
gamma = tslearn.metrics.gamma_soft_dtw(dataset=X_formatted)
```

## API Test: `get_backend`

### Signature
```python
def get_backend(self)
```
_Source: tslearn/tslearn/backend/backend.py:84_

### Goal
Retrieves the currently active computation backend instance (either `NumPyBackend` or `PyTorchBackend`) associated with the `Backend` object.

### Parameters
- `self`: The `Backend` instance initialized with a specific engine (e.g., "torch" or "numpy").

### Input
The caller must have an initialized `tslearn.backend.Backend` object. The backend engine must have been previously set or instantiated via the `Backend` constructor or `set_backend` method.

### Output
Returns an instance of the active backend class (e.g., `tslearn.backend.PyTorchBackend` or `tslearn.backend.NumPyBackend`), which provides the low-level operations for time-series metrics.

### Valid Call Patterns
```python
from tslearn.backend import Backend

# Initialize backend and retrieve the instance
backend_obj = Backend("torch")
be = backend_obj.get_backend()
```

### LLM Instruction Prompt
- Use `get_backend()` only on an instance of `tslearn.backend.Backend`.
- Do not assume the return type is a string; it returns a backend object instance capable of performing numerical operations.
- Ensure the `pytorch` package is installed if the backend is set to "torch".

### Prompt Snippet
```text
# Retrieve the active backend instance from a Backend object
be = backend_instance.get_backend()
```

### Common Failure Modes
- Calling `get_backend()` on an uninitialized or improperly configured `Backend` object.
- Assuming the return value is a string (e.g., "torch") rather than the backend object itself.
- Attempting to use a "torch" backend in an environment where `pytorch` is not installed, leading to runtime errors during subsequent metric computations.

### Fix Code Hint
```python
from tslearn.backend import Backend

# Ensure the backend is correctly instantiated before calling get_backend
backend_obj = Backend("numpy")
be = backend_obj.get_backend()
assert hasattr(be, "is_numpy")
```

## API Test: `get_cluster_probas`

### Signature
```python
def get_cluster_probas(self, Xi)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:215_

_Source doc:_ Compute cluster probability :math:`P(c_k | Xi)`. This quantity is computed using the following formula: .. math:: P(c_k | Xi) = \frac{s_k(Xi)}{\sum_j s_j(Xi)} where .. math:: s_k(Xi) = \frac{1}{1 + \exp{-\lambda \Delta_k(Xi)}} with .. math:: \Delta_k(Xi) = \frac{\bar{D} - d(Xi, c_k)}{\bar{D}} and :math:`\bar{D}` is the average of the distances between `Xi` and the cluster centers. Parameters ---------- Xi: numpy array, shape (t, d) A time series observed up to time t Returns ------- probas : numpy array, shape (n_clusters, ) Examples -------- >>> dataset = to_time_series_dataset([[1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [3, 2, 1, 1, 2, 3], ...                                   [3, 2, 1, 1, 2, 3]]) >>> y = [0, 0, 0, 1, 1, 1, 0, 0] >>> ts0 = to_time_series([1, 2]) >>> model = NonMyopicEarlyClassifier(n_clusters=3, lamb=0., ...                                  random_state=0) >>> probas = model.fit(dataset, y).get_cluster_probas(ts0) >>> probas.shape (3,) >>> probas  # doctest: +ELLIPSIS array([0.33..., 0.33..., 0.33...]) >>> model = NonMyopicEarlyClassifier(n_clusters=3, lamb=10000., ...                                  random_state=0) >>> probas = model.fit(dataset, y).get_cluster_probas(ts0) >>> probas.shape (3,) >>> probas array([0.5, 0.5, 0. ]) >>> ts1 = to_time_series([3, 2]) >>> model.get_cluster_probas(ts1)

### Goal
Computes the probability distribution of a time series belonging to each cluster defined by a `NonMyopicEarlyClassifier` model based on the distance to cluster centers.

### Parameters
- `self`: An instance of `NonMyopicEarlyClassifier` that has already been fitted to a dataset.
- `Xi`: A `numpy` array of shape `(t, d)` representing a single time series observed up to time `t`.

### Input
- **Preconditions:** The `NonMyopicEarlyClassifier` instance must be fitted using the `fit(X, y)` method before calling this function.
- **Data Format:** `Xi` must be a time series formatted as a 2D `numpy` array of shape `(t, d)`. Use `tslearn.utils.to_time_series` to ensure the input is correctly formatted.

### Output
- Returns a `numpy` array of shape `(n_clusters,)` containing the probability values :math:`P(c_k | Xi)` for each cluster.

### Valid Call Patterns
```python
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.utils import to_time_series, to_time_series_dataset

# Setup
dataset = to_time_series_dataset([[1, 2, 3, 4, 5, 6], [3, 2, 1, 1, 2, 3]])
y = [0, 1]
ts = to_time_series([1, 2])

# Execution
model = NonMyopicEarlyClassifier(n_clusters=2).fit(dataset, y)
probas = model.get_cluster_probas(ts)
print(probas.shape)
```

### LLM Instruction Prompt
- Always ensure the `NonMyopicEarlyClassifier` is fitted before calling `get_cluster_probas`.
- Ensure the input `Xi` is converted to the standard `tslearn` time-series format using `to_time_series` if it is a raw list or 1D array.
- The output is a probability distribution; verify that the sum of the returned array equals 1.0.

### Prompt Snippet
```text
model.get_cluster_probas(Xi)
```

### Common Failure Modes
- **NotFittedError:** Calling `get_cluster_probas` before calling `fit` on the classifier.
- **Shape Mismatch:** Passing a 3D dataset array instead of a 2D time-series array `(t, d)` for `Xi`.
- **Invalid Input Type:** Passing a list or non-NumPy array without converting it via `to_time_series`.

### Fix Code Hint
```python
# Ensure input is 2D (t, d)
from tslearn.utils import to_time_series
ts_formatted = to_time_series(my_raw_data)
probas = model.get_cluster_probas(ts_formatted)
```

## API Test: `get_config`

### Signature
```python
def get_config(self)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:160  (+1 more definition site/overload)_

### Goal
Retrieves the configuration parameters of a `tslearn` estimator, typically used for serialization or inspecting the hyper-parameters of a fitted model.

### Parameters
- `self`: The estimator instance (e.g., `LearningShapelets`) from which the configuration is to be retrieved.

### Input
- The caller must provide an initialized or fitted `tslearn` estimator instance. No additional arguments are required.

### Output
Returns `unspecified` — The return value is a dictionary containing the estimator's configuration parameters (e.g., hyper-parameters used during initialization).

### Valid Call Patterns
```python
# Inferred from signature:
# Assuming an instance of an estimator like LearningShapelets
config = estimator.get_config()
```

### LLM Instruction Prompt
- Use `get_config` only on `tslearn` estimator objects that implement the `get_config` method (e.g., `LearningShapelets`).
- Do not pass any arguments to the function.
- Use the returned dictionary to inspect or clone the estimator's hyper-parameters.

### Prompt Snippet
```text
# Retrieve configuration from a tslearn estimator
params = estimator.get_config()
```

### Common Failure Modes
- Calling `get_config` on a class instead of an instance.
- Assuming the return type is a specific object type when it is implementation-dependent (dictionary).
- Attempting to pass arguments to the function, which will result in a `TypeError`.

### Fix Code Hint
```python
# Ensure the estimator is instantiated before calling get_config
from tslearn.shapelets import LearningShapelets
clf = LearningShapelets()
config = clf.get_config()
print(config)
```

## API Test: `get_early_predict_generator`

### Signature
```python
def get_early_predict_generator(self, n_ts=1)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:627_

_Source doc:_ Allows streaming incoming timestamps of a time series and retrieving current predicted class as well as estimated delay before prediction timestamps. Prediction timestamps are timestamps at which a prediction is made in early classification setting. Parameters ---------- n_ts: int (default 1) The number of time series that will be predicted by the generator Returns ------- generator Use the `send` method of the generator to stream timestamps as they become available and retrieve associated predictions and delays. This method takes an array-like, shape (n_ts, n_timestamps, n_features), representing freshly aquired data for all timeseries as input. This new data is concatenated with previously sent data, if any, to output the predicted classes and estimated delays before optimal prediction timestamps for all timeseries based all on available data (same output as :func:`~early_predict`). Use n_timestamps = 1 for step by step feeding. Examples -------- >>> dataset = to_time_series_dataset([[1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [3, 2, 1, 1, 2, 3], ...                                   [3, 2, 1, 1, 2, 3]]) >>> y = [0, 0, 0, 1, 1, 1, 0, 0] >>> model = NonMyopicEarlyClassifier(n_clusters=3, lamb=1000., ...                                  cost_time_parameter=.1, ...                                  random_state=0).fit(dataset, y) >>> incoming_timestamps = np.array([1, 2, 3, 3, 1, 2]).reshape(6, 1, 1, 1) >>> gen = model.get_early_predict_generator() >>> for x in incoming_timestamps: ...     print(gen.send(x)) (array([0]), array([3])) (array([0]), array([2])) (array([0]), array([1])) (array([1]), array([0])) (array([1]), array([0])) (array([1]), array([0]))

### Goal
Provides a stateful generator for real-time early classification, allowing incremental updates of time-series data to retrieve current class predictions and estimated remaining time until an optimal prediction can be made.

### Parameters
- `self`: An instance of an early classification model (e.g., `NonMyopicEarlyClassifier`) that has already been fitted.
- `n_ts`, default `1`: The number of time series instances that will be processed by the generator.

### Input
- The model must be fitted using `fit(X, y)` before calling this method.
- The generator's `send(x)` method expects `x` to be an array-like of shape `(n_ts, n_timestamps, n_features)`, representing the new time-series segments to be appended to the internal state.

### Output
- Returns a Python generator object. Calling `send(x)` on this generator returns a tuple containing the predicted classes and the estimated delays (in number of timestamps) until the optimal prediction point.

### Valid Call Patterns
```python
import numpy as np
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.utils import to_time_series_dataset

# Setup and fit
dataset = to_time_series_dataset([[1, 2, 3], [3, 2, 1]])
y = [0, 1]
model = NonMyopicEarlyClassifier(n_clusters=2).fit(dataset, y)

# Initialize generator
gen = model.get_early_predict_generator(n_ts=1)
# Stream data
prediction, delay = gen.send(np.array([[[1]]]))
print(f"Prediction: {prediction}, Delay: {delay}")
```

### LLM Instruction Prompt
- Always ensure the model is fitted before invoking `get_early_predict_generator`.
- Use the `send()` method on the returned generator to feed data incrementally.
- Ensure the input to `send()` matches the `n_ts` defined during generator initialization and has the shape `(n_ts, n_timestamps, n_features)`.

### Prompt Snippet
```text
gen = model.get_early_predict_generator(n_ts=...)
result = gen.send(new_data_segment)
```

### Common Failure Modes
- Calling `send()` before the model has been fitted, resulting in an `AttributeError` or `NotFittedError`.
- Mismatch between the `n_ts` parameter provided to the generator and the shape of the data passed to `send()`.
- Passing data with incorrect dimensions (must be 3D: `n_ts, n_timestamps, n_features`).

### Fix Code Hint
```python
# Ensure data is 3D before sending to the generator
# If data is 1D (n_features=1, n_timestamps=1), reshape to (n_ts, 1, 1)
x_reshaped = x.reshape(n_ts, 1, n_features)
prediction, delay = gen.send(x_reshaped)
```

## API Test: `get_early_predict_proba_generator`

### Signature
```python
def get_early_predict_proba_generator(self, n_ts=1)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:677_

_Source doc:_ Allows streaming incoming timestamps of a time series and retrieving current probability estimates as well as estimated delay before prediction timestamps. Prediction timestamps are timestamps at which a prediction is made in early classification setting. Parameters ---------- n_ts: int (default 1) The number of time series that will be predicted by the generator Returns ------- generator Use the `send` method of the generator to stream timestamps as they become available and retrieve associated prediction probalities and delays. This method takes an array-like, shape (n_ts, n_timestamps, n_features), representing freshly aquired data for all timeseries as input. This new data is concatenated with previously sent data, if any, to output the predicted probability estimates and estimated delays before optimal prediction timestamps for all timeseries based all on available data (same output as :func:`~early_predict_proba`). Use n_timestamps = 1 for step by step feeding. Examples -------- >>> dataset = to_time_series_dataset([[1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 4, 5, 6], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [1, 2, 3, 3, 2, 1], ...                                   [3, 2, 1, 1, 2, 3], ...                                   [3, 2, 1, 1, 2, 3]]) >>> y = [0, 0, 0, 1, 1, 1, 0, 0] >>> model = NonMyopicEarlyClassifier(n_clusters=3, lamb=1000., ...                                  cost_time_parameter=.1, ...                                  random_state=0).fit(dataset, y) >>> incoming_timestamps = np.array([1, 2, 3, 3, 1, 2]).reshape(6, 1, 1, 1) >>> gen = model.get_early_predict_proba_generator() >>> for x in incoming_timestamps: ...     print(gen.send(x)) (array([[1., 0.]]), array([3])) (array([[1., 0.]]), array([2])) (array([[1., 0.]]), array([1])) (array([[0., 1.]]), array([0])) (array([[0., 1.]]), array([0])) (array([[0., 1.]]), array([0]))

### Goal
Provides a generator object for real-time, incremental early classification, allowing the user to stream incoming time-series data points and receive updated class probability estimates and predicted delays until the optimal classification time.

### Parameters
- `self`: An instance of a fitted early classification model (e.g., `NonMyopicEarlyClassifier`).
- `n_ts`, default `1`: The number of time series that will be processed by the generator.

### Input
- The model must be fitted using `fit(X, y)` before calling this method.
- The generator's `send()` method expects an array-like input of shape `(n_ts, n_timestamps, n_features)` representing the newly acquired time-series segments.

### Output
- Returns a Python generator object. Calling `generator.send(data)` returns a tuple containing:
    - `probabilities`: An array of shape `(n_ts, n_classes)` representing the current class probability estimates.
    - `delays`: An array of shape `(n_ts,)` representing the estimated number of time steps remaining before the optimal prediction time.

### Valid Call Patterns
```python
import numpy as np
from tslearn.early_classification import NonMyopicEarlyClassifier
from tslearn.utils import to_time_series_dataset

# Setup and fit
dataset = to_time_series_dataset([[1, 2, 3], [3, 2, 1]])
y = [0, 1]
model = NonMyopicEarlyClassifier(n_clusters=2).fit(dataset, y)

# Initialize generator
gen = model.get_early_predict_proba_generator(n_ts=1)
next(gen) # Prime the generator
result = gen.send(np.array([[[1]]])) # Send new data point
print(result)
```

### LLM Instruction Prompt
- Always ensure the model is fitted before requesting the generator.
- Use `gen.send(data)` to feed data incrementally.
- The input to `send` must be 3D: `(n_ts, n_timestamps, n_features)`.
- The generator maintains internal state; do not reuse the same generator instance for different time series unless `n_ts` matches.

### Prompt Snippet
```text
# Example of streaming data to the generator
gen = model.get_early_predict_proba_generator(n_ts=1)
next(gen) # Initialize
for step_data in stream:
    probs, delays = gen.send(step_data)
```

### Common Failure Modes
- Calling `send()` before the generator is initialized (must call `next(gen)` or `gen.send(None)` first).
- Providing input data with dimensions inconsistent with the `n_ts` parameter or the model's expected feature count.
- Attempting to use the generator with an unfitted model, which will raise a `NotFittedError`.

### Fix Code Hint
```python
# Ensure the generator is primed before sending data
gen = model.get_early_predict_proba_generator(n_ts=1)
next(gen) 
# Now safe to send
probs, delays = gen.send(new_data)
```

## API Test: `get_weights`

### Signature
```python
def get_weights(self, layer_name=None)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:828_

_Source doc:_ Return model weights (or weights for a given layer if `layer_name` is provided). Parameters ---------- layer_name: str or None (default: None) Name of the layer for which  weights should be returned. If None, all model weights are returned. Available layer names with weights are: - "shapelets_i_j" with i an integer for the shapelet id and j an integer for the dimension - "classification" for the final classification layer Returns ------- list list of model (or layer) weights Examples -------- >>> from tslearn.generators import random_walk_blobs >>> X, y = random_walk_blobs(n_ts_per_blob=100, sz=256, d=1, n_blobs=3) >>> clf = LearningShapelets(n_shapelets_per_size={10: 5}, max_iter=1, ...                     verbose=0) >>> clf.fit(X, y).get_weights("classification")[0].shape (5, 3) >>> clf.get_weights("shapelets_0")[0].shape (5, 10, 1) >>> len(clf.get_weights("shapelets_0")) 1

### Goal
Retrieves the internal weight parameters of a trained `LearningShapelets` model, either for the entire model or for a specific layer identified by name.

### Parameters
- `self`: The fitted `LearningShapelets` estimator instance.
- `layer_name`, default `None`: A string specifying the layer to extract weights from (e.g., `"classification"` or `"shapelets_i_j"`). If `None`, returns all model weights.

### Input
- **Preconditions:** The `LearningShapelets` estimator must be fitted using the `.fit(X, y)` method before calling `get_weights`.
- **Data Format:** The estimator expects 3D NumPy arrays of shape `(n_ts, max_sz, d)` for the training data.

### Output
Returns `list`: A list containing the requested model or layer weights as NumPy arrays.

### Valid Call Patterns
```python
# Extract weights for the classification layer
weights_clf = clf.get_weights("classification")

# Extract weights for a specific shapelet
weights_shp = clf.get_weights("shapelets_0")

# Extract all model weights
all_weights = clf.get_weights()
```

### LLM Instruction Prompt
- Always ensure the `LearningShapelets` estimator has been successfully fitted before invoking `get_weights`.
- Use the `layer_name` parameter to target specific components like `"classification"` or specific shapelet indices (`"shapelets_i_j"`).
- Remember that the returned object is a list of NumPy arrays; index into the list to access specific weight tensors.

### Prompt Snippet
```text
# Ensure the model is fitted
clf.fit(X, y)
# Access specific layer weights
weights = clf.get_weights(layer_name="classification")
```

### Common Failure Modes
- **NotFittedError:** Calling `get_weights` before the estimator has been fitted.
- **Invalid Layer Name:** Providing a `layer_name` string that does not match the internal naming convention (e.g., typos in `"shapelets_i_j"`).
- **Backend Mismatch:** Assuming the output format is a PyTorch tensor when the estimator is operating in a standard NumPy-based environment.

### Fix Code Hint
```python
# Ensure the model is fitted before access
if not hasattr(clf, "model_"):
    clf.fit(X, y)
weights = clf.get_weights("classification")
```

## API Test: `grabocka_params_to_shapelet_size_dict`

### Signature
```python
def grabocka_params_to_shapelet_size_dict(n_ts, ts_sz, n_classes, l, r)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:235_

_Source doc:_ Compute number and length of shapelets. This function uses the heuristic from [1]_. Parameters ---------- n_ts: int Number of time series in the dataset ts_sz: int Length of time series in the dataset n_classes: int Number of classes in the dataset l: float Fraction of the length of time series to be used for base shapelet length r: int Number of different shapelet lengths to use Returns ------- dict Dictionary giving, for each shapelet length, the number of such shapelets to be generated Examples -------- >>> d = grabocka_params_to_shapelet_size_dict( ...         n_ts=100, ts_sz=100, n_classes=3, l=0.1, r=2) >>> keys = sorted(d.keys()) >>> print(keys) [10, 20] >>> print([d[k] for k in keys]) [4, 4] References ---------- .. [1] J. Grabocka et al. Learning Time-Series Shapelets. SIGKDD 2014.

### Goal
Computes the optimal number and length of shapelets for a given dataset using the heuristic proposed by Grabocka et al. (2014) to facilitate effective time-series classification.

### Parameters
- `n_ts`: `int`. The total number of time series samples in the training dataset.
- `ts_sz`: `int`. The length (number of time steps) of the time series in the dataset.
- `n_classes`: `int`. The number of distinct target classes in the classification task.
- `l`: `float`. The fraction of the total time series length to be used as the base shapelet length.
- `r`: `int`. The number of distinct shapelet lengths to generate.

### Input
The function requires scalar integer and float inputs representing dataset metadata. It does not require actual time-series data arrays, only the dimensions of the dataset intended for shapelet-based analysis.

### Output
Returns a `dict` where keys are integers representing the generated shapelet lengths and values are integers representing the number of shapelets to be generated for each corresponding length.

### Valid Call Patterns
```python
from tslearn.shapelets import grabocka_params_to_shapelet_size_dict

# Example based on documentation
params = grabocka_params_to_shapelet_size_dict(n_ts=100, ts_sz=100, n_classes=3, l=0.1, r=2)
print(params)
# Output: {10: 4, 20: 4}
```

### LLM Instruction Prompt
- Use this function to determine hyper-parameters for `LearningShapelets` estimators.
- Ensure `n_ts`, `ts_sz`, `n_classes`, and `r` are positive integers.
- Ensure `l` is a float representing a fraction (typically between 0 and 1).

### Prompt Snippet
```text
from tslearn.shapelets import grabocka_params_to_shapelet_size_dict
shapelet_config = grabocka_params_to_shapelet_size_dict(n_ts=..., ts_sz=..., n_classes=..., l=..., r=...)
```

### Common Failure Modes
- Passing non-integer values for `n_ts`, `ts_sz`, `n_classes`, or `r`, which may cause arithmetic errors or unexpected dictionary keys.
- Providing an `l` value that results in a base shapelet length of 0 or greater than `ts_sz`.

### Fix Code Hint
```python
from tslearn.shapelets import grabocka_params_to_shapelet_size_dict
# Verify correctness witness
d = grabocka_params_to_shapelet_size_dict(n_ts=100, ts_sz=100, n_classes=3, l=0.1, r=2)
assert d == {10: 4, 20: 4}, f"Expected { {10: 4, 20: 4} }, got {d}"
print("Correctness witness verified: grabocka_params_to_shapelet_size_dict output matches expected heuristic.")
```

## API Test: `grad`

### Signature
```python
def grad(self)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:1137_

_Source doc:_ Compute gradient of soft-DTW w.r.t. D by dynamic programming. Returns ------- grad: array-like, shape=(m, n) Gradient w.r.t. D.

### Goal
Computes the gradient of the soft-DTW loss function with respect to the pairwise distance matrix `D` using dynamic programming.

### Parameters
- `self`: An instance of a soft-DTW implementation class (e.g., `SoftDTW`) that has been initialized and contains the necessary state for gradient computation.

### Input
- **Preconditions:** The instance `self` must have been initialized with valid parameters (such as `gamma`). The internal state must be populated such that the dynamic programming table required for the gradient calculation is available.
- **Data Format:** The underlying distance matrix `D` is typically derived from the input time-series sequences of shape `(n_ts, max_sz, d)`.

### Output
Returns `grad`: An array-like object of shape `(m, n)`, representing the gradient of the soft-DTW loss with respect to the pairwise distance matrix `D`.

### Valid Call Patterns
```python
# Inferred from signature:
# Assuming 'sdtw' is an initialized instance of a soft-DTW class
gradient = sdtw.grad()
```

### LLM Instruction Prompt
- When using `grad`, ensure the parent object has been properly initialized.
- Note that this method is specific to soft-DTW variants and relies on the internal dynamic programming state of the object.
- If using the `PyTorch` backend, prefer automatic differentiation features where possible, as `grad` is a lower-level manual implementation of the gradient computation.

### Prompt Snippet
```text
# Example of accessing the gradient from a soft-DTW instance
sdtw = SoftDTW(gamma=1.0)
# ... (internal state setup)
grad_matrix = sdtw.grad()
```

### Common Failure Modes
- **Uninitialized State:** Calling `grad()` before the soft-DTW computation has been performed or before the internal DP table is populated will result in an error or incorrect values.
- **Backend Mismatch:** Attempting to use this method when the backend is not configured to support the specific dynamic programming implementation expected by the class.
- **Shape Mismatch:** If the input time series dimensions `m` and `n` are not correctly aligned with the expected distance matrix dimensions, the returned gradient shape will be invalid.

### Fix Code Hint
```python
# Ensure the instance is ready before calling grad
if hasattr(sdtw, 'grad'):
    gradient = sdtw.grad()
else:
    raise AttributeError("The provided object does not support the grad() method.")
```

## API Test: `in_file_string_replace`

### Signature
```python
def in_file_string_replace(filename, old_string, new_string)
```
_Source: tslearn/tslearn/datasets/datasets.py:57_

_Source doc:_ String replacement within a text file. It is used to fix typos in downloaded csv file. The code was modified from "https://stackoverflow.com/questions/4128144/" Parameters ---------- filename : str Path to the file where strings should be replaced old_string : str The string to be replaced in the file. new_string : str The new string that will replace old_string

### Goal
Performs an in-place string substitution within a specified text file, typically used to sanitize or correct formatting errors in downloaded time-series datasets.

### Parameters
- `filename`: `str`. The filesystem path to the target text or CSV file.
- `old_string`: `str`. The literal substring to be located and replaced within the file.
- `new_string`: `str`. The replacement string to substitute for every occurrence of `old_string`.

### Input
The function expects a valid file path to a readable and writable text-based file. The file must exist on the local filesystem, and the process must have sufficient permissions to read the file and write the modified content back to the disk.

### Output
Returns `None`. The operation is performed in-place on the file specified by `filename`.

### Valid Call Patterns
```python
from tslearn.datasets import in_file_string_replace
import tempfile

# Create a dummy file to demonstrate
with tempfile.NamedTemporaryFile(mode='w+', delete=False) as tmp:
    tmp.write("data_v1,1,2,3")
    tmp_path = tmp.name

# Perform replacement
in_file_string_replace(tmp_path, "data_v1", "data_v2")

# Verify result
with open(tmp_path, 'r') as f:
    print(f.read()) # Output: data_v2,1,2,3
```

### LLM Instruction Prompt
- Use `in_file_string_replace` only when you need to modify a local text file (e.g., a CSV) to fix typos or formatting issues before loading it into a `tslearn` dataset structure.
- Ensure the `filename` provided is an absolute or relative path to an existing file.
- Note that this function modifies the file in-place; ensure you have backups if the data is critical.

### Prompt Snippet
```text
from tslearn.datasets import in_file_string_replace
in_file_string_replace(filename="path/to/data.csv", old_string="typo", new_string="correct")
```

### Common Failure Modes
- **FileNotFoundError**: Occurs if the provided `filename` path does not exist.
- **PermissionError**: Occurs if the user does not have write access to the file or the directory.
- **Memory Errors**: Since the function reads the file content into memory, attempting to process extremely large files (multi-gigabyte) may cause the process to crash.

### Fix Code Hint
```python
# Ensure the file exists before calling
import os
if os.path.exists(filename):
    in_file_string_replace(filename, old_string, new_string)
else:
    raise FileNotFoundError(f"The file {filename} was not found.")
```

## API Test: `instantiate_backend`

### Signature
```python
def instantiate_backend(*args)
```
_Source: tslearn/tslearn/backend/backend.py:7_

_Source doc:_ Select backend. Parameter --------- *args : Input arguments can be Backend instance or string or array or None Arguments used to define the backend instance. Returns ------- backend : Backend instance The backend instance.

### Goal
Initializes and returns the appropriate computation backend (NumPy or PyTorch) used by `tslearn` for time-series metric computations and alignment operations.

### Parameters
- `*args`: Variable arguments used to define the backend. Can be a `Backend` instance, a string identifier (e.g., `"pytorch"`), an array (to infer the backend), or `None`.

### Input
- **Data/Formats:** Accepts `numpy` arrays or `torch` tensors.
- **Preconditions:** If requesting the `"pytorch"` backend, the `pytorch` package must be installed in the environment.
- **Type-Selection Rules:** The function performs a priority check: if an input is a `Torch` tensor, it defaults to `PyTorchBackend`; if it is a `NumPy` array, it defaults to `NumPyBackend`.

### Output
Returns `Backend instance` — An object providing the computation engine interface (e.g., `NumPyBackend` or `PyTorchBackend`) used to execute time-series operations.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend

# Initialize by string identifier
be = instantiate_backend("pytorch")

# Initialize by inferring from an array
be = instantiate_backend(my_numpy_array)
```

### LLM Instruction Prompt
- Always use `instantiate_backend` to retrieve the computation engine when performing custom metric calculations or when passing a `be` parameter to `tslearn` functions.
- If the user provides `torch` tensors, ensure the backend is set to `"pytorch"` to support automatic differentiation.
- Do not assume the backend type; always use the returned instance to check properties (e.g., `backend.is_numpy`).

### Prompt Snippet
```text
# Example of backend initialization for PyTorch
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch")
```

### Common Failure Modes
- **Missing Dependency:** Calling `instantiate_backend("pytorch")` without `torch` installed will raise an `ImportError` or `ValueError`.
- **Invalid Argument:** Passing unsupported types or unrecognized strings as `*args` may result in a failure to resolve the backend.
- **Backend Mismatch:** Attempting to use a `NumPyBackend` instance with `torch` tensors for operations requiring automatic differentiation will fail.

### Fix Code Hint
```python
# Ensure the backend matches the data type
import numpy as np
import torch
from tslearn.backend import instantiate_backend

# Correct way to handle dynamic backend selection
data = torch.tensor([1.0, 2.0])
be = instantiate_backend(data)
assert not be.is_numpy  # Confirms PyTorch backend is active
```

## API Test: `intercept_`

### Signature
```python
def intercept_(self)
```
_Source: tslearn/tslearn/svm/svm.py:39_

### Goal
Retrieves the intercept term (bias) learned by a `TimeSeriesSVC` estimator after the model has been fitted.

### Parameters
- `self`: An instance of a `tslearn` estimator (specifically `TimeSeriesSVC`) that has already been trained via the `fit` method.

### Input
The caller must ensure the estimator instance has been successfully fitted on a 3D time-series dataset (shape `(n_ts, max_sz, d)`) using the `fit()` method. Accessing this attribute before calling `fit()` will result in an `AttributeError`.

### Output
Returns `unspecified` — The intercept value(s) as a NumPy array or scalar, depending on the number of classes and the specific SVM configuration used during training.

### Valid Call Patterns
```python
from tslearn.svm import TimeSeriesSVC
# Assuming X and y are pre-formatted 3D time-series data
clf = TimeSeriesSVC(kernel="gak").fit(X, y)
intercept = clf.intercept_
print(f"Model intercept: {intercept}")
```

### LLM Instruction Prompt
- Only call `intercept_` on `tslearn` estimators that support SVM-based classification (e.g., `TimeSeriesSVC`).
- Ensure the estimator has been fitted; otherwise, the attribute will not exist.
- Do not attempt to set or modify this attribute directly, as it is managed by the `fit` process.

### Prompt Snippet
```text
# Ensure the model is fitted before accessing the intercept
clf.fit(X_train, y_train)
bias = clf.intercept_
```

### Common Failure Modes
- **AttributeError**: Occurs if `intercept_` is accessed before the `fit()` method has been called on the estimator.
- **Type Mismatch**: Attempting to access `intercept_` on estimators that do not implement SVM-based logic (e.g., `TimeSeriesKMeans`), as they do not define this attribute.

### Fix Code Hint
```python
# Check if the estimator is fitted before accessing internal attributes
if hasattr(clf, "intercept_"):
    print(clf.intercept_)
else:
    print("Model must be fitted before accessing intercept_.")
```

## API Test: `inv_transform_1d_sax`

### Signature
```python
def inv_transform_1d_sax(dataset_sax, breakpoints_avg_middle_, breakpoints_slope_middle_, original_size)
```
_Source: tslearn/tslearn/metrics/cysax.py:189_

_Source doc:_ Compute time series corresponding to given 1d-SAX representations. Parameters ---------- dataset_sax : array-like, shape=(n_ts, sz, 2 * d), dtype=float64 (Linux and MacOS) or float32 (Windows) A dataset of SAX series. breakpoints_avg_middle_ : array-like, ndim=1, dtype=float64 breakpoints_slope_middle_ : array-like, ndim=1, dtype=float64 original_size : int64 (Linux and MacOS) or int32 (Windows) Length of the original time series. Returns ------- dataset_out : array-like, shape=(n_ts, original_size, d), dtype=float64 A dataset of time series corresponding to the provided representation.

### Goal
Reconstructs time-series data from their 1d-SAX (Symbolic Aggregate approximation) representations using provided breakpoint information.

### Parameters
- `dataset_sax`: A 3D array-like object of shape `(n_ts, sz, 2 * d)` containing the SAX-encoded time series.
- `breakpoints_avg_middle_`: A 1D array of float64 values representing the middle points of the average value intervals used during SAX transformation.
- `breakpoints_slope_middle_`: A 1D array of float64 values representing the middle points of the slope intervals used during SAX transformation.
- `original_size`: An integer representing the length of the time series prior to SAX transformation.

### Input
The function expects `dataset_sax` to be a 3D array with shape `(n_ts, sz, 2 * d)`. The `breakpoints` must be 1D arrays of `float64` type. The `original_size` must be an integer matching the intended output time-series length.

### Output
Returns a 3D array of shape `(n_ts, original_size, d)` containing the reconstructed time series as `float64` values.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import inv_transform_1d_sax

# Inferred usage based on signature
n_ts, sz, d = 1, 5, 1
dataset_sax = np.zeros((n_ts, sz, 2 * d))
breakpoints_avg = np.array([-1.0, 0.0, 1.0])
breakpoints_slope = np.array([-0.5, 0.0, 0.5])
original_size = 10

dataset_out = inv_transform_1d_sax(
    dataset_sax, 
    breakpoints_avg, 
    breakpoints_slope, 
    original_size
)
print(f"Reconstructed shape: {dataset_out.shape}")
```

### LLM Instruction Prompt
- Always ensure `dataset_sax` has a third dimension size equal to `2 * d`.
- Ensure `breakpoints` are provided as 1D NumPy arrays of type `float64`.
- The `original_size` must be a positive integer.

### Prompt Snippet
```text
from tslearn.metrics import inv_transform_1d_sax
# Ensure inputs are prepared as:
# dataset_sax: (n_ts, sz, 2 * d)
# breakpoints_avg_middle_: 1D array
# breakpoints_slope_middle_: 1D array
# original_size: int
reconstructed = inv_transform_1d_sax(dataset_sax, avg_bp, slope_bp, orig_sz)
```

### Common Failure Modes
- **Shape Mismatch:** Providing a `dataset_sax` where the last dimension is not `2 * d` will cause an index error or incorrect reconstruction.
- **Type Mismatch:** Passing lists instead of NumPy arrays for `breakpoints` may lead to internal C-extension failures.
- **Platform Dtype Sensitivity:** Note that `dataset_sax` dtype may vary between `float64` (Linux/macOS) and `float32` (Windows), which can cause issues if the input array is explicitly cast to the wrong precision for the host OS.

### Fix Code Hint
```python
import numpy as np
# Ensure inputs are cast to the expected types
dataset_sax = np.ascontiguousarray(dataset_sax, dtype=np.float64)
breakpoints_avg = np.ascontiguousarray(breakpoints_avg, dtype=np.float64)
breakpoints_slope = np.ascontiguousarray(breakpoints_slope, dtype=np.float64)
```

## API Test: `inv_transform_paa`

### Signature
```python
def inv_transform_paa(dataset_paa, original_size)
```
_Source: tslearn/tslearn/metrics/cysax.py:11_

_Source doc:_ Compute time series corresponding to given PAA representations. Parameters ---------- dataset_paa : array-like, shape=(n_ts, sz, d), dtype=float64 A dataset of PAA series. original_size : int32 Length of the original time series. Returns ------- dataset_out : array-like, shape=(n_ts, original_size, d), dtype=float64 A dataset of time series corresponding to the provided representation.

### Goal
Reconstructs an approximation of the original time series dataset from its Piecewise Aggregate Approximation (PAA) representation by repeating the mean values across the original time segments.

### Parameters
- `dataset_paa`: array-like, shape=(n_ts, sz, d), dtype=float64. The PAA-transformed dataset where `n_ts` is the number of series, `sz` is the number of segments, and `d` is the dimensionality.
- `original_size`: int32. The target length of the reconstructed time series.

### Input
The input must be a 3D array-like structure. Precondition: The `dataset_paa` must have been generated by a PAA transformation process. The `original_size` must be an integer greater than or equal to the number of segments (`sz`) in the PAA representation to ensure a valid reconstruction.

### Output
Returns `dataset_out` — A 3D NumPy array of shape `(n_ts, original_size, d)` containing the reconstructed time series, where each segment of the original series is filled with the corresponding PAA mean value.

### Valid Call Patterns
```python
import numpy as np
from tslearn.preprocessing import PiecewiseAggregateApproximation
from tslearn.utils import inv_transform_paa

# Setup: 1 time series, length 10, 1 dimension
X = np.array([[[1.0], [2.0], [3.0], [4.0], [5.0], [6.0], [7.0], [8.0], [9.0], [10.0]]])
paa = PiecewiseAggregateApproximation(n_segments=2)
X_paa = paa.fit_transform(X)

# Reconstruct
X_rec = inv_transform_paa(X_paa, original_size=10)
print(f"Reconstructed shape: {X_rec.shape}")
# Witness: Reconstructed shape: (1, 10, 1)
```

### LLM Instruction Prompt
- Use `tslearn.utils.inv_transform_paa` to perform inverse PAA transformations.
- Ensure the input `dataset_paa` is a 3D array.
- Always provide an integer for `original_size` that matches the intended output length.

### Prompt Snippet
```text
from tslearn.utils import inv_transform_paa
# X_paa: (n_ts, sz, d), original_size: int
reconstructed = inv_transform_paa(X_paa, original_size)
```

### Common Failure Modes
- **Shape Mismatch:** Providing an `original_size` that is incompatible with the segment logic or passing a 2D array instead of the required 3D `(n_ts, sz, d)` format.
- **Type Errors:** Passing non-integer values for `original_size` or non-numeric data in `dataset_paa`.
- **Missing Imports:** Attempting to call the function without importing it from `tslearn.utils`.

### Fix Code Hint
```python
# Ensure input is 3D and original_size is an integer
import numpy as np
dataset_paa = np.array(dataset_paa)
if dataset_paa.ndim != 3:
    dataset_paa = dataset_paa.reshape(dataset_paa.shape[0], dataset_paa.shape[1], -1)
reconstructed = inv_transform_paa(dataset_paa, int(original_size))
```

## API Test: `inv_transform_sax`

### Signature
```python
def inv_transform_sax(dataset_sax, breakpoints_middle_, original_size)
```
_Source: tslearn/tslearn/metrics/cysax.py:78_

_Source doc:_ Compute time series corresponding to given SAX representations. Parameters ---------- dataset_sax : array-like, shape=(n_ts, sz, d), dtype=float64 (Linux and MacOS) or float32 (Windows) A dataset of SAX series. breakpoints_middle_ : array-like, ndim=1, dtype=float64 original_size : int64 (Linux and MacOS) or int32 (Windows) Length of the original time series. Returns ------- dataset_out : array-like, shape=(n_ts, original_size, d), dtype=float64

### Goal
Reconstructs approximate time-series data from their Symbolic Aggregate approXimation (SAX) representations using provided breakpoint midpoints.

### Parameters
- `dataset_sax`: A 3D array-like structure of shape `(n_ts, sz, d)` containing the SAX-encoded time series.
- `breakpoints_middle_`: A 1D array containing the middle values of the SAX alphabet intervals used during the original transformation.
- `original_size`: An integer representing the length of the time series prior to SAX transformation.

### Input
- **Data Format:** Requires a 3D NumPy array for `dataset_sax`.
- **Preconditions:** The `breakpoints_middle_` must correspond to the alphabet intervals used during the initial SAX encoding. The `original_size` must match the intended length of the reconstructed series.

### Output
- Returns a 3D NumPy array of shape `(n_ts, original_size, d)` containing the reconstructed time series values (dtype: `float64`).

### Valid Call Patterns
```python
import numpy as np
from tslearn.piecewise import inv_transform_sax

# Inferred usage based on signature
sax_data = np.array([[[0.0], [1.0]]]) # Example SAX representation
breakpoints = np.array([-1.0, 0.0, 1.0])
original_len = 4
reconstructed = inv_transform_sax(sax_data, breakpoints, original_len)
print(f"Reconstructed shape: {reconstructed.shape}")
```

### LLM Instruction Prompt
- Use `tslearn.piecewise.inv_transform_sax` to perform inverse SAX transformation.
- Ensure `dataset_sax` is a 3D array.
- Ensure `breakpoints_middle_` is a 1D array matching the alphabet used in the encoding.
- The `original_size` must be an integer representing the target length of the output time series.

### Prompt Snippet
```text
from tslearn.piecewise import inv_transform_sax
# dataset_sax: (n_ts, sz, d), breakpoints_middle_: (n_alphabet,), original_size: int
result = inv_transform_sax(dataset_sax, breakpoints_middle_, original_size)
```

### Common Failure Modes
- **Shape Mismatch:** Providing a 2D array instead of a 3D array for `dataset_sax` will cause an error.
- **Breakpoint Mismatch:** Using `breakpoints_middle_` that do not align with the original SAX encoding will result in incorrect reconstructed values.
- **Type Errors:** Passing non-integer values for `original_size` or non-array-like structures for the dataset.

### Fix Code Hint
```python
import numpy as np
# Ensure inputs are properly formatted as numpy arrays
dataset_sax = np.asarray(dataset_sax)
breakpoints_middle_ = np.asarray(breakpoints_middle_)
# Ensure original_size is an integer
original_size = int(original_size)
```

## API Test: `inverse_transform`

### Signature
```python
def inverse_transform(self, X)
```
_Source: tslearn/tslearn/piecewise/piecewise.py:242  (+2 more definition site/overload)_

_Source doc:_ Compute time series corresponding to given PAA representations. Parameters ---------- X : array-like of shape (n_ts, sz_paa, d) A dataset of PAA series. Returns ------- numpy.ndarray of shape (n_ts, sz_original_ts, d) A dataset of time series corresponding to the provided representation.

### Goal
Reconstructs time-series data from their Piecewise Aggregate Approximation (PAA) representations by mapping the PAA segments back to the original time-series dimensionality.

### Parameters
- `self`: The fitted PAA transformer instance (e.g., `PiecewiseAggregateApproximation`).
- `X`: array-like of shape `(n_ts, sz_paa, d)`. The dataset of PAA-transformed time series to be inverted.

### Input
- **Data Format:** Must be a 3D array-like structure `(n_ts, sz_paa, d)`.
- **Preconditions:** The transformer instance must have been previously fitted on a dataset to establish the original time-series length (`sz_original_ts`) and the PAA segment configuration.

### Output
Returns `numpy.ndarray` of shape `(n_ts, sz_original_ts, d)`. This represents the reconstructed time series dataset where each PAA segment has been expanded to its original temporal resolution.

### Valid Call Patterns
```python
from tslearn.piecewise import PiecewiseAggregateApproximation
import numpy as np

# Inferred usage pattern based on signature and PAA workflow
paa = PiecewiseAggregateApproximation(n_segments=2)
X_paa = np.random.rand(10, 2, 1) # (n_ts, sz_paa, d)
X_reconstructed = paa.inverse_transform(X_paa)
print(f"Reconstructed shape: {X_reconstructed.shape}")
```

### LLM Instruction Prompt
- When calling `inverse_transform`, ensure the input `X` matches the `n_segments` and `d` (dimensionality) used during the `fit` process of the transformer.
- The output will always be a 3D NumPy array, even if the input dimensionality `d` is 1.

### Prompt Snippet
```text
# Reconstruct PAA data
X_orig = transformer.inverse_transform(X_paa)
```

### Common Failure Modes
- **Shape Mismatch:** Passing an array `X` where the number of segments (`sz_paa`) does not match the `n_segments` parameter defined during the initialization of the transformer.
- **Unfitted Estimator:** Calling `inverse_transform` before calling `fit` or `fit_transform`, which prevents the transformer from knowing the `sz_original_ts` required for reconstruction.
- **Dimensionality Mismatch:** Providing an input `X` with a different number of features (`d`) than the data used to fit the transformer.

### Fix Code Hint
```python
# Ensure the transformer is fitted before inversion
if not hasattr(transformer, "n_segments"):
    transformer.fit(X_train)
X_reconstructed = transformer.inverse_transform(X_paa)
```

## API Test: `is_array`

### Signature
```python
def is_array(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:102_

### Goal
Determines whether the provided input object is a recognized array-like structure within the `tslearn` backend ecosystem.

### Parameters
- `x`: The object to be checked. Can be a NumPy array, a PyTorch tensor, or other data structures.

### Input
- The input `x` can be any Python object.
- This is a low-level backend helper used to verify if an object is compatible with array-based operations in `tslearn`.

### Output
Returns `bool` — `True` if the input is identified as an array-like structure by the backend, `False` otherwise.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import is_array
import numpy as np

# Check a standard NumPy array
x = np.array([1, 2, 3])
assert is_array(x) is True

# Check a non-array type
assert is_array(123) is False
```

### LLM Instruction Prompt
- Use `is_array` to verify the type of input data before passing it to `tslearn` estimators or metric functions if you need to ensure compatibility with the `NumPy` or `PyTorch` backends.
- Always import `is_array` from `tslearn.backend.numpy_backend`.

### Prompt Snippet
```text
from tslearn.backend.numpy_backend import is_array
# Verify if data is an array before processing
if is_array(data):
    # proceed with tslearn operations
```

### Common Failure Modes
- **Import Errors:** Attempting to import `is_array` from `tslearn` directly instead of the specific backend module.
- **Misinterpretation:** Assuming `is_array` validates the shape or content of the time series; it only checks if the object is an array-like structure.
- **Backend Mismatch:** Relying on this check for `PyTorch` tensors when the environment does not have `torch` installed, which may lead to unexpected behavior depending on the backend implementation.

### Fix Code Hint
```python
# Ensure correct import path
from tslearn.backend.numpy_backend import is_array

# Correct usage
if not is_array(my_data):
    # Convert to expected format using tslearn.utils.to_time_series_dataset
    from tslearn.utils import to_time_series_dataset
    my_data = to_time_series_dataset(my_data)
```

## API Test: `is_float`

### Signature
```python
def is_float(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:106  (+1 more definition site/overload)_

### Goal
Determines whether a given input value or array element is of a floating-point type within the context of the active `tslearn` backend.

### Parameters
- `x`: The value, scalar, or array-like object to be checked for floating-point type compatibility.

### Input
- **Data:** Can be a scalar, a NumPy array, or a PyTorch tensor depending on the active backend.
- **Preconditions:** The function is typically accessed via a backend instance (e.g., `NumPyBackend` or `PyTorchBackend`) initialized via `tslearn.backend.instantiate_backend`.

### Output
Returns `bool` — `True` if the input `x` is a floating-point type, `False` otherwise.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend
import numpy as np

# Using the backend instance to check a value
be = instantiate_backend("numpy")
val = np.array([1.5])
assert be.is_float(val) is True
```

### LLM Instruction Prompt
- Always access `is_float` through a backend instance (e.g., `be.is_float(x)`) rather than calling it as a standalone function, as it is defined within the backend classes.
- Ensure the backend is initialized using `tslearn.backend.instantiate_backend` before calling this method.

### Prompt Snippet
```text
# Check if a value is float using the tslearn backend
is_f = be.is_float(my_value)
```

### Common Failure Modes
- **AttributeError:** Calling `is_float` directly from `tslearn` (e.g., `tslearn.is_float`) instead of through an instantiated backend object.
- **Type Mismatch:** Passing complex objects or non-numeric types that the specific backend implementation cannot evaluate for floating-point status.

### Fix Code Hint
```python
from tslearn.backend import instantiate_backend

# Correct usage:
be = instantiate_backend("numpy")
if be.is_float(my_data):
    # Proceed with float-specific operations
    pass
```

## API Test: `is_float32`

### Signature
```python
def is_float32(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:110  (+1 more definition site/overload)_

### Goal
Determines whether the provided input array or scalar is represented using 32-bit floating-point precision.

### Parameters
- `x`: The input object (typically a NumPy array or scalar) to be inspected for its data type.

### Input
- **Data:** Accepts NumPy arrays or scalar values.
- **Preconditions:** The input should be a numeric type compatible with NumPy's `dtype` inspection. While `tslearn` primarily operates on 3D arrays, this helper function is agnostic to the shape of `x`.

### Output
Returns `bool` — `True` if the input `x` has a `float32` data type, `False` otherwise.

### Valid Call Patterns
```python
import numpy as np
from tslearn.backend.numpy_backend import is_float32

# Check a float32 array
arr = np.array([1.0, 2.0], dtype=np.float32)
print(f"Is float32: {is_float32(arr)}")
# Witness: Is float32: True
```

### LLM Instruction Prompt
- Use `is_float32` to verify the precision of time-series data before passing it to backend-specific operations that may require strict type matching.
- Always import the function explicitly from `tslearn.backend.numpy_backend`.

### Prompt Snippet
```text
from tslearn.backend.numpy_backend import is_float32
# Check if array is float32
is_f32 = is_float32(my_array)
```

### Common Failure Modes
- **Non-NumPy inputs:** Passing objects that do not have a `.dtype` attribute (e.g., standard Python lists) may raise an `AttributeError`.
- **Implicit casting:** Assuming an array is `float32` when it has been implicitly cast to `float64` by NumPy operations; always verify with `is_float32` if precision is critical for memory or backend compatibility.

### Fix Code Hint
```python
import numpy as np
from tslearn.backend.numpy_backend import is_float32

# Ensure data is float32 if the check fails
if not is_float32(x):
    x = x.astype(np.float32)
```

## API Test: `is_float64`

### Signature
```python
def is_float64(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:114  (+1 more definition site/overload)_

### Goal
Checks whether the provided input object or array has a `float64` data type, typically used for internal backend validation of numerical precision.

### Parameters
- `x`: The input object, array, or tensor to be inspected for its data type.

### Input
- The input `x` should be a numerical object, typically a `numpy.ndarray` or a scalar value.
- No specific preprocessing is required, but the function is intended for use within the `tslearn` backend logic to ensure compatibility with `float64` precision requirements.

### Output
Returns `bool` — `True` if the input `x` is of type `float64`, `False` otherwise.

### Valid Call Patterns
```python
import numpy as np
from tslearn.backend.numpy_backend import is_float64

# Check a float64 array
arr = np.array([1.0, 2.0], dtype=np.float64)
assert is_float64(arr) is True

# Check a float32 array
arr_32 = np.array([1.0, 2.0], dtype=np.float32)
assert is_float64(arr_32) is False

print(f"Correctness witness: is_float64 verified for float64 and float32 types.")
```

### LLM Instruction Prompt
- Use `is_float64` when you need to verify the precision of a NumPy array before passing it to a `tslearn` estimator or metric that strictly requires `float64` input.
- Always import from `tslearn.backend.numpy_backend`.
- Do not assume the input is an array; the function handles various types, but it is most commonly used for NumPy-based time-series data.

### Prompt Snippet
```text
from tslearn.backend.numpy_backend import is_float64
# Verify precision before metric computation
if not is_float64(X):
    X = X.astype(np.float64)
```

### Common Failure Modes
- Passing non-NumPy objects that do not have a `dtype` attribute, which may raise an `AttributeError` depending on the implementation context.
- Assuming `is_float64` will automatically cast the data; it only performs a check. The user must perform the cast manually if the check returns `False`.

### Fix Code Hint
```python
# If is_float64(x) is False, cast explicitly:
x = x.astype(np.float64)
```

## API Test: `is_numpy`

### Signature
```python
def is_numpy(self)
```
_Source: tslearn/tslearn/backend/backend.py:77_

### Goal
Checks whether the current `tslearn` backend instance is configured to use `NumPy` for time-series computations.

### Parameters
- `self`: An instance of a `tslearn` backend object (e.g., `tslearn.backend.Backend`).

### Input
The caller must provide a backend instance initialized via `tslearn.backend.instantiate_backend` or the `Backend` constructor. No specific data format is required as input to the method itself, as it inspects the internal state of the backend object.

### Output
Returns `bool`: `True` if the backend is set to `NumPyBackend`, `False` otherwise (e.g., if set to `PyTorchBackend`).

### Valid Call Patterns
```python
from tslearn.backend import Backend

# Check backend state
backend = Backend("numpy")
assert backend.is_numpy is True

backend.set_backend("torch")
assert backend.is_numpy is False
```

### LLM Instruction Prompt
- Use `is_numpy` to verify the active computation engine before performing operations that might be backend-specific (e.g., checking for `PyTorch` automatic differentiation support).
- Access this as a property/method on a `Backend` instance.

### Prompt Snippet
```text
# Verify backend before metric computation
if not backend.is_numpy:
    # Handle PyTorch-specific logic
    pass
```

### Common Failure Modes
- **Attribute Access:** Attempting to call `is_numpy` on a non-backend object or a module rather than an instantiated backend instance.
- **Misconfiguration:** Assuming `is_numpy` is `True` when the environment has been explicitly switched to `torch` via `set_backend`.
- **Type Confusion:** Confusing the `is_numpy` property with the `is_pytorch` property; always verify the specific backend required for the intended operation.

### Fix Code Hint
```python
from tslearn.backend import instantiate_backend

# Ensure the backend is correctly initialized before checking
be = instantiate_backend("numpy")
if be.is_numpy:
    print("Using NumPy backend")
```

## API Test: `is_pytorch`

### Signature
```python
def is_pytorch(self)
```
_Source: tslearn/tslearn/backend/backend.py:81_

### Goal
Determines whether the current `tslearn` backend instance is configured to use the PyTorch computation engine.

### Parameters
- `self`: The `Backend` instance currently managing the computation engine.

### Input
The `Backend` object must be initialized. No specific data input is required for this property check, but the backend must have been previously set (e.g., via `instantiate_backend` or the `Backend` constructor).

### Output
Returns `bool` — `True` if the backend is set to PyTorch, `False` otherwise.

### Valid Call Patterns
```python
from tslearn.backend import Backend

# Check backend status
backend_ = Backend("torch")
is_pt = backend_.is_pytorch
print(f"Is PyTorch: {is_pt}")
```

### LLM Instruction Prompt
- Use `is_pytorch` as a property access on a `tslearn.backend.Backend` instance to verify the active computation engine before performing operations that require automatic differentiation or GPU acceleration.

### Prompt Snippet
```text
# Verify backend before PyTorch-specific operations
if backend.is_pytorch:
    # Proceed with torch-based metric computation
```

### Common Failure Modes
- **Accessing on uninitialized backend:** Attempting to check `is_pytorch` on a `Backend` object that has not been properly instantiated or configured.
- **Assuming PyTorch availability:** Calling this on a system where `pytorch` is not installed may lead to errors if the backend was forced to "torch" without the dependency present.
- **Confusing with `is_numpy`:** Expecting `is_pytorch` to return `True` when the backend is set to `numpy` (the default).

### Fix Code Hint
```python
from tslearn.backend import Backend

# Ensure the backend is explicitly set if unsure
be = Backend("torch")
if not be.is_pytorch:
    be.set_backend("torch")
```

## API Test: `iscomplex`

### Signature
```python
def iscomplex(x)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:153_

### Goal
Determines whether the provided input array or tensor contains complex-valued data.

### Parameters
- `x`: The input data structure (typically a NumPy array or PyTorch tensor) to be inspected for complex values.

### Input
The input `x` should be a numerical array-like structure. While `tslearn` primarily operates on real-valued time-series data, this utility is used by the backend to verify data types before performing alignment or distance computations.

### Output
Returns `unspecified` — typically a boolean value (or a boolean tensor/array depending on the backend) indicating if the input contains complex numbers.

### Valid Call Patterns
```python
from tslearn.backend.pytorch_backend import iscomplex
import numpy as np

# Check a real-valued array
x = np.array([1.0, 2.0, 3.0])
result = iscomplex(x)
print(f"Is complex: {result}")
```

### LLM Instruction Prompt
- Use `iscomplex` only when verifying the data type of time-series inputs before passing them to distance metrics or estimators that expect real-valued inputs.
- Do not assume the return type is a Python `bool` if the backend is set to `pytorch`; it may return a tensor.

### Prompt Snippet
```text
# Check if input data contains complex numbers before processing
if iscomplex(X):
    raise ValueError("Complex-valued time series are not supported.")
```

### Common Failure Modes
- **Backend Mismatch:** Passing an object that does not conform to the expected array-like interface of the active backend.
- **Type Errors:** Passing non-numerical types (e.g., strings or lists of objects) which may cause the underlying backend implementation to raise an `AttributeError` or `TypeError`.

### Fix Code Hint
```python
import numpy as np
# Ensure input is cast to a standard float array if complex values are detected
if iscomplex(x):
    x = np.real(x)
```

## API Test: `itakura_mask`

### Signature
```python
def itakura_mask(sz1, sz2, max_slope=2.0, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1702  (+1 more definition site/overload)_

_Source doc:_ Compute the Itakura mask. Parameters ---------- sz1 : int The size of the first time series sz2 : int The size of the second time series. max_slope : float (default = 2) The maximum slope of the parallelogram. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- mask : array-like, shape=(sz1, sz2) Itakura mask. Examples -------- >>> itakura_mask(6, 6, max_slope=3) array([[ True, False, False, False, False, False], [False,  True,  True,  True, False, False], [False,  True,  True,  True,  True, False], [False,  True,  True,  True,  True, False], [False, False,  True,  True,  True, False], [False, False, False, False, False,  True]])

### Goal
Computes a boolean mask representing the Itakura parallelogram constraint, used to restrict the search space in Dynamic Time Warping (DTW) alignments to prevent pathological warping.

### Parameters
- `sz1`: `int`. The length of the first time series.
- `sz2`: `int`. The length of the second time series.
- `max_slope`, default `2.0`: `float`. The maximum slope allowed for the warping path within the Itakura parallelogram.
- `be`, default `None`: `Backend object` or `str` or `None`. The computation engine to use (e.g., `"numpy"` or `"pytorch"`). If `None`, the backend is inferred.

### Input
The function requires two integer dimensions (`sz1`, `sz2`) representing the lengths of the time series to be aligned. No specific data arrays are required as input, only the dimensions.

### Output
Returns `mask`: `array-like` of shape `(sz1, sz2)`. A boolean matrix where `True` values indicate valid warping path indices within the Itakura constraint.

### Valid Call Patterns
```python
from tslearn.metrics import itakura_mask

# Standard usage with default slope
mask = itakura_mask(6, 6)

# Usage with custom slope and explicit backend
mask_torch = itakura_mask(10, 10, max_slope=2.5, be="pytorch")
```

### LLM Instruction Prompt
- Use `itakura_mask` when you need to generate a constraint matrix for DTW-based alignment algorithms.
- Ensure `sz1` and `sz2` are positive integers.
- If performing differentiable operations, explicitly set `be="pytorch"`.

### Prompt Snippet
```text
# Generate an Itakura mask for two time series of length 10
from tslearn.metrics import itakura_mask
mask = itakura_mask(10, 10, max_slope=2.0)
```

### Common Failure Modes
- Passing non-integer values for `sz1` or `sz2` will raise a `TypeError`.
- Providing a `max_slope` that is too restrictive (e.g., < 1.0) may result in an empty or invalid path constraint.
- Mismatching the `be` backend with the environment (e.g., requesting `"pytorch"` without the `torch` package installed) will raise an `ImportError` or `RuntimeError`.

### Fix Code Hint
```python
import numpy as np
from tslearn.metrics import itakura_mask

# Ensure dimensions are integers
sz1, sz2 = 10, 10
mask = itakura_mask(sz1, sz2, max_slope=2.0)

# Verify the mask is a boolean array
assert isinstance(mask, np.ndarray)
print(f"Mask shape: {mask.shape}, Correctness: {mask.dtype == bool}")
```

## API Test: `jacobian_product`

### Signature
```python
def jacobian_product(self, E)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:1218_

_Source doc:_ Compute the product between the Jacobian (a linear map from m x d to m x n) and a matrix E. Parameters ---------- E: array-like, shape=(m, n) Second time series. Returns ------- G: array-like, shape=(m, d) Product with Jacobian. ([m x d, m x n] * [m x n] = [m x d]).

### Goal
Computes the product of the Jacobian matrix of a soft-DTW alignment with a provided matrix `E` to facilitate gradient-based optimization in time-series learning.

### Parameters
- `self`: An instance of a soft-DTW-based estimator or metric class that maintains the state of the alignment Jacobian.
- `E`: array-like, shape=(m, n); the matrix representing the second time series or a gradient component to be projected through the Jacobian.

### Input
- **Data Format:** `E` must be an array-like structure (NumPy array or PyTorch tensor) with shape `(m, n)`.
- **Preconditions:** The `self` object must have been initialized or fitted such that the internal Jacobian mapping is defined. This is typically used within the context of `tslearn`'s differentiable metrics (e.g., `SoftDTW`).

### Output
Returns `G`: array-like, shape=(m, d); the resulting product of the Jacobian linear map and the input matrix `E`.

### Valid Call Patterns
```python
# Inferred from signature: usage within a SoftDTW-based context
# Assuming 'model' is an instance of a class implementing jacobian_product
G = model.jacobian_product(E)
```

### LLM Instruction Prompt
- When calling `jacobian_product`, ensure the input `E` matches the dimensions `(m, n)` expected by the internal state of the `self` object.
- This method is intended for internal gradient computation; ensure the backend (NumPy or PyTorch) is consistent with the `self` object's configuration.

### Prompt Snippet
```text
# Example of computing the Jacobian product
# E must be shape (m, n)
G = estimator.jacobian_product(E)
```

### Common Failure Modes
- **Shape Mismatch:** Providing an `E` matrix that does not match the `(m, n)` dimensions required by the internal Jacobian state will raise a `ValueError` or `RuntimeError`.
- **Uninitialized State:** Calling `jacobian_product` on an estimator that has not been fitted or initialized will result in an `AttributeError` or `NoneType` access error.
- **Backend Incompatibility:** Mixing NumPy arrays with a `PyTorch` backend (or vice versa) may cause type errors during the linear map operation.

### Fix Code Hint
```python
import numpy as np
# Ensure E is the correct shape (m, n)
E = np.random.rand(m, n)
# Verify the estimator is initialized before calling
if hasattr(estimator, 'jacobian_product'):
    G = estimator.jacobian_product(E)
```

## API Test: `kneighbors`

### Signature
```python
def kneighbors(self, X=None, n_neighbors=None, return_distance=True)
```
_Source: tslearn/tslearn/neighbors/neighbors.py:101  (+1 more definition site/overload)_

_Source doc:_ Finds the K-neighbors of a point. Returns indices of and distances to the neighbors of each point. Parameters ---------- X : array-like, shape (n_ts, sz, d) The query time series. If not provided, neighbors of each indexed point are returned. In this case, the query point is not considered its own neighbor. n_neighbors : int Number of neighbors to get (default is the value passed to the constructor). return_distance : boolean, optional. Defaults to True. If False, distances will not be returned Returns ------- dist : array Array representing the distance to points, only present if return_distance=True ind : array Indices of the nearest points in the population matrix.

### Goal
Computes the nearest neighbors for a set of query time series based on the distance metric configured in the estimator.

### Parameters
- `self`: The fitted `KNeighborsTimeSeries` or `KNeighborsTimeSeriesClassifier` estimator instance.
- `X`, default `None`: The query time series data, expected as a 3D array of shape `(n_ts, sz, d)`. If `None`, returns neighbors for the training set.
- `n_neighbors`, default `None`: The number of neighbors to retrieve. If `None`, defaults to the `n_neighbors` value specified during estimator initialization.
- `return_distance`, default `True`: Whether to return the distance values alongside the indices.

### Input
- **Data Format:** Must be a 3D NumPy array of shape `(n_ts, sz, d)`.
- **Preconditions:** The estimator must be fitted using the `.fit(X)` method before calling `kneighbors`.
- **Compatibility:** If input data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` prior to passing it to the estimator.

### Output
Returns a tuple `(dist, ind)` if `return_distance=True`, or just `ind` if `return_distance=False`.
- `dist`: A 2D array of shape `(n_ts, n_neighbors)` containing the calculated distances.
- `ind`: A 2D array of shape `(n_ts, n_neighbors)` containing the indices of the nearest neighbors in the training set.

### Valid Call Patterns
```python
import numpy as np
from tslearn.neighbors import KNeighborsTimeSeries

n, sz, d = 5, 10, 3
X = np.random.randn(n, sz, d)
model = KNeighborsTimeSeries(n_neighbors=2).fit(X)

# Retrieve indices and distances
dist, ind = model.kneighbors(X, n_neighbors=2, return_distance=True)
print(f"Witness: Found {ind.shape[1]} neighbors for {ind.shape[0]} samples.")
```

### LLM Instruction Prompt
- Always ensure the estimator is fitted before calling `kneighbors`.
- If `X` is provided, ensure it matches the dimensionality `(n_ts, sz, d)` of the training data.
- When `return_distance` is `False`, expect only the indices array as the return value.

### Prompt Snippet
```text
# Example usage of kneighbors
dist, ind = estimator.kneighbors(X_query, n_neighbors=k, return_distance=True)
```

### Common Failure Modes
- **NotFittedError:** Calling `kneighbors` before `fit`.
- **Shape Mismatch:** Providing `X` with a different number of features (`d`) or time steps (`sz`) than the training data, depending on the metric.
- **Invalid Metric:** Passing an unsupported metric string to the estimator constructor, which will raise a `ValueError` during `fit`.

### Fix Code Hint
```python
# Ensure data is 3D and estimator is fitted
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X_raw)
model.fit(X_formatted)
dist, ind = model.kneighbors(X_formatted)
```

## API Test: `lb_envelope`

### Signature
```python
def lb_envelope(ts, radius=1, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2170_

_Source doc:_ Compute time series envelope as required by LB_Keogh. LB_Keogh was originally presented in [1]_. Parameters ---------- ts : array-like, shape=(sz, d) or (sz,) Time series for which the envelope should be computed. If shape is (sz,), the time series is assumed to be univariate. radius : int (default: 1) Radius to be used for the envelope generation (the envelope at time index i will be generated based on all observations from the time series at indices comprised between i-radius and i+radius). be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- envelope_down : array-like, shape=(sz, d) Lower-side of the envelope. envelope_up : array-like, shape=(sz, d) Upper-side of the envelope. Examples -------- >>> ts1 = [1, 2, 3, 2, 1] >>> env_low, env_up = lb_envelope(ts1, radius=1) >>> env_low array([[1.], [1.], [2.], [1.], [1.]]) >>> env_up array([[2.], [3.], [3.], [3.], [2.]]) See also -------- lb_keogh : Compute LB_Keogh similarity References ---------- .. [1] Keogh, E. Exact indexing of dynamic time warping. In International Conference on Very Large Data Bases, 2002. pp 406-417.

### Goal
Computes the upper and lower bounding envelopes of a time series, which are essential for calculating the LB_Keogh lower bound for Dynamic Time Warping (DTW) distance.

### Parameters
- `ts`: The input time series data. Can be a 1D array of shape `(sz,)` or a 2D array of shape `(sz, d)`.
- `radius`, default `1`: The window size used to define the envelope; at each index `i`, the envelope is determined by values in the range `[i-radius, i+radius]`.
- `be`, default `None`: The computation backend. Accepts `None` (auto-detect), `"numpy"`, `"pytorch"`, or specific backend instances.

### Input
The input `ts` must be a numerical array-like structure. If using the `PyTorch` backend, `ts` should be a `torch.Tensor`. For `NumPy` backend, it should be a `numpy.ndarray`.

### Output
Returns a tuple `(envelope_down, envelope_up)`, where both are arrays of shape `(sz, d)` representing the lower and upper bounds of the time series respectively.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import lb_envelope

ts1 = [1, 2, 3, 2, 1]
env_low, env_up = lb_envelope(ts1, radius=1)
print(f"Lower: {env_low.shape}, Upper: {env_up.shape}")
```

### LLM Instruction Prompt
- Always ensure the input `ts` is a valid array-like structure.
- If performing differentiable operations, explicitly set `be="pytorch"` and ensure `ts` is a `torch.Tensor` with `requires_grad=True`.
- The `radius` must be a non-negative integer.

### Prompt Snippet
```text
from tslearn.metrics import lb_envelope
# Compute envelopes for LB_Keogh
env_low, env_up = lb_envelope(ts_data, radius=r)
```

### Common Failure Modes
- Passing a 3D array `(n_ts, sz, d)` directly: `lb_envelope` expects a single time series `(sz, d)` or `(sz,)`. Use a loop or list comprehension for datasets.
- Mismatched backend: Providing a `PyTorch` tensor while using the default `NumPy` backend may cause type errors or performance degradation.
- Invalid `radius`: Providing a negative integer for `radius` will result in an error.

### Fix Code Hint
```python
# If you have a dataset X of shape (n_ts, sz, d), process one by one:
envelopes = [lb_envelope(ts, radius=1) for ts in X]
```

## API Test: `lb_keogh`

### Signature
```python
def lb_keogh(ts_query, ts_candidate=None, radius=1, envelope_candidate=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2008_

_Source doc:_ Compute LB_Keogh. LB_Keogh was originally presented in [1]_. Parameters ---------- ts_query : array-like, shape=(sz1, 1) or (sz1,) Univariate query time series to compare to the envelope of the candidate. ts_candidate : None or array-like, shape=(sz2, 1) or (sz2,) (default: None) Univariate candidate time series. None means the envelope is provided via `envelope_candidate` parameter and hence does not need to be computed again. radius : int (default: 1) Radius to be used for the envelope generation (the envelope at time index i will be generated based on all observations from the candidate time series at indices comprised between i-radius and i+radius). Not used if `ts_candidate` is None. envelope_candidate: pair of array-like (envelope_down, envelope_up) or None (default: None) Pre-computed envelope of the candidate time series. If set to None, it is computed based on `ts_candidate`. Notes ----- This method requires a `ts_query` and `ts_candidate` (or `envelope_candidate`, depending on the call) to be of equal size. Returns ------- float Distance between the query time series and the envelope of the candidate time series. Examples -------- >>> ts1 = [1, 2, 3, 2, 1] >>> ts2 = [0, 0, 0, 0, 0] >>> env_low, env_up = lb_envelope(ts1, radius=1) >>> float(lb_keogh(ts_query=ts2, ...                envelope_candidate=(env_low, env_up)))  # doctest: +ELLIPSIS 2.8284... >>> float(lb_keogh(ts_query=ts2, ...                ts_candidate=ts1, ...                radius=1))  # doctest: +ELLIPSIS 2.8284... See also -------- lb_envelope : Compute LB_Keogh-related envelope References ---------- .. [1] Keogh, E. Exact indexing of dynamic time warping. In International Conference on Very Large Data Bases, 2002. pp 406-417.

### Goal
Computes the LB_Keogh lower bound distance between a query time series and the envelope of a candidate time series, providing an efficient approximation for DTW-based similarity search.

### Parameters
- `ts_query`: Univariate query time series, shape `(sz1, 1)` or `(sz1,)`.
- `ts_candidate`, default `None`: Univariate candidate time series, shape `(sz2, 1)` or `(sz2,)`. Required if `envelope_candidate` is `None`.
- `radius`, default `1`: Integer radius for envelope generation; defines the window size `[i-radius, i+radius]` for the candidate series.
- `envelope_candidate`, default `None`: A pair of array-like objects `(envelope_down, envelope_up)` representing the pre-computed envelope of the candidate.

### Input
- **Data Format:** Accepts array-like structures (NumPy arrays or lists).
- **Preconditions:** `ts_query` and `ts_candidate` (or the provided `envelope_candidate`) must be of equal size.
- **Constraint:** This function is designed for univariate time series.

### Output
Returns `float`: The lower bound distance between the query time series and the candidate's envelope.

### Valid Call Patterns
```python
from tslearn.metrics import lb_keogh, lb_envelope
import numpy as np

ts1 = np.array([1, 2, 3, 2, 1])
ts2 = np.array([0, 0, 0, 0, 0])

# Pattern 1: Compute using candidate series
dist1 = lb_keogh(ts_query=ts2, ts_candidate=ts1, radius=1)

# Pattern 2: Compute using pre-computed envelope
env_low, env_up = lb_envelope(ts1, radius=1)
dist2 = lb_keogh(ts_query=ts2, envelope_candidate=(env_low, env_up))

print(f"Witness: {dist1 == dist2}")
```

### LLM Instruction Prompt
- When calling `lb_keogh`, ensure the query and candidate series are univariate.
- If providing `envelope_candidate`, ensure it is a tuple or list containing exactly two arrays (lower and upper bounds).
- If `ts_candidate` is provided, `radius` must be an integer.

### Prompt Snippet
```text
from tslearn.metrics import lb_keogh
# Ensure ts_query and ts_candidate are equal length
dist = lb_keogh(ts_query=query_arr, ts_candidate=cand_arr, radius=r)
```

### Common Failure Modes
- **Shape Mismatch:** Passing `ts_query` and `ts_candidate` of different lengths will raise a `ValueError` or index error.
- **Missing Arguments:** Failing to provide either `ts_candidate` or `envelope_candidate` will result in an invalid computation.
- **Multivariate Input:** Passing 3D arrays (standard `tslearn` dataset format) directly without selecting a univariate series will cause dimension errors.

### Fix Code Hint
```python
# Ensure univariate shape (sz,) or (sz, 1)
ts_query = ts_query.flatten()
ts_candidate = ts_candidate.flatten()
# Verify lengths
assert len(ts_query) == len(ts_candidate)
```

## API Test: `lcss`

### Signature
```python
def lcss(s1, s2, eps=1.0, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2383_

_Source doc:_ Compute the Longest Common Subsequence (LCSS) similarity measure between (possibly multidimensional) time series and return the similarity. LCSS is computed by matching indexes that are met up until the eps threshold, so it leaves some points unmatched and focuses on the similar parts of two sequences. The matching can occur even if the time indexes are different. One can set additional constraints to the set of acceptable paths: the Sakoe-Chiba band which is parametrized by a radius or the Itakura parallelogram which is parametrized by a maximum slope. Both these constraints consists in forcing paths to lie close to the diagonal. To retrieve a meaningful similarity value from the length of the longest common subsequence, the percentage of that value regarding the length of the shortest time series is returned. According to this definition, the values returned by LCSS range from 0 to 1, the highest value taken when two time series fully match, and vice-versa. It is not required that both time series share the same size, but they must be the same dimension. LCSS was originally presented in [1]_ and is discussed in more details in our :ref:`dedicated user-guide page <lcss>`. Notes ----- Contrary to Dynamic Time Warping and variants, an LCSS path does not need to be contiguous. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. eps : float (default: 1.) Maximum matching distance threshold. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for LCSS. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used.

### Goal
Computes the Longest Common Subsequence (LCSS) similarity between two time series, returning a normalized score between 0 and 1 that represents the percentage of matching points relative to the shortest sequence.

### Parameters
- `s1`: Array-like, shape `(sz1, d)` or `(sz1,)`. The first time series.
- `s2`: Array-like, shape `(sz2, d)` or `(sz2,)`. The second time series.
- `eps`, default `1.0`: Float. The maximum distance threshold for two points to be considered a match.
- `global_constraint`, default `None`: String, either `"itakura"` or `"sakoe_chiba"`. Restricts the alignment path.
- `sakoe_chiba_radius`, default `None`: Integer. The radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float. The maximum slope for the Itakura parallelogram constraint.
- `be`, default `None`: Backend object or string (e.g., `"numpy"`). Specifies the computation engine.

### Input
Input time series must be of the same dimension `d`. They do not need to be the same length. If inputs are 1D, they are treated as univariate.

### Output
Returns a `float` representing the similarity score (0 to 1).

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Basic usage
sim = tslearn.metrics.lcss([1, 2, 3], [1.0, 2.0, 2.0, 3.0])

# With constraints and backend
sim = tslearn.metrics.lcss([1, 2, 3], [-2.0, 5.0, 7.0], eps=3, be="numpy")
```

### LLM Instruction Prompt
- Always import `tslearn.metrics` to access `lcss`.
- Ensure `s1` and `s2` have the same number of dimensions.
- Use `eps` to control the sensitivity of the matching threshold.
- If using `global_constraint`, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is provided or understood to default.

### Prompt Snippet
```text
from tslearn.metrics import lcss
score = lcss(s1, s2, eps=0.5)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing time series with different dimensions `d` will raise an error.
- **Constraint Conflict:** Providing both `sakoe_chiba_radius` and `itakura_max_slope` without a valid `global_constraint` may trigger a `RuntimeWarning` and ignore the constraints.
- **Backend Errors:** Passing an unsupported backend string or object.

### Fix Code Hint
```python
# Ensure dimensions match
s1 = np.array([[1], [2]])
s2 = np.array([[1], [2], [3]])
# Correct call
sim = tslearn.metrics.lcss(s1, s2, eps=0.1)
```

## API Test: `lcss_accumulated_matrix`

### Signature
```python
def lcss_accumulated_matrix(s1, s2, eps, mask, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2270_

_Source doc:_ Compute the longest common subsequence similarity score between two time series. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) First time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Second time series. If shape is (sz2,), the time series is assumed to be univariate. eps : float Matching threshold. mask : array-like, shape=(sz1, sz2) Mask. Unconsidered cells must have False values. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- acc_cost_mat : array-like, shape=(sz1 + 1, sz2 + 1) Accumulated cost matrix.

### Goal
Computes the accumulated cost matrix for the Longest Common Subsequence (LCSS) metric, facilitating the alignment of two time series based on a matching threshold.

### Parameters
- `s1`: First time series; array-like of shape `(sz1, d)` or `(sz1,)`.
- `s2`: Second time series; array-like of shape `(sz2, d)` or `(sz2,)`.
- `eps`: Float; the matching threshold used to determine if two points are considered "matching".
- `mask`: Array-like of shape `(sz1, sz2)`; a boolean mask where `False` values indicate cells that should not be considered in the alignment.
- `be`, default `None`: Backend object or string (`"numpy"` or `"pytorch"`) to specify the computation engine.

### Input
- **Data Format:** Input series must be array-like. Univariate series are accepted as `(sz,)`, while multivariate series must be `(sz, d)`.
- **Preconditions:** The `mask` must have dimensions matching the lengths of `s1` and `s2`. If using the `PyTorch` backend, inputs must be compatible tensors.

### Output
Returns `acc_cost_mat` — An array-like object of shape `(sz1 + 1, sz2 + 1)` representing the accumulated cost matrix of the LCSS alignment.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import lcss_accumulated_matrix

s1 = np.array([1.0, 2.0, 3.0])
s2 = np.array([1.1, 2.1, 4.0])
mask = np.ones((3, 3), dtype=bool)
eps = 0.5

acc_mat = lcss_accumulated_matrix(s1, s2, eps=eps, mask=mask)
print(f"Accumulated matrix shape: {acc_mat.shape}")
# Witness: Accumulated matrix shape: (4, 4)
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` are provided as compatible array-like structures.
- Ensure the `mask` dimensions strictly correspond to `(len(s1), len(s2))`.
- If performing differentiable operations, explicitly set `be="pytorch"` and ensure inputs are `torch.Tensor` objects with `requires_grad=True`.

### Prompt Snippet
```text
from tslearn.metrics import lcss_accumulated_matrix
# Ensure mask is (len(s1), len(s2))
acc_mat = lcss_accumulated_matrix(s1, s2, eps=threshold, mask=mask_array)
```

### Common Failure Modes
- **Shape Mismatch:** Providing a `mask` that does not match the lengths of `s1` and `s2` will raise a `ValueError`.
- **Backend Incompatibility:** Passing a `PyTorch` tensor to the default `NumPy` backend (or vice versa) without specifying `be` may lead to type errors or performance degradation.
- **Threshold Sensitivity:** An `eps` value that is too small may result in an empty alignment (all zeros in the accumulated matrix).

### Fix Code Hint
```python
# Ensure inputs are properly shaped for the backend
import numpy as np
s1 = np.asarray(s1)
s2 = np.asarray(s2)
mask = np.ones((len(s1), len(s2)), dtype=bool)
# Use lcss_accumulated_matrix(s1, s2, eps, mask)
```

## API Test: `lcss_accumulated_matrix_from_dist_matrix`

### Signature
```python
def lcss_accumulated_matrix_from_dist_matrix(dist_matrix, eps, mask, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2865_

_Source doc:_ Compute the accumulated cost matrix score between two time series using a precomputed distance matrix. Parameters ---------- dist_matrix : array-like, shape=(sz1, sz2) Array containing the pairwise distances. eps : float (default: 1.) Maximum matching distance threshold. mask : array-like, shape=(sz1, sz2) Mask. Unconsidered cells must have False values. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is `NumPyBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- acc_cost_mat : array-like, shape=(sz1 + 1, sz2 + 1) Accumulated cost matrix.

### Goal
Computes the accumulated cost matrix for the Longest Common Subsequence (LCSS) metric based on a precomputed pairwise distance matrix between two time series.

### Parameters
- `dist_matrix`: array-like, shape=(sz1, sz2). The precomputed pairwise distance matrix between two time series.
- `eps`: float. The maximum matching distance threshold used to determine if two points are considered a match.
- `mask`: array-like, shape=(sz1, sz2). A boolean mask where `False` values indicate cells that should be ignored during the accumulation process.
- `be`, default `None`: Backend object, string ("numpy" or "pytorch"), or `None`. Determines the computational engine used for the matrix operations.

### Input
The caller must provide a precomputed distance matrix of shape `(sz1, sz2)` and a corresponding boolean mask of the same shape. The inputs should be compatible with the selected backend (NumPy arrays for `numpy` backend, or PyTorch tensors for `pytorch` backend).

### Output
Returns `acc_cost_mat`: an array-like object of shape `(sz1 + 1, sz2 + 1)` representing the accumulated cost matrix used for LCSS calculation.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import lcss_accumulated_matrix_from_dist_matrix

dist_matrix = np.array([[0.1, 0.5], [0.8, 0.2]])
mask = np.array([[True, True], [True, True]])
acc_mat = lcss_accumulated_matrix_from_dist_matrix(dist_matrix, eps=0.3, mask=mask)
assert acc_mat.shape == (3, 3)
print(f"Correctness witness: Accumulated matrix shape {acc_mat.shape}")
```

### LLM Instruction Prompt
- Use `lcss_accumulated_matrix_from_dist_matrix` when you need to perform custom LCSS alignment steps on precomputed distance data.
- Ensure `dist_matrix` and `mask` have matching dimensions `(sz1, sz2)`.
- Explicitly set `be` if you are working within a PyTorch-based pipeline to ensure gradient tracking compatibility.

### Prompt Snippet
```text
from tslearn.metrics import lcss_accumulated_matrix_from_dist_matrix
# Ensure dist_matrix and mask are provided as arrays of shape (sz1, sz2)
acc_mat = lcss_accumulated_matrix_from_dist_matrix(dist_matrix, eps=eps_val, mask=mask_val)
```

### Common Failure Modes
- **Shape Mismatch:** Providing `dist_matrix` and `mask` with different shapes will raise a `ValueError`.
- **Backend Incompatibility:** Passing a PyTorch tensor to the function while the default `NumPyBackend` is active (or vice versa) may cause type errors; explicitly set `be="pytorch"` if using tensors.
- **Invalid Masking:** Providing a mask that does not correctly account for the desired alignment constraints can lead to unexpected LCSS scores.

### Fix Code Hint
```python
# Ensure inputs are cast to the correct backend type before calling
import numpy as np
dist_matrix = np.array(dist_matrix)
mask = np.array(mask, dtype=bool)
acc_mat = lcss_accumulated_matrix_from_dist_matrix(dist_matrix, eps=0.5, mask=mask)
```

## API Test: `lcss_path`

### Signature
```python
def lcss_path(s1, s2, eps=1, global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2697_

_Source doc:_ Compute the Longest Common Subsequence (LCSS) similarity measure between (possibly multidimensional) time series and return both the path and the similarity. LCSS is computed by matching indexes that are met up until the eps threshold, so it leaves some points unmatched and focuses on the similar parts of two sequences. The matching can occur even if the time indexes are different. One can set additional constraints to the set of acceptable paths: the Sakoe-Chiba band which is parametrized by a radius or the Itakura parallelogram which is parametrized by a maximum slope. Both these constraints consists in forcing paths to lie close to the diagonal. To retrieve a meaningful similarity value from the length of the longest common subsequence, the percentage of that value regarding the length of the shortest time series is returned. According to this definition, the values returned by LCSS range from 0 to 1, the highest value taken when two time series fully match, and vice-versa. It is not required that both time series share the same size, but they must be the same dimension. LCSS was originally presented in [1]_ and is discussed in more details in our :ref:`dedicated user-guide page <lcss>`. Notes ----- Contrary to Dynamic Time Warping and variants, an LCSS path does not need to be contiguous. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. eps : float (default: 1.) Maximum matching distance threshold. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for LCSS. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. be : Backend object or string or None

### Goal
Computes the Longest Common Subsequence (LCSS) similarity and the optimal alignment path between two time series, allowing for non-contiguous matches within a specified distance threshold.

### Parameters
- `s1`: Array-like, shape=(sz1, d) or (sz1,). The first time series.
- `s2`: Array-like, shape=(sz2, d) or (sz2,). The second time series.
- `eps`, default `1`: Float. The maximum distance threshold for considering two points as a match.
- `global_constraint`, default `None`: String, either "itakura" or "sakoe_chiba". Restricts the search space for the alignment path.
- `sakoe_chiba_radius`, default `None`: Integer. The radius for the Sakoe-Chiba band constraint.
- `itakura_max_slope`, default `None`: Float. The maximum slope for the Itakura parallelogram constraint.
- `be`, default `None`: Backend object, string, or None. Specifies the computation backend (e.g., "numpy" or "pytorch").

### Input
- Time series must be provided as array-like structures.
- Both `s1` and `s2` must have the same dimension `d`.
- If using the `PyTorch` backend, inputs should be `torch.Tensor` objects.

### Output
Returns a tuple `(path, similarity)`:
- `path`: A list of tuples representing the indices of matched points.
- `similarity`: A float between 0 and 1 representing the percentage of the longest common subsequence relative to the length of the shortest input series.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

s1 = [1.0, 2.0, 3.0]
s2 = [1.0, 2.0, 2.0, 3.0]
path, sim = tslearn.metrics.lcss_path(s1, s2, eps=1.0)
print(f"Similarity: {sim}, Path: {path}")
```

### LLM Instruction Prompt
- Always specify `eps` if the default threshold of 1.0 is not appropriate for the scale of your data.
- If using `global_constraint`, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is set or understood to use the default.
- Ensure `s1` and `s2` are of the same dimension `d`.

### Prompt Snippet
```text
path, sim = tslearn.metrics.lcss_path(s1, s2, eps=..., global_constraint=..., be=...)
```

### Common Failure Modes
- Dimension Mismatch: Providing `s1` and `s2` with different dimensions `d` will raise an error.
- Incorrect Constraint Usage: Setting both `sakoe_chiba_radius` and `itakura_max_slope` without a valid `global_constraint` may trigger a `RuntimeWarning`.
- Backend Incompatibility: Passing a `torch.Tensor` while expecting `NumPy` behavior without specifying `be="pytorch"` may lead to unexpected type errors.

### Fix Code Hint
```python
# Ensure inputs are 2D if multidimensional, or 1D if univariate
# Use to_time_series_dataset if inputs are lists of varying lengths
from tslearn.utils import to_time_series_dataset
s1_formatted = to_time_series_dataset([s1])[0]
s2_formatted = to_time_series_dataset([s2])[0]
path, sim = tslearn.metrics.lcss_path(s1_formatted, s2_formatted, eps=0.5)
```

## API Test: `lcss_path_from_metric`

### Signature
```python
def lcss_path_from_metric(s1, s2=None, eps=1, metric='euclidean', global_constraint=None, sakoe_chiba_radius=None, itakura_max_slope=None, be=None, **kwds)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2908_

_Source doc:_ Compute the Longest Common Subsequence (LCSS) similarity measure between (possibly multidimensional) time series using a distance metric defined by the user and return both the path and the similarity. Having the length of the longest commom subsequence between two time series, the similarity is computed as the percentage of that value regarding the length of the shortest time series. It is not required that both time series share the same size, but they must be the same dimension. LCSS was originally presented in [1]_. Valid values for metric are the same as for scikit-learn `pairwise_distances`_ function i.e. a string (e.g. "euclidean", "sqeuclidean", "hamming") or a function that is used to compute the pairwise distances. See `scikit`_ and `scipy`_ documentations for more information about the available metrics. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) if metric!="precomputed", (sz1, sz2) otherwise A time series or an array of pairwise distances between samples. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,), optional (default: None) A second time series, only allowed if metric != "precomputed". If shape is (sz2,), the time series is assumed to be univariate. eps : float (default: 1.) Maximum matching distance threshold. metric : string or callable (default: "euclidean") Function used to compute the pairwise distances between each points of `s1` and `s2`. If metric is "precomputed", `s1` is assumed to be a distance matrix. If metric is an other string, it must be one of the options compatible with sklearn.metrics.pairwise_distances. Alternatively, if metric is a callable function, it is called on pairs of rows of `s1` and `s2`. The callable should take two 1 dimensional arrays as input and return a value indicating the distance between them. global_constraint : {"itakura", "sakoe_chiba"} or None (default: None) Global constraint to restrict admissible paths for LCSS. sakoe_chiba_radius : int or None (default: None) Radius to be used for Sakoe-Chiba band global constraint. The Sakoe-Chiba radius corresponds to the parameter :math:`\delta` mentioned in [1]_, it controls how far in time we can go in order to match a given point from one time series to a point in another time series. If None and `global_constraint` is set to "sakoe_chiba", a radius of 1 is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used. itakura_max_slope : float or None (default: None) Maximum slope for the Itakura parallelogram constraint. If None and `global_constraint` is set to "itakura", a maximum slope of 2. is used. If both `sakoe_chiba_radius` and `itakura_max_slope` are set, `global_constraint` is used to infer which constraint to use among the two. In this case, if `global_constraint` corresponds to no global constraint, a `RuntimeWarning` is raised and no global constraint is used.

### Goal
Computes the Longest Common Subsequence (LCSS) alignment path and similarity score between two time series using a custom or standard distance metric.

### Parameters
- `s1`: The first time series (shape `(sz1, d)`) or a precomputed distance matrix (shape `(sz1, sz2)` if `metric="precomputed"`).
- `s2`, default `None`: The second time series (shape `(sz2, d)`). Required unless `metric="precomputed"`.
- `eps`, default `1`: The matching threshold; points are considered matching if their distance is less than or equal to `eps`.
- `metric`, default `'euclidean'`: The distance metric to use. Can be a string (e.g., "sqeuclidean") or a callable function accepting two 1D arrays.
- `global_constraint`, default `None`: Restricts the alignment path search space; options are `"itakura"` or `"sakoe_chiba"`.
- `sakoe_chiba_radius`, default `None`: The window size for the Sakoe-Chiba constraint.
- `itakura_max_slope`, default `None`: The maximum slope for the Itakura parallelogram constraint.
- `be`, default `None`: Backend object (e.g., `NumPyBackend` or `PyTorchBackend`) for computation.
- `**kwds`: Additional keyword arguments passed to the distance metric function.

### Input
- Data must be provided as NumPy arrays or PyTorch tensors.
- Time series must have the same dimension `d`.
- If `metric` is not `"precomputed"`, `s1` and `s2` must be 1D or 2D arrays.

### Output
Returns a tuple `(path, similarity)`, where `path` is a list of `(index1, index2)` tuples representing the alignment, and `similarity` is a float representing the percentage of matching points relative to the shortest series.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

s1 = np.random.rand(10, 1)
s2 = np.random.rand(15, 1)

# Standard usage with string metric
path, sim = tslearn.metrics.lcss_path_from_metric(s1, s2, metric="sqeuclidean")

# Usage with custom callable metric
def custom_dist(x, y):
    return np.sum((x - y)**2)

path, sim = tslearn.metrics.lcss_path_from_metric(s1, s2, metric=custom_dist)
```

### LLM Instruction Prompt
- Always specify the `metric` if not using the default "euclidean".
- Ensure `s1` and `s2` have the same number of dimensions `d`.
- If using `global_constraint`, ensure the corresponding parameter (`sakoe_chiba_radius` or `itakura_max_slope`) is set appropriately.

### Prompt Snippet
```text
from tslearn.metrics import lcss_path_from_metric
# path, sim = lcss_path_from_metric(s1, s2, metric='...')
```

### Common Failure Modes
- Dimension mismatch: Passing time series with different feature dimensions `d` will raise a `ValueError`.
- Missing `s2`: Failing to provide `s2` when `metric` is not `"precomputed"`.
- Invalid `metric`: Providing a string not supported by `sklearn.metrics.pairwise_distances`.

### Fix Code Hint
```python
# Ensure inputs are 2D (n_samples, n_features)
s1 = s1.reshape(-1, 1) if s1.ndim == 1 else s1
s2 = s2.reshape(-1, 1) if s2.ndim == 1 else s2
path, sim = tslearn.metrics.lcss_path_from_metric(s1, s2, metric="euclidean")
```

## API Test: `list_cached_datasets`

### Signature
```python
def list_cached_datasets(self)
```
_Source: tslearn/tslearn/datasets/ucr_uea.py:236_

_Source doc:_ List datasets from the UCR/UEA archive that are available in cache. Examples -------- >>> beetlefly = UCR_UEA_datasets().load_dataset("BeetleFly") >>> l = UCR_UEA_datasets().list_cached_datasets() >>> "BeetleFly" in l True

### Goal
Retrieves a list of UCR/UEA time-series datasets currently stored in the local cache directory associated with the `UCR_UEA_datasets` instance.

### Parameters
- `self`: An instance of the `tslearn.datasets.UCR_UEA_datasets` class.

### Input
- **Preconditions:** The `UCR_UEA_datasets` instance must be initialized. The cache directory is determined by the `root_dir` parameter provided during initialization or the default `tslearn` data home directory.

### Output
Returns `list` — A list of strings representing the names of the datasets found in the cache directory.

### Valid Call Patterns
```python
from tslearn.datasets import UCR_UEA_datasets

# Initialize the loader
loader = UCR_UEA_datasets()

# List cached datasets
cached = loader.list_cached_datasets()
print(f"Cached datasets: {cached}")
```

### LLM Instruction Prompt
- Always instantiate `UCR_UEA_datasets` before calling `list_cached_datasets`.
- Use the instance method `list_cached_datasets()` on the object returned by `UCR_UEA_datasets()`.
- Do not assume any specific datasets are cached unless `load_dataset` has been called previously.

### Prompt Snippet
```text
# Check if a dataset is cached
loader = UCR_UEA_datasets()
is_cached = "Trace" in loader.list_cached_datasets()
```

### Common Failure Modes
- **Uninitialized Instance:** Calling the method without an instance of `UCR_UEA_datasets` (e.g., attempting to call it as a static method).
- **Environment Pathing:** If `XDG_DATA_HOME` or `root_dir` is modified, the method will only report datasets cached in that specific path, which may differ from the default system cache.
- **Empty Cache:** The method returns an empty list if no datasets have been downloaded to the specified `root_dir` yet.

### Fix Code Hint
```python
from tslearn.datasets import UCR_UEA_datasets
from pathlib import Path

# Ensure the loader points to the correct directory
loader = UCR_UEA_datasets(root_dir=Path("./my_data_cache"))
# If empty, load a dataset first to populate the cache
loader.load_dataset("Trace")
cached = loader.list_cached_datasets()
assert "Trace" in cached
print(f"Correctness witness: {cached}")
```

## API Test: `list_datasets`

### Signature
```python
def list_datasets(self)
```
_Source: tslearn/tslearn/datasets/cached.py:26  (+1 more definition site/overload)_

_Source doc:_ List cached datasets. Examples -------- >>> from tslearn.datasets import CachedDatasets >>> cached = CachedDatasets().list_datasets() >>> "Trace" in cached True Returns ------- list of str: A list of names of all cached (univariate and multivariate) dataset names.

### Goal
Retrieves the collection of available time-series datasets that are pre-cached within the `tslearn` environment for immediate use in benchmarking or testing.

### Parameters
- `self`: An instance of the `CachedDatasets` class.

### Input
- No external data or file inputs are required. The method operates on the internal state of the `CachedDatasets` object.

### Output
Returns `list of str` — A list containing the names of all univariate and multivariate datasets currently available in the local cache.

### Valid Call Patterns
```python
from tslearn.datasets import CachedDatasets
data_loader = CachedDatasets()
cached_names = data_loader.list_datasets()
```

### LLM Instruction Prompt
- Always instantiate `tslearn.datasets.CachedDatasets` before calling `list_datasets`.
- Do not pass any arguments to the `list_datasets` method.
- Use the returned list to verify if a specific dataset name exists before attempting to load it via `load_dataset`.

### Prompt Snippet
```text
from tslearn.datasets import CachedDatasets
datasets = CachedDatasets().list_datasets()
```

### Common Failure Modes
- Calling `list_datasets` as a static method (e.g., `CachedDatasets.list_datasets()`) without instantiating the class first, which will raise a `TypeError` due to the missing `self` argument.
- Assuming the list is exhaustive for all UCR datasets; this method only returns datasets that are specifically cached within the `tslearn` package.

### Fix Code Hint
```python
# Incorrect:
# names = CachedDatasets.list_datasets()

# Correct:
from tslearn.datasets import CachedDatasets
names = CachedDatasets().list_datasets()
assert "Trace" in names, "Trace dataset should be available in cache"
```

## API Test: `list_multivariate_datasets`

### Signature
```python
def list_multivariate_datasets(self)
```
_Source: tslearn/tslearn/datasets/ucr_uea.py:194_

_Source doc:_ List multivariate datasets in the UCR/UEA archive. Examples -------- >>> l = UCR_UEA_datasets().list_multivariate_datasets() >>> "PenDigits" in l True Returns ------- list of str: A list of the names of all multivariate dataset namas.

### Goal
Retrieves a list of all multivariate time-series dataset names available within the UCR/UEA archive integration.

### Parameters
- `self`: An instance of the `UCR_UEA_datasets` class.

### Input
- **Preconditions:** The caller must instantiate the `UCR_UEA_datasets` class before invoking this method. No external data files are required as input, but the method relies on the internal archive structure provided by `tslearn`.

### Output
Returns `list of str`: A list containing the names of all multivariate datasets currently supported or indexed in the UCR/UEA archive.

### Valid Call Patterns
```python
from tslearn.datasets import UCR_UEA_datasets

# Instantiate the dataset handler and list multivariate datasets
datasets = UCR_UEA_datasets()
multivariate_list = datasets.list_multivariate_datasets()
print(f"Found {len(multivariate_list)} multivariate datasets.")
```

### LLM Instruction Prompt
- Always instantiate `tslearn.datasets.UCR_UEA_datasets` before calling `list_multivariate_datasets`.
- Do not pass any arguments to `list_multivariate_datasets`.
- Use the returned list to verify if a specific dataset name exists in the archive before attempting to load it.

### Prompt Snippet
```text
from tslearn.datasets import UCR_UEA_datasets
loader = UCR_UEA_datasets()
available_multivariate = loader.list_multivariate_datasets()
```

### Common Failure Modes
- **AttributeError:** Calling `list_multivariate_datasets` directly on the module `tslearn.datasets` instead of an instance of `UCR_UEA_datasets`.
- **Network/Environment:** While the method accesses internal metadata, ensure the `tslearn` installation is complete, as missing archive metadata files may cause unexpected behavior.

### Fix Code Hint
```python
# Incorrect:
# from tslearn.datasets import list_multivariate_datasets
# list_multivariate_datasets() 

# Correct:
from tslearn.datasets import UCR_UEA_datasets
l = UCR_UEA_datasets().list_multivariate_datasets()
```

## API Test: `list_univariate_datasets`

### Signature
```python
def list_univariate_datasets(self)
```
_Source: tslearn/tslearn/datasets/ucr_uea.py:174_

_Source doc:_ List univariate datasets in the UCR/UEA archive. Examples -------- >>> l = UCR_UEA_datasets().list_univariate_datasets() >>> len(l) 128 Returns ------- list of str: A list of the names of all univariate datasets.

### Goal
Retrieves the collection of names for all univariate time-series datasets available within the UCR/UEA archive integration.

### Parameters
- `self`: An instance of the `UCR_UEA_datasets` class.

### Input
- **Preconditions:** The `tslearn` package must be installed. This method does not require external network access if the local cache is populated, but it relies on the `UCR_UEA_datasets` object being instantiated first.

### Output
Returns `list of str`: A list containing the string identifiers (names) of all univariate datasets currently supported or available in the UCR/UEA archive.

### Valid Call Patterns
```python
from tslearn.datasets import UCR_UEA_datasets

# Instantiate the dataset handler and list univariate names
datasets = UCR_UEA_datasets()
univariate_list = datasets.list_univariate_datasets()
```

### LLM Instruction Prompt
- Always instantiate `UCR_UEA_datasets()` before calling `list_univariate_datasets()`.
- Do not pass arguments to `list_univariate_datasets()`.
- Use the returned list to filter or select datasets for subsequent loading via `load_dataset`.

### Prompt Snippet
```text
# Example of listing univariate datasets
from tslearn.datasets import UCR_UEA_datasets
names = UCR_UEA_datasets().list_univariate_datasets()
```

### Common Failure Modes
- **AttributeError:** Calling `list_univariate_datasets` as a static method (e.g., `UCR_UEA_datasets.list_univariate_datasets()`) instead of on an instance.
- **ImportError:** Failing to import `UCR_UEA_datasets` from `tslearn.datasets`.

### Fix Code Hint
```python
from tslearn.datasets import UCR_UEA_datasets

# Correct: Instantiate first
loader = UCR_UEA_datasets()
names = loader.list_univariate_datasets()

# Verification
assert isinstance(names, list)
print(f"Configured correctness witness: Found {len(names)} univariate datasets.")
```

## API Test: `load_dataset`

### Signature
```python
def load_dataset(self, dataset_name)
```
_Source: tslearn/tslearn/datasets/cached.py:45  (+1 more definition site/overload)_

_Source doc:_ Load a cached dataset from its name. Parameters ---------- dataset_name : str Name of the dataset. Should be in the list returned by :meth:`~list_datasets`. Returns ------- numpy.ndarray of shape (n_ts_train, sz, d) or None Training time series. None if unsuccessful. numpy.ndarray of integers with shape (n_ts_train, ) or None Training labels. None if unsuccessful. numpy.ndarray of shape (n_ts_test, sz, d) or None Test time series. None if unsuccessful. numpy.ndarray of integers with shape (n_ts_test, ) or None Test labels. None if unsuccessful. Examples -------- >>> data_loader = CachedDatasets() >>> X_train, y_train, X_test, y_test = data_loader.load_dataset( ...                                        "Trace") >>> print(X_train.shape) (100, 275, 1) >>> print(y_train.shape) (100,) Raises ------ IOError If the dataset does not exist or cannot be read.

### Goal
Loads a standardized time-series dataset from the local cache, returning training and test sets formatted as 3D NumPy arrays compatible with `tslearn` estimators.

### Parameters
- `self`: An instance of a dataset loader class, such as `CachedDatasets` or `UCR_UEA_datasets`.
- `dataset_name`: A string representing the name of the dataset to load; must be present in the list returned by the loader's `list_datasets()` method.

### Input
The caller must provide a valid `dataset_name` string. Precondition: The dataset must either be already cached in the loader's `root_dir` or available for download/caching by the loader instance.

### Output
Returns a tuple of four `numpy.ndarray` objects: `(X_train, y_train, X_test, y_test)`. `X` arrays are 3D with shape `(n_ts, sz, d)`, and `y` arrays are 1D integer arrays of shape `(n_ts,)`. Returns `None` for any component if the load is unsuccessful.

### Valid Call Patterns
```python
from tslearn.datasets import UCR_UEA_datasets
data_loader = UCR_UEA_datasets()
X_train, y_train, X_test, y_test = data_loader.load_dataset("Trace")
print(f"Witness: X_train shape is {X_train.shape}")
assert X_train.shape == (100, 275, 1)
```

### LLM Instruction Prompt
- Always instantiate a dataset loader (e.g., `UCR_UEA_datasets`) before calling `load_dataset`.
- Ensure the `dataset_name` matches the exact string identifier used by the `tslearn` dataset registry.
- Handle the returned tuple by unpacking it into four variables: `X_train, y_train, X_test, y_test`.

### Prompt Snippet
```text
# Load dataset using tslearn
from tslearn.datasets import UCR_UEA_datasets
loader = UCR_UEA_datasets()
X_train, y_train, X_test, y_test = loader.load_dataset("DatasetName")
```

### Common Failure Modes
- `IOError`: Raised if the `dataset_name` is invalid or the file cannot be read from the cache.
- `AttributeError`: Occurs if calling `load_dataset` as a static method instead of on an instance of a loader class.
- `None` returns: If the dataset is not found or the download fails, the function may return `None` for the arrays, leading to downstream `TypeError` when passing to estimators.

### Fix Code Hint
```python
from tslearn.datasets import UCR_UEA_datasets
try:
    loader = UCR_UEA_datasets()
    data = loader.load_dataset("Trace")
    if data is not None:
        X_train, y_train, X_test, y_test = data
except IOError as e:
    print(f"Failed to load dataset: {e}")
```

## API Test: `load_dict`

### Signature
```python
def load_dict(filename, group)
```
_Source: tslearn/tslearn/hdftools/hdftools.py:118_

_Source doc:_ Recursively load a dict from an hdf5 group in a file. Parameters ---------- filename : str full path to the hdf5 file group : str Name of the group that contains the dict to load Returns ------- d : dict dict loaded from the specified hdf5 group.

### Goal
`load_dict` provides a utility to deserialize a dictionary structure from an HDF5 file, facilitating the persistent storage and retrieval of time-series datasets or model parameters.

### Parameters
- `filename`: `str`. The full filesystem path to the HDF5 file containing the serialized data.
- `group`: `str`. The specific name of the HDF5 group within the file from which the dictionary should be loaded.

### Input
The function expects an existing HDF5 file created via `tslearn.hdftools.save_dict`. The file must be accessible in the current environment, and the specified `group` must exist within the HDF5 hierarchy.

### Output
Returns `dict`. A dictionary containing the data structures (typically NumPy arrays) that were previously serialized into the specified HDF5 group.

### Valid Call Patterns
```python
from tslearn import hdftools
import os

# Assuming 'data.hdf5' exists and contains a group named 'my_group'
d = hdftools.load_dict(filename='data.hdf5', group='my_group')
print(f"Loaded keys: {list(d.keys())}")
```

### LLM Instruction Prompt
- Use `tslearn.hdftools.load_dict` only when retrieving data previously saved with `tslearn.hdftools.save_dict`.
- Ensure the `filename` is a valid string path and the `group` matches the identifier used during the save operation.
- The function returns a standard Python dictionary; verify the contents (e.g., check keys or array shapes) before passing them to `tslearn` estimators.

### Prompt Snippet
```text
from tslearn import hdftools
data = hdftools.load_dict(filename="path/to/file.hdf5", group="group_name")
```

### Common Failure Modes
- **FileNotFoundError**: Occurs if the `filename` path is incorrect or the file does not exist.
- **KeyError / GroupNotFoundError**: Occurs if the specified `group` does not exist within the HDF5 file.
- **Type Mismatch**: If the HDF5 file was not created using `tslearn.hdftools.save_dict`, the internal structure may not be compatible with the recursive loading logic, leading to unexpected data types or errors.

### Fix Code Hint
```python
import os
from tslearn import hdftools

# Ensure the file exists before attempting to load
if os.path.exists(fname):
    try:
        data = hdftools.load_dict(fname, 'my_group')
    except Exception as e:
        print(f"Failed to load dictionary: {e}")
```

## API Test: `load_time_series_txt`

### Signature
```python
def load_time_series_txt(fname)
```
_Source: tslearn/tslearn/utils/utils.py:393_

_Source doc:_ Loads a time series dataset from disk. Parameters ---------- fname : string Path to the file from which time series should be read. Returns ------- numpy.ndarray or array of numpy.ndarray The dataset of time series. Examples -------- >>> dataset = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3]]) >>> save_time_series_txt("tmp-tslearn-test.txt", dataset) >>> reloaded_dataset = load_time_series_txt("tmp-tslearn-test.txt") See Also -------- save_time_series_txt : Save time series to disk

### Goal
Loads a time-series dataset from a text file on disk into a format compatible with `tslearn` estimators.

### Parameters
- `fname`: A string representing the file path to the text file containing the time-series data.

### Input
The function expects a text file previously generated by `save_time_series_txt`. The data must be structured such that it can be parsed into a `numpy.ndarray` or an array of `numpy.ndarray` objects representing time series.

### Output
Returns a `numpy.ndarray` (for fixed-length series) or an array of `numpy.ndarray` (for variable-length series) containing the time-series dataset.

### Valid Call Patterns
```python
from tslearn.utils import load_time_series_txt, to_time_series_dataset, save_time_series_txt
import os

# Setup: Create and save a dataset
dataset = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3]])
save_time_series_txt("tmp-tslearn-test.txt", dataset)

# Execution: Load the dataset
reloaded_dataset = load_time_series_txt("tmp-tslearn-test.txt")

# Cleanup
if os.path.exists("tmp-tslearn-test.txt"):
    os.remove("tmp-tslearn-test.txt")
```

### LLM Instruction Prompt
- Use `load_time_series_txt` only when reading files specifically created by `save_time_series_txt`.
- Ensure the file path provided in `fname` is valid and accessible.
- Always verify the output shape if the dataset is expected to be uniform (3D array) versus variable-length (list of arrays).

### Prompt Snippet
```text
from tslearn.utils import load_time_series_txt
data = load_time_series_txt(fname="path/to/dataset.txt")
```

### Common Failure Modes
- **FileNotFoundError**: Occurs if the `fname` path is incorrect or the file does not exist.
- **Parsing Errors**: Occurs if the text file format does not match the expected structure produced by `save_time_series_txt`.
- **Memory Errors**: Attempting to load extremely large text files into memory as a single `numpy.ndarray`.

### Fix Code Hint
```python
# Ensure the file exists before loading
import os
if os.path.exists(fname):
    data = load_time_series_txt(fname)
else:
    raise FileNotFoundError(f"The file {fname} was not found.")
```

## API Test: `locate`

### Signature
```python
def locate(self, X)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:625_

_Source doc:_ Compute shapelet match location for a set of time series. Parameters ---------- X : array-like of shape=(n_ts, sz, d) Time series dataset. Returns ------- array of shape=(n_ts, n_shapelets) Location of the shapelet matches for the provided time series. Examples -------- >>> from tslearn.generators import random_walk_blobs >>> X = numpy.zeros((3, 10, 1)) >>> X[0, 4:7, 0] = numpy.array([1, 2, 3]) >>> y = [1, 0, 0] >>> # Data is all zeros except a motif 1-2-3 in the first time series >>> clf = LearningShapelets(n_shapelets_per_size={3: 1}, max_iter=1, ...                     verbose=0) >>> _ = clf.fit(X, y) >>> weights_shapelet = [ ...     numpy.array([[[1], [2], [3]]]) ... ] >>> clf.set_weights(weights_shapelet, layer_name="shapelets_0") >>> clf.locate(X) array([[4], [0], [0]])

### Goal
Computes the starting indices of the best-matching shapelets within a provided time-series dataset for a trained `LearningShapelets` estimator.

### Parameters
- `self`: An instance of `tslearn.shapelets.LearningShapelets` that has already been fitted to a dataset.
- `X`: array-like of shape=(n_ts, sz, d). The time-series dataset to search for shapelet occurrences.

### Input
- **Format:** Must be a 3D array of shape `(n_ts, sz, d)`. If the input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling `locate`.
- **Preconditions:** The estimator `self` must be fitted (i.e., `fit(X, y)` has been called) so that internal shapelet weights are initialized.

### Output
Returns `numpy.ndarray` of shape `(n_ts, n_shapelets)`. Each entry `(i, j)` represents the starting index (time step) where the `j`-th shapelet achieves its minimum distance within the `i`-th time series.

### Valid Call Patterns
```python
import numpy as np
from tslearn.shapelets import LearningShapelets
from tslearn.utils import to_time_series_dataset

# Example usage based on project test suite
time_series = to_time_series_dataset([[1, 2, 3, 4, 5], [3, 2, 1]])
y = [0, 1]
clf = LearningShapelets(n_shapelets_per_size={2: 1}, max_iter=10, verbose=0)
clf.fit(time_series, y)

# Locate matches
locations = clf.locate(time_series)
print(f"Shapelet locations: {locations}")
```

### LLM Instruction Prompt
- Always ensure the `LearningShapelets` estimator is fitted before calling `locate`.
- Ensure input `X` is a 3D array; use `tslearn.utils.to_time_series_dataset` if the input is a list of sequences or a 2D array.
- The output is a 2D array of indices; interpret these as the starting time-step of the best match for each shapelet.

### Prompt Snippet
```text
# Ensure X is 3D before calling locate
X_formatted = to_time_series_dataset(X)
locations = clf.locate(X_formatted)
```

### Common Failure Modes
- **NotFittedError:** Calling `locate` before `fit` has been executed on the estimator.
- **Shape Mismatch:** Passing a 2D array directly without converting it to the required 3D `(n_ts, sz, d)` format.
- **Incompatible Dimensions:** Providing time series shorter than the length of the discovered shapelets, which may lead to indexing errors or empty search ranges.

### Fix Code Hint
```python
# If you encounter a shape error, ensure your data is 3D:
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(X)
# Then verify the estimator is trained:
if not hasattr(clf, "shapelets_"):
    clf.fit(X, y)
locations = clf.locate(X)
```

## API Test: `mae`

### Signature
```python
def mae(y_true, y_pred, ts_weights=None, timestamps_weights=None, multioutput='uniform_average')
```
_Source: tslearn/tslearn/metrics/performance.py:12_

_Source doc:_ Mean absolute error (MAE) MAE is a measure of the prediction accuracy for forecasting and regression tasks that computes the average of the deviations between ground truth and predicted values. Parameters ---------- y_true: array like, shape (n_ts, sz, d) Target dataset of ground_truth values y_pred: array like, shape (n_ts, sz, d) Estimated dataset of predicted values ts_weights: array like, shape (n_ts,) or None (default: None) Weights to apply to the time-series in the datasets if non-uniform. Use none for uniform weights. timestamps_weights: array like, shape (sz,) or None (default: None) Weights to apply to the timestamps of each-time series if non-uniform. Use none for uniform weights. multioutput: {'uniform_average', 'raw_values'} or array-like, shape (d,) (default: 'uniform_average') for multivariate timeseries, defines the aggregation of per feature results, if any. 'raw_values': no aggregation, result is per feature 'uniform_average': errors of all features are averaged with uniform weights array-like: errors of all features are averaged using the given weights Returns ------- float or array-like, shape (d,) If multioutput is ‘raw_values’, then mean absolute error is returned for each feature. Otherwise, the average of each feature is returned.

### Goal
Computes the Mean Absolute Error (MAE) between ground truth and predicted time-series datasets, supporting weighted aggregation across time steps, series, and features.

### Parameters
- `y_true`: Ground truth time-series dataset, expected as an array-like of shape `(n_ts, sz, d)`.
- `y_pred`: Predicted time-series dataset, expected as an array-like of shape `(n_ts, sz, d)`.
- `ts_weights`, default `None`: Optional weights of shape `(n_ts,)` to apply to individual time series.
- `timestamps_weights`, default `None`: Optional weights of shape `(sz,)` to apply to time steps within each series.
- `multioutput`, default `'uniform_average'`: Strategy for aggregating multivariate features; options are `'uniform_average'`, `'raw_values'`, or an array-like of shape `(d,)`.

### Input
Input data must be 3D arrays of shape `(n_ts, sz, d)`. If data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this function.

### Output
Returns a `float` (if `multioutput` is `'uniform_average'`) or an `array-like` of shape `(d,)` (if `multioutput` is `'raw_values'` or custom weights) representing the calculated error.

### Valid Call Patterns
```python
from tslearn.metrics import performance
import numpy as np

y_true = [[[1, 2], [2, 3], [3, 4]]]
y_pred = [[[0, 1], [1, 2], [2, 3]]]

# Basic usage
mae_val = performance.mae(y_true, y_pred)

# Weighted usage
mae_weighted = performance.mae(y_true, y_pred, ts_weights=[1], timestamps_weights=[0, 1, 0])
```

### LLM Instruction Prompt
- Always import `mae` from `tslearn.metrics.performance`.
- Ensure `y_true` and `y_pred` are 3D arrays; use `tslearn.utils.to_time_series_dataset` if necessary.
- When `multioutput='raw_values'`, expect an array of shape `(d,)` as the return value.

### Prompt Snippet
```text
from tslearn.metrics import performance
# Ensure inputs are 3D: (n_ts, sz, d)
error = performance.mae(y_true, y_pred, multioutput='uniform_average')
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without converting to 3D `(n_ts, sz, d)` will raise a `ValueError`.
- **Weight Dimension Mismatch:** Providing `ts_weights` or `timestamps_weights` with lengths inconsistent with `n_ts` or `sz` respectively will cause indexing errors.
- **Backend Incompatibility:** Attempting to pass `torch` tensors without ensuring the environment supports them or using the appropriate backend-aware wrappers.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Convert 2D list to 3D dataset before calling mae
y_true_3d = to_time_series_dataset(y_true)
y_pred_3d = to_time_series_dataset(y_pred)
result = performance.mae(y_true_3d, y_pred_3d)
```

## API Test: `mase`

### Signature
```python
def mase(y_true, y_pred, train_data, seasonal_period=1, ts_weights=None, timestamps_weights=None, multioutput='uniform_average')
```
_Source: tslearn/tslearn/metrics/performance.py:153_

_Source doc:_ Mean absolute scaled error (MASE) MASE is a measure of the prediction accuracy for forecasting and regression tasks that computes the scaled average of the deviations between ground truth and predicted values. The scaling factor is computed as the MAE of the naive m-seasonal forecast on the in-sample dataset. Parameters ---------- y_true: array like, shape (n_ts, sz, d) Target dataset of ground_truth values y_pred: array like, shape (n_ts, sz, d) Estimated dataset of predicted values train_data: array like the in-sample dataset, used to compute the scaling factor seasonal_period: int (default: 1) seasonal period used to compute the scaling factor ts_weights: array like, shape (n_ts,) or None (default: None) Weights to apply to the time-series in the datasets if non-uniform. Use none for uniform weights. timestamps_weights: array like, shape (sz,) or None (default: None) Weights to apply to the timestamps of each-time series if non-uniform. Use none for uniform weights. multioutput: {'uniform_average', 'raw_values'} or array-like, shape (d,) (default: 'uniform_average') for multivariate timeseries, defines the aggregation of per feature results, if any. 'raw_values': no aggregation, result is per feature 'uniform_average': errors of all features are averaged with uniform weights array-like: errors of all features are averaged using the given weights Returns ------- float or array-like, shape (d,) If multioutput is ‘raw_values’, then mean squared error is returned for each feature. Otherwise, the average of each feature is returned.

### Goal
Computes the Mean Absolute Scaled Error (MASE) to evaluate forecasting accuracy by scaling the mean absolute error of predictions against the mean absolute error of a naive seasonal in-sample forecast.

### Parameters
- `y_true`: Ground truth time-series dataset, shape `(n_ts, sz, d)`.
- `y_pred`: Predicted time-series dataset, shape `(n_ts, sz, d)`.
- `train_data`: In-sample dataset used to calculate the naive seasonal scaling factor.
- `seasonal_period`, default `1`: The seasonal lag used for the naive forecast scaling factor.
- `ts_weights`, default `None`: Optional weights for each time series in the dataset, shape `(n_ts,)`.
- `timestamps_weights`, default `None`: Optional weights for each timestamp, shape `(sz,)`.
- `multioutput`, default `'uniform_average'`: Aggregation strategy for multivariate data; can be `'uniform_average'`, `'raw_values'`, or an array of weights for each feature.

### Input
Expects 3D NumPy arrays of shape `(n_ts, sz, d)`. If data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this function.

### Output
Returns a `float` (if `multioutput='uniform_average'`) or an `array-like` of shape `(d,)` (if `multioutput='raw_values'` or custom weights) representing the scaled error.

### Valid Call Patterns
```python
from tslearn.metrics import performance
import numpy as np

# Basic usage
mase_val = performance.mase(y_pred, y_true, y_train)

# Multivariate with raw output
mase_raw = performance.mase(y_pred, y_true, y_train, multioutput="raw_values")

# With seasonal period and custom feature weights
mase_weighted = performance.mase(y_pred, y_true, y_train, seasonal_period=5, multioutput=[0.2, 0.8])
```

### LLM Instruction Prompt
- Always ensure `y_true`, `y_pred`, and `train_data` are 3D arrays.
- Use `multioutput='raw_values'` when you need to inspect error per feature in multivariate time series.
- Be aware that `seasonal_period` must be a positive integer.

### Prompt Snippet
```text
# Example of calling mase
from tslearn.metrics import performance
score = performance.mase(y_pred, y_true, train_data, seasonal_period=12)
```

### Common Failure Modes
- **Divide by Zero:** If `train_data` is constant (e.g., all zeros), the naive forecast error is zero, resulting in `np.inf` and a `RuntimeWarning`.
- **Shape Mismatch:** Passing 2D arrays without converting them to 3D `(n_ts, sz, d)` will cause indexing errors.
- **Incompatible `seasonal_period`:** Setting a `seasonal_period` larger than the length of the `train_data` will lead to invalid scaling calculations.

### Fix Code Hint
```python
# Ensure data is 3D before passing to mase
from tslearn.utils import to_time_series_dataset
y_true_3d = to_time_series_dataset(y_true)
y_pred_3d = to_time_series_dataset(y_pred)
train_data_3d = to_time_series_dataset(train_data)
score = performance.mase(y_true_3d, y_pred_3d, train_data_3d)
```

## API Test: `mse`

### Signature
```python
def mse(y_true, y_pred, ts_weights=None, timestamps_weights=None, multioutput='uniform_average')
```
_Source: tslearn/tslearn/metrics/performance.py:83_

_Source doc:_ Mean squared error (MSE) MSE is a measure of the prediction accuracy for forecasting and regression tasks that computes the average of the squared deviations between ground truth and predicted values. Parameters ---------- y_true: array like, shape (n_ts, sz, d) Target dataset of ground_truth values y_pred: array like, shape (n_ts, sz, d) Estimated dataset of predicted values ts_weights: array like, shape (n_ts,) or None (default: None) Weights to apply to the time-series in the datasets if non-uniform. Use none for uniform weights. timestamps_weights: array like, shape (sz,) or None (default: None) Weights to apply to the timestamps of each-time series if non-uniform. Use none for uniform weights. multioutput: {'uniform_average', 'raw_values'} or array-like, shape (d,) (default: 'uniform_average') for multivariate timeseries, defines the aggregation of per feature results, if any. 'raw_values': no aggregation, result is per feature 'uniform_average': errors of all features are averaged with uniform weights array-like: errors of all features are averaged using the given weights Returns ------- float or array-like, shape (d,) If multioutput is ‘raw_values’, then mean squared error is returned for each feature. Otherwise, the average of each feature is returned.

### Goal
Computes the mean squared error between ground truth and predicted time-series datasets to evaluate the accuracy of forecasting or regression models.

### Parameters
- `y_true`: Ground truth time-series dataset, expected as an array-like structure of shape `(n_ts, sz, d)`.
- `y_pred`: Predicted time-series dataset, expected as an array-like structure of shape `(n_ts, sz, d)`.
- `ts_weights`, default `None`: Optional weights of shape `(n_ts,)` to apply to individual time series in the dataset.
- `timestamps_weights`, default `None`: Optional weights of shape `(sz,)` to apply to specific time steps across all series.
- `multioutput`, default `'uniform_average'`: Defines aggregation for multivariate data (`d > 1`). Options are `'uniform_average'`, `'raw_values'`, or a custom weight array of shape `(d,)`.

### Input
Input data must be 3D arrays of shape `(n_ts, sz, d)`. If data is provided as 1D or 2D lists/arrays, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this function.

### Output
Returns a `float` (if `multioutput` is `'uniform_average'`) or an `array-like` of shape `(d,)` (if `multioutput` is `'raw_values'` or a custom weight array), representing the calculated mean squared error.

### Valid Call Patterns
```python
from tslearn.metrics import performance
import numpy as np

y_true = np.array([[[0.5, 2], [2, 3], [1, 4]]])
y_pred = np.array([[[1, 2], [2, 3], [3, 4]]])

# Standard usage
mse_val = performance.mse(y_true, y_pred)

# Multivariate raw values
mse_raw = performance.mse(y_true, y_pred, multioutput="raw_values")
```

### LLM Instruction Prompt
- Always ensure `y_true` and `y_pred` have identical shapes `(n_ts, sz, d)`.
- If working with multivariate data, explicitly define `multioutput` if you require per-feature results rather than the default uniform average.
- Use `tslearn.utils.to_time_series_dataset` to ensure input compatibility before passing arrays to `mse`.

### Prompt Snippet
```text
from tslearn.metrics import performance
# Ensure inputs are 3D arrays (n_ts, sz, d)
mse_score = performance.mse(y_true, y_pred, multioutput='uniform_average')
```

### Common Failure Modes
- **Shape Mismatch:** Passing arrays where `n_ts`, `sz`, or `d` do not match between `y_true` and `y_pred` will raise a `ValueError`.
- **Dimensionality Error:** Passing 2D arrays directly without converting them to 3D `(n_ts, sz, d)` format via `to_time_series_dataset`.
- **Weight Mismatch:** Providing `ts_weights` or `timestamps_weights` with lengths that do not match the corresponding dimensions of the input arrays.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Convert 2D input to 3D if necessary
y_true_3d = to_time_series_dataset(y_true)
y_pred_3d = to_time_series_dataset(y_pred)
mse_val = performance.mse(y_true_3d, y_pred_3d)
```

## API Test: `n_iter_`

### Signature
```python
def n_iter_(self)
```
_Source: tslearn/tslearn/svm/svm.py:279  (+1 more definition site/overload)_

### Goal
Retrieves the actual number of iterations performed by the estimator during the fitting process.

### Parameters
- `self`: An instance of a `tslearn` estimator (e.g., `TimeSeriesSVC` or other iterative models) that has already been fitted using the `fit()` method.

### Input
- **Preconditions:** The estimator must have been successfully trained via a call to `fit(X, y)`. Calling this attribute before `fit()` will result in an `AttributeError` as the attribute is only populated during the training phase.

### Output
Returns `unspecified` — Typically an integer representing the number of iterations run by the underlying solver to reach convergence or the maximum iteration limit.

### Valid Call Patterns
```python
# Inferred from signature: Estimator must be fitted first
from tslearn.svm import TimeSeriesSVC
clf = TimeSeriesSVC().fit(X_train, y_train)
iterations = clf.n_iter_
print(f"Model converged after {iterations} iterations.")
```

### LLM Instruction Prompt
- Always ensure the estimator instance has been fitted before accessing `n_iter_`.
- Do not assume `n_iter_` is available on all `tslearn` estimators; check the specific estimator documentation to confirm it tracks iteration counts.
- Use this attribute primarily for monitoring convergence or debugging training duration.

### Prompt Snippet
```text
# Ensure the model is fitted before accessing n_iter_
model.fit(X, y)
n_iters = model.n_iter_
```

### Common Failure Modes
- **AttributeError:** Occurs if `n_iter_` is accessed on an estimator that has not yet been fitted.
- **Missing Attribute:** Some `tslearn` estimators may not implement `n_iter_` if the underlying algorithm (e.g., certain distance-based classifiers) does not perform iterative optimization.
- **Type Mismatch:** Attempting to call `n_iter_` as a function (e.g., `model.n_iter_()`) instead of accessing it as an attribute.

### Fix Code Hint
```python
# Correct access pattern
if hasattr(model, "n_iter_"):
    print(model.n_iter_)
else:
    print("This estimator does not track iteration counts.")
```

## API Test: `ndim`

### Signature
```python
def ndim(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:118  (+1 more definition site/overload)_

### Goal
Determines the number of dimensions of a time-series array or tensor, providing a backend-agnostic way to inspect the rank of the input data.

### Parameters
- `x`: The input time-series data, typically a NumPy array or a PyTorch tensor representing a dataset or a single time series.

### Input
- **Data Format:** Accepts 3D NumPy arrays `(n_ts, max_sz, d)` or equivalent PyTorch tensors.
- **Preconditions:** The input must be a valid array-like structure supported by the active backend (NumPy or PyTorch). If the input is a list or non-array structure, it should be converted to a standard format (e.g., via `to_time_series_dataset`) before inspection.

### Output
Returns `unspecified` — The function returns an integer representing the number of dimensions (rank) of the input array `x`.

### Valid Call Patterns
```python
from tslearn.backend import ndim
import numpy as np

# Check dimensions of a standard 3D time-series dataset
X = np.zeros((10, 20, 1))
print(ndim(X))  # Output: 3
```

### LLM Instruction Prompt
- Use `ndim(x)` to verify the rank of time-series data before passing it to estimators or metrics.
- Ensure the input `x` is already formatted as a `tslearn`-compatible array (3D) to avoid ambiguity in dimension interpretation.

### Prompt Snippet
```text
# Verify input dimensions before processing
if ndim(X) != 3:
    raise ValueError("Input must be a 3D array (n_ts, sz, d)")
```

### Common Failure Modes
- **Scalar/List Inputs:** Passing a raw Python list or a scalar instead of an array-like object may cause the function to fail or return unexpected results depending on the backend implementation.
- **Backend Mismatch:** If the input is a PyTorch tensor but the environment is not configured to handle it, or if the backend is not correctly instantiated, the function may raise a `TypeError`.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
from tslearn.backend import ndim

# Ensure data is 3D before checking dimensions
X_formatted = to_time_series_dataset(X)
rank = ndim(X_formatted)
print(f"Data rank: {rank}")
```

## API Test: `njit_accumulated_matrix`

### Signature
```python
def njit_accumulated_matrix(s1, s2, mask)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:74_

_Source doc:_ Compute the accumulated cost matrix score between two time series. Parameters ---------- s1 : array-like, shape=(sz1, d) First time series. s2 : array-like, shape=(sz2, d) Second time series. mask : array-like, shape=(sz1, sz2) Mask. Unconsidered cells must have False values. Returns ------- mat : array-like, shape=(sz1, sz2) Accumulated cost matrix.

### Goal
Computes the accumulated cost matrix for time-series alignment, typically used as a low-level backend helper for constrained Dynamic Time Warping (DTW) calculations.

### Parameters
- `s1`: array-like, shape=(sz1, d). The first time series sequence.
- `s2`: array-like, shape=(sz2, d). The second time series sequence.
- `mask`: array-like, shape=(sz1, sz2). A boolean mask where `False` values indicate cells that should be excluded from the accumulated cost calculation.

### Input
- **Data Format:** Expects 2D array-like structures (sequences of length `sz` with `d` dimensions).
- **Preconditions:** The `mask` must be a boolean array of shape `(sz1, sz2)`. This function is a low-level Numba-jitted helper; inputs should be pre-formatted as NumPy arrays for optimal performance.

### Output
Returns `mat` — A 2D array of shape `(sz1, sz2)` representing the accumulated cost matrix, where each cell `(i, j)` contains the cumulative cost of the optimal alignment path up to that point, subject to the provided `mask`.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.dtw_variants import njit_accumulated_matrix

s1 = np.array([[1.0], [2.0], [3.0]])
s2 = np.array([[1.0], [2.0], [3.0]])
mask = np.ones((3, 3), dtype=bool)

mat = njit_accumulated_matrix(s1, s2, mask)
print(f"Correctness witness: Accumulated matrix shape {mat.shape}")
```

### LLM Instruction Prompt
- Use `njit_accumulated_matrix` only when performing low-level alignment computations where a custom constraint mask is required.
- Ensure `s1` and `s2` are 2D arrays (shape `(sz, d)`).
- Ensure `mask` is a boolean array of shape `(len(s1), len(s2))`.
- This function is a backend helper; do not use it for high-level estimator workflows (use `tslearn.metrics` functions instead).

### Prompt Snippet
```text
from tslearn.metrics.dtw_variants import njit_accumulated_matrix
# Ensure s1, s2 are (sz, d) and mask is (sz1, sz2) boolean
mat = njit_accumulated_matrix(s1, s2, mask)
```

### Common Failure Modes
- **Shape Mismatch:** Passing arrays that do not match the `(sz, d)` requirement or a `mask` that does not match the `(sz1, sz2)` dimensions will trigger a Numba-related error.
- **Type Errors:** Passing non-numeric data for `s1` or `s2` will cause the JIT-compiled function to fail.
- **Masking Logic:** Providing a `mask` where all values are `False` may result in an accumulated matrix filled with infinity or invalid values depending on the implementation logic.

### Fix Code Hint
```python
# Ensure inputs are numpy arrays and mask is boolean
s1 = np.asarray(s1)
s2 = np.asarray(s2)
mask = np.asarray(mask, dtype=bool)
mat = njit_accumulated_matrix(s1, s2, mask)
```

## API Test: `njit_accumulated_matrix_from_dist_matrix`

### Signature
```python
def njit_accumulated_matrix_from_dist_matrix(dist_matrix, mask)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:426_

_Source doc:_ Compute the accumulated cost matrix score between two time series using a precomputed distance matrix. Parameters ---------- dist_matrix : array-like, shape=(sz1, sz2) Array containing the pairwise distances. mask : array-like, shape=(sz1, sz2) Mask. Unconsidered cells must have False values. Returns ------- mat : array-like, shape=(sz1, sz2) Accumulated cost matrix.

### Goal
Computes the accumulated cost matrix for time-series alignment by aggregating pairwise distances while respecting a provided boolean mask to constrain the alignment path.

### Parameters
- `dist_matrix`: array-like, shape=(sz1, sz2). A precomputed matrix containing the pairwise distances between two time series.
- `mask`: array-like, shape=(sz1, sz2). A boolean mask where `False` values indicate cells that should be excluded from the accumulated cost calculation.

### Input
- **Data Format:** Requires two 2D array-like structures (typically NumPy arrays) of identical shape `(sz1, sz2)`.
- **Preconditions:** The `dist_matrix` must contain non-negative distance values. The `mask` must be boolean-compatible, where `True` represents valid cells for the alignment path and `False` represents restricted cells.

### Output
Returns `mat`: array-like, shape=(sz1, sz2). The resulting accumulated cost matrix where each cell `(i, j)` represents the minimum cost to reach that point from the origin, considering the constraints imposed by the mask.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.dtw_variants import njit_accumulated_matrix_from_dist_matrix

dist_matrix = np.array([[1.0, 2.0], [3.0, 4.0]])
mask = np.array([[True, True], [True, True]])
acc_matrix = njit_accumulated_matrix_from_dist_matrix(dist_matrix, mask)
print(f"Correctness witness: {acc_matrix.shape == (2, 2)}")
```

### LLM Instruction Prompt
- Always ensure `dist_matrix` and `mask` have identical dimensions `(sz1, sz2)`.
- Use this function only when you have already computed the pairwise distance matrix and need the specific accumulated cost structure for alignment analysis.
- Ensure the `mask` correctly identifies the valid search space for the alignment path to avoid invalid cost accumulation.

### Prompt Snippet
```text
from tslearn.metrics.dtw_variants import njit_accumulated_matrix_from_dist_matrix
# Ensure dist_matrix and mask are 2D arrays of shape (sz1, sz2)
acc_mat = njit_accumulated_matrix_from_dist_matrix(dist_matrix, mask)
```

### Common Failure Modes
- **Shape Mismatch:** Passing `dist_matrix` and `mask` with different shapes will raise a `ValueError` or index error during the Numba-jitted execution.
- **Invalid Masking:** Providing a mask where all paths to the end are blocked (all `False` in critical regions) may result in unexpected accumulated values (e.g., infinity or zero depending on implementation).
- **Type Errors:** Passing non-array-like objects or non-numeric types for `dist_matrix` will cause the underlying Numba function to fail.

### Fix Code Hint
```python
# Ensure inputs are numpy arrays of the same shape
import numpy as np
dist_matrix = np.asarray(dist_matrix)
mask = np.asarray(mask, dtype=bool)
assert dist_matrix.shape == mask.shape, "Shape mismatch"
```

## API Test: `njit_lcss_accumulated_matrix`

### Signature
```python
def njit_lcss_accumulated_matrix(s1, s2, eps, mask)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2233_

_Source doc:_ Compute the longest common subsequence similarity score between two time series. Parameters ---------- s1 : array-like, shape=(sz1, d) First time series. s2 : array-like, shape=(sz2, d) Second time series. eps : float Matching threshold. mask : array-like, shape=(sz1, sz2) Mask. Unconsidered cells must have False values. Returns ------- acc_cost_mat : array-like, shape=(sz1 + 1, sz2 + 1) Accumulated cost matrix.

### Goal
Computes the accumulated cost matrix for the Longest Common Subsequence (LCSS) similarity measure between two time series, supporting constrained alignment via a boolean mask.

### Parameters
- `s1`: array-like, shape=(sz1, d). The first time series.
- `s2`: array-like, shape=(sz2, d). The second time series.
- `eps`: float. The matching threshold; two points are considered matching if their distance is less than or equal to `eps`.
- `mask`: array-like, shape=(sz1, sz2). A boolean mask where `False` values indicate cells that should be excluded from the alignment path.

### Input
- Data must be provided as array-like structures (NumPy arrays preferred).
- Time series must be 2D arrays of shape `(sz, d)`. If working with 3D datasets, extract individual series first.
- `mask` must be a boolean array of shape `(sz1, sz2)` corresponding to the dimensions of the two input series.

### Output
- Returns `acc_cost_mat`: A 2D array of shape `(sz1 + 1, sz2 + 1)` representing the accumulated cost matrix used to derive the LCSS similarity.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.dtw_variants import njit_lcss_accumulated_matrix

s1 = np.array([[1.0], [2.0], [3.0]])
s2 = np.array([[1.1], [2.1], [4.0]])
mask = np.ones((3, 3), dtype=bool)
eps = 0.5

acc_mat = njit_lcss_accumulated_matrix(s1, s2, eps, mask)
print(f"Accumulated matrix shape: {acc_mat.shape}")
assert acc_mat.shape == (4, 4)
```

### LLM Instruction Prompt
- Use `njit_lcss_accumulated_matrix` when you need the raw accumulated cost matrix for LCSS rather than the final scalar similarity score.
- Ensure `s1` and `s2` are 2D arrays.
- Ensure `mask` dimensions strictly match `(len(s1), len(s2))`.

### Prompt Snippet
```text
from tslearn.metrics.dtw_variants import njit_lcss_accumulated_matrix
# Ensure inputs are 2D arrays and mask is boolean (sz1, sz2)
acc_mat = njit_lcss_accumulated_matrix(s1, s2, eps, mask)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 1D arrays instead of 2D `(sz, d)` arrays will cause indexing errors.
- **Mask Dimension Error:** Providing a mask that does not match the lengths of `s1` and `s2` will raise a `ValueError`.
- **Type Error:** Passing non-numeric data for `s1`, `s2`, or `eps` will result in failure during the Numba-jitted computation.

### Fix Code Hint
```python
# Ensure 2D shape for univariate series
s1 = s1.reshape(-1, 1) if s1.ndim == 1 else s1
s2 = s2.reshape(-1, 1) if s2.ndim == 1 else s2
# Ensure mask is boolean
mask = mask.astype(bool)
```

## API Test: `njit_lcss_accumulated_matrix_from_dist_matrix`

### Signature
```python
def njit_lcss_accumulated_matrix_from_dist_matrix(dist_matrix, eps, mask)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:2831_

_Source doc:_ Compute the accumulated cost matrix score between two time series using a precomputed distance matrix. Parameters ---------- dist_matrix : array-like, shape=(sz1, sz2) Array containing the pairwise distances. eps : float (default: 1.) Maximum matching distance threshold. mask : array-like, shape=(sz1, sz2) Mask. Unconsidered cells must have False values. Returns ------- acc_cost_mat : array-like, shape=(sz1 + 1, sz2 + 1) Accumulated cost matrix.

### Goal
Computes the Longest Common Subsequence (LCSS) accumulated cost matrix from a precomputed pairwise distance matrix, allowing for threshold-based matching.

### Parameters
- `dist_matrix`: array-like, shape=(sz1, sz2). The precomputed pairwise distance matrix between two time series.
- `eps`: float. The maximum distance threshold for considering two points as a match.
- `mask`: array-like, shape=(sz1, sz2). A boolean mask where `False` values indicate cells that should be ignored during the accumulation process.

### Input
- The input `dist_matrix` and `mask` must be 2D arrays of compatible shapes `(sz1, sz2)`.
- `dist_matrix` typically contains non-negative values representing distances between time series points.
- `mask` must be a boolean array where `True` represents valid cells and `False` represents cells to be excluded from the LCSS calculation.

### Output
Returns `acc_cost_mat` — A 2D array of shape `(sz1 + 1, sz2 + 1)` representing the accumulated cost matrix used to derive the LCSS similarity.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.dtw_variants import njit_lcss_accumulated_matrix_from_dist_matrix

# Example usage with dummy data
sz1, sz2 = 3, 3
dist_matrix = np.zeros((sz1, sz2))
mask = np.ones((sz1, sz2), dtype=bool)
eps = 1.0

acc_mat = njit_lcss_accumulated_matrix_from_dist_matrix(dist_matrix, eps, mask)
print(f"Correctness witness: Accumulated matrix shape is {acc_mat.shape}")
assert acc_mat.shape == (sz1 + 1, sz2 + 1)
```

### LLM Instruction Prompt
- Use `njit_lcss_accumulated_matrix_from_dist_matrix` only when you have a precomputed distance matrix and need the LCSS accumulated cost matrix.
- Ensure `dist_matrix` and `mask` have matching dimensions `(sz1, sz2)`.
- The output matrix will always have an extra row and column compared to the input matrix (size `sz1+1` by `sz2+1`).

### Prompt Snippet
```text
from tslearn.metrics.dtw_variants import njit_lcss_accumulated_matrix_from_dist_matrix
# Ensure dist_matrix and mask are (sz1, sz2)
acc_mat = njit_lcss_accumulated_matrix_from_dist_matrix(dist_matrix, eps, mask)
```

### Common Failure Modes
- Passing arrays with mismatched dimensions for `dist_matrix` and `mask`.
- Providing a `mask` that is not boolean or does not match the expected `(sz1, sz2)` shape.
- Assuming the output is the same shape as the input; the output is always `(sz1 + 1, sz2 + 1)`.

### Fix Code Hint
```python
# Ensure mask is boolean and matches dist_matrix shape
mask = np.ones_like(dist_matrix, dtype=bool)
# If using a window constraint, update mask:
# mask[np.abs(i - j) > window] = False
```

## API Test: `njit_sakoe_chiba_mask`

### Signature
```python
def njit_sakoe_chiba_mask(sz1, sz2, radius=1)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1487_

_Source doc:_ Compute the Sakoe-Chiba mask. Parameters ---------- sz1 : int The size of the first time series sz2 : int The size of the second time series. radius : int The radius of the band. Returns ------- mask : array-like, shape=(sz1, sz2) Sakoe-Chiba mask. Examples -------- >>> njit_sakoe_chiba_mask(4, 4, 1) array([[ True,  True, False, False], [ True,  True,  True, False], [False,  True,  True,  True], [False, False,  True,  True]]) >>> njit_sakoe_chiba_mask(7, 3, 1) array([[ True,  True, False], [ True,  True,  True], [ True,  True,  True], [ True,  True,  True], [ True,  True,  True], [ True,  True,  True], [False,  True,  True]])

### Goal
Computes a boolean Sakoe-Chiba band mask for constrained Dynamic Time Warping (DTW) alignment between two time series of specified lengths.

### Parameters
- `sz1`: int; The length of the first time series.
- `sz2`: int; The length of the second time series.
- `radius`, default `1`: int; The radius of the Sakoe-Chiba band, defining the allowed deviation from the diagonal.

### Input
The function expects two integers representing the lengths of the time series to be aligned. No specific data structures are required as input, only the scalar dimensions.

### Output
Returns `mask`: A boolean array of shape `(sz1, sz2)` where `True` values indicate indices within the Sakoe-Chiba band and `False` values indicate indices outside the band.

### Valid Call Patterns
```python
from tslearn.metrics.dtw_variants import njit_sakoe_chiba_mask
mask = njit_sakoe_chiba_mask(4, 4, radius=1)
print(mask.shape)
# Witness: (4, 4)
```

### LLM Instruction Prompt
- Use `njit_sakoe_chiba_mask` when you need to generate a constraint mask for DTW-based alignment algorithms.
- Ensure `sz1` and `sz2` are positive integers.
- The `radius` parameter must be a non-negative integer.

### Prompt Snippet
```text
from tslearn.metrics.dtw_variants import njit_sakoe_chiba_mask
# Generate a mask for two series of length 10 with a band radius of 2
mask = njit_sakoe_chiba_mask(10, 10, radius=2)
```

### Common Failure Modes
- Passing non-integer values for `sz1` or `sz2`, which will cause a type error in the underlying Numba-jitted code.
- Providing a `radius` that is too small relative to the difference in series lengths, resulting in a mask that may exclude valid alignment paths.

### Fix Code Hint
```python
import numpy as np
from tslearn.metrics.dtw_variants import njit_sakoe_chiba_mask

# Ensure inputs are integers
sz1, sz2 = 10, 10
radius = 2
mask = njit_sakoe_chiba_mask(int(sz1), int(sz2), radius=int(radius))
assert mask.shape == (10, 10)
print("Correctness witness: Mask generated successfully.")
```

## API Test: `normal`

### Signature
```python
def normal(loc=0.0, scale=1.0, size=(1,))
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:250_

### Goal
Generates a tensor of random numbers sampled from a normal (Gaussian) distribution using the configured backend.

### Parameters
- `loc`, default `0.0`: The mean (center) of the distribution.
- `scale`, default `1.0`: The standard deviation (spread) of the distribution.
- `size`, default `(1,)`: The shape of the output tensor.

### Input
The function requires the `tslearn` backend to be initialized. It is typically used within the context of the `PyTorch` backend to generate tensors for synthetic time-series data or weight initialization.

### Output
Returns a tensor (type depends on the active backend, e.g., `torch.Tensor`) containing random samples drawn from the specified normal distribution.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend

be = instantiate_backend("pytorch")
# Generate a 10x5 tensor of normally distributed values
data = be.normal(loc=0.0, scale=1.0, size=(10, 5))
print(f"Generated tensor shape: {data.shape}")
```

### LLM Instruction Prompt
- Always use the `be` (backend) object returned by `instantiate_backend` to call `normal`.
- Ensure the `size` parameter is provided as a tuple matching the required dimensions for your time-series array (e.g., `(n_ts, sz, d)`).
- Do not use this function if the backend is not explicitly set to a framework that supports `normal` (e.g., `pytorch`).

### Prompt Snippet
```text
# Example of generating synthetic noise for time series
be = instantiate_backend("pytorch")
noise = be.normal(loc=0.0, scale=0.1, size=(n_ts, sz, d))
```

### Common Failure Modes
- **Backend Mismatch:** Calling `normal` on a backend that does not implement it (e.g., some `numpy` configurations might require `numpy.random.normal` instead).
- **Shape Mismatch:** Providing an incorrect `size` tuple that does not align with the expected 3D time-series structure `(n_ts, sz, d)`.
- **Missing Dependency:** Attempting to use the `pytorch` backend without the `pytorch` package installed in the environment.

### Fix Code Hint
```python
# Ensure the backend is initialized correctly before calling
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch")
# Use the backend-specific method
data = be.normal(loc=0.0, scale=1.0, size=(10, 100, 1))
```

## API Test: `normalized_cc`

### Signature
```python
def normalized_cc(s1, s2, norm1=-1.0, norm2=-1.0)
```
_Source: tslearn/tslearn/metrics/cycc.py:10_

_Source doc:_ Normalize cc. Parameters ---------- s1 : array-like, shape=(sz, d), dtype=float64 A time series. s2 : array-like, shape=(sz, d), dtype=float64 Another time series. norm1 : float64, default=-1.0 norm2 : float64, default=-1.0 Returns ------- norm_cc : array-like, shape=(2 * sz - 1), dtype=float64

### Goal
Computes the normalized cross-correlation between two time series of equal length.

### Parameters
- `s1`: A time series of shape `(sz, d)` and `dtype=float64`.
- `s2`: Another time series of shape `(sz, d)` and `dtype=float64`.
- `norm1`, default `-1.0`: Normalization factor for the first time series. If `-1.0`, the value is computed internally.
- `norm2`, default `-1.0`: Normalization factor for the second time series. If `-1.0`, the value is computed internally.

### Input
The function expects two time series as array-like objects with shape `(sz, d)`. Both series must have the same length `sz` and dimensionality `d`. Inputs should be provided as `float64` arrays.

### Output
Returns `norm_cc` as an array-like object of shape `(2 * sz - 1)` and `dtype=float64`, representing the normalized cross-correlation sequence.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import normalized_cc

s1 = np.array([[1.0], [2.0], [3.0]])
s2 = np.array([[3.0], [2.0], [1.0]])
result = normalized_cc(s1, s2)
print(f"Correctness witness: {result.shape == (5,)}")
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` have identical shapes `(sz, d)`.
- If the series are not already `float64`, cast them before passing to the function.
- The output length is strictly `2 * sz - 1`.

### Prompt Snippet
```text
from tslearn.metrics import normalized_cc
# Ensure s1 and s2 are (sz, d) float64 arrays
result = normalized_cc(s1, s2)
```

### Common Failure Modes
- Passing time series of unequal lengths, which will result in a shape mismatch error.
- Providing 1D arrays instead of 2D arrays `(sz, d)`, which violates the expected input shape.
- Providing non-float data types, which may cause issues with the underlying C-extension.

### Fix Code Hint
```python
import numpy as np
# Ensure 2D shape (sz, d) and float64 type
s1 = np.asarray(s1, dtype=np.float64).reshape(-1, 1) if s1.ndim == 1 else np.asarray(s1, dtype=np.float64)
s2 = np.asarray(s2, dtype=np.float64).reshape(-1, 1) if s2.ndim == 1 else np.asarray(s2, dtype=np.float64)
```

## API Test: `pairwise_distances`

### Signature
```python
def pairwise_distances(X, Y=None, metric='euclidean')
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:177_

### Goal
Computes the pairwise distance matrix between time-series datasets using a specified metric.

### Parameters
- `X`: A 3D array-like structure of shape `(n_ts, max_sz, d)` representing the first set of time series.
- `Y`, default `None`: A 3D array-like structure of shape `(m_ts, max_sz, d)` representing the second set of time series. If `None`, computes the pairwise distances within `X`.
- `metric`, default `'euclidean'`: The distance metric to use for the computation.

### Input
- **Data Format:** Input data must be 3D NumPy arrays or tensors of shape `(n_ts, max_sz, d)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` prior to calling this function.
- **Compatibility:** The function operates within the `tslearn` backend framework; ensure the input types are consistent with the intended backend (NumPy or PyTorch).

### Output
Returns `unspecified` — A 2D distance matrix of shape `(n_ts, n_ts)` (if `Y` is `None`) or `(n_ts, m_ts)` (if `Y` is provided) containing the computed distances.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import pairwise_distances
from tslearn.utils import to_time_series_dataset

# Inferred from signature: Compute pairwise distances for a single dataset
X = to_time_series_dataset(np.random.rand(5, 10, 1))
dist_matrix = pairwise_distances(X, metric='euclidean')
assert dist_matrix.shape == (5, 5)
print(f"Correctness witness: Computed distance matrix of shape {dist_matrix.shape}")
```

### LLM Instruction Prompt
- Always ensure input arrays are 3D `(n_ts, max_sz, d)` before passing them to `pairwise_distances`.
- If the input is not 3D, use `tslearn.utils.to_time_series_dataset` to format the data first.
- When using `PyTorch` tensors, ensure the backend context is correctly initialized.

### Prompt Snippet
```text
from tslearn.metrics import pairwise_distances
# Ensure X is 3D: (n_ts, max_sz, d)
dist = pairwise_distances(X, Y=Y, metric='euclidean')
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError` or index error.
- **Metric Incompatibility:** Providing a metric string not supported by the current backend implementation.
- **Backend Mismatch:** Mixing `PyTorch` tensors and `NumPy` arrays in the same call may lead to type errors depending on the active backend.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure input is 3D
X_formatted = to_time_series_dataset(X)
dist = pairwise_distances(X_formatted, metric='euclidean')
```

## API Test: `partial_fit`

### Signature
```python
def partial_fit(self, X, y, *args, **kwargs)
```
_Source: tslearn/tslearn/neural_network/neural_network.py:62  (+1 more definition site/overload)_

_Source doc:_ Update the model with a single iteration over the given data. Parameters ---------- X : array-like, shape (n_ts, sz, d) The input data. y : array-like, shape (n_ts, ) or (n_ts, dim_y) Target values. *args, **kwargs : arguments for the underlying MLPClassifier's method from scikit-learn Returns ------- TimeSeriesMLPClassifier The fitted estimator

### Goal
Performs an incremental update of the `TimeSeriesMLPClassifier` model using a single iteration over the provided time-series dataset.

### Parameters
- `self`: The `TimeSeriesMLPClassifier` instance.
- `X`: array-like, shape (n_ts, sz, d). The input time-series data.
- `y`: array-like, shape (n_ts, ) or (n_ts, dim_y). The target values corresponding to the input time series.
- `*args`: Positional arguments passed to the underlying scikit-learn `MLPClassifier.partial_fit` method.
- `**kwargs`: Keyword arguments passed to the underlying scikit-learn `MLPClassifier.partial_fit` method.

### Input
- **Data Format:** Input `X` must be a 3D array of shape `(n_ts, sz, d)`. If data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this method.
- **Preconditions:** The estimator must be initialized. This method is intended for online learning or scenarios where the dataset is too large to fit into memory at once.

### Output
- Returns the `TimeSeriesMLPClassifier` instance itself, updated with the new batch of data.

### Valid Call Patterns
```python
from tslearn.neural_network import TimeSeriesMLPClassifier
import numpy as np

# Inferred usage pattern
clf = TimeSeriesMLPClassifier(hidden_layer_sizes=(10,))
X = np.random.rand(5, 10, 1)
y = np.array([0, 1, 0, 1, 0])
clf.partial_fit(X, y)
print("Model updated successfully.")
```

### LLM Instruction Prompt
- Use `partial_fit` only when working with `TimeSeriesMLPClassifier`.
- Ensure `X` is formatted as a 3D NumPy array `(n_ts, sz, d)` before invocation.
- Pass additional parameters for the underlying scikit-learn MLP via `*args` and `**kwargs` if necessary.

### Prompt Snippet
```text
# Example of incremental training
clf.partial_fit(X_batch, y_batch)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will raise a `ValueError` or shape-related exception.
- **Uninitialized Estimator:** Calling `partial_fit` on an estimator that has not been properly configured for the input dimensions may lead to runtime errors during the first iteration.
- **Incompatible Backend:** Attempting to pass data types not supported by the underlying scikit-learn MLP implementation.

### Fix Code Hint
```python
# Ensure data is 3D before calling partial_fit
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X_raw)
clf.partial_fit(X_formatted, y)
```

## API Test: `pdist`

### Signature
```python
def pdist(x, metric='euclidean', p=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:193_

### Goal
Computes the pairwise distances between time series in a dataset using a specified distance metric.

### Parameters
- `x`: A 3D array-like structure of shape `(n_ts, max_sz, d)` representing the time-series dataset.
- `metric`, default `'euclidean'`: The distance metric to use for the pairwise computation.
- `p`, default `None`: The power parameter for the Minkowski metric, used only when `metric='minkowski'`.

### Input
The input `x` must be a 3D NumPy array or equivalent tensor of shape `(n_ts, max_sz, d)`. If the data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this function.

### Output
Returns `unspecified` — A pairwise distance matrix of shape `(n_ts, n_ts)` containing the computed distances between all pairs of time series in the input dataset.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics import pdist
from tslearn.utils import to_time_series_dataset

# Example inferred from signature
X = to_time_series_dataset(np.random.rand(5, 10, 1))
dist_matrix = pdist(X, metric='euclidean')
print(f"Correctness witness: {dist_matrix.shape == (5, 5)}")
```

### LLM Instruction Prompt
- When calling `pdist`, ensure the input `x` is a 3D array.
- If using `metric='minkowski'`, you must provide a value for `p`.
- The function returns a square matrix representing the distance between every pair of time series in the input.

### Prompt Snippet
```text
from tslearn.metrics import pdist
# Ensure input is 3D: (n_ts, sz, d)
dist_matrix = pdist(X, metric='euclidean')
```

### Common Failure Modes
- Passing 2D arrays directly without using `to_time_series_dataset`, resulting in shape mismatch errors.
- Omitting the `p` parameter when `metric='minkowski'` is selected.
- Providing non-numeric data or arrays with inconsistent dimensions across the time-series axis.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure data is 3D before calling pdist
X_formatted = to_time_series_dataset(X)
dist_matrix = pdist(X_formatted, metric='euclidean')
```

## API Test: `predict`

### Signature
```python
def predict(self, X)
def predict(self, X=None, n=1)
```
_Source: tslearn/tslearn/forecasting/_arima.py:351  (+12 more definition site/overload)_

_Source doc:_ Forecasts n timestamps of the given data if any, otherwise forecasts n timestamps for the fitted data. Parameters ---------- X : array-like, shape (n_ts, sz, d), optional Time-series dataset to forecast. If None, the fitted data is forecasted otherwise the fitted model is applied to the given data. n : int (default: 1) The number of timestamps to forecast, a.k.a. the horizon. Returns ------- array, shape = (n_ts, n, d) Array of forecasted timestamps

### Goal
Predicts future time-series values or class labels by applying a fitted `tslearn` estimator to input data.

### Parameters
- `self`: The fitted `tslearn` estimator instance.
- `X`, default `None`: The time-series dataset to predict or forecast, provided as an array-like structure of shape `(n_ts, sz, d)`. If `None`, the estimator uses the data it was fitted on.
- `n`, default `1`: The forecasting horizon, representing the number of future timestamps to predict.

### Input
- **Format:** 3D NumPy arrays of shape `(n_ts, sz, d)`.
- **Preconditions:** The estimator must be fitted (via `fit` or `fit_transform`) before calling `predict`. If the input data is 1D or 2D, it must be converted to the required 3D format using `tslearn.utils.to_time_series_dataset` prior to prediction.

### Output
Returns `array` — An array of shape `(n_ts, n, d)` containing the forecasted values or predicted labels.

### Valid Call Patterns
```python
# Standard classification prediction
predictions = knn.predict(X_scaled)

# Forecasting with a specific horizon
forecasts = model.predict(X=X_test, n=5)

# Forecasting on fitted data
forecasts = model.predict(n=10)
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array `(n_ts, sz, d)`. If the input is a list of sequences, wrap it in `tslearn.utils.to_time_series_dataset` first.
- When using forecasting models, ensure the model has been fitted.
- If `X` is provided, it must match the dimensionality expected by the fitted model.

### Prompt Snippet
```text
# Ensure X is 3D before calling predict
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
predictions = model.predict(X_formatted)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without using `to_time_series_dataset` will raise a `ValueError`.
- **Unfitted Estimator:** Calling `predict` before `fit` will raise a `NotFittedError` (or equivalent `RuntimeError` depending on the estimator).
- **Incompatible Dimensions:** Providing `X` with a different number of features `d` than the data used during `fit` will cause a dimension mismatch error.

### Fix Code Hint
```python
# If you encounter a shape error, verify your input:
import numpy as np
from tslearn.utils import to_time_series_dataset

# Ensure input is (n_ts, sz, d)
X_3d = to_time_series_dataset(X)
assert X_3d.ndim == 3
predictions = model.predict(X_3d)
```

## API Test: `predict_class_and_earliness`

### Signature
```python
def predict_class_and_earliness(self, X)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:399_

_Source doc:_ Provide predicted class as well as prediction timestamps. Prediction timestamps are timestamps at which a prediction is made in early classification setting. Parameters ---------- X : array-like of shape (n_series, n_timestamps, n_features) Vector to be scored, where `n_series` is the number of time series, `n_timestamps` is the number of timestamps in the series and `n_features` is the number of features recorded at each timestamp. Returns ------- array, shape (n_series,) Predicted classes. array-like of shape (n_series, ) Prediction timestamps.

### Goal
Performs early classification on time-series data, returning both the predicted class labels and the specific timestamps at which the model reached a decision.

### Parameters
- `self`: An instance of an early classification estimator (e.g., `RNNEarlyClassifier`).
- `X`: array-like of shape (n_series, n_timestamps, n_features). The input time-series dataset to be scored.

### Input
- **Data Format:** Must be a 3D array of shape `(n_series, n_timestamps, n_features)`.
- **Preconditions:** If input data is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this method. The estimator must be fitted using `fit` before calling this method.

### Output
Returns a tuple containing:
- `y_pred`: array of shape `(n_series,)` containing the predicted class labels.
- `timestamps`: array-like of shape `(n_series,)` containing the timestamps at which the predictions were triggered.

### Valid Call Patterns
```python
# Inferred from signature and scikit-learn estimator patterns
y_pred, timestamps = estimator.predict_class_and_earliness(X_test)
```

### LLM Instruction Prompt
- When using `predict_class_and_earliness`, ensure the input `X` is a 3D NumPy array.
- Always unpack the return value into two variables: the class predictions and the prediction timestamps.
- Verify that the estimator has been trained via `fit` before invoking this method.

### Prompt Snippet
```text
# Example usage for early classification
y_pred, timestamps = model.predict_class_and_earliness(X_test)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will raise a `ValueError`. Use `tslearn.utils.to_time_series_dataset` to reshape.
- **Unfitted Estimator:** Calling this method before `fit` will result in a `NotFittedError` (standard scikit-learn behavior).
- **Backend Incompatibility:** If the model was trained with a specific backend (e.g., PyTorch), ensure the input `X` is compatible with that backend's expected tensor types.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure 3D shape (n_ts, sz, d)
X_formatted = to_time_series_dataset(X_raw)
y_pred, timestamps = model.predict_class_and_earliness(X_formatted)
```

## API Test: `predict_log_proba`

### Signature
```python
def predict_log_proba(self, X)
```
_Source: tslearn/tslearn/svm/svm.py:370_

_Source doc:_ Predict class log-probabilities for a given set of time series. Note that probability estimates are not guaranteed to match predict output. See our :ref:`dedicated user guide section <kernels-ml>` for more details. Parameters ---------- X : array-like of shape=(n_ts, sz, d) Time series dataset. Returns ------- array of shape=(n_ts, n_classes), Class probability matrix.

### Goal
Computes the logarithm of class probabilities for a provided time-series dataset using a trained classifier, typically used for probabilistic classification tasks.

### Parameters
- `self`: The trained `tslearn` estimator instance (e.g., `TimeSeriesSVC`).
- `X`: The input time-series dataset of shape `(n_ts, sz, d)`, where `n_ts` is the number of samples, `sz` is the time series length, and `d` is the number of features.

### Input
- **Data Format:** Must be a 3D array-like structure. If input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this method.
- **Preconditions:** The estimator must have been fitted (via `fit`) on training data before calling `predict_log_proba`.

### Output
Returns `array` — A matrix of shape `(n_ts, n_classes)` containing the log-probabilities for each class per time series.

### Valid Call Patterns
```python
# Inferred from signature and scikit-learn compatibility
# Assuming 'clf' is a fitted tslearn classifier
log_probs = clf.predict_log_proba(X_test)
print(f"Log-probability matrix shape: {log_probs.shape}")
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array `(n_ts, sz, d)`.
- Verify the estimator has been trained via `fit` before invoking this method.
- Note that log-probabilities are not guaranteed to align with the hard labels returned by `predict`.

### Prompt Snippet
```text
# Ensure X is 3D before calling predict_log_proba
from tslearn.utils import to_time_series_dataset
X_formatted = to_time_series_dataset(X)
log_probs = estimator.predict_log_proba(X_formatted)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays without using `to_time_series_dataset` will raise a `ValueError`.
- **Unfitted Estimator:** Calling the method before `fit` will result in a `NotFittedError` (standard scikit-learn behavior).
- **Backend Incompatibility:** Using custom data types that do not conform to the expected NumPy/PyTorch backend requirements.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset

# Ensure input is 3D: (n_ts, sz, d)
X_ready = to_time_series_dataset(X_raw)
# Check if estimator is fitted
if hasattr(clf, "classes_"):
    log_probs = clf.predict_log_proba(X_ready)
else:
    print("Estimator must be fitted first.")
```

## API Test: `predict_proba`

### Signature
```python
def predict_proba(self, X)
```
_Source: tslearn/tslearn/neural_network/neural_network.py:112  (+4 more definition site/overload)_

_Source doc:_ Probability estimates. The returned estimates for all classes are ordered by the label of classes. Parameters ---------- X : array-like of shape (n_series, n_timestamps, n_features) Vector to be scored, where `n_series` is the number of time series, `n_timestamps` is the number of timestamps in the series and `n_features` is the number of features recorded at each timestamp. Returns ------- array-like of shape (n_series, n_classes) Probability of the sample for each class in the model, where classes are ordered as they are in ``self.classes_``.

### Goal
Computes the probability estimates for each class for the provided time-series samples, following the scikit-learn estimator interface.

### Parameters
- `self`: The trained `tslearn` estimator instance (e.g., a classifier that supports probabilistic output).
- `X`: array-like of shape `(n_series, n_timestamps, n_features)`. The time-series dataset to be scored.

### Input
- **Data Format:** Must be a 3D NumPy array of shape `(n_series, n_timestamps, n_features)`.
- **Preconditions:** The estimator must be fitted using the `fit` method prior to calling `predict_proba`. If the input data has variable lengths, it must be pre-processed or formatted according to the specific estimator's requirements (e.g., using `tslearn.utils.to_time_series_dataset`).

### Output
Returns `array-like` of shape `(n_series, n_classes)`. Each row corresponds to a time series, and each column represents the probability of that series belonging to the class at that index, ordered by `self.classes_`.

### Valid Call Patterns
```python
import numpy as np
from tslearn.neighbors import KNeighborsTimeSeriesClassifier

# Inferred usage pattern based on scikit-learn compatibility
X = np.random.rand(10, 5, 1)
y = np.array([0, 1, 0, 1, 0, 1, 0, 1, 0, 1])
clf = KNeighborsTimeSeriesClassifier(n_neighbors=1)
clf.fit(X, y)
probas = clf.predict_proba(X)
print(f"Correctness witness: {probas.shape == (10, 2)}")
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array. If the input is a list of variable-length sequences, use `tslearn.utils.to_time_series_dataset` before passing it to `predict_proba`.
- Verify that the estimator has been trained via `fit` before calling this method.
- The output columns correspond to the classes stored in `self.classes_`.

### Prompt Snippet
```text
# Ensure X is 3D
X_formatted = tslearn.utils.to_time_series_dataset(X_raw)
probas = estimator.predict_proba(X_formatted)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will raise a `ValueError`.
- **Unfitted Estimator:** Calling `predict_proba` before `fit` will raise a `NotFittedError` (or equivalent scikit-learn exception).
- **Incompatible Estimator:** Not all `tslearn` estimators support `predict_proba`; ensure the specific model class implements probabilistic output.

### Fix Code Hint
```python
# Ensure input is 3D (n_ts, sz, d)
if X.ndim == 2:
    X = X.reshape((X.shape[0], X.shape[1], 1))
probas = model.predict_proba(X)
```

## API Test: `predict_proba_and_earliness`

### Signature
```python
def predict_proba_and_earliness(self, X)
```
_Source: tslearn/tslearn/early_classification/early_classification.py:453_

_Source doc:_ Provide probability estimates as well as prediction timestamps. Prediction timestamps are timestamps at which a prediction is made in early classification setting. The returned estimates for all classes are ordered by the label of classes. Parameters ---------- X : array-like of shape (n_series, n_timestamps, n_features) Vector to be scored, where `n_series` is the number of time series, `n_timestamps` is the number of timestamps in the series and `n_features` is the number of features recorded at each timestamp. Returns ------- array-like of shape (n_series, n_classes) Probability of the sample for each class in the model, where classes are ordered as they are in ``self.classes_``. array-like of shape (n_series, ) Prediction timestamps.

### Goal
Computes class probability estimates and the corresponding decision timestamps for early time-series classification models.

### Parameters
- `self`: An instance of an early classification estimator (e.g., `tslearn.early_classification.NonContiguousIntervalClassifier`).
- `X`: array-like of shape (n_series, n_timestamps, n_features). The input time-series dataset to be scored.

### Input
- **Format:** Must be a 3D array-like structure. If input is 1D or 2D, it must be converted using `tslearn.utils.to_time_series_dataset` prior to calling this method.
- **Preconditions:** The estimator must be fitted (`fit` method called) before invoking this method.

### Output
Returns a tuple containing:
- `probabilities`: array-like of shape (n_series, n_classes) representing the probability of each sample for each class.
- `timestamps`: array-like of shape (n_series, ) representing the specific timestamp at which the early classification decision was triggered.

### Valid Call Patterns
```python
# Inferred from signature:
probs, timestamps = estimator.predict_proba_and_earliness(X)
```

### LLM Instruction Prompt
- When using `predict_proba_and_earliness`, ensure the input `X` is a 3D array formatted for `tslearn`.
- Always unpack the return value into two variables: one for the probability matrix and one for the prediction timestamps.
- Ensure the estimator has been trained on a compatible dataset before calling this method.

### Prompt Snippet
```text
# Example usage:
probs, timestamps = model.predict_proba_and_earliness(X_test)
print(f"Class probabilities: {probs}")
print(f"Decision timestamps: {timestamps}")
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays instead of 3D arrays will raise a `ValueError`. Use `tslearn.utils.to_time_series_dataset` to fix.
- **Unfitted Estimator:** Calling this method before `fit` will result in a `NotFittedError` (standard scikit-learn behavior).
- **Incompatible Estimator:** This method is specific to early classification models; calling it on standard classifiers (e.g., `KNeighborsTimeSeriesClassifier`) will raise an `AttributeError`.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure X is 3D
X_formatted = to_time_series_dataset(X)
# Ensure model is fitted
model.fit(X_train, y_train)
probs, timestamps = model.predict_proba_and_earliness(X_formatted)
```

## API Test: `random_walk_blobs`

### Signature
```python
def random_walk_blobs(n_ts_per_blob=100, sz=256, d=1, n_blobs=2, noise_level=1.0, random_state=None)
```
_Source: tslearn/tslearn/generators/generators.py:57_

_Source doc:_ Blob-based random walk time series generator. Generate n_ts_per_blobs * n_blobs time series of size sz and dimensionality d. Generated time series follow the model: .. math:: ts[t] = ts[t - 1] + a where :math:`a` is drawn from a normal distribution of mean mu and standard deviation std. Each blob contains time series derived from a same seed time series with added white noise. Parameters ---------- n_ts_per_blob : int (default: 100) Number of time series in each blob sz : int (default: 256) Length of time series (number of time instants) d : int (default: 1) Dimensionality of time series n_blobs : int (default: 2) Number of blobs noise_level : float (default: 1.) Standard deviation of white noise added to time series in each blob random_state : integer or numpy.RandomState or None (default: None) Generator used to draw the time series. If an integer is given, it fixes the seed. Defaults to the global numpy random number generator. Returns ------- numpy.ndarray A dataset of random walk time series numpy.ndarray Labels associated to random walk time series (blob id) Examples -------- >>> X, y = random_walk_blobs(n_ts_per_blob=100, sz=256, d=5, n_blobs=3) >>> X.shape (300, 256, 5) >>> y.shape (300,)

### Goal
Generates synthetic time-series datasets consisting of multiple clusters (blobs) of random walk sequences, useful for testing clustering and classification algorithms.

### Parameters
- `n_ts_per_blob`, default `100`: Number of time series to generate for each distinct cluster.
- `sz`, default `256`: The temporal length (number of time steps) of each generated time series.
- `d`, default `1`: The dimensionality (number of features) of each time series.
- `n_blobs`, default `2`: The total number of distinct clusters to generate.
- `noise_level`, default `1.0`: The standard deviation of the white noise added to the seed time series within each blob.
- `random_state`, default `None`: Seed for the random number generator to ensure reproducibility.

### Input
No input data is required; this is a synthetic data generator. The function returns data in the standard `tslearn` 3D format `(n_ts, sz, d)`.

### Output
Returns a tuple containing:
1. `X`: A `numpy.ndarray` of shape `(n_ts_per_blob * n_blobs, sz, d)` containing the generated time series.
2. `y`: A `numpy.ndarray` of shape `(n_ts_per_blob * n_blobs,)` containing the integer labels identifying the blob membership for each time series.

### Valid Call Patterns
```python
from tslearn.generators import random_walk_blobs

# Generate a dataset with 3 blobs, 100 series each, length 256, 5 dimensions
X, y = random_walk_blobs(n_ts_per_blob=100, sz=256, d=5, n_blobs=3, random_state=42)
assert X.shape == (300, 256, 5)
assert y.shape == (300,)
print(f"Generated {X.shape[0]} series with {X.shape[2]} dimensions.")
```

### LLM Instruction Prompt
- Always import `random_walk_blobs` from `tslearn.generators`.
- Ensure the output `X` is treated as a 3D NumPy array compatible with `tslearn` estimators.
- Use `random_state` when reproducibility is required for benchmarking or testing.

### Prompt Snippet
```text
from tslearn.generators import random_walk_blobs
X, y = random_walk_blobs(n_ts_per_blob=..., sz=..., d=..., n_blobs=...)
```

### Common Failure Modes
- Passing non-integer values for `n_ts_per_blob`, `sz`, `d`, or `n_blobs`, which will cause errors in array allocation.
- Assuming the output is 2D; remember that `tslearn` generators return 3D arrays `(n_ts, sz, d)` even if `d=1`.

### Fix Code Hint
```python
# Ensure dimensions are integers
X, y = random_walk_blobs(n_ts_per_blob=int(100), sz=int(256), d=int(1))
```

## API Test: `random_walks`

### Signature
```python
def random_walks(n_ts=100, sz=256, d=1, mu=0.0, std=1.0, random_state=None)
```
_Source: tslearn/tslearn/generators/generators.py:7_

_Source doc:_ Random walk time series generator. Generate n_ts time series of size sz and dimensionality d. Generated time series follow the model: .. math:: ts[t] = ts[t - 1] + a where :math:`a` is drawn from a normal distribution of mean mu and standard deviation std. Parameters ---------- n_ts : int (default: 100) Number of time series. sz : int (default: 256) Length of time series (number of time instants). d : int (default: 1) Dimensionality of time series. mu : float (default: 0.) Mean of the normal distribution from which random walk steps are drawn. std : float (default: 1.) Standard deviation of the normal distribution from which random walk steps are drawn. random_state : integer or numpy.RandomState or None (default: None) Generator used to draw the time series. If an integer is given, it fixes the seed. Defaults to the global numpy random number generator. Returns ------- numpy.ndarray A dataset of random walk time series Examples -------- >>> random_walks(n_ts=100, sz=256, d=5, mu=0., std=1.).shape (100, 256, 5)

### Goal
Generates synthetic time-series datasets based on a random walk model, useful for testing clustering, classification, and forecasting estimators.

### Parameters
- `n_ts`, default `100`: The number of time-series samples to generate.
- `sz`, default `256`: The length (number of time steps) of each generated time series.
- `d`, default `1`: The dimensionality (number of features) of each time series.
- `mu`, default `0.0`: The mean of the normal distribution used to draw the random walk steps.
- `std`, default `1.0`: The standard deviation of the normal distribution used to draw the random walk steps.
- `random_state`, default `None`: Seed or `numpy.random.RandomState` instance for deterministic generation.

### Input
No external input required; this is a synthetic data generator. The function produces data in the standard `tslearn` 3D format: `(n_ts, sz, d)`.

### Output
Returns a `numpy.ndarray` of shape `(n_ts, sz, d)` containing the generated random walk time-series data.

### Valid Call Patterns
```python
from tslearn.generators import random_walks

# Generate default dataset
X = random_walks(n_ts=20, sz=16, d=1)

# Generate with specific parameters
X_custom = random_walks(n_ts=100, sz=10, d=3, std=0.0, random_state=42)
```

### LLM Instruction Prompt
- Use `random_walks` to generate synthetic 3D NumPy arrays for testing `tslearn` estimators.
- Always ensure the output shape matches `(n_ts, sz, d)`.
- If reproducibility is required, always provide an integer to the `random_state` parameter.

### Prompt Snippet
```text
from tslearn.generators import random_walks
X = random_walks(n_ts=100, sz=256, d=1)
```

### Common Failure Modes
- Passing non-integer values for `n_ts`, `sz`, or `d`, which will cause internal array allocation errors.
- Expecting the output to be 2D; the generator always returns a 3D array `(n_ts, sz, d)`, even if `d=1`.
- Assuming `std=0` results in random noise; it results in a constant time series (all zeros).

### Fix Code Hint
```python
import numpy as np
from tslearn.generators import random_walks

# Verify output shape for a specific configuration
X = random_walks(n_ts=50, sz=100, d=2)
assert X.shape == (50, 100, 2)
print(f"Correctness witness: Generated array with shape {X.shape}")
```

## API Test: `sakoe_chiba_mask`

### Signature
```python
def sakoe_chiba_mask(sz1, sz2, radius=1, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1536  (+1 more definition site/overload)_

_Source doc:_ Compute the Sakoe-Chiba mask. Parameters ---------- sz1 : int The size of the first time series sz2 : int The size of the second time series. radius : int The radius of the band. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- mask : array-like, shape=(sz1, sz2) Sakoe-Chiba mask. Examples -------- >>> sakoe_chiba_mask(4, 4, radius=2) array([[ True,  True,  True, False], [ True,  True,  True,  True], [ True,  True,  True,  True], [False,  True,  True,  True]])

### Goal
Computes a boolean Sakoe-Chiba band mask of shape `(sz1, sz2)` to constrain the alignment path in time-series distance metrics like Dynamic Time Warping (DTW).

### Parameters
- `sz1`: `int`. The length of the first time series.
- `sz2`: `int`. The length of the second time series.
- `radius`, default `1`: `int`. The radius of the Sakoe-Chiba band; defines the maximum allowed deviation from the diagonal.
- `be`, default `None`: `Backend object` or `string` or `None`. Specifies the computation engine (`"numpy"` or `"pytorch"`). If `None`, the backend is inferred.

### Input
The function requires two integer dimensions (`sz1`, `sz2`) representing the lengths of the time series to be aligned. No specific data array is required as input, only the dimensions.

### Output
Returns `mask`: An `array-like` object of shape `(sz1, sz2)` containing boolean values, where `True` indicates that the cell is within the Sakoe-Chiba band and `False` indicates it is outside.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Standard usage with NumPy backend
mask = tslearn.metrics.sakoe_chiba_mask(4, 4, radius=1)
print(f"Correctness witness: {mask.shape == (4, 4)}")

# Explicit backend usage
mask_pt = tslearn.metrics.sakoe_chiba_mask(7, 3, radius=1, be="numpy")
assert isinstance(mask_pt, np.ndarray)
```

### LLM Instruction Prompt
- Use `tslearn.metrics.sakoe_chiba_mask` to generate alignment constraints.
- Always provide `sz1` and `sz2` as integers.
- If performing differentiable operations, explicitly set `be="pytorch"`.
- Ensure the `radius` is a non-negative integer.

### Prompt Snippet
```text
from tslearn.metrics import sakoe_chiba_mask
mask = sakoe_chiba_mask(sz1=len_a, sz2=len_b, radius=r)
```

### Common Failure Modes
- Passing non-integer values for `sz1` or `sz2`, which will cause indexing errors in the underlying backend.
- Providing a `radius` that is too small, resulting in a mask that excludes valid alignment paths.
- Mismatching the `be` backend type with the data types used in subsequent distance calculations (e.g., using a NumPy mask with PyTorch tensors).

### Fix Code Hint
```python
# Ensure dimensions are integers and radius is valid
sz1, sz2 = int(series1.shape[0]), int(series2.shape[0])
mask = tslearn.metrics.sakoe_chiba_mask(sz1, sz2, radius=2)
```

## API Test: `save_dict`

### Signature
```python
def save_dict(d, filename, group, raise_type_fail=True)
```
_Source: tslearn/tslearn/hdftools/hdftools.py:8_

_Source doc:_ Recursively save a dict to an hdf5 group in a new file. Parameters ---------- d: dict dict to save as an hdf5 file filename : str Full path to save the file to. File must not already exist. group : str group name to save the dict to raise_type_fail : bool If True: raise an exception if saving a part of the dict fails. If False: prints a warning instead and saves the object's __str__() return value. Returns ------- None Raises ------ FileExistsError If the path specified by the `filename` parameter already exists. TypeError If a particular entry within the dict cannot be saved to hdf5 AND the argument `raise_type_fail` is set to `True`

### Goal
Persists a dictionary containing time-series data or model parameters into an HDF5 file structure for efficient storage and retrieval.

### Parameters
- `d`: `dict` — The dictionary containing data (e.g., NumPy arrays) to be serialized.
- `filename`: `str` — The full filesystem path where the HDF5 file will be created.
- `group`: `str` — The HDF5 group name under which the dictionary contents will be stored.
- `raise_type_fail`, default `True`: `bool` — If `True`, raises a `TypeError` if an entry cannot be serialized; if `False`, logs a warning and stores the string representation of the object.

### Input
- The `filename` must point to a path that does not already exist on the filesystem.
- The dictionary `d` should contain types compatible with HDF5 storage (e.g., NumPy arrays, scalars).

### Output
Returns `None`. The function performs a side-effect by writing the dictionary to the specified HDF5 file.

### Valid Call Patterns
```python
from tslearn.hdftools import save_dict
import numpy as np

data = {"ts_data": np.random.rand(10, 5)}
save_dict(data, filename="my_data.hdf5", group="experiment_1")
```

### LLM Instruction Prompt
- Use `tslearn.hdftools.save_dict` to persist dictionaries to disk.
- Always ensure the `filename` does not exist before calling, as the function will raise a `FileExistsError` otherwise.
- If the dictionary contains complex custom objects, set `raise_type_fail=False` to avoid crashes, noting that data will be saved as strings.

### Prompt Snippet
```text
from tslearn.hdftools import save_dict
# Ensure filename is unique
save_dict(my_dict, filename="path/to/file.hdf5", group="my_group")
```

### Common Failure Modes
- `FileExistsError`: Occurs if the `filename` provided already exists on the disk.
- `TypeError`: Occurs if an object in the dictionary cannot be mapped to an HDF5-compatible format and `raise_type_fail` is `True`.

### Fix Code Hint
```python
import os
# Check for existence before saving
if not os.path.exists(filename):
    save_dict(d, filename, group)
else:
    print(f"File {filename} already exists.")
```

## API Test: `save_time_series_txt`

### Signature
```python
def save_time_series_txt(fname, dataset, fmt='%.18e')
```
_Source: tslearn/tslearn/utils/utils.py:364_

_Source doc:_ Writes a time series dataset to disk. Parameters ---------- fname : string Path to the file in which time series should be written. dataset : array-like The dataset of time series to be saved. fmt : string (default: "%.18e") Format to be used to write each value. Examples -------- >>> dataset = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3]]) >>> save_time_series_txt("tmp-tslearn-test.txt", dataset) See Also -------- load_time_series_txt : Load time series from disk

### Goal
Persists a time-series dataset to a text file on disk, enabling serialization of processed or raw time-series data for later retrieval.

### Parameters
- `fname`: `str`. The file path where the dataset will be saved.
- `dataset`: `array-like`. The time-series dataset to be saved, typically a 3D NumPy array of shape `(n_ts, max_sz, d)`.
- `fmt`, default `'%.18e'`: `str`. The printf-style format string used to write each numerical value in the dataset.

### Input
The `dataset` must be compatible with `tslearn`'s internal representation. If the input is a list of sequences or a 2D array, it should be pre-processed using `tslearn.utils.to_time_series_dataset` to ensure it conforms to the required 3D `(n_ts, max_sz, d)` structure before calling this function.

### Output
Returns `None`. The function performs a side-effect operation by writing the dataset to the specified file path.

### Valid Call Patterns
```python
from tslearn.utils import to_time_series_dataset, save_time_series_txt
import numpy as np

# Prepare data
data = to_time_series_dataset([[1, 2, 3, 4], [1, 2, 3]])
# Save to disk
save_time_series_txt("ts_data.txt", data, fmt='%.4f')
```

### LLM Instruction Prompt
- Always ensure the `dataset` is in the 3D `(n_ts, max_sz, d)` format expected by `tslearn` before passing it to `save_time_series_txt`.
- Use `to_time_series_dataset` if the input data is a list of variable-length sequences.
- The `fmt` parameter defaults to scientific notation; adjust this only if specific precision or formatting is required for downstream compatibility.

### Prompt Snippet
```text
from tslearn.utils import save_time_series_txt
# Ensure dataset is 3D via to_time_series_dataset(data)
save_time_series_txt(fname="output.txt", dataset=my_dataset)
```

### Common Failure Modes
- **Shape Mismatch:** Passing a 1D or 2D array directly without using `to_time_series_dataset` will result in an error or an incorrectly formatted file.
- **Permission Errors:** Providing a `fname` path where the process lacks write permissions will raise an `IOError` or `OSError`.
- **Invalid Format String:** Providing an incompatible `fmt` string (e.g., one that does not support floating-point numbers) will cause a `ValueError` during the write operation.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset, save_time_series_txt
import numpy as np

# Ensure data is 3D before saving
X = to_time_series_dataset(my_raw_data)
save_time_series_txt("data.txt", X)
```

## API Test: `select_backend`

### Signature
```python
def select_backend(data)
```
_Source: tslearn/tslearn/backend/backend.py:31_

_Source doc:_ Select backend. Parameter --------- data : array-like or string or None Indicates the backend to choose. Optional, default equals None. Returns ------- backend : class The backend class. If data is a Numpy array or data equals 'numpy' or data is None, backend equals NumpyBackend(). If data is a PyTorch array or data equals 'pytorch', backend equals PytorchBackend().

### Goal
Determines and returns the appropriate computation backend (NumPy or PyTorch) based on the provided input data type or a string identifier to facilitate backend-agnostic time-series operations.

### Parameters
- `data`: `array-like`, `string`, or `None`. Indicates the backend to choose. If `None`, defaults to `NumpyBackend`.

### Input
- **Formats:** Accepts `numpy.ndarray`, `torch.Tensor`, or strings `'numpy'` / `'pytorch'`.
- **Preconditions:** If passing an array, it should be formatted as a 3D time-series dataset `(n_ts, max_sz, d)`. If using the `'pytorch'` backend, the `pytorch` package must be installed in the environment.

### Output
Returns `backend` : `class`. An instance of either `NumpyBackend` or `PytorchBackend` capable of performing time-series metric computations.

### Valid Call Patterns
```python
from tslearn.backend import select_backend

# Default (NumPy)
be = select_backend(None)

# Explicit string selection
be_torch = select_backend("pytorch")

# Inference from data
import numpy as np
be_np = select_backend(np.array([[[1.0]]]))
```

### LLM Instruction Prompt
- Use `select_backend` to dynamically resolve the computation engine when writing generic time-series functions that must support both NumPy and PyTorch.
- Always prefer passing the data object itself to `select_backend` to allow the function to infer the correct backend automatically.

### Prompt Snippet
```text
# Example of backend-agnostic metric usage
from tslearn.backend import select_backend
be = select_backend(data)
distance = be.dtw(data1, data2)
```

### Common Failure Modes
- **Missing Dependency:** Passing `'pytorch'` or a `torch.Tensor` when the `pytorch` library is not installed will raise an `ImportError` or `TypeError`.
- **Invalid Input:** Passing an unsupported type (e.g., a list of lists without conversion to a 3D array) may lead to unexpected backend selection or runtime errors during subsequent metric calculations.

### Fix Code Hint
```python
import numpy as np
from tslearn.backend import select_backend

# Ensure data is 3D before backend selection
data = np.array([[[1.0], [2.0]]])
be = select_backend(data)

# Verify correctness witness
print(f"Selected backend: {type(be).__name__}")
assert "Backend" in type(be).__name__
```

## API Test: `set_backend`

### Signature
```python
def set_backend(self, data=None)
```
_Source: tslearn/tslearn/backend/backend.py:87_

### Goal
Updates the internal computation engine of a `Backend` instance to either `numpy` or `pytorch` based on the provided data or explicit string identifier.

### Parameters
- `self`: The `Backend` instance being configured.
- `data`, default `None`: An optional input (e.g., a NumPy array or PyTorch tensor) used to infer the appropriate backend, or a string identifier ("numpy" or "torch").

### Input
The caller must provide a `Backend` instance. If `data` is provided, it should be a NumPy array or a PyTorch tensor to allow the backend to auto-detect the required computation engine. If `data` is `None`, the method relies on internal state or explicit configuration.

### Output
Returns `None`. The method modifies the `Backend` instance in-place, updating its internal `is_pytorch` and `is_numpy` flags.

### Valid Call Patterns
```python
from tslearn.backend import Backend

# Initialize and switch backend explicitly
backend_ = Backend("torch")
backend_.set_backend("numpy")

# Verify state change
assert backend_.is_numpy
```

### LLM Instruction Prompt
- Use `set_backend` only on an existing `Backend` object instance.
- When switching backends, ensure the target environment has the necessary dependencies (e.g., `torch` must be installed to use the "torch" backend).
- Prefer passing a string ("numpy" or "torch") to `set_backend` for explicit control, or pass a data object if you want the backend to infer the type.

### Prompt Snippet
```text
# Example of switching backend to numpy
backend_instance.set_backend("numpy")
```

### Common Failure Modes
- **Dependency Missing:** Calling `set_backend("torch")` when `pytorch` is not installed in the environment will raise an `ImportError` or `RuntimeError`.
- **Invalid Argument:** Passing an unsupported string (e.g., "tensorflow") will result in an error as `tslearn` only supports `numpy` and `pytorch` backends.
- **Instance Mismatch:** Attempting to call `set_backend` on a class rather than an instantiated `Backend` object.

### Fix Code Hint
```python
from tslearn.backend import Backend

# Ensure the backend is set correctly before performing metric computations
be = Backend("numpy")
try:
    be.set_backend("torch")
except Exception as e:
    print(f"Failed to switch backend: {e}")
```

## API Test: `set_weights`

### Signature
```python
def set_weights(self, weights, layer_name=None)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:866_

_Source doc:_ Set model weights (or weights for a given layer if `layer_name` is provided). Parameters ---------- weights: list of ndarrays Weights to set for the model / target layer layer_name: str or None (default: None) Name of the layer for which  weights should be set. If None, all model weights are set. Available layer names with weights are: - "shapelets_i_j" with i an integer for the shapelet id and j an integer for the dimension - "classification" for the final classification layer Examples -------- >>> from tslearn.generators import random_walk_blobs >>> X, y = random_walk_blobs(n_ts_per_blob=10, sz=16, d=1, n_blobs=3) >>> clf = LearningShapelets(n_shapelets_per_size={3: 1}, max_iter=1, ...                     verbose=0) >>> _ = clf.fit(X, y) >>> weights_shapelet = [ ...     numpy.array([[[1], [2], [3]]]) ... ] >>> clf.set_weights(weights_shapelet, layer_name="shapelets_0") >>> clf.shapelets_as_time_series_ array([[[1.], [2.], [3.]]])

### Goal
Manually update the internal parameters of a `LearningShapelets` estimator, allowing for custom initialization or fine-tuning of shapelet values and classification layers.

### Parameters
- `self`: The `LearningShapelets` estimator instance.
- `weights`: A list of `numpy.ndarray` objects containing the weight values to be assigned to the model or specific layer.
- `layer_name`, default `None`: A string identifier for the target layer (e.g., `"shapelets_i_j"` or `"classification"`). If `None`, updates all model weights.

### Input
- The estimator must be initialized and typically fitted (or at least have its architecture defined) before weights can be set.
- `weights` must be provided as a list of `numpy.ndarray` objects matching the expected shape of the target layer.

### Output
Returns `None`. The method modifies the estimator instance in-place.

### Valid Call Patterns
```python
import numpy as np
from tslearn.shapelets import LearningShapelets

# Initialize and fit
clf = LearningShapelets(n_shapelets_per_size={3: 1}, max_iter=1)
clf.fit(X, y)

# Update specific shapelet layer
weights_shapelet = [np.array([[[1], [2], [3]]])]
clf.set_weights(weights_shapelet, layer_name="shapelets_0")
```

### LLM Instruction Prompt
- Use `set_weights` only on `LearningShapelets` instances.
- Ensure the `weights` list contains `numpy.ndarray` objects that match the dimensions of the target layer defined during initialization.
- Use the `layer_name` parameter to target specific components like `"shapelets_i_j"` or `"classification"`.

### Prompt Snippet
```text
# Example of setting weights for a specific shapelet layer
clf.set_weights([new_weights_array], layer_name="shapelets_0")
```

### Common Failure Modes
- **Shape Mismatch:** Providing `weights` arrays that do not match the expected dimensions of the target layer will raise a `ValueError` or `IndexError`.
- **Uninitialized Model:** Calling `set_weights` on an estimator that has not been initialized or has an incompatible architecture for the provided `layer_name`.
- **Invalid Layer Name:** Passing a `layer_name` that does not exist in the model architecture.

### Fix Code Hint
```python
# Ensure weights are in a list and match the expected shape
# Check layer_name against model.shapelets_ or model.classification_layer_
clf.set_weights([np.array(data)], layer_name="shapelets_0")
```

## API Test: `shape`

### Signature
```python
def shape(self, data)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:202_

### Goal
Returns the shape of the provided time-series data structure as a tuple, consistent with the backend's internal representation.

### Parameters
- `self`: The backend instance (e.g., `PyTorchBackend` or `NumPyBackend`) used to perform the operation.
- `data`: The time-series dataset or tensor whose dimensions are to be retrieved.

### Input
The `data` argument should be a 3D array-like structure (NumPy array or PyTorch tensor) with dimensions `(n_ts, max_sz, d)`. If the input is 1D or 2D, it must be converted to a 3D time-series dataset using `tslearn.utils.to_time_series_dataset` prior to calling this method.

### Output
Returns `unspecified` — A tuple representing the dimensions of the input `data` (e.g., `(n_samples, n_timestamps, n_features)`).

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend
import numpy as np

be = instantiate_backend("numpy")
data = np.random.rand(10, 20, 3)
print(be.shape(data))
# Output: (10, 20, 3)
```

### LLM Instruction Prompt
- Always use an instantiated backend object (`be`) to call `shape`.
- Ensure the input `data` is a 3D array or tensor; if the data is variable-length or 2D, ensure it has been processed by `to_time_series_dataset` first.
- Do not assume the return type is a specific container; treat it as a tuple of integers.

### Prompt Snippet
```text
# Example of retrieving data shape via backend
shape_tuple = be.shape(my_time_series_data)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing 1D or 2D arrays directly without using `to_time_series_dataset` will result in incorrect shape reporting or runtime errors.
- **Backend Mismatch:** Attempting to pass a PyTorch tensor to a `NumPyBackend` instance may cause type-specific errors; always use `instantiate_backend` to ensure the backend matches the data type.
- **NoneType Input:** Passing `None` or an empty list will raise an `AttributeError` or `TypeError` depending on the backend implementation.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset
# Ensure data is 3D before calling shape
data_3d = to_time_series_dataset(data)
dims = be.shape(data_3d)
```

## API Test: `shapelets_`

### Signature
```python
def shapelets_(self)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:441_

### Goal
Retrieves the learned shapelets from a `LearningShapelets` estimator after the model has been fitted to the training data.

### Parameters
- `self`: An instance of `tslearn.shapelets.LearningShapelets` that has already been trained via the `fit` method.

### Input
- **Preconditions:** The `LearningShapelets` estimator must have been successfully trained using the `fit(X, y)` method.
- **Data Format:** The underlying model must have been initialized with `n_shapelets_per_size` to define the shapelet structure.

### Output
Returns `numpy.ndarray` — A collection of learned shapelets, typically represented as a 3D array of shape `(n_shapelets, shapelet_length, d)`.

### Valid Call Patterns
```python
from tslearn.shapelets import LearningShapelets
import numpy as np

# Setup and fit
n, sz, d = 15, 10, 2
X = np.random.randn(n, sz, d)
y = np.random.randint(2, size=n)
clf = LearningShapelets(n_shapelets_per_size={2: 5}, max_iter=1)
clf.fit(X, y)

# Access learned shapelets
learned_shapelets = clf.shapelets_
print(f"Shapelets shape: {learned_shapelets.shape}")
```

### LLM Instruction Prompt
- Always ensure `fit()` has been called on the `LearningShapelets` instance before accessing `shapelets_`.
- Use `shapelets_` to inspect the learned features; use `shapelets_as_time_series_` if you need the shapelets formatted specifically for time-series visualization or comparison.
- Note that the returned array format depends on the backend (e.g., PyTorch, TensorFlow, or JAX) used during training.

### Prompt Snippet
```text
# Accessing learned shapelets from a fitted model
shapelets = model.shapelets_
```

### Common Failure Modes
- **AttributeError:** Occurs if `shapelets_` is accessed before calling `fit()`.
- **Backend Mismatch:** If using a non-NumPy backend (e.g., PyTorch), the returned object may be a tensor; ensure conversion to NumPy if performing standard array operations.
- **Empty Model:** If `n_shapelets_per_size` is not provided or is empty, the model will not contain learned shapelets.

### Fix Code Hint
```python
# Ensure the model is fitted
if not hasattr(clf, "shapelets_"):
    clf.fit(X_train, y_train)
# Access safely
shapelets = clf.shapelets_
```

## API Test: `shapelets_as_time_series_`

### Signature
```python
def shapelets_as_time_series_(self)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:455_

_Source doc:_ Set of time-series shapelets formatted as a ``tslearn`` time series dataset. Examples -------- >>> from tslearn.generators import random_walk_blobs >>> X, y = random_walk_blobs(n_ts_per_blob=10, sz=256, d=1, n_blobs=3) >>> model = LearningShapelets(n_shapelets_per_size={3: 2, 4: 1}, ...                       max_iter=1) >>> _ = model.fit(X, y) >>> model.shapelets_as_time_series_.shape (3, 4, 1)

### Goal
Retrieves the learned shapelets from a `LearningShapelets` estimator, formatted as a standard 3D `tslearn` time-series dataset.

### Parameters
- `self`: An instance of `tslearn.shapelets.LearningShapelets` that has already been fitted to a dataset using the `fit` method.

### Input
- **Preconditions:** The `LearningShapelets` estimator must have been successfully trained via `fit(X, y)`.
- **Data Format:** The internal shapelets are stored as a collection of variable-length sequences; this property converts them into a uniform 3D NumPy array of shape `(n_shapelets, max_shapelet_length, d)`.

### Output
- Returns a 3D `numpy.ndarray` of shape `(n_shapelets, max_shapelet_length, d)` containing the learned shapelets, where `NaN` values are used to pad shorter shapelets to the length of the longest shapelet.

### Valid Call Patterns
```python
from tslearn.shapelets import LearningShapelets
model = LearningShapelets(n_shapelets_per_size={3: 2, 4: 1}, max_iter=1)
model.fit(X, y)
shapelets_dataset = model.shapelets_as_time_series_
```

### LLM Instruction Prompt
- Always ensure the `LearningShapelets` model has been fitted before accessing `shapelets_as_time_series_`.
- Treat the output as a 3D NumPy array compatible with other `tslearn` preprocessing or visualization tools.
- Note that the output includes padding (NaNs) to maintain the 3D array structure.

### Prompt Snippet
```text
# Accessing learned shapelets from a fitted LearningShapelets model
shapelets = model.shapelets_as_time_series_
```

### Common Failure Modes
- **AttributeError:** Accessing `shapelets_as_time_series_` before calling `fit()` on the estimator.
- **Shape Mismatch:** Assuming all shapelets have the same length; the output is padded with `NaN` to accommodate the longest shapelet in the set.

### Fix Code Hint
```python
import numpy as np
from tslearn.shapelets import LearningShapelets

# Ensure model is fitted
model = LearningShapelets(n_shapelets_per_size={3: 2}, max_iter=1)
model.fit(X, y)

# Access and handle padding if necessary
shapelets = model.shapelets_as_time_series_
# To remove padding for analysis:
clean_shapelets = [s[~np.isnan(s).any(axis=1)] for s in shapelets]
print(f"Retrieved {len(clean_shapelets)} shapelets.")
```

## API Test: `sigma_gak`

### Signature
```python
def sigma_gak(dataset, n_samples=100, random_state=None, be=None)
```
_Source: tslearn/tslearn/metrics/_gak.py:20  (+1 more definition site/overload)_

_Source doc:_ Compute sigma value to be used for GAK. This method was originally presented in [1]_. Parameters ---------- dataset : array-like, shape=(n_ts, sz, d) or (n_ts, sz1) or (sz,) A dataset of time series. If shape is (sz,), the dataset is composed of a unique univariate time series. n_samples : int (default: 100) Number of samples on which median distance should be estimated. random_state : integer or numpy.RandomState or None (default: None) The generator used to draw the samples. If an integer is given, it fixes the seed. Defaults to the global numpy random number generator. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- float Suggested bandwidth (:math:`\sigma`) for the Global Alignment kernel. Examples -------- >>> dataset = [[1, 2, 2, 3], [1., 2., 3., 4.]] >>> float(sigma_gak(dataset=dataset, ...                 n_samples=200, ...                 random_state=0))  # doctest: +ELLIPSIS 2.0... See Also -------- gak : Compute Global Alignment kernel cdist_gak : Compute cross-similarity matrix using Global Alignment kernel References ---------- .. [1] M. Cuturi, "Fast global alignment kernels," ICML 2011.

### Goal
Computes the optimal bandwidth parameter ($\sigma$) for the Global Alignment Kernel (GAK) by estimating the median distance between time series samples in the provided dataset.

### Parameters
- `dataset`: Array-like, shape `(n_ts, sz, d)`, `(n_ts, sz)`, or `(sz,)`. The collection of time series used to estimate the bandwidth.
- `n_samples`, default `100`: The number of random pairs to sample from the dataset to estimate the median distance.
- `random_state`, default `None`: Seed or random number generator instance to ensure reproducibility of the sampling process.
- `be`, default `None`: The backend engine to use for computation. Can be `"numpy"`, `"pytorch"`, or an instance of a backend class. If `None`, the backend is inferred from the input array type.

### Input
- **Data Format:** Must be a 3D array `(n_ts, sz, d)` or compatible 2D/1D structures. Use `tslearn.utils.to_time_series_dataset` to ensure input is in the standard `tslearn` format before passing to this function.
- **Preconditions:** The dataset must contain enough samples to perform the requested `n_samples` calculation.

### Output
- Returns a `float` representing the suggested bandwidth ($\sigma$) for the Global Alignment kernel.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

dataset = [[1, 2, 2, 3], [1., 2., 3., 4.]]
sigma = tslearn.metrics.sigma_gak(dataset, n_samples=200, random_state=0)
print(f"Calculated sigma: {sigma}")
# Witness: Calculated sigma: 2.0
```

### LLM Instruction Prompt
- Always use `tslearn.metrics.sigma_gak` to compute the bandwidth before calling `gak` or `cdist_gak` to ensure kernel stability.
- If using `PyTorch` tensors as input, explicitly set `be="pytorch"` or ensure the input is already a tensor to trigger backend inference.
- Ensure `dataset` is formatted as a 3D array if working with multivariate data.

### Prompt Snippet
```text
from tslearn.metrics import sigma_gak
sigma = sigma_gak(dataset, n_samples=100, random_state=42)
```

### Common Failure Modes
- **ZeroDivisionError:** Occurs if the dataset contains identical time series or insufficient variance, leading to a median distance of zero.
- **Shape Mismatch:** Passing data that is not properly formatted as a `tslearn` time-series dataset (e.g., raw lists without conversion).
- **Backend Mismatch:** Attempting to use `be="pytorch"` without the `torch` package installed or when passing incompatible NumPy arrays.

### Fix Code Hint
```python
# Ensure data is 3D and use a fixed random_state for deterministic results
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(my_data)
sigma = tslearn.metrics.sigma_gak(X, n_samples=100, random_state=42)
```

## API Test: `silhouette_score`

### Signature
```python
def silhouette_score(X, labels, metric=None, sample_size=None, metric_params=None, n_jobs=None, verbose=0, random_state=None, **kwds)
```
_Source: tslearn/tslearn/clustering/utils.py:70_

_Source doc:_ Compute the mean Silhouette Coefficient of all samples (cf.  [1]_ and [2]_). Read more in the `scikit-learn documentation <http://scikit-learn.org/stable/modules/clustering.html\ #silhouette-coefficient>`_. Parameters ---------- X : array [n_ts, n_ts] if metric == "precomputed", or, \ [n_ts, sz, d] otherwise Array of pairwise distances between time series, or a time series dataset. labels : array, shape = [n_ts] Predicted labels for each time series. metric : string, callable or None (default: None) The metric to use when calculating distance between time series. Should be one of {'dtw', 'softdtw', 'euclidean'} or a callable distance function or None. If 'softdtw' is passed, a normalized version of Soft-DTW is used that is defined as `sdtw_(x,y) := sdtw(x,y) - 1/2(sdtw(x,x)+sdtw(y,y))`. If X is the distance array itself, use ``metric="precomputed"``. If None, dtw is used. sample_size : int or None (default: None) The size of the sample to use when computing the Silhouette Coefficient on a random subset of the data. If ``sample_size is None``, no sampling is used. metric_params : dict or None (default: None) Parameter values for the chosen metric. For metrics that accept parallelization of the cross-distance matrix computations, `n_jobs` key passed in `metric_params` is overridden by the `n_jobs` argument. n_jobs : int or None, optional (default=None) The number of jobs to run in parallel for cross-distance matrix computations. Ignored if the cross-distance matrix cannot be computed using parallelization. ``None`` means 1 unless in a :obj:`joblib.parallel_backend` context. ``-1`` means using all processors. See scikit-learns' `Glossary <https://scikit-learn.org/stable/glossary.html#term-n_jobs>`_ for more details. verbose : int (default: 0) If nonzero, print information about the inertia while learning the model and joblib progress messages are printed. random_state : int, RandomState instance or None, optional (default: None) The generator used to randomly select a subset of samples.  If int, random_state is the seed used by the random number generator; If RandomState instance, random_state is the random number generator; If None, the random number generator is the RandomState instance used by `np.random`. Used when ``sample_size is not None``. **kwds : optional keyword parameters Any further parameters are passed directly to the distance function, just as for the `metric_params` parameter. Returns ------- silhouette : float

### Goal
Computes the mean Silhouette Coefficient for a set of time series, evaluating the quality of clustering results based on intra-cluster cohesion and inter-cluster separation.

### Parameters
- `X`: Time series dataset of shape `(n_ts, sz, d)` or a precomputed distance matrix of shape `(n_ts, n_ts)`.
- `labels`: Array of shape `(n_ts,)` containing the cluster assignments for each time series.
- `metric`, default `None`: Distance metric to use; options include `'dtw'`, `'softdtw'`, `'euclidean'`, or a callable. Defaults to `'dtw'`.
- `sample_size`, default `None`: Number of samples to use for random subset computation; if `None`, uses the entire dataset.
- `metric_params`, default `None`: Dictionary of parameters passed to the chosen distance metric.
- `n_jobs`, default `None`: Number of parallel jobs for distance matrix computation.
- `verbose`, default `0`: Verbosity level for progress reporting.
- `random_state`, default `None`: Seed or `RandomState` instance for reproducible sampling when `sample_size` is set.
- `**kwds`: Additional keyword arguments passed directly to the distance function.

### Input
Requires a 3D NumPy array `(n_ts, sz, d)` for standard metrics. If `metric="precomputed"`, `X` must be a square 2D distance matrix `(n_ts, n_ts)`. Ensure data is formatted via `tslearn.utils.to_time_series_dataset` if input is variable-length or non-3D.

### Output
Returns `float`: The mean Silhouette Coefficient, ranging from -1 (incorrect clustering) to +1 (highly dense, well-separated clusters).

### Valid Call Patterns
```python
from tslearn.clustering import silhouette_score
from tslearn.metrics import dtw

# Standard usage with DTW
score = silhouette_score(X, labels, metric="dtw")

# Using a callable metric
score = silhouette_score(X, labels, metric=dtw)

# Using precomputed distances
score = silhouette_score(dist_matrix, labels, metric="precomputed")
```

### LLM Instruction Prompt
- Always verify that `X` is 3D unless `metric="precomputed"` is specified.
- When using `sample_size`, ensure `random_state` is provided for deterministic results.
- Prefer `metric="dtw"` or `metric="softdtw"` for time-series specific clustering evaluation.

### Prompt Snippet
```text
# Example: Compute silhouette score for clustering
from tslearn.clustering import silhouette_score
score = silhouette_score(X, labels, metric="dtw")
```

### Common Failure Modes
- Passing 2D arrays directly without using `to_time_series_dataset` when the data is not already in `(n_ts, sz, d)` format.
- Mismatch between the number of samples in `X` and the length of `labels`.
- Using `metric="precomputed"` while passing a 3D array instead of a 2D distance matrix.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset
# Ensure 3D shape (n_ts, sz, d)
X_formatted = to_time_series_dataset(X)
score = silhouette_score(X_formatted, labels, metric="dtw")
```

## API Test: `soft_dtw`

### Signature
```python
def soft_dtw(ts1, ts2, gamma=1.0, be=None, compute_with_backend=False)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:530_

_Source doc:_ Compute Soft-DTW metric between two time series. Soft-DTW was originally presented in [1]_ and is discussed in more details in our :ref:`user-guide page on DTW and its variants<dtw>`. Soft-DTW is computed as: .. math:: \text{soft-DTW}_{\gamma}(X, Y) = \min_{\pi}{}^\gamma \sum_{(i, j) \in \pi} \|X_i, Y_j\|^2 where :math:`\min^\gamma` is the soft-min operator of parameter :math:`\gamma`. In the limit case :math:`\gamma = 0`, :math:`\min^\gamma` reduces to a hard-min operator and soft-DTW is defined as the square of the DTW similarity measure. Parameters ---------- ts1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. ts2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. gamma : float (default 1.) Gamma parameter for Soft-DTW. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. compute_with_backend : bool, default=False This parameter has no influence when the NumPy backend is used. When a backend different from NumPy is used (cf parameter `be`): If `True`, the computation is done with the corresponding backend. If `False`, a conversion to the NumPy backend can be used to accelerate the computation. Returns ------- float Similarity Examples -------- >>> float(soft_dtw([1, 2, 2, 3], ...                [1., 2., 3., 4.], ...                gamma=1.))  # doctest: +NORMALIZE_WHITESPACE +ELLIPSIS -0.89... >>> float(soft_dtw([1, 2, 3, 3], ...                [1., 2., 2.1, 3.2], ...                 gamma=0.01))  # doctest: +NORMALIZE_WHITESPACE +ELLIPSIS 0.089... The PyTorch backend can be used to compute gradients:

### Goal
Computes the differentiable Soft-DTW similarity measure between two time series, providing a smooth approximation of the standard DTW distance.

### Parameters
- `ts1`: A time series of shape `(sz1, d)` or `(sz1,)`. Univariate series are assumed if shape is `(sz1,)`.
- `ts2`: A time series of shape `(sz2, d)` or `(sz2,)`. Univariate series are assumed if shape is `(sz2,)`.
- `gamma`, default `1.0`: The smoothing parameter for the soft-min operator. As `gamma` approaches 0, the result converges to the square of the standard DTW distance.
- `be`, default `None`: The backend engine to use. Can be a `NumPyBackend` instance, `PyTorchBackend` instance, or the strings `"numpy"` or `"pytorch"`. If `None`, the backend is inferred from the input arrays.
- `compute_with_backend`, default `False`: If using a non-NumPy backend, setting this to `True` forces computation within that backend; `False` may allow conversion to NumPy for performance.

### Input
Input time series must be array-like. While `tslearn` estimators typically expect 3D arrays `(n_ts, sz, d)`, this metric function accepts individual time series as 1D or 2D arrays.

### Output
Returns a `float` representing the Soft-DTW similarity value.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Basic usage with NumPy
ts1 = np.array([1.0, 2.0, 2.0, 3.0])
ts2 = np.array([1.0, 2.0, 3.0, 4.0])
sim = tslearn.metrics.soft_dtw(ts1, ts2, gamma=1.0)

# Explicit backend usage
sim_be = tslearn.metrics.soft_dtw(ts1, ts2, gamma=1.0, be="numpy", compute_with_backend=True)
```

### LLM Instruction Prompt
- Always prefer explicit backend specification if the user intends to use PyTorch for automatic differentiation.
- Ensure input arrays are formatted as 1D or 2D arrays; do not pass 3D datasets directly unless indexing into them.
- When `gamma` is set to a very small value, warn that the computation may become numerically unstable or behave like standard DTW.

### Prompt Snippet
```text
tslearn.metrics.soft_dtw(ts1, ts2, gamma=1.0, be=None, compute_with_backend=False)
```

### Common Failure Modes
- Passing 3D arrays `(n_ts, sz, d)` directly: The function expects individual time series `(sz, d)` or `(sz,)`.
- Backend mismatch: Attempting to use `PyTorch` features (like `requires_grad`) without passing `be="pytorch"` or providing PyTorch tensors.
- Numerical instability: Using extremely small `gamma` values can lead to overflow or precision issues in the soft-min calculation.

### Fix Code Hint
```python
# Ensure input is 2D for multivariate or 1D for univariate
ts1_clean = ts1.reshape(-1, 1) if ts1.ndim == 1 else ts1
ts2_clean = ts2.reshape(-1, 1) if ts2.ndim == 1 else ts2
sim = tslearn.metrics.soft_dtw(ts1_clean, ts2_clean, gamma=0.1)
```

## API Test: `soft_dtw_alignment`

### Signature
```python
def soft_dtw_alignment(ts1, ts2, gamma=1.0, be=None, compute_with_backend=False)
```
_Source: tslearn/tslearn/metrics/softdtw_variants.py:641_

_Source doc:_ Compute Soft-DTW metric between two time series and return both the similarity measure and the alignment matrix. Soft-DTW was originally presented in [1]_ and is discussed in more details in our :ref:`user-guide page on DTW and its variants<dtw>`. Soft-DTW is computed as: .. math:: \text{soft-DTW}_{\gamma}(X, Y) = \min_{\pi}{}^\gamma \sum_{(i, j) \in \pi} \|X_i, Y_j\|^2 where :math:`\min^\gamma` is the soft-min operator of parameter :math:`\gamma`. In the limit case :math:`\gamma = 0`, :math:`\min^\gamma` reduces to a hard-min operator and soft-DTW is defined as the square of the DTW similarity measure. Parameters ---------- ts1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. ts2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. gamma : float (default 1.) Gamma parameter for Soft-DTW. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. compute_with_backend : bool, default=False This parameter has no influence when the NumPy backend is used. When a backend different from NumPy is used (cf parameter `be`): If `True`, the computation is done with the corresponding backend. If `False`, a conversion to the NumPy backend can be used to accelerate the computation. Returns ------- array-like, shape=(sz1, sz2) Soft-alignment matrix float Similarity Examples -------- >>> a, dist = soft_dtw_alignment([1, 2, 2, 3], ...                              [1., 2., 3., 4.], ...                              gamma=1.)  # doctest: +ELLIPSIS >>> float(dist) -0.89... >>> a  # doctest: +ELLIPSIS +NORMALIZE_WHITESPACE array([[1.00...e+00, 1.88...e-01, 2.83...e-04, 4.19...e-11],

### Goal
Computes the Soft-DTW similarity measure and the corresponding soft-alignment matrix between two time series, providing a differentiable alternative to standard DTW.

### Parameters
- `ts1`: A time series of shape `(sz1, d)` or `(sz1,)`. If univariate, shape is `(sz1,)`.
- `ts2`: A time series of shape `(sz2, d)` or `(sz2,)`. If univariate, shape is `(sz2,)`.
- `gamma`, default `1.0`: The smoothing parameter for the soft-min operator. As `gamma` approaches 0, the result converges to the square of the standard DTW distance.
- `be`, default `None`: The backend engine to use. Can be `None`, a string (`"numpy"`, `"pytorch"`), or a backend object. Determines the computation engine and device compatibility.
- `compute_with_backend`, default `False`: If `True` and using a non-NumPy backend, forces computation within that backend; if `False`, may convert to NumPy for performance.

### Input
Input time series must be array-like. While `tslearn` estimators typically expect 3D arrays `(n_ts, sz, d)`, this metric function accepts individual time series as 1D or 2D arrays. Ensure inputs are compatible with the selected backend (e.g., `torch.Tensor` for `PyTorchBackend`).

### Output
Returns a tuple containing:
- `alignment_matrix`: An array of shape `(sz1, sz2)` representing the soft-alignment.
- `similarity`: A float representing the Soft-DTW similarity value.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

# Standard usage with NumPy
a, dist = tslearn.metrics.soft_dtw_alignment([1, 2, 2, 3], [1., 2., 3., 4.], gamma=1.)

# Usage with explicit backend
s1 = np.random.rand(10, 2)
s2 = np.random.rand(30, 2)
matrix_path, dist = tslearn.metrics.soft_dtw_alignment(s1, s2, gamma=0.0, be="numpy")
```

### LLM Instruction Prompt
- Always specify `gamma` when the default `1.0` is not desired.
- When using `PyTorch`, ensure `be="pytorch"` is passed to enable automatic differentiation.
- If the input arrays are already tensors, `be` can be left as `None` to auto-detect the backend.

### Prompt Snippet
```text
# Example of computing Soft-DTW alignment
from tslearn.metrics import soft_dtw_alignment
alignment, similarity = soft_dtw_alignment(ts1, ts2, gamma=0.5)
```

### Common Failure Modes
- **Shape Mismatch:** Providing arrays with incompatible dimensions (e.g., different feature counts `d` in `(sz1, d)` and `(sz2, d)`).
- **Backend Mismatch:** Attempting to use `PyTorch` features (like `requires_grad`) without passing `be="pytorch"` or providing `torch.Tensor` inputs.
- **Gamma Sensitivity:** Setting `gamma` too high can lead to numerical instability or loss of alignment structure; setting `gamma=0` requires careful handling as it becomes the hard-min DTW square.

### Fix Code Hint
```python
# Ensure inputs are 2D if multivariate
ts1 = np.array(ts1).reshape(-1, 1) if len(np.array(ts1).shape) == 1 else ts1
ts2 = np.array(ts2).reshape(-1, 1) if len(np.array(ts2).shape) == 1 else ts2
alignment, similarity = soft_dtw_alignment(ts1, ts2, gamma=1.0)
```

## API Test: `softdtw_barycenter`

### Signature
```python
def softdtw_barycenter(X, gamma=1.0, weights=None, method='L-BFGS-B', tol=0.001, max_iter=50, init=None)
```
_Source: tslearn/tslearn/barycenters/softdtw.py:38_

_Source doc:_ Compute barycenter (time series averaging) under the soft-DTW geometry. Soft-DTW was originally presented in [1]_. Parameters ---------- X : array-like, shape=(n_ts, sz, d) Time series dataset. gamma: float Regularization parameter. Lower is less smoothed (closer to true DTW). weights: None or array Weights of each X[i]. Must be the same size as len(X). If None, uniform weights are used. method: string Optimization method, passed to `scipy.optimize.minimize`. Default: L-BFGS. tol: float Tolerance of the method used. max_iter: int Maximum number of iterations. init: array or None (default: None) Initial barycenter to start from for the optimization process. If `None`, euclidean barycenter is used as a starting point. Returns ------- numpy.array of shape (bsz, d) where `bsz` is the size of the `init` array \ if provided or `sz` otherwise Soft-DTW barycenter of the provided time series dataset. Examples -------- >>> time_series = [[1, 2, 3, 4], [1, 2, 4, 5]] >>> softdtw_barycenter(time_series, max_iter=5) array([[1.25161574], [2.03821705], [3.5101956 ], [4.36140605]]) >>> time_series = [[1, 2, 3, 4], [1, 2, 3, 4, 5]] >>> softdtw_barycenter(time_series, max_iter=5) array([[1.21349933], [1.8932251 ], [2.67573269], [3.51057026], [4.33645802]]) References ---------- .. [1] M. Cuturi, M. Blondel "Soft-DTW: a Differentiable Loss Function for Time-Series," ICML 2017.

### Goal
Computes a representative time-series average (barycenter) for a dataset by minimizing the soft-DTW loss, providing a differentiable and smoothed alternative to standard DTW barycenters.

### Parameters
- `X`: Time series dataset of shape `(n_ts, sz, d)`.
- `gamma`, default `1.0`: Regularization parameter; smaller values result in less smoothing and behavior closer to standard DTW.
- `weights`, default `None`: Optional array of weights for each time series in `X`. Must have length `n_ts`. If `None`, uniform weights are applied.
- `method`, default `'L-BFGS-B'`: Optimization algorithm passed to `scipy.optimize.minimize`.
- `tol`, default `0.001`: Convergence tolerance for the optimization process.
- `max_iter`, default `50`: Maximum number of iterations for the optimization.
- `init`, default `None`: Initial barycenter array. If `None`, the Euclidean barycenter is used as the starting point.

### Input
- **Format:** 3D `numpy` array of shape `(n_ts, sz, d)`.
- **Preconditions:** If input data is not in the required 3D format, use `tslearn.utils.to_time_series_dataset` to convert it before passing to the function.

### Output
Returns a `numpy.array` of shape `(bsz, d)`, representing the computed soft-DTW barycenter, where `bsz` is the length of the `init` array (if provided) or the length of the input time series `sz`.

### Valid Call Patterns
```python
import numpy as np
import tslearn.barycenters
# Standard usage
time_series = np.random.randn(15, 10, 3)
barycenter = tslearn.barycenters.softdtw_barycenter(time_series, max_iter=5)
print(f"Barycenter shape: {barycenter.shape}")
```

### LLM Instruction Prompt
- Always import `tslearn.barycenters` to access the function.
- Ensure input `X` is a 3D array; if provided as a list of lists, wrap it in `tslearn.utils.to_time_series_dataset`.
- When setting `max_iter=0`, the function effectively computes the Euclidean barycenter.

### Prompt Snippet
```text
from tslearn.barycenters import softdtw_barycenter
# Ensure X is (n_ts, sz, d)
result = softdtw_barycenter(X, gamma=1.0, max_iter=50)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays without converting them to 3D `(n_ts, sz, d)` format.
- **Weight Mismatch:** Providing a `weights` array that does not match the number of time series in `X`.
- **Convergence:** Setting `max_iter` too low for complex datasets, leading to an under-optimized barycenter.

### Fix Code Hint
```python
import tslearn.utils
# Convert 2D list to 3D dataset if necessary
X_3d = tslearn.utils.to_time_series_dataset(X_list)
barycenter = tslearn.barycenters.softdtw_barycenter(X_3d, max_iter=100)
```

## API Test: `sqrt`

### Signature
```python
def sqrt(self, x, out=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:207_

### Goal
Computes the element-wise square root of the input array or tensor using the configured backend engine.

### Parameters
- `self`: The backend instance (e.g., `NumPyBackend` or `PyTorchBackend`) used to perform the operation.
- `x`: The input array or tensor for which to compute the square root.
- `out`, default `None`: Optional output array or tensor to store the result.

### Input
- **Data:** Accepts `numpy.ndarray` or `torch.Tensor` depending on the backend instance.
- **Preconditions:** The input `x` must contain non-negative values. The backend instance must be initialized via `tslearn.backend.instantiate_backend` to match the input type.

### Output
Returns an array or tensor of the same shape as `x` containing the square root of each element.

### Valid Call Patterns
```python
from tslearn.backend import instantiate_backend
import numpy as np

# Example using the NumPy backend
be = instantiate_backend("numpy")
x = np.array([4.0, 9.0, 16.0])
result = be.sqrt(x)
print(f"Correctness witness: {result}")
```

### LLM Instruction Prompt
- Always use the `sqrt` method via an instantiated backend object (`be.sqrt(...)`) rather than calling it as a standalone function.
- Ensure the input `x` is compatible with the backend (e.g., `numpy` array for `NumPyBackend`, `torch.Tensor` for `PyTorchBackend`).
- Do not pass negative values to `x` as this will result in `NaN` or runtime errors depending on the backend.

### Prompt Snippet
```text
# Use the backend instance to compute square root
result = backend.sqrt(data)
```

### Common Failure Modes
- **Backend Mismatch:** Passing a `torch.Tensor` to a `NumPyBackend` instance or vice versa.
- **Negative Input:** Passing negative values in `x` will result in `NaN` values in the output.
- **Uninitialized Backend:** Attempting to call `sqrt` on a backend object that has not been properly instantiated via `instantiate_backend`.

### Fix Code Hint
```python
# Ensure backend matches input type
from tslearn.backend import instantiate_backend
be = instantiate_backend(x) # Automatically selects backend based on input type
result = be.sqrt(x)
```

## API Test: `str_to_time_series`

### Signature
```python
def str_to_time_series(ts_str)
```
_Source: tslearn/tslearn/utils/utils.py:326_

_Source doc:_ Reads a time series from its string representation (used when loading time series from disk). Parameters ---------- ts_str : string String representation of the time-series. Returns ------- numpy.ndarray Represented time-series. Examples -------- >>> str_to_time_series("1 2 3 4") array([[1.], [2.], [3.], [4.]]) >>> str_to_time_series("1 2|3 4") array([[1., 3.], [2., 4.]]) See Also -------- load_time_series_txt : Load time series from disk time_series_to_str : Transform a time series into a string

### Goal
Converts a string-encoded time series representation into a 2D `numpy.ndarray` suitable for `tslearn` processing.

### Parameters
- `ts_str`: A string where values are separated by spaces (for time steps) and the pipe character `|` is used to separate dimensions.

### Input
- **Format:** A string representing numerical values.
- **Preconditions:** The string must follow the `tslearn` serialization format where space-delimited values represent consecutive time points, and `|` delimiters denote distinct dimensions.

### Output
Returns `numpy.ndarray` — A 2D array of shape `(n_timestamps, n_features)` containing the parsed time-series data.

### Valid Call Patterns
```python
from tslearn.utils import str_to_time_series
import numpy as np

# Single dimension example
ts1 = str_to_time_series("1 2 3 4")
assert ts1.shape == (4, 1)

# Multi-dimension example
ts2 = str_to_time_series("1 2|3 4")
assert ts2.shape == (2, 2)
print(f"Correctness witness: {ts2.tolist()}")
```

### LLM Instruction Prompt
- Use `str_to_time_series` only when parsing raw string data retrieved from disk or legacy text formats.
- Ensure the input string uses `|` to separate dimensions if the time series is multivariate.
- Always verify the resulting array shape matches the expected `(n_timestamps, n_features)` before passing it to `to_time_series_dataset`.

### Prompt Snippet
```text
from tslearn.utils import str_to_time_series
# Parse string to array
ts_array = str_to_time_series("val1 val2|val3 val4")
```

### Common Failure Modes
- **Malformed Strings:** Passing strings with non-numeric characters or inconsistent dimension separators will raise a `ValueError` or `IndexError` during parsing.
- **Dimension Mismatch:** If the number of values between `|` separators is inconsistent, the resulting array may not be constructible as a standard `numpy` array.
- **Input Type:** Passing a list or a non-string object will cause an error; ensure the input is explicitly a string.

### Fix Code Hint
```python
# Ensure input is a string and handle potential parsing errors
try:
    ts = str_to_time_series(str(raw_data))
except ValueError as e:
    print(f"Failed to parse time series string: {e}")
```

## API Test: `subsequence_cost_matrix`

### Signature
```python
def subsequence_cost_matrix(subseq, longseq, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1236_

_Source doc:_ Compute the accumulated cost matrix score between a subsequence and a reference time series. Parameters ---------- subseq : array-like, shape=(sz1, d) or (sz1,) Subsequence time series. If shape is (sz1,), the time series is assumed to be univariate. longseq : array-like, shape=(sz2, d) or (sz2,) Reference time series. If shape is (sz2,), the time series is assumed to be univariate. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- mat : array-like, shape=(sz1, sz2) Accumulated cost matrix.

### Goal
Computes the accumulated cost matrix for subsequence dynamic time warping, identifying the optimal alignment cost between a shorter query sequence and a longer reference time series.

### Parameters
- `subseq`: The query time series of shape `(sz1, d)` or `(sz1,)`.
- `longseq`: The reference time series of shape `(sz2, d)` or `(sz2,)`.
- `be`, default `None`: The computation backend; accepts `"numpy"`, `"pytorch"`, or a backend instance. If `None`, the backend is inferred from the input array types.

### Input
- **Data Format:** Accepts array-like structures (NumPy arrays or PyTorch tensors).
- **Preconditions:** Inputs should be formatted as time series. If inputs are 1D or 2D, they are treated as univariate; for multivariate data, ensure the last dimension `d` matches. Use `tslearn.utils.to_time_series` to ensure compatibility.
- **Backend:** If using `PyTorch`, the `pytorch` package must be installed.

### Output
Returns `mat`: An accumulated cost matrix of shape `(sz1, sz2)` representing the cumulative alignment costs.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np
from tslearn.utils import to_time_series

subseq = to_time_series(np.array([1, 4]))
longseq = to_time_series(np.array([1.0, 2.0, 2.0, 3.0, 4.0]))

# Standard call with inferred backend
cost_matrix = tslearn.metrics.subsequence_cost_matrix(subseq, longseq)
print(f"Correctness witness: Matrix shape {cost_matrix.shape}")
```

### LLM Instruction Prompt
- Always ensure `subseq` and `longseq` are converted to `tslearn` compatible time-series format using `to_time_series` before passing to the function.
- If performing gradient-based optimization, explicitly set `be="pytorch"` and ensure inputs are `torch.Tensor` objects with `requires_grad=True`.

### Prompt Snippet
```text
tslearn.metrics.subsequence_cost_matrix(subseq, longseq, be=...)
```

### Common Failure Modes
- **Shape Mismatch:** Providing arrays where the feature dimension `d` does not match between `subseq` and `longseq`.
- **Backend Incompatibility:** Passing a PyTorch tensor while the `NumPyBackend` is explicitly requested, or vice versa.
- **Dimension Errors:** Passing raw 1D lists without converting them to the expected `(n_ts, sz, d)` or `(sz, d)` format required by `tslearn` metrics.

### Fix Code Hint
```python
# Ensure inputs are properly shaped for tslearn metrics
from tslearn.utils import to_time_series
subseq = to_time_series(subseq)
longseq = to_time_series(longseq)
cost_matrix = tslearn.metrics.subsequence_cost_matrix(subseq, longseq)
```

## API Test: `subsequence_path`

### Signature
```python
def subsequence_path(acc_cost_mat, idx_path_end, be=None)
```
_Source: tslearn/tslearn/metrics/dtw_variants.py:1367_

_Source doc:_ Compute the optimal path through an accumulated cost matrix given the endpoint of the sequence. Parameters ---------- acc_cost_mat: array-like, shape=(sz1, sz2) Accumulated cost matrix comparing subsequence from a longer sequence. idx_path_end: int The end position of the matched subsequence in the longer sequence. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- path: list of tuples of integer pairs Matching path represented as a list of index pairs. In each pair, the first index corresponds to `subseq` and the second one corresponds to `longseq`. The startpoint of the Path is :math:`P_0 = (0, ?)` and it ends at :math:`P_L = (len(subseq)-1, idx\_path\_end)` Examples -------- >>> acc_cost_mat = numpy.array([[1., 0., 0., 1., 4.], ...                             [5., 1., 1., 0., 1.]]) >>> # calculate the globally optimal path >>> optimal_end_point = numpy.argmin(acc_cost_mat[-1, :]) >>> path = subsequence_path(acc_cost_mat, optimal_end_point) >>> path [(0, 2), (1, 3)] See Also -------- dtw_subsequence_path : Get the similarity score for DTW subsequence_cost_matrix: Calculate the required cost matrix

### Goal
Computes the optimal alignment path through an accumulated cost matrix for a subsequence matching task, identifying the sequence of index pairs that minimize the total cost from the start of the subsequence to a specified endpoint in the longer sequence.

### Parameters
- `acc_cost_mat`: array-like, shape=(sz1, sz2). The accumulated cost matrix computed via `subsequence_cost_matrix`.
- `idx_path_end`: int. The column index in the longer sequence where the optimal path terminates.
- `be`, default `None`: Backend object, string ("numpy" or "pytorch"), or None. Determines the computational engine used for path backtracking.

### Input
The caller must provide an accumulated cost matrix (typically generated by `tslearn.metrics.subsequence_cost_matrix`) and a valid integer index representing the endpoint in the longer sequence. If using `PyTorch` tensors, ensure the `be` parameter is explicitly set to `"pytorch"` or a `PyTorchBackend` instance to maintain consistency.

### Output
Returns a `list` of `tuples` of integer pairs `(i, j)`, where `i` is the index in the subsequence and `j` is the index in the longer sequence.

### Valid Call Patterns
```python
import tslearn.metrics
import numpy as np

acc_cost_mat = np.array([[1., 0., 0., 1., 4.], [5., 1., 1., 0., 1.]])
optimal_end_point = np.argmin(acc_cost_mat[-1, :])
path = tslearn.metrics.subsequence_path(acc_cost_mat, optimal_end_point)
print(path)
# Witness: [(0, 2), (1, 3)]
```

### LLM Instruction Prompt
- Always use `tslearn.metrics.subsequence_path` to retrieve the alignment path.
- Ensure `acc_cost_mat` is the result of a prior `subsequence_cost_matrix` call.
- If the input arrays are `torch.Tensor` objects, explicitly pass `be="pytorch"` to the function.

### Prompt Snippet
```text
# Calculate path for subsequence alignment
path = tslearn.metrics.subsequence_path(acc_cost_mat, end_idx, be=backend)
```

### Common Failure Modes
- Passing an `idx_path_end` that is out of bounds for the `acc_cost_mat` columns.
- Mismatch between the backend of the input `acc_cost_mat` and the `be` parameter (e.g., passing a PyTorch tensor without specifying the PyTorch backend).
- Providing a standard cost matrix instead of an accumulated cost matrix.

### Fix Code Hint
```python
# Ensure the matrix is accumulated before calling
from tslearn.metrics import subsequence_cost_matrix, subsequence_path
cost_mat = subsequence_cost_matrix(subseq, longseq)
path = subsequence_path(cost_mat, idx_path_end=3)
```

## API Test: `support_`

### Signature
```python
def support_(self)
```
_Source: tslearn/tslearn/svm/svm.py:24_

### Goal
Retrieves the support vectors from a fitted `TimeSeriesSVC` or `LearningShapelets` estimator.

### Parameters
- `self`: An instance of a `tslearn` estimator (e.g., `TimeSeriesSVC` or `LearningShapelets`) that has already been fitted to training data.

### Input
- **Preconditions:** The estimator must have been successfully trained using the `fit()` method.
- **Data Format:** The underlying model must contain support vectors, which are typically stored as a 3D NumPy array of shape `(n_support_vectors, max_sz, d)`.

### Output
Returns `unspecified` — The support vectors identified during the training process, typically returned as a NumPy array.

### Valid Call Patterns
```python
# Inferred from signature: usage on a fitted estimator
from tslearn.svm import TimeSeriesSVC
clf = TimeSeriesSVC().fit(X_train, y_train)
support_vectors = clf.support_
print(f"Number of support vectors: {len(support_vectors)}")
```

### LLM Instruction Prompt
- Always ensure the estimator instance has been fitted via `fit()` before accessing `support_`.
- Treat the returned value as a read-only attribute representing the subset of training data that defines the decision boundary.

### Prompt Snippet
```text
# Accessing support vectors from a fitted tslearn estimator
support_vectors = estimator.support_
```

### Common Failure Modes
- **AttributeError:** Occurs if `support_` is accessed before calling `fit()` on the estimator.
- **Type Mismatch:** Attempting to access `support_` on an estimator class that does not implement support vector logic (e.g., `TimeSeriesKMeans`).

### Fix Code Hint
```python
# Ensure the model is fitted first
if hasattr(estimator, "support_"):
    sv = estimator.support_
else:
    raise ValueError("Estimator must be fitted and support support_ vectors.")
```

## API Test: `support_vectors_`

### Signature
```python
def support_vectors_(self)
```
_Source: tslearn/tslearn/svm/svm.py:285  (+1 more definition site/overload)_

### Goal
Retrieves the support vectors identified during the training of a time-series Support Vector Machine (SVM) estimator.

### Parameters
- `self`: An instance of a fitted `tslearn` SVM estimator (e.g., `TimeSeriesSVC` or `LearningShapelets`).

### Input
- **Preconditions:** The estimator must have been successfully trained using the `fit()` method on a 3D NumPy array of shape `(n_ts, max_sz, d)`.
- **Data Format:** The input data must be formatted as a 3D time-series dataset, typically prepared via `tslearn.utils.to_time_series_dataset`.

### Output
Returns `unspecified` — The support vectors used by the model, typically returned as a 3D NumPy array of shape `(n_support_vectors, max_sz, d)`.

### Valid Call Patterns
```python
# Inferred from signature:
# Assuming 'clf' is a fitted tslearn SVM estimator
support_vectors = clf.support_vectors_
```

### LLM Instruction Prompt
- When calling `support_vectors_`, ensure the estimator instance has already been fitted.
- Access this as an attribute (property) rather than a method call, as it is a post-fit attribute in `scikit-learn` style estimators.
- Verify that the estimator is an SVM-based model, as this attribute is not present in all `tslearn` estimators.

### Prompt Snippet
```text
# Accessing support vectors from a fitted TimeSeriesSVC
sv = model.support_vectors_
```

### Common Failure Modes
- **AttributeError:** Occurs if `support_vectors_` is accessed before calling `fit()` on the estimator.
- **TypeError:** Occurs if the estimator instance does not support SVM-based attributes (e.g., calling this on a `TimeSeriesKMeans` object).
- **Data Shape Mismatch:** If the underlying model was trained on variable-length data, the returned support vectors may contain padding values to maintain the 3D array structure.

### Fix Code Hint
```python
from tslearn.svm import TimeSeriesSVC
# Ensure the model is fitted before accessing the attribute
clf = TimeSeriesSVC().fit(X_train, y_train)
if hasattr(clf, "support_vectors_"):
    sv = clf.support_vectors_
```

## API Test: `time_series_to_str`

### Signature
```python
def time_series_to_str(ts, fmt='%.18e')
```
_Source: tslearn/tslearn/utils/utils.py:289_

_Source doc:_ Transforms a time series to its representation as a string (used when saving time series to disk). Parameters ---------- ts : array-like Time series to be represented. fmt : string (default: "%.18e") Format to be used to write each value (only ASCII characters). Returns ------- string String representation of the time-series. Examples -------- >>> time_series_to_str([1, 2, 3, 4], fmt="%.1f") '1.0 2.0 3.0 4.0' >>> time_series_to_str([[1, 3], [2, 4]], fmt="%.1f") '1.0 2.0|3.0 4.0' See Also -------- load_time_series_txt : Load time series from disk str_to_time_series : Transform a string into a time series

### Goal
Serializes a time-series array into a formatted ASCII string representation suitable for storage in text-based files.

### Parameters
- `ts`: array-like; The time-series data to be serialized. Can be a 1D sequence or a 2D array (where rows represent time steps and columns represent dimensions).
- `fmt`, default `'%.18e'`: string; The format specifier used to convert numerical values to strings, following standard Python string formatting conventions.

### Input
- **Data:** Accepts list-like or NumPy array-like structures.
- **Preconditions:** The input should represent a single time series. If working with a dataset of multiple time series, this function must be called iteratively on individual samples.

### Output
Returns `string` — A string representation where values are space-separated within a time step, and time steps are separated by the pipe character (`|`) if the series is multi-dimensional or multi-step.

### Valid Call Patterns
```python
from tslearn.utils import time_series_to_str

# Serialize a 1D time series
s1 = [1, 2, 3, 4]
print(time_series_to_str(s1, fmt="%.1f"))

# Serialize a 2D time series (time steps x dimensions)
s2 = [[1, 3], [2, 4]]
print(time_series_to_str(s2, fmt="%.1f"))
```

### LLM Instruction Prompt
- Use `tslearn.utils.time_series_to_str` to convert time-series data into a string format for disk persistence.
- Always specify the `fmt` parameter if the default scientific notation is not desired.
- Ensure the input `ts` is a single time series (1D or 2D array), not a 3D dataset array.

### Prompt Snippet
```text
from tslearn.utils import time_series_to_str
# Example:
# serialized = time_series_to_str(my_ts, fmt="%.4f")
```

### Common Failure Modes
- **Passing 3D arrays:** Passing a full dataset (shape `(n_ts, sz, d)`) directly will result in an incorrectly formatted string; iterate over the dataset instead.
- **Invalid Format Strings:** Providing a `fmt` string incompatible with float formatting will raise a `ValueError` during string conversion.

### Fix Code Hint
```python
# If passing a 3D dataset, iterate over it:
from tslearn.utils import time_series_to_str
serialized_data = [time_series_to_str(ts, fmt="%.2f") for ts in X]
```

## API Test: `to_cesium_dataset`

### Signature
```python
def to_cesium_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:655_

_Source doc:_ Transform a tslearn-compatible dataset into a cesium dataset. Parameters ---------- X: array, shape = (n_ts, sz, d), where n_ts=1 tslearn-formatted dataset to be cast to cesium format Returns ------- list of cesium TimeSeries cesium-formatted dataset (cf. `link <http://cesium-ml.org/docs/api/cesium.time_series.html#cesium.time_series.TimeSeries>`_) Examples -------- >>> tslearn_arr = numpy.random.randn(3, 16, 1) >>> cesium_ds = to_cesium_dataset(tslearn_arr) >>> len(cesium_ds) 3 >>> cesium_ds[0].measurement.shape (16,) >>> tslearn_arr = numpy.random.randn(3, 16, 2) >>> cesium_ds = to_cesium_dataset(tslearn_arr) >>> len(cesium_ds) 3 >>> cesium_ds[0].measurement.shape (2, 16) >>> tslearn_arr = [[1, 2, 3], [1, 2, 3, 4]] >>> cesium_ds = to_cesium_dataset(tslearn_arr) >>> len(cesium_ds) 2 >>> cesium_ds[0].measurement.shape (3,) Notes ----- Conversion from/to cesium format requires cesium to be installed.

### Goal
Converts a `tslearn`-formatted time-series dataset into the `cesium` library's `TimeSeries` object format for interoperability with the Cesium machine learning platform.

### Parameters
- `X`: A `tslearn`-compatible dataset, typically a 3D NumPy array of shape `(n_ts, sz, d)` or a list of variable-length time series.

### Input
- **Data Format:** Must be a 3D array `(n_ts, sz, d)` or a list of sequences.
- **Preconditions:** The `cesium` Python package must be installed in the environment. If the input is not already in `tslearn` format, it is recommended to pass it through `tslearn.utils.to_time_series_dataset` first.

### Output
Returns a `list` of `cesium.time_series.TimeSeries` objects representing the dataset.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_cesium_dataset

# Standard 3D array conversion
tslearn_dataset = np.random.randn(15, 10, 3)
cesium_ds = to_cesium_dataset(tslearn_dataset)
```

### LLM Instruction Prompt
- Always verify that the `cesium` library is available before suggesting this function.
- Ensure the input `X` is a valid `tslearn` dataset (3D array or list of sequences).
- Use this function only when the user explicitly requires interoperability with the Cesium ML ecosystem.

### Prompt Snippet
```text
from tslearn.utils import to_cesium_dataset
# Ensure cesium is installed; X is a tslearn-formatted dataset
cesium_data = to_cesium_dataset(X)
```

### Common Failure Modes
- **Missing Dependency:** Raises an `ImportError` or `ModuleNotFoundError` if the `cesium` package is not installed in the current Python environment.
- **Invalid Input Shape:** Passing data that does not conform to `tslearn`'s expected 3D structure or list-of-sequences format may result in unexpected behavior or shape errors during the internal conversion process.

### Fix Code Hint
```python
# Ensure dependency is present
try:
    import cesium
    cesium_ds = to_cesium_dataset(X)
except ImportError:
    print("The 'cesium' package is required for this conversion.")
```

## API Test: `to_hdf5`

### Signature
```python
def to_hdf5(self, path)
```
_Source: tslearn/tslearn/shapelets/shapelets.py:986  (+1 more definition site/overload)_

_Source doc:_ Save model to a HDF5 file. Requires ``h5py`` http://docs.h5py.org/ Parameters ---------- path : str Full file path. File must not already exist. Raises ------ FileExistsError If a file with the same path already exists.

### Goal
Serializes a trained `tslearn` estimator to a HDF5 file format for persistent storage and later retrieval.

### Parameters
- `self`: The trained `tslearn` estimator instance (e.g., `TimeSeriesKMeans`, `KNeighborsTimeSeriesClassifier`).
- `path`: A string representing the full file path where the model should be saved.

### Input
- **Preconditions:** The estimator must be fitted (trained) before calling `to_hdf5`.
- **Dependencies:** The `h5py` library must be installed in the environment.
- **File System:** The specified `path` must not point to an existing file.

### Output
Returns `None`. The method performs an in-place file system operation to write the model state.

### Valid Call Patterns
```python
# Inferred from signature: Saving a trained estimator
from tslearn.clustering import TimeSeriesKMeans
model = TimeSeriesKMeans(n_clusters=3)
model.fit(X)
model.to_hdf5("my_model.h5")
```

### LLM Instruction Prompt
- Use `to_hdf5` only on `tslearn` estimators that support serialization.
- Ensure the `path` provided does not exist to avoid `FileExistsError`.
- Always verify that the estimator has been trained via `fit()` before attempting to save.

### Prompt Snippet
```text
# Ensure the model is trained before saving
model.fit(X)
model.to_hdf5("path/to/model.h5")
```

### Common Failure Modes
- **FileExistsError:** Occurs if the provided `path` already exists on the disk.
- **ImportError:** Occurs if the `h5py` package is not installed in the Python environment.
- **NotFittedError:** Occurs if the estimator is saved before calling `fit()`.
- **Incompatibility:** Not all `tslearn` objects are serializable; for example, `LearningShapelets` is explicitly noted as not HDF5 serializable.

### Fix Code Hint
```python
import os
import h5py

# Check for existence before saving
if not os.path.exists("model.h5"):
    model.to_hdf5("model.h5")
else:
    print("File already exists, choose a different path.")
```

## API Test: `to_json`

### Signature
```python
def to_json(self, path)
```
_Source: tslearn/tslearn/bases/bases.py:259_

_Source doc:_ Save model to a JSON file. Parameters ---------- path : str Full file path.

### Goal
Serializes a trained `tslearn` estimator to a JSON-formatted file for persistence and later retrieval.

### Parameters
- `self`: The trained `tslearn` estimator instance (e.g., `TimeSeriesKMeans`, `KNeighborsTimeSeriesClassifier`).
- `path`: A `str` representing the full file path where the JSON representation of the model will be saved.

### Input
- **Preconditions:** The estimator must have been fitted (i.e., `fit()` has been called) prior to calling `to_json` to ensure the model parameters are populated.
- **Format:** The `path` must be a valid string pointing to a writable location on the filesystem.

### Output
Returns `None`. The method performs a side-effect by writing the model state to the specified file path.

### Valid Call Patterns
```python
# Inferred from signature: Saving a trained estimator
from tslearn.clustering import TimeSeriesKMeans
model = TimeSeriesKMeans(n_clusters=3)
model.fit(X)
model.to_json("my_model.json")
```

### LLM Instruction Prompt
- Use `to_json` only on `tslearn` estimator instances that have already been trained via the `fit` method.
- Ensure the `path` argument is a string representing a valid file path.
- Note that this method is intended for model serialization; verify that the target environment has write permissions to the specified path.

### Prompt Snippet
```text
# Example of saving a model
estimator.to_json(path="model_checkpoint.json")
```

### Common Failure Modes
- **Unfitted Estimator:** Calling `to_json` on an estimator that has not been fitted will result in an incomplete or empty JSON file, as the model parameters are not yet initialized.
- **Permission Denied:** Providing a `path` to a directory where the user lacks write permissions will raise an `OSError` or `PermissionError`.
- **Invalid Path:** Providing a path with non-existent parent directories will cause the operation to fail.

### Fix Code Hint
```python
import os
# Ensure directory exists before saving
os.makedirs(os.path.dirname(path), exist_ok=True)
model.to_json(path)
```

## API Test: `to_numpy`

### Signature
```python
def to_numpy(x)
```
_Source: tslearn/tslearn/backend/numpy_backend.py:122  (+1 more definition site/overload)_

### Goal
Converts the input time-series data structure into a standard NumPy array representation to ensure compatibility with `tslearn`'s NumPy-based backend operations.

### Parameters
- `x`: The input data structure (e.g., list, array-like, or tensor) containing time-series data to be converted.

### Input
- **Data Format:** Accepts standard Python lists, NumPy arrays, or other array-like structures.
- **Preconditions:** The input should represent time-series data. If the data is intended for use in `tslearn` estimators, it should ideally be formatted as a 3D array `(n_ts, max_sz, d)` prior to or following conversion, typically using `to_time_series_dataset` if the input is a collection of variable-length sequences.

### Output
Returns `unspecified` — A NumPy array containing the data from `x`.

### Valid Call Patterns
```python
from tslearn.backend.numpy_backend import to_numpy
import numpy as np

# Example: Converting a list to a numpy array
data = [1, 2, 3, 4]
result = to_numpy(data)
print(f"Correctness witness: {type(result)}")
```

### LLM Instruction Prompt
- Use `to_numpy` when you need to ensure that a data structure is compatible with NumPy-based backend functions in `tslearn`.
- Always import `to_numpy` explicitly from `tslearn.backend.numpy_backend`.
- Do not assume the output is a specific shape; verify the dimensions if the downstream estimator requires a 3D array.

### Prompt Snippet
```text
from tslearn.backend.numpy_backend import to_numpy
# Ensure data is in numpy format before metric computation
X_np = to_numpy(X)
```

### Common Failure Modes
- **Type Mismatch:** Passing objects that cannot be coerced into a NumPy array (e.g., complex nested objects or non-numeric types).
- **Dimension Mismatch:** Passing data that, once converted to NumPy, does not meet the 3D `(n_ts, max_sz, d)` requirement expected by `tslearn` estimators.
- **Backend Conflict:** Attempting to use `to_numpy` on a `torch.Tensor` that is currently attached to a computation graph; this may detach the tensor or cause errors depending on the environment.

### Fix Code Hint
```python
# Ensure input is 3D before passing to estimators
from tslearn.utils import to_time_series_dataset
from tslearn.backend.numpy_backend import to_numpy

X_raw = [[1, 2], [3, 4, 5]]
X_formatted = to_time_series_dataset(X_raw)
X_final = to_numpy(X_formatted)
```

## API Test: `to_pickle`

### Signature
```python
def to_pickle(self, path)
```
_Source: tslearn/tslearn/bases/bases.py:307_

_Source doc:_ Save model to a pickle file. Parameters ---------- path : str Full file path.

### Goal
Persists a trained `tslearn` estimator to disk using Python's `pickle` serialization format for later retrieval.

### Parameters
- `self`: The `tslearn` estimator instance (e.g., `TimeSeriesKMeans`, `KNeighborsTimeSeriesClassifier`) that has already been fitted to data.
- `path`: A `str` representing the full file path where the model should be saved.

### Input
- **Preconditions:** The estimator must be in a valid state (typically after calling `.fit()`).
- **Format:** The `path` must be a valid string representing a writable file location on the local filesystem.

### Output
Returns `None`. The method performs a side-effect by writing the serialized object to the specified file path.

### Valid Call Patterns
```python
from tslearn.clustering import TimeSeriesKMeans
import tempfile

# Inferred from signature:
model = TimeSeriesKMeans(n_clusters=2)
# ... fit model ...
model.to_pickle("my_model.pkl")
```

### LLM Instruction Prompt
- Use `to_pickle` only on instantiated and fitted `tslearn` estimator objects.
- Ensure the `path` argument is a string pointing to a valid directory where the process has write permissions.
- Do not attempt to use `to_pickle` on non-estimator objects or raw data arrays; it is intended for model serialization.

### Prompt Snippet
```text
# Serialize a fitted tslearn model
estimator.to_pickle(path="path/to/model.pkl")
```

### Common Failure Modes
- **PermissionError:** Occurs if the provided `path` points to a directory without write access or a read-only file.
- **NotFittedError:** Occurs if the estimator is pickled before calling `.fit()`, potentially leading to an incomplete model state upon reloading.
- **FileNotFoundError:** Occurs if the directory structure in `path` does not exist.

### Fix Code Hint
```python
import os

# Ensure directory exists before saving
model_path = "models/my_model.pkl"
os.makedirs(os.path.dirname(model_path), exist_ok=True)
model.to_pickle(model_path)
```

## API Test: `to_pyflux_dataset`

### Signature
```python
def to_pyflux_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:409_

_Source doc:_ Transform a tslearn-compatible dataset into a pyflux dataset. Parameters ---------- X: array, shape = (n_ts, sz, d), where n_ts=1 tslearn-formatted dataset to be cast to pyflux format Returns ------- Pandas data-frame pyflux-formatted dataset (cf. `link <https://pyflux.readthedocs.io/en/latest/getting_started.html>`_) Examples -------- >>> tslearn_arr = numpy.random.randn(1, 16, 1) >>> pyflux_df = to_pyflux_dataset(tslearn_arr) >>> pyflux_df.shape (16, 1) >>> pyflux_df.columns[0] 'dim_0' >>> tslearn_arr = numpy.random.randn(1, 16, 2) >>> pyflux_df = to_pyflux_dataset(tslearn_arr) >>> pyflux_df.shape (16, 2) >>> pyflux_df.columns[1] 'dim_1' >>> tslearn_arr = numpy.random.randn(10, 16, 1) >>> to_pyflux_dataset(tslearn_arr)  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: Array should be made of a single time series (10 here) Notes ----- Conversion from/to pyflux format requires pandas to be installed.

### Goal
Converts a single time-series dataset from the standard `tslearn` 3D array format into a `pandas.DataFrame` compatible with the `pyflux` library.

### Parameters
- `X`: `array`, shape = `(n_ts, sz, d)`. The input dataset. Must contain exactly one time series (`n_ts=1`).

### Input
- **Format:** A 3D NumPy array with shape `(1, sz, d)`.
- **Preconditions:** The `pandas` library must be installed in the environment. The input array must represent a single time series; multi-series arrays will raise a `ValueError`.

### Output
Returns a `pandas.DataFrame` where columns are named `dim_0` through `dim_{d-1}`, representing the time-series dimensions.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_pyflux_dataset

# Single time series with 16 time steps and 2 dimensions
tslearn_arr = np.random.randn(1, 16, 2)
pyflux_df = to_pyflux_dataset(tslearn_arr)

print(f"DataFrame shape: {pyflux_df.shape}")
# Witness: DataFrame shape: (16, 2)
```

### LLM Instruction Prompt
- Always verify that the input array `X` has a first dimension of size 1 before calling `to_pyflux_dataset`.
- Ensure `pandas` is available in the execution environment.
- Use this function only when the downstream task requires `pyflux` compatibility.

### Prompt Snippet
```text
from tslearn.utils import to_pyflux_dataset
# Ensure X has shape (1, sz, d)
df = to_pyflux_dataset(X)
```

### Common Failure Modes
- **`ValueError`**: Occurs if `X` contains more than one time series (e.g., `n_ts > 1`).
- **`ImportError`**: Occurs if `pandas` is not installed in the current Python environment.
- **Shape Mismatch**: Passing a 1D or 2D array directly without first converting it to the required 3D `(1, sz, d)` format using `tslearn.utils.to_time_series_dataset`.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset, to_pyflux_dataset

# If input is 2D (sz, d), convert to 3D (1, sz, d) first
X_raw = np.random.randn(16, 2)
X_3d = to_time_series_dataset([X_raw])
pyflux_df = to_pyflux_dataset(X_3d)
```

## API Test: `to_pyts_dataset`

### Signature
```python
def to_pyts_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:64_

_Source doc:_ Transform a tslearn-compatible dataset into a pyts dataset. Parameters ---------- X: array, shape = (n_ts, sz, d) tslearn-formatted dataset to be cast to pyts format Returns ------- array, shape=(n_ts, sz) if d=1, (n_ts, d, sz) otherwise pyts-formatted dataset Examples -------- >>> tslearn_arr = numpy.random.randn(10, 16, 1) >>> pyts_arr = to_pyts_dataset(tslearn_arr) >>> pyts_arr.shape (10, 16) >>> tslearn_arr = numpy.random.randn(10, 16, 2) >>> pyts_arr = to_pyts_dataset(tslearn_arr) >>> pyts_arr.shape (10, 2, 16) >>> tslearn_arr = [numpy.random.randn(16, 1), numpy.random.randn(10, 1)] >>> to_pyts_dataset(tslearn_arr)  # doctest: +IGNORE_EXCEPTION_DETAIL Traceback (most recent call last): ... ValueError: All the time series in the array should be of equal lengths

### Goal
Converts a `tslearn`-formatted 3D time-series dataset into the format expected by the `pyts` library.

### Parameters
- `X`: `array`, shape = `(n_ts, sz, d)`. The `tslearn`-formatted dataset containing `n_ts` time series of length `sz` with `d` dimensions.

### Input
The input must be a 3D NumPy array or a compatible structure where all time series have equal lengths (`sz`). If the data is currently in a variable-length format or 1D/2D, it must first be converted using `tslearn.utils.to_time_series_dataset`.

### Output
Returns an `array` with shape `(n_ts, sz)` if the input is univariate (`d=1`), or `(n_ts, d, sz)` if the input is multivariate (`d > 1`).

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_pyts_dataset

# Univariate case
tslearn_arr = np.random.randn(10, 16, 1)
pyts_arr = to_pyts_dataset(tslearn_arr)
print(f"Univariate shape: {pyts_arr.shape}") # (10, 16)

# Multivariate case
tslearn_arr_multi = np.random.randn(10, 16, 2)
pyts_arr_multi = to_pyts_dataset(tslearn_arr_multi)
print(f"Multivariate shape: {pyts_arr_multi.shape}") # (10, 2, 16)
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array of shape `(n_ts, sz, d)` before calling `to_pyts_dataset`.
- If the input is a list of variable-length arrays, you must call `tslearn.utils.to_time_series_dataset(X)` first to pad the sequences to equal length.
- Do not pass lists of unequal length arrays directly, as this will raise a `ValueError`.

### Prompt Snippet
```text
from tslearn.utils import to_pyts_dataset
# Ensure X is 3D and uniform length
pyts_data = to_pyts_dataset(X)
```

### Common Failure Modes
- **Unequal Lengths:** Passing a list of arrays with different lengths will trigger a `ValueError`. Use `tslearn.utils.to_time_series_dataset` to pad them first.
- **Incorrect Dimensions:** Passing a 2D array (e.g., `(n_ts, sz)`) without the third dimension `d` will cause shape mismatch errors. Ensure the input is explicitly 3D.

### Fix Code Hint
```python
from tslearn.utils import to_time_series_dataset, to_pyts_dataset

# If data is variable length or 2D:
X_formatted = to_time_series_dataset(X)
X_pyts = to_pyts_dataset(X_formatted)
```

## API Test: `to_seglearn_dataset`

### Signature
```python
def to_seglearn_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:141_

_Source doc:_ Transform a tslearn-compatible dataset into a seglearn dataset. Parameters ---------- X: array, shape = (n_ts, sz, d) tslearn-formatted dataset to be cast to seglearn format Returns ------- array of arrays, shape=(n_ts, ) seglearn-formatted dataset. i-th sub-array in the list has shape (sz_i, d) Examples -------- >>> tslearn_arr = numpy.random.randn(10, 16, 1) >>> seglearn_arr = to_seglearn_dataset(tslearn_arr) >>> seglearn_arr.shape (10, 16, 1) >>> tslearn_arr = numpy.random.randn(10, 16, 2) >>> seglearn_arr = to_seglearn_dataset(tslearn_arr) >>> seglearn_arr.shape (10, 16, 2) >>> tslearn_arr = [numpy.random.randn(16, 2), numpy.random.randn(10, 2)] >>> seglearn_arr = to_seglearn_dataset(tslearn_arr) >>> seglearn_arr.shape (2,) >>> seglearn_arr[0].shape (16, 2) >>> seglearn_arr[1].shape (10, 2)

### Goal
Converts a `tslearn`-formatted time-series dataset (either a 3D NumPy array or a list of variable-length arrays) into the format expected by the `seglearn` library.

### Parameters
- `X`: The time-series dataset to convert. Can be a 3D NumPy array of shape `(n_ts, sz, d)` or a list of 2D arrays with varying time-series lengths.

### Input
- **Data Format:** Must be a `tslearn`-compatible dataset. If the input is a 3D array, it must have shape `(n_ts, sz, d)`.
- **Preconditions:** If the input is not already in `tslearn` format, use `tslearn.utils.to_time_series_dataset` prior to calling this function to ensure compatibility.

### Output
Returns an `array of arrays` (a list of NumPy arrays) where the i-th sub-array has shape `(sz_i, d)`, representing the `seglearn` dataset structure.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_seglearn_dataset

# Example with fixed-length 3D array
tslearn_arr = np.random.randn(10, 16, 1)
seglearn_arr = to_seglearn_dataset(tslearn_arr)
print(f"Witness: {seglearn_arr.shape}")

# Example with variable-length list
tslearn_list = [np.random.randn(16, 2), np.random.randn(10, 2)]
seglearn_list = to_seglearn_dataset(tslearn_list)
print(f"Witness: {len(seglearn_list)} arrays, first shape {seglearn_list[0].shape}")
```

### LLM Instruction Prompt
- Always import `to_seglearn_dataset` from `tslearn.utils`.
- Ensure the input `X` is a valid `tslearn` dataset (3D array or list of 2D arrays) before conversion.
- Use this function only when interoperability with `seglearn` is required.

### Prompt Snippet
```text
from tslearn.utils import to_seglearn_dataset
seglearn_data = to_seglearn_dataset(tslearn_data)
```

### Common Failure Modes
- **Shape Mismatch:** Passing a 1D or 2D array that has not been converted to `tslearn` 3D format first will result in errors or incorrect dimensions.
- **Type Errors:** Passing non-array-like objects (e.g., raw Python lists of scalars) will fail; ensure data is structured as time-series sequences.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset, to_seglearn_dataset

# Ensure input is 3D or list-of-2D before casting
X_raw = [np.random.rand(10, 1), np.random.rand(5, 1)]
X_tslearn = to_time_series_dataset(X_raw)
X_seglearn = to_seglearn_dataset(X_tslearn)
```

## API Test: `to_sklearn_dataset`

### Signature
```python
def to_sklearn_dataset(dataset, dtype=float, return_dim=False)
```
_Source: tslearn/tslearn/utils/cast.py:21_

_Source doc:_ Transforms a time series dataset so that it fits the format used in ``sklearn`` estimators. Parameters ---------- dataset : array-like The dataset of time series to be transformed. dtype : data type (default: float64) Data type for the returned dataset. return_dim : boolean  (optional, default: False) Whether the dimensionality (third dimension should be returned together with the transformed dataset). Returns ------- numpy.ndarray of shape (n_ts, sz * d) The transformed dataset of time series. int (optional, if return_dim=True) The dimensionality of the original tslearn dataset (third dimension) Examples -------- >>> to_sklearn_dataset([[1, 2]], return_dim=True) (array([[1., 2.]]), 1) >>> to_sklearn_dataset([[1, 2], [1, 4, 3]]) array([[ 1.,  2., nan], [ 1.,  4.,  3.]]) See Also -------- to_time_series_dataset : Transforms a time series dataset to ``tslearn`` format.

### Goal
Converts a `tslearn`-formatted 3D time-series dataset into a 2D flat array structure compatible with `scikit-learn` estimators.

### Parameters
- `dataset`: array-like; The collection of time series to be transformed.
- `dtype`, default `float`: The data type for the resulting array.
- `return_dim`, default `False`: If `True`, returns a tuple containing the transformed array and the original dimensionality (third dimension) of the input.

### Input
- **Format:** Expects a 3D array-like structure `(n_ts, sz, d)`.
- **Preconditions:** If the input is not already in `tslearn` 3D format, it is recommended to first process it using `tslearn.utils.to_time_series_dataset`.

### Output
- Returns a `numpy.ndarray` of shape `(n_ts, sz * d)`.
- If `return_dim=True`, returns a tuple: `(numpy.ndarray, int)`.

### Valid Call Patterns
```python
from tslearn.utils import to_sklearn_dataset
import numpy as np

# Basic transformation
data = np.array([[[1, 2], [3, 4]]])
transformed = to_sklearn_dataset(data)

# Transformation with dimension return
transformed, dim = to_sklearn_dataset(data, return_dim=True)
print(f"Correctness witness: Transformed shape {transformed.shape}, Dim {dim}")
```

### LLM Instruction Prompt
- Use `to_sklearn_dataset` when preparing `tslearn` data for use in standard `scikit-learn` models that do not support 3D input.
- Always ensure the input is a valid time-series dataset (3D) before calling.
- If the original dimensionality is needed for inverse transformations, set `return_dim=True`.

### Prompt Snippet
```text
from tslearn.utils import to_sklearn_dataset
# Usage: to_sklearn_dataset(dataset, dtype=float, return_dim=False)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 1D or 2D arrays that have not been converted to `tslearn` 3D format first will result in unexpected flattening or errors.
- **NaN Padding:** When input time series have variable lengths, the function pads the resulting 2D array with `nan` values to match the maximum length; ensure the downstream `scikit-learn` estimator can handle `nan` values.

### Fix Code Hint
```python
# Ensure input is 3D before flattening
from tslearn.utils import to_time_series_dataset, to_sklearn_dataset
X_3d = to_time_series_dataset(raw_data)
X_sklearn = to_sklearn_dataset(X_3d)
```

## API Test: `to_sktime_dataset`

### Signature
```python
def to_sktime_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:288_

_Source doc:_ Transform a tslearn-compatible dataset into a sktime dataset. Parameters ---------- X: array, shape = (n_ts, sz, d) tslearn-formatted dataset to be cast to sktime format Returns ------- Pandas data-frame sktime-formatted dataset Examples -------- >>> tslearn_arr = numpy.random.randn(10, 16, 1) >>> sktime_arr = to_sktime_dataset(tslearn_arr) >>> sktime_arr.shape (10, 1) >>> sktime_arr["dim_0"][0].shape (16,) >>> tslearn_arr = numpy.random.randn(10, 16, 2) >>> sktime_arr = to_sktime_dataset(tslearn_arr) >>> sktime_arr.shape (10, 2) >>> sktime_arr["dim_1"][0].shape (16,) Notes ----- Conversion from/to sktime format requires pandas to be installed.

### Goal
Converts a `tslearn`-formatted 3D time-series dataset into a `pandas` DataFrame compatible with the `sktime` library's nested data format.

### Parameters
- `X`: `array`, shape = `(n_ts, sz, d)`. The `tslearn`-formatted dataset containing `n_ts` time series, each of length `sz` with `d` dimensions.

### Input
- **Data Format:** Must be a 3D NumPy array. If the data is 1D or 2D, it must first be converted using `tslearn.utils.to_time_series_dataset`.
- **Preconditions:** The `pandas` library must be installed in the environment, as the output format relies on `pandas` structures.

### Output
Returns a `pandas.DataFrame` representing the dataset in `sktime` format, where each column corresponds to a dimension of the time series and each cell contains a `pandas.Series` of the time-series values.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_sktime_dataset

# Standard 3D array conversion
tslearn_dataset = np.random.randn(10, 16, 3)
sktime_df = to_sktime_dataset(tslearn_dataset)

print(f"Converted dataset shape: {sktime_df.shape}")
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D NumPy array before calling `to_sktime_dataset`.
- If the input is not 3D, use `tslearn.utils.to_time_series_dataset(X)` first.
- Verify that `pandas` is available in the environment, as this function will raise an `ImportError` if `pandas` is missing.

### Prompt Snippet
```text
from tslearn.utils import to_sktime_dataset
# Ensure input is 3D (n_ts, sz, d)
sktime_data = to_sktime_dataset(X)
```

### Common Failure Modes
- **Missing Dependency:** Calling the function without `pandas` installed will result in an `ImportError`.
- **Incorrect Input Shape:** Passing a 2D array directly will cause a shape mismatch error; ensure the input is 3D.
- **Data Type Mismatch:** Passing non-numeric types or objects that cannot be cast to a 3D NumPy array will result in a `ValueError` or `TypeError`.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_sktime_dataset, to_time_series_dataset

# If input is 2D, fix by converting to 3D first
X_raw = np.random.randn(10, 16)
X_3d = to_time_series_dataset(X_raw)
sktime_df = to_sktime_dataset(X_3d)
```

## API Test: `to_stumpy_dataset`

### Signature
```python
def to_stumpy_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:214_

_Source doc:_ Transform a tslearn-compatible dataset into a stumpy dataset. Parameters ---------- X: array, shape = (n_ts, sz, d) tslearn-formatted dataset to be cast to stumpy format Returns ------- list of arrays of shape=(d, sz_i) if d > 1 or (sz_i, ) otherwise stumpy-formatted dataset. Examples -------- >>> tslearn_arr = numpy.random.randn(10, 16, 1) >>> stumpy_arr = to_stumpy_dataset(tslearn_arr) >>> len(stumpy_arr) 10 >>> stumpy_arr[0].shape (16,) >>> tslearn_arr = numpy.random.randn(10, 16, 2) >>> stumpy_arr = to_stumpy_dataset(tslearn_arr) >>> len(stumpy_arr) 10 >>> stumpy_arr[0].shape (2, 16)

### Goal
Converts a 3D `tslearn`-formatted time-series dataset into the list-of-arrays format required by the `stumpy` library for matrix profile computations.

### Parameters
- `X`: `array`, shape = (n_ts, sz, d). The `tslearn`-formatted dataset containing `n_ts` time series, each of length `sz` with `d` dimensions.

### Input
- **Data Format:** Must be a 3D NumPy array. If the data is 1D or 2D, it must first be converted to the standard `tslearn` 3D format using `tslearn.utils.to_time_series_dataset`.
- **Preconditions:** The input must be a valid `tslearn` dataset where the last dimension represents the number of features/dimensions per time step.

### Output
Returns a `list` of arrays. Each element in the list corresponds to a time series from the input. If the original data had `d > 1` dimensions, each array in the list has shape `(d, sz_i)`. If `d == 1`, each array has shape `(sz_i,)`.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_stumpy_dataset

# Example with univariate data
tslearn_arr = np.random.randn(10, 16, 1)
stumpy_arr = to_stumpy_dataset(tslearn_arr)

# Example with multivariate data
tslearn_arr_multi = np.random.randn(10, 16, 2)
stumpy_arr_multi = to_stumpy_dataset(tslearn_arr_multi)
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D NumPy array before calling `to_stumpy_dataset`.
- If the input is a list of variable-length sequences, pass it through `tslearn.utils.to_time_series_dataset` first.
- Use this function specifically when preparing data for `stumpy` matrix profile algorithms.

### Prompt Snippet
```text
from tslearn.utils import to_stumpy_dataset
# X must be 3D (n_ts, sz, d)
stumpy_data = to_stumpy_dataset(X)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing a 2D array directly will raise an error; ensure the input is 3D.
- **Non-NumPy Input:** Passing a list of lists directly without using `to_time_series_dataset` first will cause shape errors.
- **Incompatible Backend:** This function is strictly for `NumPy` arrays; it does not support `PyTorch` tensors directly.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset, to_stumpy_dataset

# Ensure input is 3D before conversion
X_3d = to_time_series_dataset(raw_data)
stumpy_data = to_stumpy_dataset(X_3d)
```

## API Test: `to_time_series`

### Signature
```python
def to_time_series(ts, remove_nans=False, be=None, dtype=float)
```
_Source: tslearn/tslearn/utils/utils.py:154_

_Source doc:_ Transforms a time series so that it fits the format used in ``tslearn`` models. Parameters ---------- ts : array-like, shape=(sz, d) or (sz,) The time series to be transformed. If shape is (sz,), the time series is assumed to be univariate. remove_nans : bool (default: False) Whether trailing NaNs at the end of the time series should be removed or not be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. dtype : data type (default: float) Data type for the returned time series, depending on the backend. Returns ------- ts_out : array-like, shape=(sz, d) The transformed time series. This is always guaranteed to be a new time series and never just a view into the old one. Examples -------- >>> to_time_series([1, 2]) array([[1.], [2.]]) >>> to_time_series([1, 2, numpy.nan]) array([[ 1.], [ 2.], [nan]]) >>> to_time_series([1, 2, numpy.nan], remove_nans=True) array([[1.], [2.]]) See Also -------- to_time_series_dataset : Transforms a dataset of time series

### Goal
Standardizes a single time-series sequence into the 2D `(sz, d)` format required by `tslearn` estimators and metrics.

### Parameters
- `ts`: The input time series, expected as an array-like object of shape `(sz, d)` or `(sz,)` for univariate series.
- `remove_nans`, default `False`: If `True`, removes trailing `NaN` values from the end of the time series.
- `be`, default `None`: The computation backend. Accepts `NumPyBackend`, `PyTorchBackend`, or strings `"numpy"`/`"pytorch"`. If `None`, the backend is inferred from the input array type.
- `dtype`, default `float`: The data type for the resulting array elements.

### Input
- **Data:** Accepts lists, NumPy arrays, or PyTorch tensors.
- **Preconditions:** The input must represent a single time series. For datasets (multiple time series), use `to_time_series_dataset` instead.

### Output
Returns `ts_out`: A new array-like object of shape `(sz, d)` containing the transformed time series. This is guaranteed to be a copy, not a view.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_time_series

# Univariate conversion
ts_formatted = to_time_series([1, 2, 3])

# Handling NaNs
ts_clean = to_time_series([1, 2, np.nan], remove_nans=True)

# Explicit backend usage
ts_torch = to_time_series([1, 2, 3], be="pytorch")
```

### LLM Instruction Prompt
- Always use `to_time_series` for single sequences and `to_time_series_dataset` for collections of sequences.
- If the input is a list or 1D array, the output will be a 2D array with `d=1`.
- When using `PyTorch` tensors as input, ensure the `be="pytorch"` argument is passed if explicit backend control is required.

### Prompt Snippet
```text
from tslearn.utils import to_time_series
# Ensure input is a single sequence, not a dataset
ts_formatted = to_time_series(my_sequence, remove_nans=True)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing a 3D array (dataset) to `to_time_series` will result in an error; use `to_time_series_dataset` for 3D inputs.
- **Backend Incompatibility:** Passing a PyTorch tensor while expecting NumPy output without specifying the backend may lead to unexpected type behavior if the backend is not correctly inferred.
- **NaN Handling:** `remove_nans=False` (default) preserves `NaN` values, which may cause downstream distance metrics (like DTW) to return `NaN` results.

### Fix Code Hint
```python
# If you get a shape error, check if your input is a dataset (3D)
# instead of a single time series (2D/1D).
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(my_list_of_sequences)
```

## API Test: `to_time_series_dataset`

### Signature
```python
def to_time_series_dataset(dataset, dtype=float, be=None)
```
_Source: tslearn/tslearn/utils/utils.py:215_

_Source doc:_ Transforms a time series dataset so that it fits the format used in ``tslearn`` models. Parameters ---------- dataset : array-like, shape=(n_ts, sz, d) or (n_ts, sz) or (sz,) The dataset of time series to be transformed. A single time series will be automatically wrapped into a dataset with a single entry. dtype : data type (default: float) Data type for the returned dataset, depending on the backend. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- dataset_out : array-like, shape=(n_ts, sz, d) The transformed dataset of time series. Examples -------- >>> to_time_series_dataset([[1, 2]]) array([[[1.], [2.]]]) >>> to_time_series_dataset([1, 2]) array([[[1.], [2.]]]) >>> to_time_series_dataset([[1, 2], [1, 4, 3]]) array([[[ 1.], [ 2.], [nan]], <BLANKLINE> [[ 1.], [ 4.], [ 3.]]]) >>> to_time_series_dataset([]).shape (0, 0, 0) See Also -------- to_time_series : Transforms a single time series

### Goal
Standardizes heterogeneous time-series input formats into the uniform 3D `(n_ts, sz, d)` array structure required by `tslearn` estimators.

### Parameters
- `dataset`: array-like; the input collection of time series. Can be a single time series, a list of lists, or a 2D/3D array.
- `dtype`, default `float`: The data type for the resulting array elements.
- `be`, default `None`: The computation backend. Accepts `NumPyBackend`, `PyTorchBackend`, or strings `"numpy"`/`"pytorch"`. If `None`, the backend is inferred from the input.

### Input
- **Data:** Accepts lists, NumPy arrays, or PyTorch tensors.
- **Preconditions:** Input must represent time series data. If variable lengths are provided, the function pads the sequences with `NaN` values to match the maximum length found in the dataset.

### Output
Returns `dataset_out` — A 3D array-like object with shape `(n_ts, sz, d)` where `n_ts` is the number of series, `sz` is the maximum length, and `d` is the number of features.

### Valid Call Patterns
```python
from tslearn.utils import to_time_series_dataset
import numpy as np

# Standard usage with list of lists
X = to_time_series_dataset([[1, 3, 4, 2], [1, 2, 4, 2]])

# Usage with variable length series
X_var = to_time_series_dataset([[1, 2], [1, 4, 3]])
```

### LLM Instruction Prompt
- Always use `to_time_series_dataset` to sanitize input data before passing it to any `tslearn` estimator.
- If the input contains variable-length sequences, expect the output to contain `NaN` padding.
- If using `PyTorch` tensors, explicitly set `be="pytorch"` or ensure the input is already a tensor to trigger backend inference.

### Prompt Snippet
```text
from tslearn.utils import to_time_series_dataset
X = to_time_series_dataset(raw_data)
```

### Common Failure Modes
- **Shape Mismatch:** Passing data that is not 1D, 2D, or 3D will raise a `ValueError`.
- **Backend Incompatibility:** Attempting to use `be="pytorch"` without the `pytorch` package installed will raise an `ImportError`.
- **Memory Overhead:** Converting very large datasets with variable lengths may result in significant memory usage due to `NaN` padding.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import to_time_series_dataset

# Deterministic check
data = [[1, 2], [1, 2, 3]]
formatted = to_time_series_dataset(data)
assert formatted.shape == (2, 3, 1)
print(f"Correctness witness: {formatted.shape}")
```

## API Test: `to_tsfresh_dataset`

### Signature
```python
def to_tsfresh_dataset(X)
```
_Source: tslearn/tslearn/utils/cast.py:527_

_Source doc:_ Transform a tslearn-compatible dataset into a tsfresh dataset. Parameters ---------- X: array, shape = (n_ts, sz, d) tslearn-formatted dataset to be cast to tsfresh format Returns ------- Pandas data-frame tsfresh-formatted dataset ("flat" data frame, as described `there <https://tsfresh.readthedocs.io/en/latest/text/data_formats.html#input-option-1-flat-dataframe-or-wide-dataframe>`_) Examples -------- >>> tslearn_arr = numpy.random.randn(1, 16, 1) >>> tsfresh_df = to_tsfresh_dataset(tslearn_arr) >>> tsfresh_df.shape (16, 3) >>> tslearn_arr = numpy.random.randn(1, 16, 2) >>> tsfresh_df = to_tsfresh_dataset(tslearn_arr) >>> tsfresh_df.shape (16, 4) Notes ----- Conversion from/to tsfresh format requires pandas to be installed.

### Goal
Converts a 3D `tslearn` time-series dataset into the "flat" DataFrame format required by the `tsfresh` library for feature extraction.

### Parameters
- `X`: `array`, shape = `(n_ts, sz, d)`. The `tslearn`-formatted dataset containing `n_ts` time series, each of length `sz` with `d` dimensions.

### Input
- **Format:** Must be a 3D NumPy array. If data is 1D or 2D, it must first be converted using `tslearn.utils.to_time_series_dataset`.
- **Preconditions:** The `pandas` library must be installed in the environment, as the output is a `pandas.DataFrame`.

### Output
Returns a `pandas.DataFrame` representing the dataset in the `tsfresh` "flat" format, where each row corresponds to a time point for a specific time series, including columns for the identifier, the time index, and the feature values.

### Valid Call Patterns
```python
import numpy as np
from tslearn.utils import to_tsfresh_dataset

# Example with 1 time series, 16 time points, 2 dimensions
tslearn_arr = np.random.randn(1, 16, 2)
tsfresh_df = to_tsfresh_dataset(tslearn_arr)

# Verification
print(f"Output shape: {tsfresh_df.shape}")
assert tsfresh_df.shape == (16, 4)
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D NumPy array before calling `to_tsfresh_dataset`.
- If the input is not 3D, use `tslearn.utils.to_time_series_dataset` first.
- Verify that `pandas` is available in the environment, as this function relies on it for the return type.

### Prompt Snippet
```text
from tslearn.utils import to_tsfresh_dataset
# Ensure X is 3D (n_ts, sz, d)
df = to_tsfresh_dataset(X)
```

### Common Failure Modes
- **Missing Dependency:** Calling the function without `pandas` installed will raise an `ImportError`.
- **Incorrect Input Shape:** Passing a 2D array directly will cause a shape mismatch error; ensure the input is 3D via `to_time_series_dataset`.
- **Data Type Mismatch:** Passing non-numeric data or non-array-like structures will result in a `ValueError` or `TypeError` during the internal conversion process.

### Fix Code Hint
```python
import tslearn.utils
# If X is 2D, convert first:
X_3d = tslearn.utils.to_time_series_dataset(X)
tsfresh_df = tslearn.utils.to_tsfresh_dataset(X_3d)
```

## API Test: `transform`

### Signature
```python
def transform(self, X, y=None)
def transform(self, X, y=None, **kwargs)
def transform(self, X, y=None, timestamps=None)
def transform(self, X)
```
_Source: tslearn/tslearn/preprocessing/preprocessing.py_

_Source doc:_ Synchronizes features of each time series with the feature of reference through linear interpolation. When timestamps are not provided, constant sampling periods are assumed for all features and identical start and stop timestamps are assumed for all features of a given times series. When timestamps are provided, features are synchronized on the temporal grid of the reference feature. Parameters ---------- X : array-like of shape (n_ts, sz, d) Time series dataset to be synchronized feature wise. y : Ignored Not used, for API consistency by convention. timestamps : np.datetime64 array-like of shape (n_ts, sz, d) or None (default: None) Acquisition timestamps, same shape as X if not None. When provided, timestamps should be increasing for each feature and should use np.datetime64('nat') for missing values. Returns ------- numpy.ndarray Time series dataset synchronized feature wise.

### Goal
Synchronizes features of a time-series dataset across a common temporal grid using linear interpolation to ensure consistent alignment.

### Parameters
- `self`: The initialized preprocessing estimator instance (e.g., `TimeSeriesResampler`).
- `X`: array-like of shape `(n_ts, sz, d)`. The time-series dataset to be synchronized.
- `y`, default `None`: Ignored; maintained for `scikit-learn` API compatibility.
- `timestamps`, default `None`: Optional `np.datetime64` array-like of shape `(n_ts, sz, d)` representing acquisition times for each feature.

### Input
- **Format:** 3D `numpy` arrays of shape `(n_ts, sz, d)`.
- **Preconditions:** If the input is 1D or 2D, it must be converted to a 3D dataset using `tslearn.utils.to_time_series_dataset` prior to calling `transform`.
- **Timestamps:** If provided, must be increasing for each feature; use `np.datetime64('nat')` to represent missing values.

### Output
- Returns `numpy.ndarray`: A 3D time-series dataset where features are synchronized on a common temporal grid.

### Valid Call Patterns
```python
import numpy as np
from tslearn.preprocessing import TimeSeriesResampler

# Example: Resampling a variable length dataset to a fixed size
X = np.random.rand(10, 20, 1)
resampler = TimeSeriesResampler(sz=10)
X_transformed = resampler.fit_transform(X)

print(f"Transformed shape: {X_transformed.shape}")
# Witness: Transformed shape: (10, 10, 1)
```

### LLM Instruction Prompt
- Always ensure the input `X` is a 3D array. If the user provides a list of lists or 2D array, suggest `tslearn.utils.to_time_series_dataset` first.
- When using `transform` on a preprocessing estimator, ensure `fit` or `fit_transform` has been called previously if the estimator requires learned parameters (e.g., scalers).

### Prompt Snippet
```text
# Ensure X is 3D before calling transform
from tslearn.utils import to_time_series_dataset
X_3d = to_time_series_dataset(X_raw)
X_transformed = estimator.transform(X_3d)
```

### Common Failure Modes
- **Shape Mismatch:** Passing 2D arrays directly without conversion to 3D `(n_ts, sz, d)` will raise a `ValueError`.
- **Unfitted Estimator:** Calling `transform` on an estimator that requires state (like `TimeSeriesScalerMinMax`) before calling `fit` will result in an `NotFittedError`.
- **Timestamp Inconsistency:** Providing `timestamps` that are not strictly increasing per feature will cause interpolation errors.

### Fix Code Hint
```python
# If you encounter a shape error, verify your input dimensions:
import numpy as np
from tslearn.utils import to_time_series_dataset

# Correct way to prepare input
X_prepared = to_time_series_dataset(my_data)
transformed = estimator.transform(X_prepared)
```

## API Test: `tril`

### Signature
```python
def tril(mat, k=0)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:217_

### Goal
Returns the lower triangular part of a matrix or tensor, with elements above the k-th diagonal set to zero.

### Parameters
- `mat`: The input matrix or tensor to be processed.
- `k`, default `0`: The diagonal above which to zero out elements. `k=0` is the main diagonal, `k>0` is above the main diagonal, and `k<0` is below the main diagonal.

### Input
The input `mat` should be a 2D array-like structure (NumPy array or PyTorch tensor). No specific preprocessing is required, but the input must be compatible with the backend being utilized.

### Output
Returns a matrix or tensor of the same shape as `mat`, containing only the elements on and below the k-th diagonal.

### Valid Call Patterns
```python
import numpy as np
from tslearn.backend.numpy_backend import NumPyBackend

be = NumPyBackend()
mat = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
result = be.tril(mat, k=0)
print(f"Correctness witness: {result.shape == (3, 3)}")
```

### LLM Instruction Prompt
- Use `tril` when you need to mask or extract the lower triangular portion of a square or rectangular matrix within a `tslearn` backend operation.
- Always ensure the input `mat` is a valid 2D structure.
- The `k` parameter follows standard NumPy/PyTorch conventions for diagonal offsets.

### Prompt Snippet
```text
# Example of using tril to mask a distance matrix
from tslearn.backend import instantiate_backend
be = instantiate_backend("numpy")
lower_tri = be.tril(distance_matrix, k=-1)
```

### Common Failure Modes
- Passing a 1D or 3D array: `tril` expects a 2D matrix; passing incompatible dimensions will raise a `ValueError` or `RuntimeError` depending on the backend.
- Backend mismatch: Ensure the input `mat` is compatible with the backend instance (e.g., do not pass a PyTorch tensor to a `NumPyBackend` instance).

### Fix Code Hint
```python
# Ensure input is 2D before calling tril
if mat.ndim != 2:
    mat = mat.reshape(mat.shape[0], -1)
result = be.tril(mat, k=0)
```

## API Test: `tril_indices`

### Signature
```python
def tril_indices(n, k=0, m=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:221_

### Goal
Returns the indices of the lower triangle of an `n` by `m` matrix, useful for masking or extracting elements below the `k`-th diagonal in time-series distance matrices.

### Parameters
- `n`: The number of rows in the matrix.
- `k`, default `0`: The diagonal above which to include elements. `k=0` is the main diagonal, `k>0` includes diagonals above, and `k<0` includes diagonals below.
- `m`, default `None`: The number of columns in the matrix. If `None`, defaults to `n`.

### Input
- `n` and `m` should be non-negative integers.
- This function is typically used in the context of backend-agnostic operations where index arrays are required for matrix manipulation.

### Output
Returns `unspecified` — A tuple of two arrays (row indices and column indices) representing the lower triangular part of the matrix.

### Valid Call Patterns
```python
from tslearn.backend.pytorch_backend import tril_indices

# Inferred from signature: Get indices for a 3x3 lower triangle
rows, cols = tril_indices(3)
print(f"Rows: {rows}, Cols: {cols}")
```

### LLM Instruction Prompt
- Use `tril_indices` when you need to generate coordinate arrays for lower-triangular matrix operations in a backend-compatible manner.
- Ensure `n` and `m` are provided as integers.
- Note that this function returns a tuple of indices; ensure your downstream logic expects this structure.

### Prompt Snippet
```text
from tslearn.backend.pytorch_backend import tril_indices
rows, cols = tril_indices(n=size, k=0)
```

### Common Failure Modes
- Passing non-integer values for `n` or `m`, which will cause indexing errors in the underlying backend.
- Assuming the output is a single array rather than a tuple of two arrays (row and column indices).
- Providing a negative value for `n` or `m`, which is invalid for matrix dimension definitions.

### Fix Code Hint
```python
# Ensure dimensions are positive integers
n_dim = int(n)
m_dim = int(m) if m is not None else n_dim
rows, cols = tril_indices(n_dim, k=0, m=m_dim)
```

## API Test: `triu`

### Signature
```python
def triu(mat, k=0)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:228_

### Goal
Returns the upper triangular part of a matrix, preserving the structure of the input tensor or array.

### Parameters
- `mat`: The input matrix or tensor from which to extract the upper triangular portion.
- `k`, default `0`: The diagonal above which to extract elements. `k=0` is the main diagonal, `k>0` is above the main diagonal, and `k<0` is below the main diagonal.

### Input
The input `mat` should be a 2D array-like structure (NumPy array or PyTorch tensor). This function is typically used within backend-agnostic operations where the input type determines the underlying computation engine.

### Output
Returns `unspecified` — The upper triangular part of the input matrix, with elements below the specified diagonal `k` set to zero. The return type matches the input type (NumPy array or PyTorch tensor).

### Valid Call Patterns
```python
import numpy as np
from tslearn.backend.numpy_backend import NumPyBackend

# Inferred usage based on signature
mat = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
result = NumPyBackend().triu(mat, k=0)
print(f"Correctness witness: {result.shape == (3, 3)}")
```

### LLM Instruction Prompt
- Use `triu` when you need to mask or extract upper triangular components of a distance or similarity matrix.
- Ensure the input `mat` is a 2D structure.
- If working with PyTorch tensors, ensure the backend context is handled appropriately to maintain gradient tracking if required.

### Prompt Snippet
```text
from tslearn.backend.numpy_backend import NumPyBackend
# ...
triu_result = NumPyBackend().triu(my_matrix, k=0)
```

### Common Failure Modes
- Passing a 3D array (time-series dataset) directly: `triu` expects a 2D matrix; ensure you are operating on a single distance/similarity matrix.
- Mismatched backend: Attempting to use `triu` on a PyTorch tensor while using a `NumPyBackend` instance may lead to type errors or loss of automatic differentiation capabilities.
- Invalid `k` value: Providing a `k` that exceeds the matrix dimensions may result in an empty or zeroed matrix depending on the backend implementation.

### Fix Code Hint
```python
import numpy as np
from tslearn.backend.numpy_backend import NumPyBackend

# Ensure input is 2D before calling triu
mat = np.random.rand(5, 5)
be = NumPyBackend()
upper_tri = be.triu(mat, k=1)
```

## API Test: `triu_indices`

### Signature
```python
def triu_indices(n, k=0, m=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:232_

### Goal
Returns the indices for the upper-triangle of an (n, m) array, useful for masking or extracting values from distance matrices in time-series analysis.

### Parameters
- `n`: The number of rows in the array.
- `k`, default `0`: The diagonal above the main diagonal to consider. `k=0` is the main diagonal, `k>0` is above, and `k<0` is below.
- `m`, default `None`: The number of columns in the array. If `None`, defaults to `n`.

### Input
- `n` and `m` must be integers representing the dimensions of the square or rectangular matrix for which indices are required.

### Output
Returns `unspecified` — A tuple of two arrays containing the row and column indices of the upper triangle of the specified matrix.

### Valid Call Patterns
```python
from tslearn.backend.pytorch_backend import triu_indices
# Inferred from signature: Get indices for a 3x3 matrix
rows, cols = triu_indices(3)
print(f"Witness: Rows {rows}, Cols {cols}")
```

### LLM Instruction Prompt
- Use `triu_indices` when you need to extract or mask elements from a distance matrix (e.g., a symmetric DTW distance matrix) to avoid redundant computations or to ignore the diagonal.
- Ensure `n` and `m` are positive integers.

### Prompt Snippet
```text
from tslearn.backend.pytorch_backend import triu_indices
rows, cols = triu_indices(n=my_n, k=my_k, m=my_m)
```

### Common Failure Modes
- Passing non-integer values for `n` or `m`, which will cause a runtime error during index generation.
- Assuming the output is a single array; the function returns a tuple of two arrays (row indices and column indices).

### Fix Code Hint
```python
# Ensure indices are used to index into a matrix of correct shape
import numpy as np
n = 3
rows, cols = triu_indices(n)
matrix = np.zeros((n, n))
matrix[rows, cols] = 1.0  # Correctly sets upper triangle
```

## API Test: `ts_size`

### Signature
```python
def ts_size(ts, be=None)
```
_Source: tslearn/tslearn/utils/utils.py:465_

_Source doc:_ Returns actual time series size. Final timesteps that have `NaN` values for all dimensions will be removed from the count. Infinity and negative infinity ar considered valid time series values. Parameters ---------- ts : array-like A time series. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- int Actual size of the time series. Examples -------- >>> ts_size([1, 2, 3, numpy.nan]) 3 >>> ts_size([1, numpy.nan]) 1 >>> ts_size([numpy.nan]) 0 >>> ts_size([[1, 2], ...          [2, 3], ...          [3, 4], ...          [numpy.nan, 2], ...          [numpy.nan, numpy.nan]]) 4 >>> ts_size([numpy.nan, 3, numpy.inf, numpy.nan]) 3

### Goal
Computes the effective length of a time series by excluding trailing timesteps that consist entirely of `NaN` values.

### Parameters
- `ts`: array-like; The time series data to evaluate.
- `be`, default `None`: Backend object or string ("numpy" or "pytorch"); specifies the computation engine. If `None`, the backend is inferred from the input array type.

### Input
- **Data Format:** Accepts array-like structures (lists, NumPy arrays, or PyTorch tensors).
- **Preconditions:** While it handles variable-length data, ensure the input is formatted consistently with the expected backend if explicitly provided.

### Output
Returns `int` — The count of timesteps excluding trailing `NaN` entries.

### Valid Call Patterns
```python
from tslearn.utils import ts_size
import numpy as np

# Standard usage with list
size = ts_size([1, 2, 3, np.nan])
assert size == 3

# Usage with multi-dimensional array
ts = [[1, 2], [2, 3], [np.nan, np.nan]]
assert ts_size(ts) == 2
```

### LLM Instruction Prompt
- Use `ts_size` to determine the valid length of a time series before performing slicing or metric computations.
- Always prefer `ts_size` over `len()` when working with datasets that contain padding or `NaN` markers for variable-length sequences.

### Prompt Snippet
```text
from tslearn.utils import ts_size
# Calculate effective length of time series 'ts'
length = ts_size(ts)
```

### Common Failure Modes
- **Dimension Mismatch:** Passing a 3D dataset (n_ts, sz, d) directly when expecting a single time series (sz, d) or (sz,).
- **Backend Mismatch:** Providing a PyTorch tensor while explicitly setting `be="numpy"`, which may cause type errors during internal processing.
- **Non-trailing NaNs:** `ts_size` only removes trailing `NaN`s; internal `NaN` values are counted as part of the size.

### Fix Code Hint
```python
# If you need to truncate a series to its effective size:
import numpy as np
from tslearn.utils import ts_size

ts = np.array([1.0, 2.0, np.nan, np.nan])
valid_ts = ts[:ts_size(ts)]
```

## API Test: `ts_zeros`

### Signature
```python
def ts_zeros(sz, d=1)
```
_Source: tslearn/tslearn/utils/utils.py:521_

_Source doc:_ Returns a time series made of zero values. Parameters ---------- sz : int Time series size. d : int (optional, default: 1) Time series dimensionality. Returns ------- numpy.ndarray A time series made of zeros. Examples -------- >>> ts_zeros(3, 2)  # doctest: +NORMALIZE_WHITESPACE array([[0., 0.], [0., 0.], [0., 0.]]) >>> ts_zeros(5).shape (5, 1)

### Goal
Generates a synthetic time-series array of a specified length and dimensionality initialized entirely with zero values.

### Parameters
- `sz`: `int`. The number of time steps (length) for the generated time series.
- `d`, default `1`: `int`. The number of features (dimensionality) per time step.

### Input
- The function accepts integer values for `sz` and `d`.
- No specific data preparation is required as this is a synthetic data generation utility.

### Output
Returns `numpy.ndarray` — A 2D array of shape `(sz, d)` containing floating-point zeros.

### Valid Call Patterns
```python
from tslearn.utils import ts_zeros

# Generate a 3-step, 2-dimensional zero time series
ts = ts_zeros(3, 2)

# Generate a 5-step, 1-dimensional zero time series (default d=1)
ts_default = ts_zeros(5)
```

### LLM Instruction Prompt
- Use `ts_zeros` when you need to initialize a placeholder time-series array of a specific shape for testing or algorithm initialization.
- Always ensure `sz` is a positive integer.
- Note that the output is a 2D array; if you require a 3D dataset (e.g., for `tslearn` estimators), you may need to wrap the result in a list or use `numpy.expand_dims`.

### Prompt Snippet
```text
from tslearn.utils import ts_zeros
# Example: Create a zero-initialized time series of length 10 and 3 dimensions
data = ts_zeros(10, 3)
```

### Common Failure Modes
- Passing non-integer values for `sz` or `d`, which will raise a `TypeError` or `ValueError` during array allocation.
- Expecting a 3D array (dataset format) directly; `ts_zeros` returns a 2D array `(sz, d)`, which is not immediately compatible with estimators expecting `(n_ts, sz, d)`.

### Fix Code Hint
```python
import numpy as np
from tslearn.utils import ts_zeros

# If you need a 3D dataset format (n_ts=1, sz=10, d=3):
ts = ts_zeros(10, 3)
dataset = ts[np.newaxis, :, :]
print(dataset.shape)  # (1, 10, 3)
```

## API Test: `uniform`

### Signature
```python
def uniform(low=0.0, high=1.0, size=(1,), dtype=None)
```
_Source: tslearn/tslearn/backend/pytorch_backend.py:256_

### Goal
Generates a tensor or array of random values sampled from a uniform distribution within the specified range, compatible with the active `tslearn` backend.

### Parameters
- `low`, default `0.0`: Lower boundary of the output interval.
- `high`, default `1.0`: Upper boundary of the output interval.
- `size`, default `(1,)`: Shape of the output array or tensor.
- `dtype`, default `None`: Desired data type of the output; if `None`, the default backend type is used.

### Input
The function requires the `tslearn` backend to be initialized. It operates on the currently active backend (NumPy or PyTorch). No specific time-series data formatting is required as this is a utility for generating synthetic data or noise.

### Output
Returns an array or tensor of shape `size` containing random samples drawn from the uniform distribution `[low, high)`.

### Valid Call Patterns
```python
import numpy as np
from tslearn.backend import instantiate_backend

# Inferred usage based on backend utility patterns
be = instantiate_backend("numpy")
samples = be.uniform(low=0.0, high=0.5, size=(100,))
assert samples.shape == (100,)
print(f"Correctness witness: {samples.mean()}")
```

### LLM Instruction Prompt
- Use `uniform` when generating synthetic noise or initializing weights within a `tslearn` workflow.
- Always ensure the backend is instantiated via `instantiate_backend` before calling this method if working within a generic pipeline.
- Prefer this over `numpy.random.uniform` or `torch.rand` to maintain backend-agnostic code compatibility.

### Prompt Snippet
```text
# Generate uniform noise for time-series augmentation
be = instantiate_backend("numpy")
noise = be.uniform(low=-0.1, high=0.1, size=X.shape)
```

### Common Failure Modes
- **Backend Mismatch:** Calling `uniform` without ensuring the correct backend is active, leading to type errors if expecting a specific tensor type (e.g., `torch.Tensor` vs `numpy.ndarray`).
- **Shape Mismatch:** Providing an invalid `size` tuple that does not match the dimensions required for subsequent time-series operations (e.g., `(n_ts, sz, d)`).

### Fix Code Hint
```python
# Ensure the backend is explicitly handled
from tslearn.backend import instantiate_backend
be = instantiate_backend("pytorch") # or "numpy"
data = be.uniform(low=0.0, high=1.0, size=(10, 5, 1))
```

## API Test: `unnormalized_gak`

### Signature
```python
def unnormalized_gak(s1, s2, sigma=1.0, be=None)
```
_Source: tslearn/tslearn/metrics/_gak.py:178  (+1 more definition site/overload)_

_Source doc:_ Compute Global Alignment Kernel (GAK) between (possibly multidimensional) time series and return it. .. math:: k(\mathbf{x}, \mathbf{y}) = \sum_{\pi \in \mathcal{A}(\mathbf{x}, \mathbf{y})} \prod_{i=1}^{ | \pi | } \exp \left( - \frac{ \left\| x_{\pi_1(i)} - y_{\pi_2{j}} \right\|^2}{2 \sigma^2} \right) It is not required that both time series share the same size, but they must be the same dimension. GAK was originally presented in [1]_. This is an unnormalized version. Parameters ---------- s1 : array-like, shape=(sz1, d) or (sz1,) A time series. If shape is (sz1,), the time series is assumed to be univariate. s2 : array-like, shape=(sz2, d) or (sz2,) Another time series. If shape is (sz2,), the time series is assumed to be univariate. sigma : float (default 1.) Bandwidth of the internal gaussian kernel used for GAK. be : Backend object or string or None Backend. If `be` is an instance of the class `NumPyBackend` or the string `"numpy"`, the NumPy backend is used. If `be` is an instance of the class `PyTorchBackend` or the string `"pytorch"`, the PyTorch backend is used. If `be` is `None`, the backend is determined by the input arrays. See our :ref:`dedicated user-guide page <backend>` for more information. Returns ------- float Kernel value Examples -------- >>> unnormalized_gak([1, 2, 3], ...                  [1., 2., 2., 3.], ...                  sigma=2.)  # doctest: +ELLIPSIS 15.358... >>> unnormalized_gak([1, 2, 3], ...                  [1., 2., 2., 3., 4.])  # doctest: +ELLIPSIS 3.166... See Also -------- gak : normalized version of GAK that ensures that k(x,x) = 1 cdist_gak : Compute cross-similarity matrix using Global Alignment kernel References ---------- .. [1] M. Cuturi, "Fast global alignment kernels," ICML 2011.

### Goal
Computes the unnormalized Global Alignment Kernel (GAK) similarity measure between two time series, accounting for non-linear temporal alignments.

### Parameters
- `s1`: Array-like, shape `(sz1, d)` or `(sz1,)`. The first time series.
- `s2`: Array-like, shape `(sz2, d)` or `(sz2,)`. The second time series.
- `sigma`, default `1.0`: Float. The bandwidth parameter for the internal Gaussian kernel.
- `be`, default `None`: Backend object, string ("numpy" or "pytorch"), or `None`. Determines the computational engine.

### Input
- **Data Format:** Accepts 1D or 2D array-like structures. If 1D, the series is treated as univariate.
- **Preconditions:** Both `s1` and `s2` must have the same dimensionality `d`.
- **Backend:** If `be` is `None`, the backend is inferred from the input array types (e.g., `torch.Tensor` triggers `PyTorchBackend`).

### Output
Returns `float`: The computed unnormalized kernel value.

### Valid Call Patterns
```python
from tslearn.metrics import unnormalized_gak

# Standard usage with default backend
val = unnormalized_gak([1, 2, 3], [1, 2, 2, 3], sigma=2.0)

# Explicit backend usage
val_torch = unnormalized_gak([1, 2, 3], [1, 2, 2, 3], sigma=2.0, be="pytorch")
```

### LLM Instruction Prompt
- Always ensure `s1` and `s2` have matching dimensions.
- If using `PyTorch` tensors, ensure the `be` parameter is set to `"pytorch"` or left as `None` for automatic detection.
- Note that this is the *unnormalized* version; use `gak` if a normalized kernel (where $k(x,x)=1$) is required.

### Prompt Snippet
```text
from tslearn.metrics import unnormalized_gak
# Example:
# result = unnormalized_gak(series_a, series_b, sigma=2.0)
```

### Common Failure Modes
- `ZeroDivisionError`: Occurs if `sigma` is set to `0`.
- `ValueError`: Occurs if `s1` and `s2` have different dimensions (e.g., one is univariate and the other is multivariate).
- `TypeError`: Occurs if the backend provided is incompatible with the input array types.

### Fix Code Hint
```python
import numpy as np
from tslearn.metrics import unnormalized_gak

# Ensure sigma > 0 to avoid ZeroDivisionError
s1 = np.array([1, 2, 3])
s2 = np.array([1, 2, 2, 3])
result = unnormalized_gak(s1, s2, sigma=1.0)
print(f"Correctness witness: {result > 0}")
```

## API Test: `y_shifted_sbd_vec`

### Signature
```python
def y_shifted_sbd_vec(ref_ts, dataset, norm_ref, norms_dataset)
```
_Source: tslearn/tslearn/metrics/cycc.py:99_

_Source doc:_ Shift a time series dataset w.r.t. a time series of reference. Parameters ---------- ref_ts : array-like, shape=(sz, d), dtype=float64 Time series of reference. dataset : array-like, shape=(n_ts, sz, d), dtype=float64 Time series dataset. norm_ref : float64 norms_dataset : array-like, shape=(n_ts,), dtype=float64 Norms of the time series dataset. Returns ------- dataset_shifted : array-like, shape=(n_ts, sz, d), dtype=float64 Shifted dataset.

### Goal
Computes the optimal shift for each time series in a dataset relative to a reference time series to minimize the Squared Euclidean distance, typically used in SBD (Shape-Based Distance) calculations.

### Parameters
- `ref_ts`: The reference time series array of shape `(sz, d)`.
- `dataset`: The collection of time series to be shifted, of shape `(n_ts, sz, d)`.
- `norm_ref`: The precomputed norm of the reference time series (`float64`).
- `norms_dataset`: An array of shape `(n_ts,)` containing the precomputed norms of each time series in the dataset.

### Input
The function expects 3D NumPy arrays for `dataset` and 2D arrays for `ref_ts` with `float64` precision. Preconditions include ensuring that `norm_ref` and `norms_dataset` are correctly calculated (typically using the L2 norm) to match the input time series, as the function relies on these for the shift calculation.

### Output
Returns `dataset_shifted`: A 3D NumPy array of shape `(n_ts, sz, d)` containing the input dataset where each time series has been circularly shifted to align with the reference.

### Valid Call Patterns
```python
import numpy as np
from tslearn.metrics.cycc import y_shifted_sbd_vec

# Inferred usage based on signature
ref = np.random.rand(10, 1)
data = np.random.rand(5, 10, 1)
norm_r = np.linalg.norm(ref)
norms_d = np.linalg.norm(data, axis=(1, 2))

shifted = y_shifted_sbd_vec(ref, data, norm_r, norms_d)
assert shifted.shape == (5, 10, 1)
print(f"Correctness witness: Output shape {shifted.shape} matches input dataset shape.")
```

### LLM Instruction Prompt
- Always ensure `dataset` is a 3D array `(n_ts, sz, d)` and `ref_ts` is a 2D array `(sz, d)`.
- Ensure `norm_ref` and `norms_dataset` are precomputed using the same norm metric (L2) used by the internal SBD logic.
- This function is a low-level backend helper; prefer higher-level distance metrics if available.

### Prompt Snippet
```text
from tslearn.metrics.cycc import y_shifted_sbd_vec
# Ensure inputs are float64 and norms are precomputed
shifted_data = y_shifted_sbd_vec(ref_ts, dataset, norm_ref, norms_dataset)
```

### Common Failure Modes
- **Shape Mismatch:** Providing a 2D array for `dataset` instead of 3D will cause an index error or shape mismatch.
- **Dtype Mismatch:** Passing non-float64 data may lead to unexpected behavior or performance degradation in the underlying Cython implementation.
- **Norm Mismatch:** Providing norms that do not correspond to the provided time series will result in incorrect shifts.

### Fix Code Hint
```python
# Ensure data is 3D and norms are calculated correctly
dataset = to_time_series_dataset(dataset)
norm_ref = np.linalg.norm(ref_ts)
norms_dataset = np.linalg.norm(dataset, axis=(1, 2))
shifted = y_shifted_sbd_vec(ref_ts, dataset, norm_ref, norms_dataset)
```

