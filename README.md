Dr. Fred Edison's incredible useful C++14 libraries
===================================================
May contain traces of industrial strength snake oil.

This is a set of small single-header libraries. They require no installation
and only need a C++14 compatible compiler. To use any of them just copy the
header file into your own project and include it where needed.
If you are using the [CMake][cmake] build system you can also add the full
project as a subdirectory and use any of the libraries by linking with
the `dfelibs` target, i.e.

    add_subdirectory(<path/to/dfelibs/folder>)
    ...
    target_link_library(<your/project/target> dfelibs)

All libraries are licensed under the terms of the [MIT license][mit_license].

Dispatcher
----------

Register arbitrary functions with the dispatcher

```cpp
#include "dfe_dispatcher.hpp"

void func1(int x, double y, std::string z) { ... }
int func2(float a, unsigned int b) { ... }

dfe::Dispatcher dispatch;
dispatch.add("a_function", func1);
dispatch.add("another_function", func2);
```

and call them by name with regular arguments

```cpp
dispatch.call("a_function", 12, 0.23, "a message");
dispatch.call("another_function", 3.14f, 23).as<int>();
```

or with a list of string arguments that are automatically parsed
into the correct types

```cpp
dispatch.call_parsed("a_function", {"12", "0.23", "a message"});
```

Histogram
---------

Compose a multi-dimensional histogram with configurable axis types

```cpp
#include "dfe_histogram.hpp"

using H3 =
  dfe::Histogram<float,
    dfe::UniformAxis<float>,
    dfe::OverflowAxis<float>,
    dfe::VariableAxis<float>>;

// axis 0: 16 bins of uniform size, no under/overflow bins
// axis 1: 8 bins of uniform size, additional under/overflow bins
// axis 2: 4 bins of variable size, no under/overflow bins
H3 h({0.0, 1.0, 16}, {-2.0, 2.0, 8}, {1.0, 10.0, 20.0, 30.0, 100.0});
```

and fill it with weighted or unweighted data

```cpp
h1.fill(0.5, 0.0, 25);       // weight = 1
h1.fill(0.25, 1.0, 65, 0.3); // weight = 0.3
h1.fill(0.25, 4.0, 65);      // axis 1 overflow
h1.fill(2.25, 1.0, 65);      // fails, due to axis 0 overflow
```

Namedtuple
----------

Add some self-awareness to a POD type

```cpp
#include "dfe_namedtuple.hpp"

struct Record {
  uint32_t x;
  float b;
  int16_t z;
  DFE_NAMEDTUPLE(Record, x, b, z);
}
```

and write it to disk in multiple formats:

```cpp
dfe::CsvNamedtupleWriter<Record> csv("records.csv"); // or
dfe::TsvNamedtupleWriter<Record> tsv("records.tsv"); // or
dfe::NpyNamedtupleWriter<Record> npy("records.npy");

csv.append(Record{1, 1.4, -2}); // same call for other writers
```

The results are either comma-separated text values

    x,b,z
    1,1.4,-2
    ...

or tab-separated text values

    x       b       z
    1       1.4     -2
    ...

or binary [NPY][npy] data. Data stored in one of the text-based formats can
also be read back in:

```cpp
dfe::TsvNamedtupleReader<Record> tsv("records.tsv");

Record data;
tsv.read(data);
```

Poly
----

Evaluate polynomial functions and their derivatives using either a
(variable-sized) container of coefficients

```cpp
#include "dfe_poly.h"

std:vector<double> coeffs = {0.0, 2.0, 3.0, 4.0};
// evaluate f(x) = 2*x + 3*x^2 + 4*x^3 at x=-1.0
double y = dfe::polynomial_val(-1.0, coeffs);
// evaluate df(x)/dx at x=-0.5
double dy = dfe::polynomial_der(-0.5, coeffs);
// evaluate both f(x) and df(x)/dx at x=2.0 at the same time
std::pair<double, double> ydy = dfe::polynomial_valder(2.0, coeffs);
```

or using the coefficients directly for fixed order polynomials:

```cpp
// evaluate f(x) = 0.25 + x + 0.75*x^2 at x=0.5
float y = dfe::polynomial_val(0.5f, {0.25f, 1.0f, 0.75f});
```


[cmake]: https://www.cmake.org
[mit_license]: https://opensource.org/licenses/MIT
[npy]: https://docs.scipy.org/doc/numpy/neps/npy-format.html
