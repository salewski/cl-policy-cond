                             POLICY-COND
                             ===========

                           By Robert Smith

License
-------

This software is licensed under BSD 3-clause license. Please see LICENSE.


Environment Introspection
-------------------------

This library provides an interface to the CLtL2 environment
functions. Currently, only DECLARATION-INFORMATION is supported.


Temporarily Setting and Restoring Optimize Policy
-------------------------------------------------

One can use WITH-POLICY to temporarily set the global optimize policy
for e.g. loading systems. The original policy will be restored upon
completion.

WITH-POLICY should not be used with local declarations. In that case,
one should use LOCALLY.


Expansion Based on Policy
-------------------------
                           
POLICY-COND is a macro in order to select certain code paths based on
the current optimize compiler policy.

For example, given the following code:

(declaim (optimize (speed 0) (safety 3)))

(defun test-cond ()
  (policy-cond
    ((> speed safety) (+ 1 1))
    ((= speed safety) (+ 2 2))
    ((< speed safety) (+ 3 3))))

The function TEST-COND will get compiled as if it were

(defun test-cond ()
  (+ 3 3))

The optimize qualities SPEED, SAFETY, SPACE, DEBUG, and
COMPILATION-SPEED are guaranteed by an implementation. They can be
used as if they are lexically bound.

Currently, any expression for the policy expression can be used. In
the future, this might change to a limited set of operators.

Also included is POLICY-IF which behaves much like POLICY-COND, except
is akin to CL:IF.

Finally there is another package, POLICY, which exports IF, which is
intended to be used with reader macros. For example,

    #+#.(policy:if (<= speed safety)) (safe-algorithm)

Note that this does not work with local declarations. See

    http://clhs.lisp.se/Body/s_declar.htm#declare

for details.


Expectations
------------

An "expectation" is something the programmer expects to be true, but
could be wrong if the consumer of the code makes a logical
error. Expectations usually have different behavior in testing and
production environments. When testing, it is permitted that code be
slower due to sanity checking, and in production (after considerable
testing), it may make sense to remove extra sanity checking and add
speed improvements.

POLICY-COND offers the notion of an expectation, which can change with
policy. The macro POLICY-COND:WITH-EXPECTATIONS accomplishes this. It
is best described with an example.

    (defun vector-item (vec n)
      (with-expectations (> speed safety)
          ((type unsigned-byte         n)
           (type (vector single-float) vec)
           (assertion (< n (length vec))))
        (aref vec n)))

If the policy expression is not satisfied, then it will expand into

    (PROGN
      (CHECK-TYPE N UNSIGNED-BYTE)
      (CHECK-TYPE VEC (VECTOR SINGLE-FLOAT))
      (ASSERT (< N (LENGTH VEC)))
      (AREF VEC N))

But if it is satisfied, it will expand into

    (LOCALLY (DECLARE (TYPE UNSIGNED-BYTE N)
                      (TYPE (VECTOR SINGLE-FLOAT) VEC))
      (AREF VEC N))

In other words, we can read

    (with-expectations POLICY (EXPECTATIONS...) BODY...)

as

    "With the expectation that EXPECTATIONS are met when POLICY is
    true, execute BODY. Otherwise, ensure that they're true."

See the documentation string for WITH-EXPECTATIONS to see the kinds of
expectation clauses supported.


See Also
--------

For similar functionality for static function dispatch, see:

    https://bitbucket.org/tarballs_are_good/parameterized-function