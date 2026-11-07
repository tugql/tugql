# Cross-database queries

One TugQL query can join and nest sources that live in different databases.
A `databases` block names each database once, and a source is qualified with
`alias::`.

```sql
databases (
  sales = 'datatug://chinook/sales?env=prod'
  geo
)
parameters (
  @MinSales integer default 1000
)
with BigInvoices as (
  from sales::Invoice as i
  where i.Total > 10
  select i.InvoiceId, i.BillingCountry, i.Total
)
from BigInvoices as b
join geo::Country as c
  on b.BillingCountry = c.Name
group by c.Name
having SUM(b.Total) >= @MinSales
select (
  c.Name as Country
  SUM(b.Total) as TotalSales
)
```

`sales` is bound to a DataTug reference; `geo` has no target, so the tool
running the query binds it.

## The `databases` block

- It is the first block of the document, before `parameters`.
- Each line declares one alias: `alias` or `alias = '<target>'`.
- An alias is an unquoted word that is not a reserved keyword. Aliases are
  unique, ignoring case.
- A target is a single-quoted string. It is usually a DataTug reference
  (`'datatug://<project>/<db>?env=<env>'`) but may be any connection string the
  executing tool understands. It must be quoted because `//` starts a comment.
- A target never carries a password. A target such as
  `'postgres://reader:secret@host/db'` is rejected (`database_credentials`);
  credentials come from the executing tool.
- An alias with no target is bound by the tool that runs the query, for
  example from its project or environment settings. Saved queries usually leave
  targets out so one query runs in every environment.

## Qualified sources

- `alias::path` qualifies a `from` or `join` source in the main query, a CTE or
  a scalar subquery: `sales::Invoice`, `sales::dbo.Invoice`,
  `crm::spaces/{SpaceID}/ext/datatug/projects`.
- `::` touches both the alias and the path.
- An unqualified source reads the default database, so a query without a
  `databases` block means what it always meant. Once a query declares
  databases, an unqualified source can only match the default database.
- A query that runs across databases qualifies every source, as the example
  above does. To read the default database in such a query, declare it as an
  alias too (see [Execution](#execution)).
- A join across two databases states its `on` explicitly. Relationship
  metadata belongs to one database, so `on` cannot be inferred across two.

## Errors

| Code | When |
|---|---|
| `undeclared_database` | a source uses an alias the block does not declare |
| `unused_database` | the block declares an alias no source uses |
| `duplicate_database` | an alias is declared twice |
| `invalid_database` | a malformed declaration, alias or target |
| `database_credentials` | a target holds a password |
| `nested_databases` | a `databases` block anywhere but the start of the root document, including an imported saved query |
| `cross_database_relationship` | a cross-database join omits `on` |

## TugQT (YAML form)

The declarations are a top-level `databases` list and a qualified source
carries `database: <alias>`:

```yaml
format: tugqtree
version: 1
databases:
    - name: geo
query:
    from:
        name: Invoice
        alias: i
        joins:
            - from:
                database: geo
                name: Country
                alias: c
              "on":
                - op: ==
                  left:
                    field: BillingCountry
                    source: i
                  right:
                    field: Name
                    source: c
    columns:
        - field: InvoiceId
          source: i
        - field: Name
          source: c
```

This is the tree of `databases ( geo )` + `from Invoice as i join geo::Country as c on i.BillingCountry = c.Name select i.InvoiceId, c.Name`:
It illustrates the tree shape only: `Invoice` is unqualified, so this query
parses and resolves but cannot run across databases (see below).

## Execution

The executing tool binds each alias to a database and runs the query through
DALgo's federated executor. Federated execution requires every source to be
qualified: a source with no `alias::` fails closed instead of guessing a
database. Each source is read from its own database; joins,
grouping and subqueries that span databases run in the engine, not in one
database's native SQL. The executing tool must check every source against its
own database's access policy.
