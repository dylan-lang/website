The STATISTICS library
**********************

.. current-library:: statistics

The ``STATISTICS`` library provides some basic statistical
functions along with optimized implementations.

.. note:: Currently, the statistical functions are only available
   for limited vectors of :drm:`<double-float>` values. This is
   expected to change in the future.

.. contents::
   :local:


The STATISTICS module
=====================

.. current-module:: statistics

Types
-----

.. type:: <double-float-vector>

   A :drm:`<vector>` that only contains :drm:`<double-float>` values.

   :equivalent: ``limited(<vector>, of: <double-float>)``

   :description:

     A :drm:`<vector>` that only contains :drm:`<double-float>` values.

     This type is used for implementations of statistical functions
     which are specialized for :drm:`<double-float>` values.

   :seealso:

     - :type:`<double-float?-vector>`
     - :type:`<numeric-sequence>`

.. type:: <double-float?-vector>

   A :drm:`<vector>` that contains values that are either
   :drm:`<double-float>` or ``#f``.

   :equivalent: ``limited(<vector>, of: false-or(<double-float>))``

   :description:

     A :drm:`<vector>` that contains values that are either
     :drm:`<double-float>` or ``#f``.

     This type is used for implementations of statistical functions
     which may need to handle missing data. By using a separate type
     from :type:`<double-float-vector>`, the implementation can limit
     any overhead from handling missing values to only being applied
     where it is needed.

     .. note:: Implementations of the statistical functions which
        handle missing data have not yet been provided.

   :seealso:

     - :type:`<double-float-vector>`
     - :type:`<numeric-sequence>`

.. type:: <numeric-sequence>

   :equivalent: ``type-union(<double-float-vector>, <double-float?-vector>)``

   :seealso:

     - :type:`<double-float-vector>`
     - :type:`<double-float?-vector>`

Coercion Functions
------------------

.. function:: double-float-vector

   Utility function for converting a sequence that contains only
   :drm:`<double-float>` values to a :type:`<double-float-vector>`
   for use with the optimized implementations of the basic statistical
   functions.

   :signature: double-float-vector (seq) => (vec)

   :parameter seq: An instance of :drm:`<sequence>`.
   :value vec: An instance of :type:`<double-float-vector>`.

   :example:

     .. code-block:: dylan

        let dv = double-float-vector(#[1.0d0, 2.0d0, 3.0d0]);

Extrema
-------

.. generic-function:: maximum
   :open:

   Returns the maximum value from a numeric sequence.

   :signature: maximum (sample) => (maximum)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value maximum: An instance of :drm:`<number>`.

   :example:

     Assuming that ``dv`` contains the values ``#[1.0d0, -1.0d0, 2.0d0]``:

     .. code-block:: dylan-console

        ? maximum(dv)
        => 2.0d0

   :seealso:

     - :gf:`maximum/trimmed`
     - :gf:`minimum`
     - :gf:`minimum/trimmed`
     - :gf:`minimum+maximum`

.. method:: maximum
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`maximum` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value maximum: An instance of :drm:`<double-float>`.

.. generic-function:: maximum/trimmed
   :open:

   Returns the maximum value from a numeric sequence that is below (or
   optionally equal to) an upper limit.

   :signature: maximum/trimmed (sample upper-limit #key inclusive?) => (maximum)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :parameter upper-limit: An instance of :drm:`<number>`.
   :parameter #key inclusive?: An instance of :drm:`<boolean>`. Default value
     is ``#t``.
   :value maximum: An instance of :drm:`<number>`.

   :description:

     Returns the maximum value from a numeric sequence that is below (or
     optionally equal to) an ``upper-limit``.

     If ``inclusive?`` is true (the default), then values equal to the
     ``upper-limit`` are included when calculating the maximum value.

   :example:

     Assuming that ``dv`` contains the values ``#[1.0d0, 2.0d0, 3.0d0, 4.0d0]``:

     .. code-block:: dylan-console

        ? maximum/trimmed(dv, 3.0d0, inclusive?: #t)
        => 3.0d0

        ? maximum/trimmed(dv, 3.0d0, inclusive?: #f)
        => 2.0d0

   :seealso:

     - :gf:`maximum`
     - :gf:`minimum`
     - :gf:`minimum/trimmed`
     - :gf:`minimum+maximum`

.. method:: maximum/trimmed
   :specializer: <double-float-vector>, <double-float>
   :sealed:

   A specialized implementation of :gf:`maximum/trimmed` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :parameter upper-limit: An instance of :drm:`<double-float>`.
   :parameter #key inclusive?: An instance of :drm:`<boolean>`.
   :value maximum: An instance of :drm:`<double-float>`.

.. generic-function:: minimum
   :open:

   Returns the minimum value from a numeric sequence.

   :signature: minimum (sample) => (minimum)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value minimum: An instance of :drm:`<number>`.

   :example:

     Assuming that ``dv`` contains the values ``#[1.0d0, -1.0d0, 2.0d0]``:

     .. code-block:: dylan-console

        ? minimum(dv)
        => -1.0d0

   :seealso:

     - :gf:`maximum`
     - :gf:`maximum/trimmed`
     - :gf:`minimum/trimmed`
     - :gf:`minimum+maximum`

.. method:: minimum
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`minimum` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value minimum: An instance of :drm:`<double-float>`.

.. generic-function:: minimum/trimmed
   :open:

   Returns the minimum value from a numeric sequence that is over (or
   optionally equal to) a ``lower-limit``.

   :signature: minimum/trimmed (sample lower-limit #key inclusive?) => (minimum)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :parameter lower-limit: An instance of :drm:`<number>`.
   :parameter #key inclusive?: An instance of :drm:`<boolean>`.
   :value minimum: An instance of :drm:`<number>`.

   :description:

     Returns the minimum value from a numeric sequence that is over (or
     optionally equal to) a ``lower-limit``.

     If ``inclusive?`` is true (the default), then values equal to the
     ``lower-limit`` are included when calculating the minimum value.

   :example:

     Assuming that ``dv`` contains the values ``#[1.0d0, 2.0d0, 3.0d0, 4.0d0]``:

     .. code-block:: dylan-console

        ? minimum/trimmed(dv, 2.0d0, inclusive?: #t)
        => 2.0d0

        ? minimum/trimmed(dv, 2.0d0, inclusive?: #f)
        => 3.0d0

   :seealso:

     - :gf:`maximum`
     - :gf:`maximum/trimmed`
     - :gf:`minimum`
     - :gf:`minimum+maximum`

.. method:: minimum/trimmed
   :specializer: <double-float-vector>, <double-float>
   :sealed:

   A specialized implementation of :gf:`minimum/trimmed` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :parameter lower-limit: An instance of :drm:`<double-float>`.
   :parameter #key inclusive?: An instance of :drm:`<boolean>`.
   :value minimum: An instance of :drm:`<double-float>`.

.. generic-function:: minimum+maximum
   :open:

   Returns both the minimum and maximum values within a numeric sequence.

   :signature: minimum+maximum (sample) => (minimum maximum)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value minimum: An instance of :drm:`<number>`.
   :value maximum: An instance of :drm:`<number>`.

   :example:

     Assuming that ``dv`` contains the values ``#[1.0d0, -1.0d0, 2.0d0]``:

     .. code-block:: dylan-console

        ? minimum+maximum(dv)
        => values(-1.0d0, 2.0d0)

   :seealso:

     - :gf:`maximum`
     - :gf:`maximum/trimmed`
     - :gf:`minimum`
     - :gf:`minimum/trimmed`

.. method:: minimum+maximum
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`minimum+maximum` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value minimum: An instance of :drm:`<double-float>`.
   :value maximum: An instance of :drm:`<double-float>`.

Means
-----

.. index:: average
.. index:: mean
.. generic-function:: mean/arithmetic
   :open:

   Returns the arithmetic mean of a numeric sequence.

   :signature: mean/arithmetic (sample) => (mean)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value mean: An instance of :drm:`<number>`.

   :description:

     Returns the arithmetic mean of a numeric sequence.

     Commonly known as just 'mean' or 'average', the arithmetic mean is
     the sum of the values of the sequence, divided by the number of values
     in the sequence. It is distinct from other ways of calculating a mean
     such as those provided by :gf:`mean/geometric` and :gf:`mean/harmonic`.

     A simple (and slightly faster) naive implementation of the arithmetic
     mean is subject to numerical inaccuracy. This implementation follows
     the method presented by Knuth in `The Art of Computer Programming`_,
     3rd edition on page 232.

   :equivalent:

     The arithmetic mean is given by:

     .. math::

        \frac{1}{n} \sum_{i=1}^{n} x_{i}

     Our implementation is computed as follows:

     .. math::

        &m_{1} = x_{1} \\
        &m_{k} = m_{k-1} + \frac{x_{k} - m_{k-1}}{k}

   :example:

     Assuming that ``dv`` contains the values ``#[1.0d0, 2.0d0, 8.0d0, 9.0d0]``:

     .. code-block:: dylan-console

        ? mean/arithmetic(dv)
        => 5.25d0

   :seealso:

     - :gf:`mean/fast`
     - :gf:`mean/geometric`
     - :gf:`mean/harmonic`
     - :gf:`standard-scores`

     - `Arithmetic Mean on Wikipedia <https://en.wikipedia.org/wiki/Arithmetic_mean>`__

.. method:: mean/arithmetic
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`mean/arithmetic` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value mean: An instance of :drm:`<double-float>`.

.. generic-function:: mean/fast
   :open:

   Returns the arithmetic mean of a numeric sequence.

   :signature: mean/fast (sample) => (mean)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value mean: An instance of :drm:`<number>`.

   :description:

     Returns the arithmetic mean of a numeric sequence.

     This differs from :gf:`mean/arithmetic` by using a naive algorithm
     that is slightly faster, but subject to numerical inaccuracy. You
     should only use this function if you're aware of the risks.

   :equivalent:

     :math:`\frac{1}{n} \sum_{i=1}^{n} x_{i}`

   :example:

     Assuming that ``dv`` contains the values ``#[1.0d0, 2.0d0, 8.0d0, 9.0d0]``:

     .. code-block:: dylan-console

        ? mean/arithmetic(dv)
        => 5.25d0

   :seealso:

     - :gf:`mean/arithmetic`
     - :gf:`mean/geometric`
     - :gf:`mean/harmonic`

.. method:: mean/fast
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`mean/fast` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value mean: An instance of :drm:`<double-float>`.

.. generic-function:: mean/geometric
   :open:

   Returns the geometric mean of a numeric sequence.

   :signature: mean/geometric (sample) => (mean)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value mean: An instance of :drm:`<number>`.

   :description:

     Returns the geometric mean of a numeric sequence.

     For greater numerical accuracy, our implementation is based on
     the exponentiation of the arithmetic mean of the natural logarithm
     of each value in ``sample``.

   :equivalent:

     The geometric mean is given by:

     .. math::

        \left(\prod_{i=1}^na_i \right)^{1/n}

     Our implementation is computed as follows:

     .. math::

        \exp\left[\frac1n\sum_{i=1}^n\ln a_i\right]

   :example:

     Assuming that ``dv`` contains the values ``#[2.0d0, 4.0d0, 8.0d0]``:

     .. code-block:: dylan-console

        ? mean/geometric(dv)
        => 4.0d0

   :seealso:

     - :gf:`mean/arithmetic`
     - :gf:`mean/fast`
     - :gf:`mean/harmonic`

     - `Geometric Mean on Wikipedia <https://en.wikipedia.org/wiki/Geometric_mean>`__

.. method:: mean/geometric
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`mean/geometric` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value mean: An instance of :drm:`<double-float>`.

.. generic-function:: mean/harmonic
   :open:

   Returns the harmonic mean of a numeric sequence.

   :signature: mean/harmonic (sample) => (mean)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value mean: An instance of :drm:`<number>`.

   :description:

     Returns the harmonic mean of a numeric sequence.

     The harmonic mean is the reciprocal of the arithmetic mean of the
     reciprocals of the values of the sequence.

   :equivalent:

     The harmonic mean is given by:

     .. math::

        \frac{n}{\sum_{i=1}^{n} \frac{1}{x_{i}}}

   :seealso:

     - :gf:`mean/arithmetic`
     - :gf:`mean/fast`
     - :gf:`mean/geometric`

     - `Harmonic Mean on Wikipedia <https://en.wikipedia.org/wiki/Harmonic_mean>`__

.. method:: mean/harmonic
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`mean/harmonic` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value mean: An instance of :drm:`<double-float>`.

Scaling
-------

.. generic-function:: scale
   :open:

   :signature: scale (sample lower-bound upper-bound) => (res)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :parameter lower-bound: An instance of :drm:`<number>`.
   :parameter upper-bound: An instance of :drm:`<number>`.
   :value res: An instance of :type:`<numeric-sequence>`.

.. method:: scale
   :specializer: <double-float-vector>, <double-float>, <double-float>
   :sealed:

   A specialized implementation of :gf:`scale` for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :parameter lower-bound: An instance of :drm:`<double-float>`.
   :parameter upper-bound: An instance of :drm:`<double-float>`.
   :value res: An instance of :type:`<double-float-vector>`.

Variance and Deviation
----------------------

.. generic-function:: standard-deviation/population
   :open:

   :signature: standard-deviation/population (sample) => (standard-deviation)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value standard-deviation: An instance of :drm:`<number>`.

   :seealso:

     - :gf:`variance/population`
     - :gf:`variance/sample`
     - :gf:`standard-deviation/sample`
     - :gf:`standard-scores`

.. method:: standard-deviation/population
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`standard-deviation/population`
   for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value standard-deviation: An instance of :drm:`<double-float>`.

.. generic-function:: standard-deviation/sample
   :open:

   :signature: standard-deviation/sample (sample) => (standard-deviation)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value standard-deviation: An instance of :drm:`<number>`.

   :description:

     The standard-deviation calculation for a sample, rather than a complete
     population, uses ``sample.size - 1`` rather than the sample size. This is
     `Bessel's Correction`_.

   :seealso:

     - :gf:`variance/population`
     - :gf:`variance/sample`
     - :gf:`standard-deviation/population`

.. method:: standard-deviation/sample
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`standard-deviation/sample`
   for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value standard-deviation: An instance of :drm:`<double-float>`.

.. generic-function:: variance/population
   :open:

   :signature: variance/population (sample) => (variance)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value variance: An instance of :drm:`<number>`.

   :seealso:

     - :gf:`variance/sample`
     - :gf:`standard-deviation/population`
     - :gf:`standard-deviation/sample`

.. method:: variance/population
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`variance/population`
   for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value variance: An instance of :drm:`<double-float>`.

.. generic-function:: variance/sample
   :open:

   :signature: variance/sample (sample) => (variance)

   :parameter sample: An instance of :type:`<numeric-sequence>`.
   :value variance: An instance of :drm:`<number>`.

   :seealso:

     - :gf:`variance/population`
     - :gf:`standard-deviation/population`
     - :gf:`standard-deviation/sample`

.. method:: variance/sample
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`variance/sample`
   for :drm:`<double-float>`.

   :parameter sample: An instance of :type:`<double-float-vector>`.
   :value variance: An instance of :drm:`<double-float>`.

.. index:: z-scores
.. index:: standardize
.. generic-function:: standard-scores
   :open:

   :signature: standard-scores (population) => (scores)

   :parameter population: An instance of :type:`<numeric-sequence>`.
   :value scores: An instance of :type:`<numeric-sequence>`.

   :equivalent:

     The standard score of a value in a sequence is given by:

     .. math::

        z = {x- \mu \over \sigma}

     Where:

     * μ is the mean of the population
     * σ is the standard deviation of the population

   :seealso:

     - :gf:`mean/arithmetic`
     - :gf:`standard-deviation/population`

.. method:: standard-scores
   :specializer: <double-float-vector>
   :sealed:

   A specialized implementation of :gf:`standard-scores`
   for :drm:`<double-float>`.

   :parameter population: An instance of :type:`<double-float-vector>`.
   :value scores: An instance of :type:`<double-float-vector>`.

.. _The Art of Computer Programming: http://www.amazon.com/Art-Computer-Programming-Seminumerical-Algorithms/dp/0201896842/
.. _Bessel's Correction: https://en.wikipedia.org/wiki/Bessel%27s_correction
