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

## References

- [GitHub Codespaces](https://github.com/features/codespaces)
- [Kerncijfers economische migratie](https://www.vlaanderen.be/datavindplaats/catalogus/abk01-jaarrapport-kerncijfers-economische-migratie)
- [Datavindplaats](https://www.vlaanderen.be/datavindplaats)
- [sqlite-utils](https://sqlite-utils.datasette.io/en/stable/cli.html)
- [Datasette](https://datasette.io/)
- [Vega Lite](https://vega.github.io/vega-lite/)
- [Vega](https://vega.github.io/vega/)
- [Pro Git](https://git-scm.com/book/en/v2)
- [everything curl](https://everything.curl.dev/project/does.html#curl-the-command-line-tool)