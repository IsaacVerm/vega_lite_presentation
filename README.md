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
- aggregate by nationality using [aggregate: 'count'](https://vega.github.io/vega-lite/examples/bar_aggregate_sort_by_encoding.html)
- [sort with biggest counts to the left of the x axis](https://vega.github.io/vega-lite/examples/bar_aggregate_sort_by_encoding.html)

### Visualisation with the same bar chart in Power BI

Requirement: get Deneb Power BI extension so you can run Vega Lite in Power BI as well.

Save Vega Lite spec as JSON file in repo.
make sure to use double quotes (if not Deneb plays up)
the Vega Lite examples do this by default

- open new blank Power BI report
- data source Web and use raw Github URL pointing to csv
- download and install Deneb (PBI certified)
- copy `bar.json` spec into Power BI
- run

This doesn't work because you need to make a couple of changes:

- increase canvas size
  - applies to both the visual and the canvas itself
  - something like 1000 x 3000 so the labels on the x axis are visible
  - lots of countries only have a single applicant
- increase row size
  - [limited to 10000 rows by default in Deneb Power BI](https://deneb.guide/docs/1.0/dataset#data-row-limits)
  - to increase in Power BI: format canvas > data limit settings > override row limit
  - if not a graph will displayed but just using a 10000 row sample
- use index to override row context
  - Power BI provides the data layer
  - https://github.com/deneb-viz/deneb/issues/86#issuecomment-893001933
  - by default our dataset doesn't have a unique identifier
  - to create one go edit query > add column > add index
  - use this new index created in power query in the values Values field of the deneb visual

> You need to force Power BI to keep all rows unique, so that the spec can do what you want. we do this by adding a column to the dataset that contains a unique value

All default Power BI filters, DAX,... still apply so you can reuse your knowledge.
Fun check: count by nationality and filter by rechtsvorm (Federale overheidsdiensten and andere federale diensten)

### Update graph and commit

Shows the benefits of git.
I added a simplified `bar_powerbi.json` bar chart.
1 line for each element in the graph (most readable in my opinion):

```json
{
    "data": {"name": "dataset"},
    "layer": [
        {"mark": "bar",
         "encoding": {
                "x": {"field": "NATIONALITEIT","sort": "-y"},
                "y": {"aggregate": "count"}
            }
        }
    ]
}
```

So 1 line for data, 1 line for mark bar and a line for each mapping in the encoding (x and y in this case).

## What to remember

### Vega Lite is text so can be checked into version control

- easy to track changes
- even easier if you keep each graphical element on a line of its own because git diff works by comparing lines

### Vega Lite syntax isn't too hard to understand

Minimum needed:

- data
- mark: what kind of graphical element do you want (bar, line,...)
- encoding: how should the data be translated to attributes of the mark?

```json
{
    "data": {"name": "dataset"},
    "mark": "bar",
    "encoding": {
      "x": {"field": "NATIONALITEIT","sort": "-y"},
      "y": {"aggregate": "count"}
    }    
}
```

Best to first do things by hand (but based on examples provided by Vega Lite).

## `git` commands to remember

- `git log --oneline -n`
- `git checkout`
- `git commit -m "{message}"`
- `git push`

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
