# Getting approximate equality with Pytest

In adding GeoJSON and SpatiaLite support to [geocode-sqlite](https://github.com/eyeseast/geocode-sqlite), I ended up with floating point coordinates that lost precision in the between database and Python. That meant my test data and results weren't _exactly_ equal.

I thought about using [dirty-equals](https://dirty-equals.helpmanual.io/types/numeric/#dirty_equals.IsApprox) for this, for its logo alone, but I was hesitant to add another dependency.

[Fortunately, this post](https://www.scivision.dev/pytest-approx-equal-assert-allclose/) pointed me to [`pytest.approx`](https://docs.pytest.org/en/latest/reference/reference.html#pytest-approx), which was exactly what I needed.

```python
assert expected["coordinates"] == pytest.approx(geometry["coordinates"])
```

In this case, it's even comparing each item in a list, which is perfect for dealing with GeoJSON.
