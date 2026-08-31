
Delta Lake is a open source storage framework that brings reliability to data lakes. 

It comes with these killer features. 

### Transaction Log (Delta Log)
- Ordered records of every transaction performed on the table.
- Single Source of Truth.
- JSON file contains commit information:
	- Operation performed + Predicates used
	- data files affected (added/removed)

-- So this functions similar to the transaction log in a relational database but on a data lake? 

300007200211484
