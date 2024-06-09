+++
date = "2024-06-09"
title = 'LCA: exit the matrix'
tags = ['sustainability', 'python']
+++

Measuring the environmental impact of a product is done using [life cycle assessment](https://en.wikipedia.org/wiki/Life-cycle_assessment) (LCA). This is a methodology that breaks down a product's life cycle into stages ([LCI](<https://www.wikiwand.com/en/Life-cycle_assessment#Life_cycle_inventory_(LCI)>)), and measures the impact of each stage on the environment ([LCIA](<https://www.wikiwand.com/en/Life-cycle_assessment#Life_cycle_impact_assessment_(LCIA)>)).

There are a few pieces of LCA software to choose from. The leading ones are [SimaPro](https://simapro.com/), [GaBi](https://sphera.com/life-cycle-assessment-lca-software/), [openLCA](https://www.openlca.org/), and [Umberto](https://www.ifu.com/umberto/). These are all proprietary software, and they're expensive. But there's a free and open source alternative: [Brightway](https://docs.brightway.dev/en/latest/).

[Christopher Mutel](https://chris.mutel.org/) (the creator of Brightway) recently left [Ecoinvent](https://ecoinvent.org/) and is now focusing on [Départ de Sentier](https://www.d-d-s.ch/). The latter is a non-profit that wants to make LCA more accessible to everyone. They do this through various initiatives, one of which is the development of Brightway. They just organized a [workshop](https://github.com/Depart-de-Sentier/Spring-School-2024/), where they shared some [tutorials](https://github.com/Depart-de-Sentier/Spring-School-2024/tree/main/class-materials/brightway-basics), which seem to be [rehashed](https://github.com/Depart-de-Sentier/Autumn-School-2022/blob/main/tutorials/datapackages/Datapackages.ipynb) from a previous workshop in 2022. These workshops seem to be academic in nature, whilst there is also [Brightcon](https://ecoinvent.org/blog/brightcon-2023-broad-open-sustainability-community/) which is more industry focused.

More Brightway learning resources can be found online. For instance, Massimo Pizzol [wrote](https://moutreach.science/2018/04/10/Teaching-experiment.html) some [notebooks](https://github.com/massimopizzol/B4B/tree/main) for his students at the university of Aalborg. There's also [this](https://github.com/maximikos/Brightway2_Intro/blob/master/BW2_tutorial.ipynb) comprehensive tutorial by Maximilian Koslowski.

---

I have a confession to make: I don't think Brightway is a good piece of software, for several reasons.

First of all, it's not user-friendly. Brightway throws too many concepts in your face right from the start. This makes it difficult for people from other disciplines to get started. I'm a data scientist, so I don't know what activities, nodes, exchanges, and other concepts Brightway mentions are. I just want to declare a product and get some results, which Brightway gives me a hard time with.

In my experience, good software hides the details and only shows them when they're needed. Brightway does the opposite: it shows you all the details right from the start, and you have to figure out which ones are important. This is not a good design. There should be abstractions that hide the complexity of the underlying model. For example, scikit-learn became everyone's favorite machine learning library because it abstracts just the right amount of complexity, for instance with its `fit` and `predict` methods.

Second, Brightway heavily relies on matrices. There's a bunch of them: the technosphere, the biosphere, and the characterization matrix. These matrices describe the inputs/outputs of different processes. There are various formulas that are applied to these matrices to calculate the environmental impact of a product. Many of these formulas seem to involve matrix inversion.

I don't think matrices are the right grammar for conducting an LCA. It's not how people think. People think in terms of processes, inputs, outputs, and impacts. They don't think in terms of matrices. Matrices are a mathematical abstraction that is not intuitive to most people. They should be hidden from the user. In fact, Brightway also offers an object-oriented way to run an LCA. Therefore, I don't understand why they even bother mentioning matrices in their documentation.

Finally, Brightway's reliance on matrices doesn't appear to be justified performance-wise. Brightway feels slow. The documentation is also sprinkled with tips on how to improve performance, which is a red flag. I'm sure some LCAs can be hairy, but still this isn't fluid dynamics or finite element analysis. It's just a bunch of processes linked together.

---

I'm playing devil's advocate. On the one hand, I have a deep respect for people who write open source software. On the other hand, I'm suspicious of software that comes from academia. The developer experience is close to non-existent. The people who use Brightway must be LCA experts themselves who can put up with Brightway's complexity. This is a shame, because LCA is a methodology that should be accessible to anyone who can write code.

There isn't really any credible open source alternative to Brightway. Someone mentioned [LCA as Code](https://github.com/kleis-technology/lcaac) to me. But it's written in Kotlin, which closes the door to a lot of people who want to contribute. That being said, I like the declarative approach of LCA as Code's interface. It feels like a step in the right direction. Brightway's semantics are somewhat more procedural, which feels cumbersome.

In the recent workshop material, there is a [toy notebook](https://github.com/Depart-de-Sentier/Spring-School-2024/blob/main/class-materials/brightway-basics/2%20-%20Building%20and%20using%20matrices%20in%20bw2calc.ipynb) to measure the footprint of a bike.

<div align="center" >
<figure style="width: 90%; margin: 0;">
    <img src="https://github.com/Depart-de-Sentier/Autumn-School-2022/blob/main/tutorials/datapackages/simple-graph.png?raw=true" style="box-shadow: none;">
</figure>
</div>

The example sounds rather straightforward. The bike is made of carbon fibre, which requires natural gas and emits CO2. However, the required Brightway required is surprisingly complex. The object-oriented approach is [simpler](https://github.com/Depart-de-Sentier/Spring-School-2024/blob/main/class-materials/brightway-basics/1%20-%20The%20supply%20chain%20graph.ipynb). But it's still too complex because it writes data to a database, which feels unnecessary. I think it could be simpler.

Here's a little Python framework I came up with:

```py
import dataclasses

@dataclasses.dataclass
class Input:
    process: 'Process'
    amount: float

@dataclasses.dataclass
class Process:
    name: str
    unit: str
    inputs: list[Input] = dataclasses.field(default_factory=list)

    def __rmul__(self, amount: float):
        return Input(self, amount)

    def measure(self, unit):
        if self.unit == unit:
            return 1
        return sum(
            inp.process.measure(unit) * inp.amount
            for inp in self.inputs
        )
```

And here's how to use it:

```py
kgCO2e = Process('CO2e', 'kgCO2e')
natural_gas = Process('Natural gas NO', 'MJ')
carbon_fiber = Process(
    'Carbon fiber DE', 'kg',
    [26.6 * kgCO2e, 237.3 * natural_gas]
)
bike_production = Process(
    'Bike production DK', 'kg',
    [2.5 * carbon_fiber]
)

print(bike_production.measure('kgCO2e'))
```

```
66.5
```

Simple, right? Granted, this just measures a point-wise carbon footprint. A more serious LCA would involve measuring the uncertainty of the results, and maybe even doing sensitivity analysis. But this is a good start because it's minimalistic. I'm sure it could be extended to handle more complexity, whilst keeping the interface simple. I'm also not worried about performance: there are many opportunities to [cache](https://docs.python.org/3/library/functools.html#functools.cache) intermediate results. I don't think matrices and matrix inversion are necessary.

Brightway's isn't just useful for LCA. It also has tooling to import emission factor databases, such as [Ecoinvent](https://docs.brightway.dev/en/latest/content/faq/ecoinvent.html) and [Agribalyse](https://doc.agribalyse.fr/documentation/utiliser-agribalyse/acces-donnees). In fact, I suspect that many people use Brightway solely for this purpose. I'm sure everyone appreciates the work that went into making these databases accessible. However, knowing how over-engineered Brightway is, I wonder if there's a simpler way to import these databases.

I want to emphasize that I'm not against Brightway. I'm just trying to figure out why it's so complex. If there are good reasons for this, then I'm not aware of them. I think it's healthy to question existing software when it doesn't feel good to use. This is how we make progress. A good recent example is [llm.c](https://github.com/karpathy/llm.c), which is an attempt by Andrej Karpathy and others to write a minimalistic alternative to PyTorch.

So what's next? I'm not entirely sure. I'm busy at Carbonfact with day-to-day operations. I don't think I have time to work on a new LCA framework. But we're going to be using Brightway for the foreseeable future. We have more and more customers who require bespoke LCAs. We also heavily rely on emission factor databases such as Ecoinvent. I hope that the Brightway community can address the concerns I raised. If not, I'm sure someone else will eventually come up with an alternative.
