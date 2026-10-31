# Lab dbt: answers to the questions

## Bronze layer

- **Why leave the bronze files untouched:** if a transformation bug is found six months later, you fix the model and rebuild silver and gold from the raw data. Nothing was lost or overwritten, and you can also audit what the source really sent.
- **Orders ingested every hour for years:** use Parquet, partitioned by date in the Hive layout, `year=/month=/day=`. Queries then only read the periods they need, and the column statistics skip the rest. A table format such as Iceberg or Delta, plus periodic compaction of small files, would be the next step.

## Silver layer

- **What happens when a view is queried:** DuckDB runs its SQL every time, so it re-reads and re-parses the CSV on S3. A view is preferable when the data is small, must always reflect the latest source, or is rarely queried. A table is better when the transformation is expensive or queried often.
- **UUID as VARCHAR:** the text form is 36 characters, so 36 bytes, more than twice the 16 bytes of the `UUID` type. DuckDB adds a string header on top of that.

## Tests

- **Strategy per dashboard:** for customer age, set invalid birthdates to `NULL` and exclude them, otherwise you display negative or absurd ages. For sales per region, keep every record: the birthdate is irrelevant there, and rejecting those orders would understate sales. Warning and monitoring is enough.
- **Detecting the random ZIP codes:** a test checking each state/ZIP pair against a reference mapping of ZIP ranges per state, as in exercise 4. The authoritative source is USPS data or a public dataset derived from it.

## Exercise 4 considerations

- **Where the reference data comes from:** an official source such as USPS, not something invented in the project.
- **Whether it should be a seed:** it suits a seed because it's small and changes rarely, and it is then versioned in Git along with the code. A large or frequently updated reference would be better ingested as a source, like the bronze files.
- **Whether validation belongs in silver:** yes. Silver is where data is validated and cleaned, so problems are caught before gold and the dashboards.
- **When the reference data changes:** update the seed, run `dbt seed`, and rebuild. Past results can change, so the change should go through a commit and review like code. Keep history if you need to reproduce old results.

## Exercise 5

- **Where the data is stored:** a view stores only the query, and the data stays in the CSV on S3. A table copies the transformed data into `lab_medallion.duckdb`.
- **When the transformation runs:** for a view, at every query. For a table, once, at `dbt build`.
- **When the CSV changes:** a view sees it immediately. A table is stale until the next `dbt build`.
- **Which uses more storage:** the table.
- **Which avoids re-reading the source:** the table.
