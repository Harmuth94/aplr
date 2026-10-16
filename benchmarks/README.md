# Standard APLR performance comparison

The comparison uses independently installed upstream and candidate wheels in
separate Python environments. Every fit runs in a fresh process, with imports
and synthetic data generation outside the fit timer. The benchmark reports fit
time, prediction time, CPU time, process peak resident memory, package versions, extension
hashes, aggregation diagnostics, and numerical differences.

All data is generated locally from seed zero. The default campaign covers
weighted Poisson/log and Tweedie/log fits with 10,000 and 50,000 rows, eight
predictors, and 0% or 75% exact duplicate feature rows. It uses explicit fixed
three-fold memberships, 100 boosting steps, 100 bins, and one fitting thread.
Each case has three repetitions for upstream, optimized APLR with aggregation
disabled, and optimized APLR with aggregation enabled.

```bash
git worktree add --detach ../aplr-baseline <upstream-base-commit>
python -m venv ../aplr-baseline/.venv
../aplr-baseline/.venv/bin/python -m pip install '../aplr-baseline[test]'
python -m venv .venv
.venv/bin/python -m pip install '.[test]'

../aplr-baseline/.venv/bin/python benchmarks/benchmark_standard.py \
  --baseline-python ../aplr-baseline/.venv/bin/python \
  --candidate-python .venv/bin/python \
  --suite parity --output benchmarks/parity-results.json

../aplr-baseline/.venv/bin/python benchmarks/benchmark_standard.py \
  --baseline-python ../aplr-baseline/.venv/bin/python \
  --candidate-python .venv/bin/python \
  --output benchmarks/standard-results.json
```

The parity suite also covers MSE/identity, binomial/logit, classification,
custom loss/link derivatives, and DataFrame preprocessing. It loads real
upstream model pickles in the candidate environment and verifies unchanged
predictions and retained time limits. Both suites require identical selected
term structures and compare coefficients, predictions, validation histories,
and CV error with `rtol=1e-10` and `atol=1e-12`.

Performance varies with hardware and the data distribution; timing is reported
rather than asserted in CI. Peak memory includes the interpreter, data,
fitted model, predictions, and retained comparison arrays. Identical-row
aggregation can cost time when few rows can be combined.
It retains original training rows for compatible final scoring, so fewer
working rows do not necessarily mean lower process peak memory.
