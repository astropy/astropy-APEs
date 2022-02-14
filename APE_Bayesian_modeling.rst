Enhance Modeling to Support Probability Techniques
--------------------------------------------------

author: Karl Gordon, Nadia, Dencheva, William Jamieson, Erik Tollerud

date-created: 2022 Mar ??

date-last-revised: 2022 Feb 14

date-accepted:

type: Standard Track

status: Discussion


Abstract
--------

Enhance astropy modeling to support probability techniques in addition to the
currently supported fequentist techniques.  Specifically to fully implement
priors as part of the model, including fitting using probabilities
(i.e., prior times likelihood), and add sampling techniques to fitting.


Detailed description
--------------------

The use of probabilistic techniques when modeling data while always happened in
astronomy, it has been become very common in the last few years (e.g., the use
of Bayesian techniques).  Currently, astropy modeling does not provide much
support for such techniques focusing the fitting on minimizing the simple chisqr
likelihood.

Probabilistic techniques work by maximizing the probability of a fit to data
and usually explicitly include priors in the fitting statistic.  The combination
of the prior and the likelihood probabilities is the posterior probability.
Priors reflect prior knowledge of model parameters.  Probabilistic based
fitting can result in the best fit model (just like minimizing the chisqr
fitting) and/or posterior functions giving the probability of the model
parameters.

***instruction text***
This section describes the need for the APE.  It should describe the existing
problem that it is trying to solve and why this APE makes the situation better.
It should include examples of how the new functionality would be used and
perhaps some use cases.


Branches and pull requests
--------------------------

***instruction text***
Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

The implementation should proceed roughly as ordered below.  Specifically,
first the priors, then the statistics subsections.  The next two subsections,
fitters and evidence may be developed in either order.

Priors
======

Three different ways of specifying a prior on a model should be supported.

* The existing `bounds` attribute of `Parameter` should be interpreted as
  a flat prior between the min/max values given in `bounds`.
* The `prior` attribute of the `Parameter` class can be set to any 1D function
  that takes a single value and returns the prior evaluated at that value.
  This could be any astropy 1D model or a user written function.
* For more complicated priors that simultaneously depend on multiple parameters,
  a user written function that takes all the parameters as input and returns
  the prior evaluated at those parameters.

The implementation of the 1st two ways will be done by having a generic prior
function that is like the 3rd way that takes all the parameters as input.
This generic prior function then uses any `Parameter.prior` functions and then
any `Parameter.bounds` implied flat prior functions.  For any `Parameters`
without any prior, this function would just give a constant prior (e.g., 1).

Statistics
==========

A statistic that combines the likelihood the data fits the model with the
prior function will be implemented.  This is called the posterior and is the
multiplication of the likelihood function by the prior function.  The maximum
of the posterior provides the best fit model.  For numerical reasons, the
natural log of the likelihood and prior function are added and the posterior
probability is reported as ln(prob).

The current set of fitters are based on optimizers that minimize the statistic
and the only currently implemented statistic is the Gaussian chisqr.  At a
minimum, a new statistic that combines a Gaussian probability (i.e.,
exp(-chisqr/2)) with the prior function is needed.  An alternative would be to
create a new class (`Posterior`?) that combined both a statistic (i.e.,
likelilhood) and prior giving the function to be maximized.

Fitters
=======

A new class of fitters need to be added.  This is the Sampler class that would
provide the interface to different samplers.  Unlike the existing Optimizers
where the best fitting model is the result, Samplers provide a sampling of the
posterior where the number of samples is set as an input.  Effectively, the
samplers provide a set of models and the set reflects the posterior. This set
of models is usually returned as a 1D array of for each model parameter
(called a chain) where the posterior is reflected as the frequency of a model
parameter in the chain or as a parallel 1D array of posterior weights.
The chain for each parameter will be stored in `Parameter.posterior` as an
astropy.uncertainty distribution.

At least two samplers will be implemented.  The specific samplers planned are
`emcee` and `dynasty`.

The current `Optimizer` class should be abstracted to a generic class
(`Solver`?) that provides the common portions of the generic `Optimizer` and
`Sampler` classes.  The `Optimizer` and `Sampler` will provide an specific
functionality needed for these two different fitting methods.  All
implementations of optimizers and samplers will be subclassed from these two
generic classes that are both subclassed from the generic `Solver` class.

Implementing the `dynasty` sampler will require a weighted distribution to be
implemented in the astropy.uncertainty.

Evidence
========

A useful statistic from probabilistic inference is the evidence.  The evidence
is the integration over all of parameters space of the posterior.  A function
to compute the evidence should be implemented (probably using the
`Parameter.posterior` attribute).

***instruction text***
This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.

Serialization
=============

As much as possible, the added functionality should be serializable.  [More
details needed by someone who can provide details.]

Backward compatibility
----------------------

***instruction text***
This section describes the ways in which the APE breaks backward compatibility.


Alternatives
------------

***instruction text***
If there were any alternative solutions to solving the same problem, they should
be discussed here, along with a justification for the chosen approach.


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
