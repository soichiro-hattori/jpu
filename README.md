# JAX + Units

**Built with [JAX](https://jax.readthedocs.io) and
[Pint](https://pint.readthedocs.io)!**

This module provides and interface between [JAX](https://jax.readthedocs.io) and
[Pint](https://pint.readthedocs.io) to allow JAX to support operations units.
For example:

```python
>>> import jax
>>> import jax.numpy as jnp
>>> import jpu
>>> u = jpu.UnitRegistry()
>>>
>>> @jax.jit
... def add_two_lengths(a, b):
...     return a + b
...
>>> add_two_lengths(3 * u.m, jnp.array([4.5, 1.2, 3.9]) * u.cm)
<Quantity([3.045 3.012 3.039], 'meter')>

```

## Installation

To install, use `pip`:

```bash
python -m pip install jpu
```

The only dependencies are `jax` and `pint`, and these will also be installed, if
not already in your environment. Take a look at [the JAX docs for more
information about installing JAX on different
systems](https://github.com/google/jax#installation).

## Usage

This is meant as a proof of concept to show how one might go about supporting
units as JAX arguments. It's non-trivial at this point to implement numpy
`ufunc`s for these custom types because of technical reasons (see
`__jax_array__` interface controversies, for example). Importantly the
`__jax_array__` and `pytree` interfaces are not currently compatible. So, until
a better custom dispatching interface is supported, the basic idea is to provide
a `pytree` type that knows about units, and then overload the `jax.numpy`
functions that we need.

Here's an example for the kind of way that you might use this proof of concept:

```python
import jax
import numpy as np
from jpu import UnitRegistry, numpy as jnpu

u = UnitRegistry()

@jax.jit
def projectile_motion(v_init, theta, time, g=u.standard_gravity):
    """Compute the motion of a projectile with support for units"""
    x = v_init * time * jnpu.cos(theta)
    y = v_init * time * jnpu.sin(theta) - 0.5 * g * jnpu.square(time)
    return x.to(u.m), y.to(u.m)

x, y = projectile_motion(
    5.0 * u.km / u.h, 60 * u.deg, np.linspace(0, 1, 50) * u.s
)
```

The key point is that this function can take parameters with any (compatible!)
units, while we can still be confident that the units will be correct when used
in the function body.
