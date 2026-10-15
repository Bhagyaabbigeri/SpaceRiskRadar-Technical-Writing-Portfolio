# SpaceRisk Radar — Technical Writing Portfolio

**Engineering Documentation, Technical Articles, and Space Technology Explainers**

This portfolio documents the engineering concepts, system architecture, data
workflows, and technical principles behind SpaceRisk Radar, a satellite-monitoring
and orbital-risk visualization project. It demonstrates how complex technical
systems can be explained through structured documentation, clear diagrams,
practical guides, and accessible engineering articles.

The documentation distinguishes established orbital-science concepts from the
project's own implementation and its simplified, illustrative calculations.
SpaceRisk Radar is an educational visualization and proximity-screening project;
its heuristic scores are not operational collision probabilities.

## Explore the portfolio

- [Start at the documentation homepage](docs/index.md)
- [Read the satellite orbital data and TLE article](docs/articles/tle-and-orbital-data.md)
- [Understand SGP4 propagation](docs/articles/sgp4-propagation.md)
- [Read the conjunction-screening explanation](docs/articles/conjunction-screening.md)
- [Explore the system architecture](docs/engineering/architecture.md)
- [Follow the data flow](docs/engineering/data-flow.md)
- [Browse the verified API reference](docs/engineering/api-reference.md)
- [Run the application locally](docs/guides/getting-started.md)
- [Use the dashboard](docs/guides/dashboard-user-guide.md)
- [Review troubleshooting notes](docs/guides/troubleshooting.md)
- [Look up space technology terms](docs/reference/glossary.md)
- [About this portfolio](docs/about.md)

## Project source

The portfolio is separate from, and does not replace, the application repository:

- [SpaceRisk Radar source repository](https://github.com/Bhagyaabbigeri/SpaceRisk-Radar)
- [Original application README](https://github.com/Bhagyaabbigeri/SpaceRisk-Radar/blob/main/README.md)

The application source is the implementation reference for claims in these
documents. Examples and descriptions are written for this portfolio rather
than copied from the application's README.

## Preview the documentation website

From this repository, install the documentation dependencies and serve the site:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements-docs.txt
python -m mkdocs serve
```

🌐 **Live Documentation Website:** [Technical Writing Portfolio](https://bhagyabiggeri.github.io/SpaceRiskRadar-Technical-Writing-Portfolio/)

```powershell
python -m mkdocs build --strict
```

This repository contains the source documentation only. The site has not been
deployed.
