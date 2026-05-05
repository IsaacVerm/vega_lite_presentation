# Vega Lite presentation 2026-05-19

## Pipeline

### Open repo with GitHub Codespace

Creating the repo and codespace is free (free tier codespaces). You just need to create GitHub account.

### Fetch data

Kerncijfers economische migratie. Found through Datavindplaats (open data Vlaanderen).

You open the page and use inspect element to find URL to fetch CSV file from:

![alt text](image.png)

```sh
curl "https://opendata.wewis.vlaanderen.be/api/explore/v2.1/catalog/datasets/abk01_jaarrapport_kerncijfers_v1/exports/csv?lang=nl&timezone=Europe%2FBrussels&use_labels=true&delimiter=%3B" -o kerncijfers_economische_migratie.csv
```

### Inspect the data

`sqlite-utils` to turn CSV in `sqlite` database.
`datasette` to run some queries.

Setup:

```sh
pip install sqlite-utils
pip install datasette
```

`datasette/sqlite-utils`:

```sh
sqlite-utils insert migration.db applications kerncijfers_economische_migratie.csv --sniff --detect-types
datasette migration.db
```

`--sniff` because semicolons instead of commas are used as delimiters.
`--detect-types` so not everything is text type.

### Choose question to answer

Number of demands by nationality applicant.

Test this query in Datasette:

```sql
select
  nationaliteit,
  count(*) as count
from
  applications
group by
  nationaliteit
order by
  count desc
```

### Visualisation with bar chart in HTML

Copy [Vega Lite example](https://vega.github.io/vega-lite/usage/embed.html#start-using-vega-lite-with-vega-embed) as `bar.html`

To view the file you need a server.
Use Live Server extension.
Benefit of Live server: live reload so any change you make to the HTML can be copied
This opens a chart which you can right away export a graph using 3 dots.

We don't want a random bar graph but the count by nationality so some modifications have to be made:

- [change input data source to CSV](https://vega.github.io/vega-lite/docs/data.html#url)
- [aggregate by nationality] using [aggregate: 'count'](https://vega.github.io/vega-lite/examples/bar_aggregate_sort_by_encoding.html)
- [sort with biggest counts to the left of the x axis](https://vega.github.io/vega-lite/examples/bar_aggregate_sort_by_encoding.html)

### Visualisation with the same bar chart in Power BI

Requirement: get Deneb Power BI extension so you can run Vega Lite in Power BI as well.

Save Vega Lite spec as JSON file in repo.
make sure to use double quotes (if not Deneb plays up)
the Vega Lite examples do this by default

- open new blank Power BI report
- data source Web and use raw Github URL pointing to csv
- download and install Deneb (PBI certified)

### Update graph and commit

Shows the benefits of git.

## References

- [GitHub Codespaces](https://github.com/features/codespaces): Linux container with Python,... pre-installed
- [Kerncijfers economische migratie](https://www.vlaanderen.be/datavindplaats/catalogus/abk01-jaarrapport-kerncijfers-economische-migratie): data used in this presentation
- [Datavindplaats](https://www.vlaanderen.be/datavindplaats): open data Vlaanderen
- [sqlite-utils](https://sqlite-utils.datasette.io/en/stable/cli.html): library to convert to `sqlite`
- [Datasette](https://datasette.io/): library to inspect `sqlite` database
- [Vega Lite](https://vega.github.io/vega-lite/): Vega Lite is a minimal version of Vega
- [Vega Lite examples](https://vega.github.io/vega-lite/examples/): take an example and modify to see how things work
- [Vega](https://vega.github.io/vega/)
- [Pro Git](https://git-scm.com/book/en/v2): reading only the introduction is sufficient for 90% of what you need to do
- [everything curl](https://everything.curl.dev/project/does.html#curl-the-command-line-tool)
- [Deneb](https://deneb.guide/): Power BI implementation of Vega Lite
- [A Layered Grammar of Graphics](https://byrneslab.net/classes/biol607/readings/wickham_layered-grammar.pdf)
- [ggplot2](https://ggplot2.tidyverse.org/articles/ggplot2.html): has the same ancestor as Vega Lite: the grammar of graphics