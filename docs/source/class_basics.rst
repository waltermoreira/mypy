Class basics
============

Instance and class attributes
*****************************

Mypy type checker detects if you are trying to access a missing
attribute, which is a very common programming error. For this to work
correctly, instance and class attributes must be defined or
initialized within the class. Mypy infers the types of attributes:

.. code-block:: python

   class A:
       def __init__(self, x: int) -> None:
           self.x = x     # Attribute x of type int

   a = A(1)
   a.x = 2       # OK
   a.y = 3       # Error: A has no attribute y

This is a bit like each class having an implicitly defined
``__slots__`` attribute. In Python semantics this is only enforced
during type checking: at runtime we use standard Python semantics. You
can selectively define a class as *dynamic*; dynamic classes have
Python-like compile-time semantics, and they allow you to assign to
arbitrary attributes anywhere in a program without the type checker
complaining:

.. code-block:: python

   from typing import Dynamic

   class A(Dynamic):
       pass

   a = A()
   a.x = 2     # OK, no need to define x explicitly.

Mypy also lets you read arbitrary attributes of dynamic class
instances. This limits type checking effectiveness, so you should only
use dynamic classes when you really need them.

.. note::

   Dynamic classes are not implemented in the current mypy version.

You can declare variables in the class body explicitly using
``Undefined`` or a type comment:

.. code-block:: python

   class A:
       x = Undefined(List[int])  # Declare attribute y of type List[int]
       y = 0  # type: Any        # Declare attribute x of type Any

   a = A()
   a.x = [1]     # OK

As in Python, a variable defined in the class body can used as a class
or an instance variable.

Similarly, you can give explicit types to instance variables defined
in a method:

.. code-block:: python

   class A:
       def __init__(self) -> None:
           self.x = Undefined(List[int])     # OK

       def f(self) -> None:
           self.y = 0 # type: Any            # OK

You can only define an instance variable within a method if you assign
to it explicitly using ``self``:

.. code-block:: python

   class A:
       def __init__(self) -> None:
           self.y = 1   # Define y
           a = self
           a.x = 1      # Error: x not defined

Overriding statically typed methods
***********************************

When overriding a statically typed method, mypy checks that the
override has a compatible signature:

.. code-block:: python

   class A:
       def f(self, x: int) -> None:
           ...

   class B(A):
       def f(self, x: str) -> None:   # Error: type of x incompatible
           ...

   class C(A):
       def f(self, x: int, y: int) -> None:  # Error: too many arguments
           ...

   class D(A):
       def f(self, x: int) -> None:   # OK
           ...

.. note::

   You can also vary return types **covariantly** in overriding. For
   example, you could override the return type ``object`` with a subtype
   such as ``int``.

You can also override a statically typed method with a dynamically
typed one. This allows dynamically typed code to override methods
defined in library classes without worrying about their type
signatures, similar to Python.

There is no runtime enforcement that the method override returns a
value that is compatible with the original return type, since types
are erased in the Python semantics:

.. code-block:: python

   class A:
       def inc(self, x: int) -> int:
           return x + 1

   class B(A):
       def inc(self, x):       # Override, dynamically typed
           return 'hello'

   b = B()
   print(b.inc(1))   # hello
   a = b # type: A
   print(a.inc(1))   # hello

Abstract base classes and multiple inheritance
**********************************************

Mypy uses Python abstract base classes for protocol types. There are
several built-in abstract base classes types (for example,
``Sequence``, ``Iterable`` and ``Iterator``). You can define abstract
base classes using the ``abc.ABCMeta`` metaclass and the
``abc.abstractmethod`` function decorator.

.. code-block:: python

   from abc import ABCMeta, abstractmethod
   import typing

   class A(metaclass=ABCMeta):
       @abstractmethod
       def foo(self, x: int) -> None: pass

       @abstractmethod
       def bar(self) -> str: pass

   class B(A):
       def foo(self, x: int) -> None: ...
       def bar(self -> str:
           return 'x'

   a = A() # Error: A is abstract
   b = B() # OK

Unlike most Python code, abstract base classes are likely to play a
significant role in many complex mypy programs.

A class can inherit any number of classes, both abstract and
concrete. As with normal overrides, a dynamically typed method can
implement a statically typed abstract method defined in an abstract
base class.

.. note::

   There are also plans to support more Python-style "duck typing" in
   the type system. The details are still open.
