---
layout: post
title: Why linearity?
tags:
  - blog
  - math
---

A lot of my teaching focuses on linearity -- I literally teach a course called "[linear systems theory](https://canvas.uw.edu/courses/1431472/)", but linearity features prominently in:  subsequent courses on "[linear multivariable control](https://canvas.uw.edu/courses/1447847)" (go figure) and "[robust control](https://canvas.uw.edu/courses/1311510)"; undergraduate course "[control system analysis](https://canvas.uw.edu/courses/1477838)"; and even special topics courses on "[hybrid systems](https://canvas.uw.edu/courses/1153606)" and "[optimization and learning for control](https://canvas.uw.edu/courses/1352564)".

All of this leads many students in my courses to ask, at various times and at varying levels of politeness: ***"why linearity?"***  This blog post is my long-winded attempt to explain my standard answer to these inquiries:  "why linearity?  because ***there's nothing else worth learning!"***

* toc placeholder -- gets replaced automagically
{:toc}

## background
The expert reader may wish to skim or skip this section, returning only when and as questions arise in subsequent sections.  But I use some not-entirely-standard notation and conventions (that I think have advantages to recommend them becoming more standard), so everyone may benefit from this resource.

There are two key suites of tools we'll use extensively in what follows:  *linear algebra* and *(the) Calculus*.  For reasons that will become clear, it is natural to discuss the former first and the latter second.  But even before we get there, it's helpful to agree on some notation and terminology that underlies both.

## background -- vector spaces (~2hrs)
I'm going to assume familiarity with [abstract vector spaces](https://en.wikipedia.org/wiki/Vector_space) and [linear functions](https://en.wikipedia.org/wiki/Linear_map) -- good, free references on these topics are the books of [Boyd & Vandenberghe](https://web.stanford.edu/~boyd/vmls/) and [Hefferon](https://joshua.smcvt.edu/linearalgebra/), though I won't necessarily adopt notation or conventions from either.

### construction
My favorite way to "construct" vector spaces is through [a standard (ab)use of exponent notation](https://math.stackexchange.com/questions/901735/meaning-of-a-set-in-the-exponent):  with $$A^B = \{ f : A \rightarrow B \}$$ denoting the set of functions from $$A$$ to $$B$$ ([viz.](https://en.wikipedia.org/wiki/Viz.) one of the *[power set](https://en.wikipedia.org/wiki/Power_set)* notations), we can [naturally](https://en.wikipedia.org/wiki/Category_theory) regard $$\mathbb{R}^S$$ as a vector space *for any set $$S$$* over the reals (you're welcome to replace $$\mathbb{R}$$ with any [field](https://en.wikipedia.org/wiki/Category_theory) $$\mathbb{F}$$ here and in what follows; also, for both clarity's and sanity's sakes, "[the reals](https://en.wikipedia.org/wiki/Real_number)" should not be confused with "[the real](https://en.wikipedia.org/wiki/The_Real)") with vector addition and scalar multiplication defined element-wise:

$$(\ast)\quad \forall s \in S,\ \alpha\in\mathbb{R},\ f,g\in\mathbb{R}^S: (\alpha \cdot f + g)(s) = \alpha\cdot f(s) + g(s). $$

Did you catch that?  The algebraic sleight-of-hand in $$(\ast)$$?  At the risk of [spoiling the joke](https://www.quora.com/Why-does-explaining-a-joke-make-it-less-funny), let me break it down.

### spoiling the joke
On the [LHS](https://en.wikipedia.org/wiki/Sides_of_an_equation), we have $$\alpha\cdot f + g$$ -- an expression that, at this point, is doubly-undefined:  I haven't told you how to interpret ***either*** the scalar-function "multiplication" $$\alpha\cdot f$$ ***or*** the function-function "addition" $$h + g$$ for $$\alpha\in\mathbb{R},\ f,g,h\in\mathbb{R}^S$$.  

  But on the [RHS](https://en.wikipedia.org/wiki/Sides_of_an_equation) we have an unambiguous expression:  since $$f,g\in\mathbb{R}^S$$ means $$f$$ and $$g$$ are scalar-valued functions $$f,g:S\rightarrow\mathbb{R}$$, and since I've already specified that $$\mathbb{R}$$ denotes the [field of "real" numbers](https://en.wikipedia.org/wiki/Real_number), this expression consists solely of scalar multiplication ($$\alpha\cdot f(s)$$) and scalar addition ($$a + b$$ with $$a = \alpha\cdot f(s)$$ and $$b = g(s)$$) of real numbers, which is something we know how to do (though maybe only [in theory](https://doi.org/10.1007/978-1-4612-0701-6)).

So $$(\ast)$$ actually accomplishes quite a lot:  it defines [binary operations](https://en.wikipedia.org/wiki/Binary_operation) (one which is *external*, i.e. a "[group action](https://en.wikipedia.org/wiki/Group_action)", though I personally don't lean into this terminology), namely $$\cdot:\mathbb{R}\times S\rightarrow S$$ and $$+:S\times S\rightarrow S$$, and in so doing, endows $$\mathbb{R}^S$$ with the structure of a(n abstract) [vector space](https://en.wikipedia.org/wiki/Vector_space) (as an instructive exercise, the reader may wish to verify the [vector space axioms](https://en.wikipedia.org/wiki/Vector_space#Definition_and_basic_properties) are satisfied).

### objections
The attentive reader may justifiably complain about the abuse of notation in $$(\ast)$$ since "$$\cdot$$" and "$$+$$" were already defined (!) -- but I claim that there is (i) no actual risk of confusion but (ii) significant benefit to permitting the abuse to continue.

At this point you may be feeling like I have already written wayy too much to explain / justify / rationalize my preferred notation for vector spaces ("[The lady doth protest too much, methinks](https://en.wikipedia.org/wiki/Vector_space#Definition_and_basic_properties)"?), so let's get to the good part:  with $$(\ast)$$ internalized, we can directly construct many of our favorite vector spaces :)

### examples

**Euclidean spaces:**
Consider the classic (I have [Nate Bottman](https://natebottman.github.io/) to thank for originally alerting me to this example):  $$\mathbb{R}^n$$ for natural number $$n\in\mathbb{N}$$.  But wait, you may be thinking -- a number $$n$$ isn't a set!  But [it is](https://en.wikipedia.org/wiki/Set-theoretic_definition_of_natural_numbers#Definition_as_von_Neumann_ordinals) or, perhaps more forgivingly, regardless of whether it is or isn't we can adopt the convention in this case that the number $$n$$ is a stand-in for an arbitrary set with $$n$$ elements.  Another notational sleight-of-hand, sure, but one that is cute enough to recommend it.  However, employing [von Neumann / Zermelo ordinals](https://en.wikipedia.org/wiki/Set-theoretic_definition_of_natural_numbers#Definition_as_von_Neumann_ordinals) confers the twin advantages of (i) not requiring any abuse of notation and (ii) naturally coinciding with our standard interpretation of $$\mathbb{R}^n$$ as a(n ordered) sequence of $$n$$ real numbers (since "$$n$$" literally equals $$\{0,1,\dots,n-1\}$$ in the von Neumann / Zermelo ordinals -- yielding the additional bonus advantage of (iii) [being zero-indexed](https://numpy.org/doc/stable/user/numpy-for-matlab-users.html)).

**matrices:**
What about $$\mathbb{R}^{m\times n}$$ for natural numbers $$m,n\in\mathbb{N}$$?  Well, if I adopt the convention that "$$m\times n$$" is a stand-in for $$\{(i,j) : i\in\{1,\dots,m\},j\in\{1,\dots,n\}\}$$ then $$(\ast)$$ tells us that "$$\mathbb{R}^{m\times n}$$" is the set of $$m\times n$$ matrices endowed with [the vector space structure we know and love](https://numpy.org/doc/stable/user/basics.broadcasting.html) (element-wise addition and scalar multiplication).

If the preceding were the only cases where this notation comes up, you would be justified in pitching a fit at this point.  But this notation is a gift that keeps on giving!

**functions:** In fields like [control theory](https://en.wikipedia.org/wiki/Control_theory), [signal processing](https://en.wikipedia.org/wiki/Signal_processing), [statistics](https://en.wikipedia.org/wiki/Reinforcement_learning) and [mathematical optimization](https://en.wikipedia.org/wiki/Mathematical_optimization) (as well as [machine learning](https://en.wikipedia.org/wiki/Machine_learning), [artificial intelligence](https://en.wikipedia.org/wiki/Artificial_intelligence), and [whatever else they're calling these things nowadays](https://dx.doi.org/10.1146/annurev-control-053018-023825)), we deal extensively with vector spaces of [signals](https://en.wikipedia.org/wiki/Discrete_time_and_continuous_time) (i.e. functions from time to state or input), [statistic](https://en.wikipedia.org/wiki/Statistic)s (ahem; i.e. functions from data to quantities of interest), [values](https://en.wikipedia.org/wiki/Value_function) (i.e. functions from decisions to costs), [policies](https://en.wikipedia.org/wiki/Reinforcement_learning) (i.e. functions from states to actions), &c.  In each case we're talking about a vector space of functions, and both (i) the space itself as well as (ii) the operations used to scale or add elements of the space can be compactly denoted using the preceding exponential notation:
* signal $$(s:[0,T]\rightarrow S) \in S^{[0,T]}$$ over ("continuous") time interval $$[0,T]\subset\mathbb{R}$$ (or "discrete" time interval $$[0,T]\subset\mathbb{N}$$) where $$S$$ is itself a vector space (e.g. state $$x(t)\in\mathbb{R}^d$$, input $$u(t)\in\mathbb{R}^d$$ for $$t\in[0,T]$$);
* statistic $$(\psi:\mathcal{D}\rightarrow V)\in \mathbb{R}^D$$ over datasets $$\mathcal{D}$$ where $$V$$ is itself a vector space (e.g. mean $$\mu(D)\in\mathbb{R}^d$$, covariance $$\Sigma(D)\in\mathbb{R}^{d\times d}$$ for $$D\subset\mathbb{R}^d$$, $$D\in\mathcal{D}$$);
* value $$(v:X\rightarrow\mathbb{R})\in\mathbb{R}^U$$ over independent variables $$x\in X$$ in optimization problem $$\min_{u\in U} c(x,u)$$ (e.g. $$v(x) = \arg\min_{u\in U} c(x,u)$$ for cost $$c:X\times U\rightarrow\mathbb{R}$$ with dependent / decision variables $$u\in U$$);
* policy $$(\pi:X\rightarrow \Delta(U))\in \Delta(U)^X$$ where $$\Delta(U)$$ denotes the set of probability distributions over finite set $$U$$ (e.g. $$\Delta(U) = \{p\in[0,1]^{\#(U)} : \sum p = 1\}$$ where $$\#(U)$$ denotes the [number of elements](https://en.wikipedia.org/wiki/Cardinality) in set $$U$$).

Still, notational compactness alone would probably not suffice to recommend this notation.  The other key advantage I see is that it emphasizes the algebraic properties satisfied by functions of interest in disparate fields.  So we don't need to be (as) intimidated by the prospect of trying to do linear algebra or ("the") Calculus in function spaces (which, at the risk of spoiling the surprise, I think are the ***only*** things we can actually do in practice).

