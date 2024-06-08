Most application is developed in layered structure. Each leyer hide complexity of another layer. One of layers is DataModel. There are many data models, and each datamodel is suitable for specifice use case. In particular, we will compare the relational model, the document model, and a few graph-based data models. 

## Relational Model Versus Document Model
SQL had become the tools of choice for most people who needed to store and query data with some kind of regular structure.

The RDBMS use cases appear in typically transaction processing (entering sales or banking transactions, airline reservations, stock-keeping in warehouses) and batch processing (customer invoicing, payroll, reporting).

The NoSql use cases are 
- A need for greater scalability than relational databases.
- A widespread preference for free and open source software over commercial database products.
- Specialized query operations that are not well supported by the relational model.
- Desire for a more dynamic and expressive data model

### Impedance mismatch
Most application development today is done in object-oriented programming languages, which leads to a common criticism of the SQL data model.
An awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns.