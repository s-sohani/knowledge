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

![[Pasted image 20240608101301.png]]


The JSON representation has better locality than the multi-table schema. If you want to fetch a profile in the relational example, you need to either perform multiple queries or perform a messy multiway join between the users table and its subordinate tables. But has some disadvatages, if that information is duplicated, all the redundant copies need to be updated. That incurs write overheads, and risks inconsistencies (where some copies of the information are updated but others aren’t).

If the database itself does not support joins:
- you have to emulate a join in application code by making multiple queries to the database.
- data has a tendency of becoming more interconnected as features are added to applications.

Example:
	Organizations and schools as entities
		In the previous description, organization (the company where the user worked) and school_name (where they studied) are just strings. Perhaps they should be references to entities instead? Then each organization, school, or university could have its own web page (with logo, news feed, etc.); each résumé could link to the
		organizations and schools that it mentions, and include their logos and other information .
	Recommendations
		Say you want to add a new feature: one user can write a recommendation for another user. The recommendation is shown on the résumé of the user who was recommended, together with the name and photo of the user making the recommendation. If the recommender updates their photo, any recommendations they have written need to reflect the new photo. Therefore, the recommendation should have a reference to the author’s profile.

### Are Document Databases Repeating History?
While many-to-many relationships and joins are routinely used in relational databases, document databases and NoSQL reopened the debate on how best to represent such relationships in a database.

The text discusses the historical debate on how to best represent relationships in databases, which dates back to the earliest computer databases and was reignited by the advent of NoSQL and document databases. In the 1970s, IBM's Information Management System (IMS) was a popular hierarchical model database, resembling the JSON structure used in modern document databases. Both IMS and document databases handle one-to-many relationships well but struggle with many-to-many relationships and do not support joins, requiring data duplication or manual reference resolution.

Two prominent models emerged to address these limitations: the relational model (which became SQL) and the network model (CODASYL model). The network model, an extension of the hierarchical model, allowed records to have multiple parents, facilitating many-to-many relationships. It used pointers to link records, requiring programmers to manually navigate access paths, which made querying and updating complex and inflexible.

Despite its efficiency for the limited hardware of the time, the network model's complexity and inflexibility made it challenging to modify an application's data model. This historical context is relevant today as developers face similar issues with modern document databases.

Although document databases store nested records similarly to the hierarchical model, both relational and document databases handle many-to-one and many-to-many relationships using unique identifiers resolved at read time.

### Relational Versus Document Databases Today
When comparing relational databases to document databases, key differences include schema flexibility, performance, and handling of data relationships. The document model offers schema flexibility, better performance due to data locality, and can align more closely with application data structures. However, it lacks robust support for joins and handling many-to-many relationships, which can complicate application code. Relational databases excel in supporting complex relationships and provide automated query optimization, simplifying code maintenance. They enforce **schemas at write time**, unlike the **schema-on-read** approach of document databases, which allows for more flexible data handling but can lead to inconsistent data structures. Despite their differences, both models are converging, with each adopting features of the other, suggesting a hybrid model as a future direction for databases.

## Query Languages for Data
When the relational model was introduced, it revolutionized querying data by using SQL, a declarative query language, unlike the imperative coding used by IMS and CODASYL. Declarative languages like SQL specify the desired result without detailing the steps to achieve it, allowing the database system to optimize query execution. This abstraction enables automatic performance improvements, hides implementation details, and facilitates parallel execution. In contrast, imperative languages require step-by-step instructions, making them harder to optimize and parallelize. Declarative queries, by focusing on what data to retrieve rather than how to retrieve it, are generally more concise and easier to work with.

### Declarative Queries on the Web
Declarative query languages have advantages beyond databases, as illustrated by web browsers. For example, using CSS to style a web page is declarative: you specify what elements should look like based on their patterns, like making the title of the selected page have a blue background. In contrast, an imperative approach in JavaScript involves detailed, complex, and error-prone steps to achieve the same result. Declarative methods, such as CSS, automatically adjust to changes and are more maintainable and efficient. This concept parallels SQL's advantages in databases, offering simplicity, ease of optimization, and better handling of changes compared to imperative code.

### MapReduce Querying
MapReduce is a programming model for processing large-scale data across multiple machines, popularized by Google and used by some NoSQL databases like MongoDB and CouchDB for read-only queries. It is a middle ground between declarative and imperative querying, using snippets of code repeatedly executed by the processing framework. An example query in MongoDB to count shark sightings per month involves writing map and reduce JavaScript functions. However, writing these functions can be complex and less optimized compared to declarative queries. To address this, MongoDB introduced the aggregation pipeline, a declarative query language similar to SQL, providing a more user-friendly and optimizable approach.

## Graph-Like Data Models