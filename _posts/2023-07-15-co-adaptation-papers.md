---
layout: post
title: Three (!) papers on co-adaptation
tags:
  - news
  - blog
  - papers
---

I am very excited to highlight three recent archival papers from my group on the theme of co-adaptation between human and machine learners. 

These papers are a major milestone in the transition my research has taken since finishing my PhD: whereas my work in grad school and first few years as faculty largely concerned [modeling and control of legged locomotion](http://www.eecs.berkeley.edu/Pubs/TechRpts/2014/EECS-2014-167.html), my postdoc and recent years as faculty shifted focus to the human/machine interface (my [CAREER proposal](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2045014) represents the bridge between these two stages).

Of course I'm biased, but I regard each of these as landmark studies -- I think we've really hit on important and timely themes, and I think the experimental work (led by [Ben Chasnov](http://students.washington.edu/bchasnov) and [Momona Yamagami](http://dx.doi.org/10.1126/science.aac6076)) is absolutely top-notch. Below I'll provide some context and informal interpretations that don't appear in the manuscripts; if you're interested in the technical content and relationship to the broader literature, you'll just have to read the papers :D

#### [Biosignal-based co-adaptive user-machine interfaces for motor control](http://faculty.washington.edu/sburden/_papers/Madduri2023cobme.pdf)

The first paper to appear is a review that [Amy Orsborn](http://faculty.washington.edu/aorsborn/) was invited to contribute to a special issue of [*Current Opinion in Biomedical Engineering*](https://www.sciencedirect.com/journal/current-opinion-in-biomedical-engineering). The special issue was curated by [Helen Huang](https://nrel.bme.unc.edu/) and [Greg Sawicki](http://pwp.gatech.edu/hpl/), and Amy very graciously invited me to pitch in on the paper together with our co-advised student [Maneeshika Madduri](https://sites.uw.edu/mmadduri/). 

Our paper reviews recent literature on the co-adaptation problem across multiple fields including neuroengineering, motor control, and human-computer interaction. [Perhaps](http://dx.doi.org/10.1101/2020.12.11.421800) [unsurprisingly](http://dx.doi.org/10.1016/j.ifacol.2023.01.109), we propose game theory as a framework for understanding and shaping outcomes when adaptive humans and machines interact. I can assert unreservedly that this is a fantastic paper since I am neither the first nor last author :)


#### [Human adaptation to adaptive machines converges to game-theoretic equilibria](http://faculty.uw.edu/sburden/_papers/Chasnov2023arxiv.pdf)

The second paper arose from my longstanding collaboration with [Lillian Ratliff](http://faculty.uw.edu/ratliffl) and our co-advised student [Ben Chasnov](http://students.washington.edu/bchasnov). We have worked on learning in games for some time: I entered into this space through a [grad school collaboration with Lillian](http://dx.doi.org/10.1109/TAC.2016.2583518) that [continued with Ben taking the lead](http://proceedings.mlr.press/v115/chasnov20a.html), and we've received both collaborative and individual grants that explore the topic in different domains (NSF CPS #[1836819](https://www.nsf.gov/awardsearch/showAward?AWD_ID=1836819), NSF CAREER #[1844729](https://www.nsf.gov/awardsearch/showAward?AWD_ID=1844729),[2045014](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2045014)). Of course, [Lillian is the authority on the topic](https://faculty.washington.edu/ratliffl/research/).

Particularly in light of recent advances in ML (e.g. RL and LLM), there is a lot of anxiety about the power of "AI agents" to emulate and influence human behavior. There's some evidence that [algorithms can influence elections](https://www.centreforpublicimpact.org/insights/good-bad-ugly-uses-machine-learning-election-campaigns). But we are not aware of scientific literature that systematically studies the strategic interaction between algorithms and *individual* people ([this review](http://dx.doi.org/10.1016/j.joep.2021.102426) provides a timely summary of the state of the field). 

To fill this gap, we conceived the simplest possible instantiation of a continuous game -- scalar actions and quadratic costs -- and tested different machine learning algorithms against a human "opponent". I'm using scare quotes to qualify the adversarial terminology, because we actually think the most interesting applications of our algorithms will be in *assistive* or *augmentative* capacities, e.g. designing rehabilitative interventions using exoskeletons or providing decision support to people in multi-agent settings. 

We find remarkable agreement between interaction outcomes and a constellation of game theory equilibria, establishing a new framework for predicting how individuals respond to algorithms.


#### [Co-adaptation improves performance in a dynamic human-machine interface](http://faculty.uw.edu/sburden/_papers/Yamagami2023biorxiv.pdf)

The third paper represents the final aim of [Momona Yamagami](https://momona-yamagami.github.io/)'s thesis. Building on Momona's wonderful experimental work studying how people learn to control dynamic machines using [different modalities](http://dx.doi.org/10.1145/3313831.3376224) and [different limbs](http://dx.doi.org/10.1109/TCYB.2021.3110187), we test what happens when an adaptive agent is inserted in-the-loop with the human and machine. 

A note on nomenclature: we decided to term the other adaptive agent the "interface", and the non-adaptive dynamics the "machine". Although we've only run experiments with simulated machines up to this point, in real-world deployment this subsystem would represent the immutable physical plant, i.e. the robot or vehicle or assistive device being controlled. We think our terminology is useful and appropriate since the *interface* between the human and the (fixed/static) machine is what learning algorithms can modify on-the-fly.

We adopted a linear systems paradigm since (a) ourselves and others (starting with [McRuer *et al*.](http://dx.doi.org/10.1109/THFE.1967.234304)) have shown how useful it can be for modeling human behavior and (b) it provides a [powerful and flexible toolkit](https://en.wikipedia.org/wiki/Robust_control) for synthesizing a robust/optimal controller (the *interface* in our case).  We used nonminimum-phase machine dynamics from a [wonderful study](http://dx.doi.org/10.1109/TCYB.2020.3027502) out of [Jesse Hoagg](https://scholar.google.com/citations?user=5eiPgckAAAAJ&hl=en)'s group because these dynamics are [fundamentally hard to control](http://dx.doi.org/10.1109/MCS.2003.1213600), so an adaptive interface may genuinely help -- rather than hinder.

Since [humans are thought to use optimization in motor control](http://dx.doi.org/10.1038/nn963), we start by modeling the human and interface as optimization-based agents. In the linear systems context, this model naturally leads to the assumption that both agents solve optimal control problems. But we show that there is a fundamental problem with this model, since there appears to be no "equilibrium" solution the interaction could converge to -- the essential problem is that controller complexity explodes, which we regard as an instance of an [infinite regress](http://www.jstor.org/stable/2628393) that has long been studied in game theory.

Acknowledging instead that [both the human and interface have bounded rationality](http://dx.doi.org/10.1126/science.aac6076), it seems implausible that this infinite regress would arise in reality. So as a starting point, we finitely-parameterize the interface and conduct experiments to see how humans respond. Since the paper's title contains spoilers, it should come as no surprise that we find that the adaptive interface confers benefits all around -- from the perspective of the human, interface, and whole closed-loop system.

