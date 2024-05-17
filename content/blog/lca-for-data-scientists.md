+++
date = "2024-05-02"
title = "Life cycle assessment for data scientists"
toc = true
tags = ['sustainability', 'data-science']
draft = true
+++

_TLDR: life cycle assessment is fundamental to measuring and reducing the environmental impact of goods and services. It is a very data-driven process, and is thus a relevant field for data scientists to apply their skillset. This post is a high-level summary of what I've learned doing LCA at Carbonfact for the past two years._

## LCA: the backbone of sustainability efforts

A growing number of companies are assessing their environmental impact. This can be for reporting purposes (e.g. public companies like Apple [publish](https://www.apple.com/environment/pdf/Apple_Environmental_Progress_Report_2023.pdf) progress reports), for compliance reasons (e.g. to obtain a [SBTI certification](https://www.wikiwand.com/en/Science_Based_Targets_initiative)), or to make their operations [eco-friendly](https://en.wikipedia.org/wiki/Environmentally_friendly).

The first step to any such initiative is to measure the impact of goods and services. This is called [life cycle assessment](https://www.wikiwand.com/en/Life-cycle_assessment) (LCA). It is composed of two steps:

1. [Life cycle inventory (LCI)](<https://www.wikiwand.com/en/Life-cycle_assessment#Life_cycle_inventory_(LCI)>) -- the process of breaking down a product/service into parts, so-called _activities_.
2. [Life cycle impact assessment (LCIA)](<https://www.wikiwand.com/en/Life-cycle_assessment#Life_cycle_impact_assessment_(LCIA)>) -- the process of measuring the impact of each activity.

I have been doing LCA at [Carbonfact](https://www.carbonfact.com/) for just over two years. We measure the environmental footprint of apparel and footwear. We've had success by incorporating data science into our solution. This post is a summary of what I've learned so far.

This is not a comprehensive guide, but rather a high-level overview of the concepts and tools involved. It is aimed at data scientists who want to get started with LCA. I've also written it for Carbonfact job applicants who wish to get a better understanding of our day-to-day work.

One bias of this article is to put emphasis on goods rather than services. The companies I work with through Carbonfact manufacture clothes. Their environmental emissions mostly come from the production of raw materials and the manufacturing processes. This is not the case for all companies. For instance, a software company's emissions mostly come from using computers, not making them.

## The footprint of a cup of coffee

A lot of people drink coffee. There are different ways to consume it. You could brew it yourself at home, and drink it from your favorite mug. You could also buy it from a coffee shop, and drink it from a disposable cup. The lifecycle is [different](https://www.visualcapitalist.com/from-bean-to-brew-the-coffee-supply-chain/) in each case. The environmental impact is therefore [not the same](https://theconversation.com/heres-how-your-cup-of-coffee-contributes-to-climate-change-196648), and comparing them is not straightforward.

For instance, a [disposable](https://www.linkedin.com/posts/bpinlova_all-the-coffee-lovers-out-there-how-do-activity-7196775980705710080-zIYZ) cup of coffee requires a cup, a lid, a sleeve, a stirrer, a maybe napkin. Each of these parts has its own sub-parts, and a lifecycle of its own. The coffee itself has a lifecycle too. It was grown, harvested, roasted, and brewed. The milk, if any, has a lifecycle too. It was produced, transported, and stored. The mug you drink from at home has a lifecycle of its own too.

In order to make an environmental comparison between different options, the first step is to define a [functional unit](https://www.youtube.com/watch?v=s5Tl1GS-gn8). This is the unit of measurement that will be used to compare the different options. In the case of a cup of coffee, it could be centiliters of coffee. The question thus becomes: what is the environmental impact of producing 1 centiliter of coffee in a disposable cup, compared to 1 centiliter of coffee in a mug? Picking a functional unit enables an apples to apples comparison.

<div align="center">
<figure >
    <img src="/img/blog/lca-for-dummies/coffee-cup.png">
    <figcaption><a href="https://www.youtube.com/watch?v=uXj3M9Z4B4g&t=2000s">Source</a></figcaption>
</figure>
</div>

Functional units lead to heated debates. For instance, in the footwear industry, the functional unit is usually the production of a pair of shoes. A pair of leather shoes thus has a higher environmental impact than a pair of sneakers made of synthetic materials. However, leather shoes last longer than sneakers. Some parties argue the functional unit should be a [fixed distance walked](https://fabrique-numerique.gitbook.io/ecobalyse/textile/durabilite), rather than a pair of shoes. This would take into account the fact that leather shoes last longer.

Another important concept is the [system boundary](https://www.youtube.com/watch?v=ftrmof_Ex_Y). This defines the limits of the system being studied. The system boundary could be the entire lifecycle of the coffee, from the farm to the cup. This would include the impact of growing the coffee beans, harvesting them, roasting them, etc. This is called a [cradle-to-grave](https://ecochain.com/blog/cradle-to-grave-in-lca/#:~:text=Cradle%2Dto%2DGrave%20is%20a,the%20end%20of%20its%20life) analysis. The system boundary could also be more limited. For instance, it could only include the impact of the coffee shop. This would exclude the impact of growing the coffee beans, and the impact of the cup. This is called a [cradle-to-gate](https://www.wikiwand.com/en/Life-cycle_assessment#Cradle-to-gate) analysis.

A wide system boundary is more comprehensive, but also more complex. It requires more data, and more assumptions. A narrow system boundary is simpler, but also less comprehensive. It is important to pick the right system boundary for the question at hand. There isn't much point capturing the impact of growing the coffee beans if the question is about the impact of the coffee shop. Moreover, I'm increasingly convinced it's relevant to collect information about things you can act upon. For instance, if you're a coffee shop owner, you can't do much about the impact of growing the coffee beans. You can however do something about the cups you serve the coffee in.

Goods usually have some kind of [bill of materials](https://www.wikiwand.com/en/Bill_of_materials) to describe their content. This is the starting point for a life cycle inventory. For instance, the bill of materials of a disposable cup of coffee would include the cup, the lid, the sleeve, the stirrer, the napkin, the coffee, etc. Each element could be represent as a row in a table. Each row would have a column for the quantity, the unit, and the source. This is wishful thinking.

In practice, the bill of materials is often non-existent, or incomplete. It often has to be constructed from [free text descriptions](/blog/carbonfact-nlp-open-problem/). This varies from one industry to another. For instance, energy labels for washing machines have existed [since 1992.](https://en.wikipedia.org/wiki/European_Union_energy_label) I expect the bill of materials for washing machines to be comprehensive and standardized. This isn't the case for the clothing industry, where regulations are barely starting to emerge.

The next step is to trace each element in the bill of materials back to its source. This is called a [supply chain](https://www.wikiwand.com/en/Supply_chain). For instance, the cup could be made of paper. The paper is made of cellulose fibers derived from wood. Said wood has been harvested from a forest, etc. Each step in the supply chain has its own environmental impact. Such steps are commonly referred to as activities in the LCA world. The goal of a life cycle inventory is to break down a product into its activities. It's a very [data-driven](https://www.fairlymade.com/blog/data-management-in-a-traceability-project-4-steps-to-know) process.

Last but not least, it is necessary to pick one or several [impact categories](https://ecochain.com/blog/impact-categories-lca/). For example, it is common to measure the impact of a product in terms of its [carbon footprint](https://www.wikiwand.com/en/Carbon_footprint). This is the amount of greenhouse gases emitted during the lifecycle of the product. This is usually expressed in kilograms of [CO2 equivalent](https://ec.europa.eu/eurostat/statistics-explained/index.php?title=Glossary:Carbon_dioxide_equivalent), or $kgCO_2$-$eq$. Other common impact categories include water usage, land usage, and toxicity. The choice of impact categories is important, and having tunnel vision on one can lead to [unintended consequences](<https://www.wikiwand.com/en/Rebound_effect_(conservation)>). That being said, many environmental indicators are in fact [correlated](https://www.carbonfact.com/blog/research/carbon-tunnel).

## LCA engines

It's boxes all the way down.

## Emission factor databases

## Collecting activity data

## Beware of false precision

- https://moutreach.science/2020/11/12/LCA-this-pseudo-science.html

Uncertainty:

- [A Review of Approaches to Treat Uncertainty in LCA](https://scholarsarchive.byu.edu/cgi/viewcontent.cgi?article=3524&context=iemssconference) by Heijungsa and Huijbregts with accompanying [slides](http://formations.cirad.fr/analyse-cycle-de-vie/pdf/Heijungs_2.pdf)

## Being problem specific vs. being general

- https://parisgoodfashion.fr/en/our-glossary/
- https://www.wikiwand.com/en/Units_of_textile_measurement
- https://ecochain.com/blog/life-cycle-assessment-lca-guide/

## Interpreting results

## Reporting

CSR report](https://www.hec.edu/en/faculty-research/centers/society-organizations-institute/think/so-institute-executive-factsheets/how-report-csr), possibly in order to obtain an [SBTI certification](https://www.wikiwand.com/en/Science_Based_Targets_initiative)

- Scopes
- Standard reports
- Spend-based reports

https://www.mckinsey.com/~/media/mckinsey/industries/retail/our%20insights/fashion%20on%20climate/fashion-on-climate-full-report.pdf

## Software

- [OpenLCA](https://www.openlca.org/). This short [tutorial](https://www.youtube.com/watch?v=r2Xdh5LT934) demonstrates how to measure the impact of a plastic bottle.
- [CMLCA](https://personal.vu.nl/R.Heijungs/CMLCA/home.html), developed at the university of Leiden where Reinout Heijungs worked. Seems a bit antique as of now.
- [EIO-LCA](http://www.eiolca.net/) -- web-based

- [SimaPro](https://pre-sustainability.com/solutions/tools/simapro/)
- [GaBi](https://gabi.sphera.com/international/index/)
- [Umberto](https://www.ifu.com/umberto/)
- [Brightway2](https://2.docs.brightway.dev/) is an opensource framework for LCA. It can be used as an interface to work with [different sources](https://2.docs.brightway.dev/notebooks.html#importing-data-example-notebooks), such as Ecoinvent, Agribalyse, or even a custom one. [This](https://github.com/maximikos/Brightway2_Intro) tutorial is a good start. There's more material [here](https://github.com/massimopizzol/B4B), [courtesty of](https://moutreach.science/2018/04/10/Teaching-experiment.html) Massimo Pizzol.

## Going further

---

Links:

- https://editor.yuka.io/eco-score-calculator
- https://ecobalyse.beta.gouv.fr/
- [Ecoinvent](https://ecoinvent.org/). Their [glossary](https://ecoinvent.org/glossary-terms/) is worth a look.
- Agribalyse
- Ecobalyse. Formally known as Wikicarbone.
- [Idemat](https://www.ecocostsvalue.com/data/idemat-and-idematlightlca/)
- [U.S. Life Cycle Inventory Database](https://www.nrel.gov/lci/)
- The functional unit - Consequential LCA
- A teaching experiment
- Science of Carbonfact
- Simulateur Eco-score
- Ecobalyse

https://chris.mutel.org/how-to-use-ecoinvent.html

- [Teaching videos](https://moutreach.science/2022/08/15/teaching-videos.html) by Massimo Pizzol from the university of Aalborg
- The [Wikipedia article](https://www.wikiwand.com/en/Life-cycle_assessment) on LCA is quite comprehensive and lists some of its pitfalls.
- [Life Cycle Analysis lecture from MIT ESD.S43 Green Supply Chain Management, Spring 2014](https://www.youtube.com/watch?v=gpuvUU0Nl4k)
- [LCA lecture](https://www.youtube.com/watch?v=uXj3M9Z4B4g) by Jeremy Faludi, along with a [hands-on workshop](https://www.youtube.com/watch?v=ATkUB9JJWHE) based on [this LCA exercice](https://venturewell.org/tools_for_design/measuring-sustainability/life-cycle-assessment-content/life-cycle-assessment-exercise/) from Venturewell
- If you understand French, I recommend checking out _C'est pas sorcier_ on YouTube. They have comprehensive half-hour videos about how materials are made, such as [cotton](https://www.youtube.com/watch?v=9Aov-vy_mgY), [paper](https://www.youtube.com/watch?v=4ZW4tX4qSHg), and [rubber](https://www.youtube.com/watch?v=kdP30T74oZE). It's definitely not wasted time and anchors LCA in the real-world.
- [Okala course](https://web.stanford.edu/class/me221/readings/Okala_Modules_1-7.pdf)
- https://moutreach.science/
- Attributional vs. consequential LCA

Here's some glossaries:

- https://consequential-lca.org/glossary/
