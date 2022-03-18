Enhance Modeling to Support Probabilistic Fitting
-------------------------------------------------

author: Karl Gordon, Nadia, Dencheva, William Jamieson, Erik Tollerud

date-created: 2022 Feb 14

date-last-revised: 2022 Mar 17

date-accepted:

type: Standard Track

status: Discussion


Abstract
--------

We propose to enhance ``astropy.modeling`` to support general probabilistic techniques in addition
to the currently supported fequentist philosophy.  This would include fully supporting
priors as part of the model, fitting using a more general Bayesian-compatible
approach to the objective function (i.e., prior times likelihood), and adding
support for sampling techniques to ``fitting``.


Detailed description
--------------------

The use of probabilistic techniques when modeling data has long been a part of
astronomy, but it has become much more common in the last few years, particularly
with the dramatic growth of Bayesian methods.  Currently, ``astropy.modeling`` does not
provide support for such techniques, as it focuses the fitting on minimizing
chi-squared likelihoods.  While this has proven to be a reasonable starting approach given the
wide range of techniques ``scipy`` provides, modern astronomy requires more
flexibility.

Probilistic fitting techniques explicitly assume data are inherently probabilistic,
and Bayesian inference takes this one step further to also allow fitted models to
be probabilistic and include the effects of assumptions before the data are measured - i.e.
priors. This is statistically formalized via Bayes' Theorem, and while there are
a wide range of Bayesian methods (more than Astropy could ever hope to support)
in astronomy, a very common use case is a physical model for the data with an
explicit data distribution around that model.  This makes it relatively
straightforward to explicitly express the prior and the likelihood function.
With an appropriate statistical sampling method (e.g. Markov Chain Monte Carlo),
these can be used to infer the posterior probability distribution of the
parameters of the model.  This leads to a more statistically rigorous and
often more useful version of "fitting a model" . That is, probabilistic
fitting can result in the best fit model (just like minimizing the existing
fitting that focuses on minimization of an objective function like chi-squared)
*and* provide a fully probabilistic description of the model that captures
uncertainties as fully as the model allows.

However, all of this is not possible with the current implementation of
``modeling``, as models assume that parameters are a specific value, not a
distribution. While this makes understanding models simpler, it also makes the
probabilistic approach to fitting impossible.  This APE aims to remove that
limitation, while still allowing the existing simpler approach to modeling for
the existing cases where it is already of clear use.

This APE will also enhance ``fitting`` to allow the use of probabilistic samplers
that are necessary for Bayesian inference.  Note though, that this APE does
*not* aim to include samplers in ``astropy`` itself, but rather provide layers that
interface with samplers already used in the astronomy community like emcee_ or
dynesty_ . This is akin to the already-existing approach ``fitting`` takes of
using ``scipy``'s fitters but wrapping them in a more astronomy-friendly
interface.

Branches and pull requests
--------------------------

N/A

(note we should probably mention the proof-of-concept tests we've done here, though.)

Exploration of some of the ideas in this APE was done in
https://urldefense.com/v3/__https://github.com/astropy/astropy/pull/10839__;!!CrWY41Z8OgsX0i-WU-0LuAcUu2o!iJ81QawZILE3axNS7djxPiZ8NspaMrnko8faGJqIfW4SFwptwnmT0JcKhBpBVBrDNA$ 

***instruction text***
Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

The implementation should proceed roughly as ordered below.  Specifically,
first the :ref:`priors<Priors>`, then the :ref:`statistics<Statistics>` subsections.
The next two subsections, :ref:`fitters<Fitters>` and :ref:`evidence<Evidence>`
may be developed in either order.

Priors
======

Three different ways of specifying a prior on a model should be supported.

* The existing ``bounds`` attribute of ``Parameter`` should be interpreted as
  a flat prior between the min/max values given in ``bounds``.
* The ``prior`` attribute of the ``Parameter`` class can be set to any 1D function
  that takes a single value and returns the prior evaluated at that value.
  This could be any astropy 1D model or a user written function.
* For more complicated priors that simultaneously depend on multiple parameters,
  can be defined as a user written function that takes all the parameters as input
  and returns the prior evaluated at those parameters.

The implementation of the first two ways will be done by having a generic ``prior``
function similar to the third way that takes all the parameters as input.
This generic ``prior`` function then uses any ``Parameter.prior`` functions and then
any ``Parameter.bounds`` implied flat prior functions.  For any ``Parameters``
without any prior, this function would just give a constant prior (e.g., 1).


.. code-block:: python

    class Parameter:
        ...
        self._prior = <user_supplied_prior>

        @property
        def prior(self):
            if self._prior is not None:
                return self._prior
            elif (not any(b is None for b in self.bounds)) and
                self.value > self.min and self.value < self.max:
                return 1.0
            else:
                return 0.0


    class Model:
        ...
        def prior(self):
            """
            Returns the combined priors of all parameters.

            Evaluates all priors and returns their product.
            """


Statistics
==========

A statistic that combines the likelihood that the data fits the model with the
prior function will be implemented.  This is called the posterior and is the
multiplication of the likelihood function by the prior function.  The maximum
of the posterior provides the best fit model.  For numerical reasons, the
natural log of the likelihood and prior function are added and the posterior
probability is reported as ln(prob).

The current set of fitters are based on optimizers that minimize the statistic
and the only currently implemented statistic is the Gaussian chisqr.  At a
minimum, a new statistic that combines a Gaussian probability (i.e.,
exp(-chisqr/2)) with the prior function is needed.  An alternative would be to
create a new class (`Posterior`?) that combines both a statistic (i.e.,
likelilhood) and prior allowing the function to be maximized.

Fitters
=======

A new class of fitters needs to be added.  This is the ``Sampler`` class that would
provide the interface to different samplers.  Unlike the existing Optimizers
where the best fitting model is the result, Samplers provide a sampling of the
posterior where the number of samples is set as an input.  Effectively, the
samplers provide a set of models and the set reflects the posterior. This set
of models is usually returned as a 1D array of for each model parameter
(called a chain) where the posterior is reflected as the frequency of a model
parameter in the chain or as a parallel 1D array of posterior weights.
The chain for each parameter will be stored in `Parameter.sampled_posterior` as an
``astropy.uncertainty`` distribution.

At least two samplers will be implemented.  The specific samplers planned are
`emcee` and `dynasty`.

The current ``Optimizer`` class should be abstracted to a generic class
(``Solver``?) that provides the common portions of the generic ```Optimizer`` and
``Sampler`` classes.  The ``Optimizer`` and ``Sampler`` will provide a specific
functionality needed for these two different fitting methods.  All
implementations of optimizers and samplers will be subclassed from these two
generic classes that are both subclassed from the generic ``Solver`` class.

Implementing the ``dynasty`` sampler will require a weighted distribution to be
implemented in ``astropy.uncertainty``.

Evidence
========

A useful statistic from probabilistic inference is the evidence.  The evidence
is the integration over all of parameters space of the posterior.  A function
to compute the evidence should be implemented (probably using the
``Parameter.sampled_posterior`` attribute).

***instruction text***
This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.

Serialization
=============

As much as possible, the added functionality should be serializable.
Currently all models are serializable using the Advanced Scientific Data Format (asdf_).
Serializing priors and posteriors of a ``Model`` will be added to asdf-astropy.
If a prior or posterior is specified as one of the models in astropy, serialization
will be automatic. If it's defined as an ```astropy.Model`` subclass the serialization
code can be written by the user. General functions cannot be serialized.

Backward compatibility
----------------------

The goal of this APE is to not break backwards compatibility at all, but
rather to allow previous code to work while also adding the new capabilities.

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

.. _emcee: https://emcee.readthedocs.io/
.. _dynesty: https://dynesty.readthedocs.io/
.. _ASDF: https://asdf-standard.readthedocs.io/
