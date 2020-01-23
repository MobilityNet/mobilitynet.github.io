# Summary

As smartphone-based Human Mobility Systems (HMSes) shift from informing
individuals to influencing public asset allocation, it is critical to assess
how well they perform. Evaluation techniques of HMSes have typically been ad
hoc, limited to single measurements and ignored power-accuracy trade-offs.
These trade-offs are important due to built-in and context-sensitive sensing
in the phones - e.g., "Find my iPhone" or "Doze mode".

Treating HMSes as physical instruments with inherent physical noise, we
propose requirements related to holistic power/accuracy trade-offs, privacy
preservation, and ground truth. We then outline a novel evaluation procedure
that uses artificial trips and multiple parallel phones to provide controlled,
repeated inputs to the HMSes under test. Artificial trips mitigate privacy
concerns and allow repeatability while efficiently exploring a wide variety of
trip contexts. Parallel sensing with control phones mitigates the effects of
context sensitive power consumption and inherent sensing error. If adopted
widely by the community, the resulting ground truthed, trade-off-aware, public
datasets can form the basis for additional HMS optimizations.

In this dataset, we use this procedure to create three artificial timelines
with 15 different modes, including ebike and escooter. We use these timelines
for a comprehensive evaluation of three different sensing settings with over
500 hours of ground-truthed data.

# Introduction

Inspired by the popularity of smartphone-based personal fitness tracking, the
transportation community aims to build Human Mobility Systems (HMSes) that can
automatically track and classify multi-modal travel patterns. Such systems can
replace expensive and infrequent travel surveys with long-term, largely
passive data collection augmented with intermittent surveys focused on
perceptual data.

While there has been much work on building HMSes, both in academia and in
industry, the procedure to _evaluate_ them has largely been an afterthought.
Careful evaluations are critical as we move from the personal to the societal
domain. Users who make decisions based on self-tracking have an intuition of
its accuracy based on their experienced ground truth. The decisions are low-
stakes lifestyle changes, which may be personally meaningful, but are not
societally contentious. However, a Metropolitan Transportation Agency picking
projects and allocating millions of dollars in funding needs to know the
accuracy of the data before making its decisions .

Computational Mobility (CM) can frame approaches to evaluate accuracy.
Consistent with interdisciplinary research principles, we can apply
computational concepts to shift the focus from ad hoc techniques to more
rigorous concepts for evaluating software systems and algorithms. These
concepts include:
1. _artificial workloads_, from Operating Systems,
1. _standard datasets_, from Machine Learning, and
1. _handling transient effects_, from Networking.

The rest of this document is structured as follows. We start with an intuitive
description of the challenges and the solution. Next, we outline the evaluation
requirements and outline an experiment procedure that meets
them. We discuss some alternative approaches and describe the reference
implementation, before concluding.

# Intuition for challenges and solution

The typical HMS evaluation procedure (e.g., Quantified Traveler, prior
versions of our work) is ad hoc and also functions as a pilot — a small (()
3-12) set of the author’s friends and family are recruited to install the app
component of the HMS on their phones, and go about their daily life for a few
days or weeks while annotating the trips with “ground truth”. The ground truth
annotation can either directly happen on the app, or through a recap at the
end of the day. Conscientious researchers may ensure that the set of
evaluators are demographically diverse, in an attempt to evaluate against a
richer set of travel patterns.

While this procedure imposes little additional researcher burden, it conflates
the _experimental_ procedure (understanding human travel behavior in the wild)
with the _evaluation_ procedure (evaluating the instrument that will measure
the human travel behavior). The first is trying to understand _behavior_ , so
it needs human diversity. The second is trying to understand _sensing_
parameters, so it needs diversity of _trip types_. The human functions as a
phone transportation mechanism during evaluation and could be profitably
replaced with a self-navigating robot if one was available.

An analogy with classic physical measurements may be useful. Consider the
situation in which a researcher wants to collect data on the weight
distribution of the population in a particular region. Since there are
currently no certifying bodies for travel diaries, let us pretend that she
cannot purchase a pre-certified scale. How would she evaluate the available
scales before starting her experiment?

The analog to ad hoc evaluation procedure would involve recruiting several of
her friends and family to weigh themselves on the scales and compare the
reported weight with their true weight. This analogy clearly reveals some
limitations of the ad hoc procedure: 1. How does she trust that the self-
reported weights are “true”? 1. If all her friends are adults weighing 55 kg –
75 kg, how does she know how the scales perform outside that range?

She can overcome the range limitations by recruiting a broader set of testers,
e.g., through an intercept survey. However, that modification makes the ground
truth limitation worse, since it is less likely that strangers will reveal
their true weight. A further modification might pay contributors to improve
the self-reported accuracy, but at this point, she is essentially running the
experiment.

A more robust evaluation procedure would involve choosing known weights across
a broad range (e.g., 0 kg to 300 kg in 10 kg increments) and comparing them to
the reported weights. Since no instrument is perfect, there is likely to be
some variation in the values reported. She would likely repeat the experiments
multiple times in order to establish error bounds.

HMS evaluation procedures need to be more sophisticated than simple physical
measurements since: 1. their operation is based on prior behavior (e.g., HMS
duty cycling, android doze mode) and the potential for feedback loops makes it
important to control the _sequence_ of evaluation operations, 1. unlike a
physical scale, which has a fixed one-time cost, they have an ongoing,
variable cost in terms of battery drain, so the evaluation must assess the
power/accuracy trade-off, and 1. unlike scalar weight data, HMSes generate
strongly correlated timeseries data, which is extremely hard to anonymize.

Therefore, the main contributions in this chapter are:

  1. We propose an **evaluation procedure** for HMSes based on predefined, ground truthed, artificial trips and outline how it addresses the above challenges,
  2. We describe the design of a cross-platform **evaluation system** that can be used to perform such evaluations reproducibly and publish the results.
  3. We highlight some lessons learned during this process that future groups might want to take into account while designing their experiments.

# Requirements for evaluating Human Mobility Systems (HMSes)

Human Mobility Systems (HMSes) need a methodology for rigorous evaluation that
allows users of the data to understand their limitations and their accuracy in
various settings. Before establishing such a method, we need to understand the
evaluation requirements, and the challenges associated with meeting each
requirement. Establishing a clear set of requirements also allows us to
understand the limitations of the proposed method and provides a roadmap for
future improvements.

## Holistic evaluation: power vs. overall accuracy

Using smartphones for collecting background sensed location data leads to
higher battery drain. Travelers are very sensitive to battery life , and there
is a clear power/accuracy trade-off for smartphone sensing. Naïve high
accuracy sensing drains the battery for almost all users (Section
[sec:modeling]), but techniques to lower battery drain also lower the accuracy
(Section [sec:apioverview]) So it is critical that the evaluation consider
both power and accuracy together. For example, if an HMS can get 95% accuracy,
but runs out of battery in 2 hours, the deployer needs to adjust the
incentives offered (e.g., money, utility, …) accordingly.

## Privacy preserving

The data collected by HMSes includes location traces, which are inherently
privacy sensitive. While a common privacy technique is to _de-link_ datasets
by replacing Personally Identifiable Information (PII) with a code , location
traces allow re-identification from the raw data alone . Intuitively, from the
location traces, we can find the _places_ where people spend most of their
time, which allows us to discover their home and work locations and uniquely
identify them. This implies that the evaluation methodology must address
privacy concerns.

## Ground truthed

In order to fully evaluate the data collected, we need ground truth for not
just the mode, but also the trip start and end times, section start and end
times and the travel trajectory. Labeling trips through prompted recall is a
low effort technique to collect mode ground truth, but it depends on accurate
trip and section segmentation. Segmentation ground truth requires recalling
the start and end _times_ at the end of the day, which is likely to be
unreliable . Similarly, for evaluating trajectories, travelers can potentially
draw out spatial ground truth but spatiotemporal ground truth is almost
impossible to obtain after the fact.

# Controlled Evaluation of context-sensitivity

Instruments are typically evaluated by repeatedly exposing them to controlled
inputs to determine their error characteristics. In the case of complex
systems such as HMSes, the evaluation needs additional controls for feedback
loops, cost/accuracy trade-offs and privacy considerations. In this section,
we outline em-eval — a procedure for HMS evaluation that addresses these
concerns with two techniques that are novel in this domain:

  1. predefined, **artificial trips** that support spatial ground truth, preserve privacy, increase the breadth of _trip types_ and support repetitions for establishing error bounds, and

  2. power and accuracy **control phones** carried at the same time as the experimental phones, that can cancel out context-sensitive variations in power and accuracy.

## Artificial timeline

The core of the experimental procedure is the predefined specification of a
sequence of artificial _trips_ , potentially with multiple _legs_ or
_sections_ per trip. The trajectory and mode of travel is also predefined. The
_data collector_ completes the timeline trips by strictly following the
specified trajectory and mode while carrying multiple phones that collect data
simultaneously using different configurations.

The specified predefined trajectories provide spatial ground truth. We do not
predefine temporal ground truth since it is extremely hard to control for
differences in walking speeds, delays due to traffic conditions, etc. We use
manual input from the data collector to collect _coarse_ temporal “ground
truth” of the transitions along the timeline. We do not use manual input for
_fine-grained_ temporal ground truth along the trajectory because:

  1. human response times are too slow for fine-grained temporal ground truth during motorized transportation, and
  2. distracting the data collector during active transportation can be risky.

Using an artificial timeline addresses several of the unique challenges
associated with HMS evaluation.

Privacy  
Since the trips are artificial, they preserve the data collector’s privacy.
Even if his adversaries would download the trips, they would not be able to
learn anything about his normal travel patterns.

Spatial ground truth  
Since even high accuracy (GPS-based) data collection has errors, predefining
spatial ground truth allows us to resolve discrepancies (Section
[sec:relaxing-constraints]) and compute the true accuracy.

Breadth and variety of trips  
Artificial trips allow efficient exploration of the breadth of the trip space.
For example, the trips could include novel modes such as e-scooters and
e-bikes, or specify different contexts for conventional modes, such as express
bus versus city bus.

Repetitions  
Since the trips are predefined, they can be repeated exactly. This allows us
to use standard variance and outlier detection to estimate error bounds on the
measured values.

## Control phones

The artificial trips give us spatial ground truth, but they do not give us
cost (power consumption) or temporal ground truth.

We control for the cost through the use of the use of multiple phones, carried
at the same time by the data collector. The phones carried by the data
collector are divided into control phones and experiment phones. The control
phones represent the baseline along each of the axes in our trade-off and the
experiment phones implement a custom sensing regime that is at some
intermediate point. The evaluation procedure allows us to determine those
points.

The power control phone captures the baseline power consumption of a phone
that is not being used for tracking by a HMS. This does not mean that the
phone is idle — phone OSes (e.g., iOS or android) are complex, context-
sensitive systems that perform their own location tracking (e.g., “Find my
iPhone”) and their own duty cycling (Figure [fig:poweraccuracyvariations]).
Using a power control allows us to identify the **additional power** consumed
by the HMS, even if it is context sensitive.

The accuracy control captures the upper bound on the accuracy of a particular
class of smartphones given sensor and OS limitations. While we would like to
compare the experimental accuracy to ground truth,

  1. all sensors have errors, so ground truth is not achievable in practice,
  2. artificial trips give us spatial but not temporal ground truth, and
  3. GIS-based trajectory specifications do not have an associated power trade-off.

Using an accuracy control allows us to compare the experimental data
collection against the best achievable data collection, in addition to the
ground truth.

# Discussion of alternative procedures

While em-eval (Section [sec:our-procedure]) addresses the complexities of HMS
evaluation, it also imposes a much higher researcher burden than the ad hoc
method. This raises the question of whether all these controls are necessary
or merely sufficient. In this section, we discuss some alternative approaches
and highlight the unexpected behavior that they would miss. This list is not
comprehensive but provides a flavor of the arguments without tedious
repetition.

|:-:| |![image](figs/double_duty_cycling_android_1.png)|
|![image](figs/spiky_range_high_accuracy_AO.png)|

## No artificial trips

Creating predetermined trips requires an upfront investment in effort, and
requires the data collector to take trips just for data collection. An
alternative would use multiple phones, but allow data collectors to go about
their regular routines and tag the modes only. We could use the accuracy
control phones to determine the ground truth trajectory.

#### No privacy  
Capturing the data collector’s regular routines compromises their privacy.
Even if the data does not include their name or phone number, a list of their
commonly visited places and trips can form a unique fingerprint that can
uniquely identify them . This sensitivity precludes evaluation data from being
published and used for reproducible research.

#### No repetition  

The behavior of the same phone with the same configuration can vary over time,
both for power and for accuracy (Figure [fig:poweraccuracyvariations]).
Repeating the same trip multiple times allows us to detect and remove
outliers. With ad hoc trips, it is unclear whether any difference in behavior
is real or caused by context-sensitive variation. And without predetermined
trips, it is challenging to repeat the same trips and trajectories over time.

#### No spatial ground truth  
No sensor is perfect and even the accuracy control phones can have sensing
errors. If we see a divergence between an experiment phone and the accuracy
control, it is unclear which one has the error (Figure
[fig:poweraccuracyvariations]).

iOS | android  
---|---  
![image](figs/ios_battery_settings_display.png) |
![](figs/android_battery_settings_display.png)  
  
## No control

Using control phones requires the researcher to purchase multiple phones of
the same make, model and approximate age. While used smartphones are
relatively cheap (USD 50 – USD 100), 4 android phones and 4 iPhones combined
will still cost USD 400 – USD 800. An alternative would be to use one phone
each for each OS, perform the timeline trips, and look at the app-specific
power consumption reported by the phone OS.

#### Sensor access attribution and the meaning of the %  

Sensor access in modern phone OSes (android and iOS) is also context-
sensitive, making it unclear how it is counted for per-app consumption. For
example, if multiple apps request a sensor reading, the OS delays returning a
result until it can batch related requests and serve all of them with a single
sensor access (e.g., Figure [fig:smoothingnavigationstartexample]). This is
why the OSes treat the sensing frequency as a hint instead of a guarantee.
Second, if sensor access is mediated by a service (e.g., fused location in
Google Play Services), it is unclear whether the sensor access is counted for
the service or the app (Figure [fig:powermeasurementchallenges]). And finally,
although android reports per app consumption as a % of the battery _capacity_
, iOS does so as a % of the battery _consumption_. This indicates that on
dedicated phones, the HMS under test will always show close to 100%, whether
it is the power control or the accuracy control (Figure
[fig:powermeasurementchallenges]). Using a control phone for the power will
cancel out these context-sensitive effects and estimate the difference in
power drain with and without the HMS app component installed.

#### Custom duty cycling increases power drain  

Sensing is not the only source of power consumption — CPU usage can also have
a significant impact on power usage. HMSes can use smart local processing to
reduce local sensing, but the increased power consumption from the CPU can
cancel out the savings from the sensing. Including an accuracy control showed
that, unlike in 2015 (Section [sec:apioverview]), the basic duty cycling
algorithm in our experiment paid for itself in low frequency sensing but
actually increased power usage for high frequency sensing (Figure
[fig:powermeasurementchallenges]).

# Related work

Human Mobility Systems (HMSes) are complex to evaluate (Section [sec:relaxing-
constraints]). Other researchers have identified similar challenges as part of
survey papers (e.g., phone context, privacy, learning, scaling , varying
metrics and time scales across research areas ). However, to the best of our
knowledge, there is no proposed solution that addresses all of them.

Papers related to instrumenting travel behavior fall into three main
categories; we list some work from each as an example. A comprehensive
classification of papers into categories is beyond the scope of this paper.

## Context sensitive sensing algorithms: power without accuracy

This research area focuses on context sensitive, adaptive power management of
sensors. Papers such as ACE and Jigsaw compare their power requirements to
naive sensing techniques. However, their accuracy evaluations focus on the
localization error (Jigsaw), or comparison to naive inferred results (e.g.
based on speed, such as ACE).

## Travel diary systems: compare to manual surveys

There is a vast variety of one-off travel diary systems that combine
smartphone based sensing with cloud-based processing to generate travel
diaries. Systems such as such as Data Mobile , Future Mobility Study (FMS) and
rMove aim to replace the paper and telephone based Household Travel Surveys
with smartphone and cloud based systems. So they evaluate the accuracy of
their systems against the traditional methods, not against ground truth. This
can show that smartphone based methods are significantly better than
traditional methods, but not provide a quantitative estimate of the accuracy
of their system. Similarly, they do not include quantitative power evaluations
- preferring statements like “Among the three types of discrepancies, the
second type, data gap due to battery drainage, was most frequently observed.”
or “The battery consumption test was simply whether, under regular usage, the
phone could make it through the day without having to be charged.” . So they
do not rigorously evaluate either the power or the accuracy side of the trade-
off.

## Mode inference: accuracy without power, non-uniform data

Mode inference of travel mode based on sensor data is an extremely popular
subject in the literature, probably because it is hard and there is much
overlap between modes other than walking. Researchers have used decision trees
, Hidden Markov Models, and neural networks to distinguish between various
subsets of travel modes. However, although the inference algorithms are
different, most such papers use similar methods for evaluating their accuracy.
They typically recruit a small sample of their friends (e.g., 16 users over one
day, 4 users over two weeks) to collect naturalistic data along with
annotations of the ground truth. The data collection focuses on the sensors
used for analysis and omits the battery. This kind of evaluation does not meet
the any of the requirements outlined above, except privacy, which is addressed
by not publishing the dataset.

# Evaluation system design

The em-eval procedure (Section [sec:our-procedure]) allows us to estimate the
power/accuracy trade-off of various sensing configurations used in Human
Mobility Systems (HMSes). One of the novel components of the procedure
involves the specification of pre-defined, artificial trips with ground-
truthed trajectories and modes.

This section explores the nuances of implementing such a procedure. We first
describe a publicly available reference implementation of a system – em-eval-
zephyr – that can be used perform this procedure. We then discuss challenges
encountered while using the system to perform an experiment in the San
Francisco Bay Area (Section [sec:accuracyevaluation]). Some of these
challenges were addressed by system improvements, while others are documented
as best practices for future data collectors.

Control configuration | Trip selection | Map leg details  
---|---|---  
![image](figs/accuracy_control_configuration.png) |
![image](figs/trip_selection.png) | ![image](figs/map_leg_details.png)  
  
## System overview

em-eval is a generic _procedure_ for HMS evaluation – it does not actually
collect any data. To use it, we need a concrete system that configures data
collection based on the spec configurations, collects coarse temporal ground
truth, periodically reads battery levels and stores data for future analysis.

We built a system – em-eval-zephyr – that combines our prior work on power
evaluations with our existing HMS platform and supports performing the em-eval
procedure. The system consists of three main parts:

#### Evaluation Specification  

The _spec_ describes an evaluation that has been performed or will be
performed in the future. In addition to mode and trajectory ground truth, it
includes the app configurations to be compared and the mapping from phones to
evaluation _roles_. The spec automatically configures both the data collection
app and the standard analysis modules.

To reduce evaluator burden, we provide preprocessing functions to fill in
trajectory information based on route waypoints for road trips and OSM
relations for public transit. We also provide sample notebooks to verify
timelines and their components before finalizing and uploading the evaluation
spec.

#### Auto-configured Smartphone App  

We have generated a custom UI skin for our e-mission platform that is focused
on evaluation. It allows evaluators to select the current spec from the public
datastore, and automatically downloads the potential comparisons to be
evaluated, the role mappings and the timeline.

Since the e-mission platform data collection settings are configurable through
the UI, the sensing configurations defined in the spec are automatically
applied based on the phone role when the data collector starts an experiment.
For example, when starting an experiment to compare high accuracy (HAHFDC)
versus medium accuracy (MAHFDC) data collection, the second experiment phone
will automatically be set to MAHFDC settings. Finally, when the data collector
performs the trips, he marks the transition ground truth in the UI, and the
app automatically displays the next step in the timeline (Figure [fig:em-eval-
zephyr-ui]).

#### Public Data + Sample Access Modules  

Since there are no privacy constraints, em-eval-zephyr uploads all collected
data to a public instance of the e-mission server. The associated repository
contains sample notebooks that can download, visualize and evaluate the data
associated with a particular spec. All the data used in this paper is publicly
available, and the notebooks can be manipulated interactively using [binder](https://mybinder.org/)

Note that although the em-eval **procedure** is general, the current
implementation of the em-eval-zephyr **system** is integrated only with the
e-mission platform. Using the procedure with other HMSes will require re-
implementing the em-eval procedure with the other HMS, or using a combination
of systems for the evaluation. For example, em-eval-zephyr can still read the
battery level periodically, display the trip sequence to the data collector,
and be used to mark the transition ground truths. However, the evaluator needs
to configure the settings for the app being tested manually, and to download,
clean and analyze the resulting data.

## System iterations and lessons learned

As we started collecting data, we had to resolve some ambiguities around
exactly when the transition ground truth should be collected. We also
discovered best practices that increased the likelihood of successful data
collection. This section outlines these lessons learned.

#### System change: capture transition complexity  

One of the big promises of using HMSes for instrumenting human travel is that
we don’t have to focus only on the primary mode. Instead, with fine-grained
data collection, we can understand the full complexity of end to end travel.

In fact, the only true unimodal trips are walking trips. Everything else is
multi-modal. Thus, a significant change to the system was to restore the
hidden complexity that is elided from user descriptions of travel diaries. For
example, consider the trip description “Drive from Mountain View Library to
Los Altos Library”. Although that appears to be a unimodal trip, it is
actually a multi-modal trip which involves implicit walk access sections to
and from the car at the source and destination respectively.

It is not possible to predetermine the ground truth for these walk access
sections since we cannot control which parking spaces are available when we
perform the trip. We address such issues by adding _shim sections_ , and
expanding the start and end from points to () 100 m polygons. We can then
relax the constraints around ground truth within the polygon by only using the
reference dataset, but still check the accuracy of the mode inference (Figure
[fig:em-eval-zephyr-ui]).

#### Best practice: Pilots are critical  

In spite of reviewing the predetermined trajectories ahead of time as part of
the validation process, and also having them displayed on the em-eval-zephyr
UI, we found that we frequently made small mistakes, during the first round of
data collection for a new timeline. Sometimes, we found that the predefined
routes, potentially suggested by Open Source Routing Machine (OSRM), felt
unsafe to bicycle on. We had to tweak the specification to pick safer routes.
The second repetition generally resolved these issues. In order to avoid a
stressful data collection experience, we suggest running through a new
timeline with a trial run before starting full-featured data collection.

#### Best practice: Mindfulness  

Remembering to mark the transition ground truths was one of the hardest parts
of the ongoing data collection and really highlights the challenges of ground
truth collection. In spite of the fact that she was performing artifical trips
to collect data for her own project, one of the authors forgot to mark wait ->
move transitions during the pilot for the long multi-modal timeline because
she had started checking her email while waiting. It is important to be
present in the moment and pay attention to the context while collecting data.

# Conclusion

Human Mobility Systems (HMSes) are complex software systems that run on
equally complex smartphone operating systems (OSes). This complexity implies
that there is rarely a simple linear relationship between their inputs and
outputs, which complicates their evaluation.

We outline a procedure, based on repeated travel over predefined artificial
_timelines_ carrying experiment and control phones, to control this
complexity. We show that it can control for outliers, and also reveal
meaningful signals about the behavior of smartphone virtual sensors that are
relevant to instrumenting human travel data.

The procedure is privacy-preserving, so it does not need human subjects
approval. It focuses on _trip_ diversity, not _demographic_ diversity, so it
can be undertaken by a small research group, or even a single researcher, as a
pre-pilot before recruiting study participants. It uses predetermined trips
and modes, so it can efficiently explore complex or newly emerging travel
patterns and modes, such as e-scooters. The procedure, and the associated
reference implementation can simplify the testing required before a study is
launched.
