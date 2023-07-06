**PRISM -- A new toolbox for parameterising battery models**

Faraday Institution Multiscale Modelling Project + EU 'Intelligent'
Project

David Howey, Brady Planden, \<*add others in due course*\>

22^nd^ June 2023 (following a workshop on 7^th^ June[^1])

**Overall objective**

PRISM will be a new open-source software library for parameterising
battery models.

**Motivation**

Battery models are complex and involve many parameters (e.g.,
resistances/capacitances, diffusion timescales, etc.). Whilst some
parameters might be obtained from first principles (e.g., via atomistic
simulations or similar), in general, battery models must be
parameterised from experimental measurements. This is challenging
because:

-   It's not clear whether the parameters are unique (in principle or in
    practice).

-   Users may not be experts in the field of system identification and
    modelling but might still wish to know whether their models are fit
    for purpose.

-   It's not always obvious what excitation signals should be used to
    gather test data, and what techniques and workflows should be used
    to parameterise models.

-   Battery models are used over wide temperature and C-rates and the
    model parameters need to be estimated accordingly.

**Academic and scientific novelty**

Although there has been a large amount of research on battery model
parameterisation, and system identification in general, it remains a
relatively uncharted area due to the complexity of battery materials and
devices. Current research on this tends to be disjointed, with
individual groups continually reinventing the wheel when it comes to
parameterisation methods. System identification of models has a long
history in control engineering, and well-established toolboxes exist for
generic model fitting e.g. the SysID toolbox in Matlab. However, these
tend to work best on simple linear time-invariant systems, whereas
battery models may involve non-stationary and non-linear behaviours, as
well as PDEs and algebraic constraints.

**Audience**

The first end-users for PRISM will be the MSM and 'intelligent' project
teams and PRISM developers -- if it isn't useful for us, then it won't
be useful for others!

**Community and values**

We aim to build a broad consortium to develop PRISM, building on and
learning from the success of the PyBaMM community. Our values are:

-   Openness (e.g., code is open source on GitHub)

-   Inclusivity and fairness (those who want to contribute may do so,
    and their input is appropriately recognised)

-   Inter-operability (aiming for modularity to enable maximum impact
    and inclusivity)

-   User-friendliness (putting user requirements first, thinking about
    handholding & workflows)

**What PRISM will do**

A list of some use cases for battery modelling is given in Appendix A.
From this, we deduce that the two key requirements for PRISM are:

-   *Parameter estimation* of battery models (could be electrochemical
    or equivalent circuit) -- to be used for e.g. pack design purposes,
    BMS design, and aging data analysis.

-   *Design optimisation* studies, e.g., layer thickness, number of
    layers, tab placement.

-   \[A tertiary requirement for PRISM might be *model synthesis*, i.e.,
    generating (low order) models automatically from data or higher
    order, more complex models. This is not in the scope of PRISM at
    this stage, though.\]

A more detailed breakdown of all requirements is given below.

**Software requirements specification**

Essential requirements are marked \*\*; other requirements are
longer-term (i.e., desirable but not essential for a first release).

1.  *Parameterisation*

    A.  Two major use cases: design optimisation and parameter
        estimation \*\*

    B.  Cost functions can be built-in or user-defined

    C.  Multiple options for simulation and optimisation are offered and
        could include deterministic or probabilistic approaches \*\*

2.  *User experience*

    A.  Self-diagnosis of performance, e.g., able to quantify the
        goodness of fit, identifiability etc. \*\*

    B.  Clear and understandable diagnostic outputs for users \*\*

    C.  Easy to use for a non-expert user (leads to workflows, examples)
        \[e.g. basic results are possible after just 1 hour of
        training\]

    D.  Flexible data ingestion (e.g. pull from CSV, a database, or
        parquet, via JSON metadata specification) \*\*

3.  *Performance*

    A.  Quick (but possibly sub-optimal) results in a short time \[\<1
        min\], more accurate results in a longer-term \[\<1 hour\]

    B.  Different levels of computational complexity are possible but
        mostly geared towards desktop solutions (i.e., not requiring a
        cluster)

4.  *Software design*

    A.  Modular approach, with clearly specified interfaces between
        sub-components \*\*

    B.  High code reusability. Within reason, don't reinvent the wheel
        -- use existing tools as much as possible \*\*

    C.  Professional software engineering development practices (e.g. CI
        from the beginning) \*\*

    D.  Close integration with PyBaMM for battery simulations (some user
        understanding of PyBaMM may be required) \*\*

    E.  Cross-platform, able to run on Windows, Mac OS, Linux \*\*

    F.  Python-based (Julia could be explored later as a secondary
        objective) \*\*

    G.  Command-line interface/notebook-based (rather than GUI --
        although this could come later) \*\*

5.  Clear, documentation with plenty of examples

**How PRISM will work**

PRISM will be a modular library for the parameterisation and
optimisation of battery models, with a particular focus on classes built
around PyBaMM models and structures. The figure below gives a conceptual
idea of how PRISM could be structured. An explanation of each box is
given below the diagram. This will likely evolve as development
progresses (e.g., in practice, some things will live "inside" other
things in a more nested way), so this is just a starting point.

![A picture containing diagram, screenshot, plan, line Description
automatically generated](media/image1.png){width="6.268055555555556in"
height="2.6465277777777776in"}

Figure 1: PRISM architecture showcasing multiple forward models and cost
functions

The modules in the above figure are defined as:

+--------+--------------------------+----------------+----------------+
| *      | **Purpose**              | **Inputs**     | **Outputs**    |
| *Box** |                          |                |                |
+========+==========================+================+================+
| Data   | Ingest data, error check | data filename\ | time series    |
|        | and clean-up, convert to | metadata       | data (current, |
|        | required PRISM formats   |                | voltage,       |
|        |                          |                | temperature,   |
|        |                          |                | time)          |
|        |                          |                |                |
|        |                          |                | metadata       |
+--------+--------------------------+----------------+----------------+
| F      | Run the model            | parameter set  | model output   |
| orward |                          |                | (time series)  |
| model  |                          | input data     |                |
|        |                          |                | covariance     |
|        |                          | initial        | propagation    |
|        |                          | conditions     |                |
|        |                          |                | solver         |
|        |                          | solver         | diagnostics    |
|        |                          | settings       |                |
|        |                          |                |                |
|        |                          | model          |                |
|        |                          | definition     |                |
+--------+--------------------------+----------------+----------------+
| Cost   | Define the cost function | model output   | cost           |
| fu     | choice (might sit inside |                |                |
| nction | optimiser). Map from     | time-series    | gr             |
|        | model output and output  | data           | adient/hessian |
|        | data to a scalar cost.   |                | estimates      |
|        |                          | priors,        | $dJ/d\theta$   |
|        |                          | assumptions    | etc.           |
+--------+--------------------------+----------------+----------------+
| Opt    | Iterates the forward     | cost           | parameter      |
| imiser | model parameters and     |                | value(s)       |
|        | simulations using the    | gr             |                |
|        | cost function            | adient/hessian | convergence    |
|        | information              | estimates      | and            |
|        |                          | $dJ/d\theta$   | diagnostics    |
|        |                          | etc.           | information    |
|        |                          |                |                |
|        |                          | parameter      |                |
|        |                          | bounds         |                |
|        |                          |                |                |
|        |                          | optimiser      |                |
|        |                          | settings       |                |
+--------+--------------------------+----------------+----------------+

**Workflow**

From a user perspective, the tool would typically be used as described
below. The reason for having the parameters/data divided into subsets is
that this is likely to be important for making parameter estimation
tractable. For example in a battery model a typical flow would be to
first fit thermal properties, then fit OCV data, then fit dynamic
electrical/electrochemical parameters.

![A picture containing text, screenshot, diagram, font Description
automatically generated](media/image2.png){width="6.268055555555556in"
height="3.4902777777777776in"}

**Open questions**

-   We're going to be handling much data. How will we handle it? Pandas
    data frame? Some database? Integrated or not with PyBaMM?

-   What needs to be done to integrate PEM (or MAP etc.) into Casadi or
    Scipy.optimize?

-   What are the minimal requirements for the PyBaMM model class?
    [[\[1\]]{.underline}](https://docs.pybamm.org/en/latest/_modules/pybamm/models/base_model.html)
    [[\[2\]]{.underline}](https://github.com/pybamm-team/PyBaMM/tree/develop/pybamm/models/full_battery_models/lithium_ion)

-   What are the large probabilistic/optimisation libraries in Python?
    [[\[1\]]{.underline}](https://www.tensorflow.org/probability)
    [[\[2\]]{.underline}](https://botorch.org/)
    [[\[3\]]{.underline}](https://www.pymc.io/welcome.html)
    [[\[4\]]{.underline}](https://pytorch.org/)
    [[\[5\]]{.underline}](https://pystan.readthedocs.io/en/latest/)
    [[\[6\]]{.underline}](https://scikit-learn.org/stable/index.html)

-   What is the best way to interface with
    [Galv](https://github.com/Battery-Intelligence-Lab/galvanalyser):

    -   API definition between Galv ↔ PRISM ↔ PyBaMM

    -   Automated parameterisation in Galv via PRISM

    -   Enabling automated digital twin predictions

**Next actions**

-   Circulate this document to those who came to the PRISM workshop for
    comments and edits.

-   Then circulate to the wider MSM team.

-   Then agree/bank the contents for now.

-   Create a couple of benchmark datasets and simple models for
    examples.

-   Discuss possible merging of some of the existing codebase from
    pybamm-param into PRISM.

-   Could we identify a few example cases and collate the minimum
    parameterisation and validation data-sets required (for those
    examples) in a common place? For example, we can share OCV, HPPC,
    rate-tests and drive cycles (done at different temperatures) which
    can them be used to develop the work flow in Figure 1 and estimate
    and validate either an ECM or EHTM type model? To basically identify
    a few examples and serval data sets (from different MSM groups) in a
    common place to start building the workflow of Figure 1. Will also
    help in addressing the first bullet point in Open Question.

**APPENDIX A -- Battery modelling use cases**

Here are some reasons why battery modelling matters. There are more!
This is just a starting point.

![A picture containing text, screenshot Description automatically
generated](media/image3.png){width="6.141411854768154in"
height="3.961538713910761in"}

[^1]: Attendees: David Howey, Brady Planden, Ferran Brosa Planella,
    Monica Marinescu, Alistair Hales, Martin Robinson, Masaki Adachi,
    Nicola Courtier, Jon Chapman and Dhammika Widanalage
