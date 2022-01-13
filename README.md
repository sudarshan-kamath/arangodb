# ArangoDB

Here are some notes made on the ArangoDB tutorial that is available in Udemy

Multi-model databases:

Supports Documents (JSON), Graphs and Key-Values in a single DB.
Supports storing the data using the AQL and again querying the same DB using AQL.
Supports in one database

Layered Approach (Other DBs) & Multi-Model approach (ArangoDB approach).

ArangoDB also provides ArangoSearch, a natively integrated text search and indexing in addition to the AQL.

AQL:
Keys are case sensitive.

First, we create a new collection and add the documents into the collection. 

Inserting a document: 
Creation of a document (an entry):
INSERT {_key:'Test'} INTO 'Airports' RETURN NEW

The RETURN keyword is a special keyword that returns the document. Here, NEW returns the newly created entry.


Fetching documents:

Similarly, we can return this entry in another manner, with the help of the key:

RETURN DOCUMENT(‘Airports/Test’)
RETURN DOCUMENT('Airports',['Test','test'])




Updating Documents:

Suppose we want to update the document with a given Key in a collection:
	UPDATE {_key:'Test'} with {newVal: 1234} IN Airports
RETURN NEW


FOR Loop:
Iteration over an array.
Supports iterating over a collection.
	
	Example:
	FOR doc IN ['test','Test']
		UPDATE doc with {newVal: 1234} IN Airports
    		RETURN NEW

Remove Operation:

This has a limitation of single remove operation per iteration of a query per collection.
 
FOR doc IN ['test','Test']
	REMOVE doc IN Airports

The Remove operation returns an empty array, therefore RETURN cannot be used to show the deleted item. The success is displayed by the number of writes.

Important Note:
The ArangoDB automatically indexes the _id and the _key as a collection primary key. _from and _to are indexed as special edge collections used to improve graph lookups.

Indexes:
Primary
Edge
Hash
Skiplists
Persistent
Geo 
Fulltext



Primary Index:
Primary Index is automatically generated. It covers the _id and the _key. It is an unsorted hash index. It is used specifically for equality lookups. 

Edge Index:
The edge index is also automatically generated when creating the edge collection for graph traversals. This covers the equality lookups for _from and _to attributes. 

Hash Index:
The hash index is user defined for custom attributes. Both the primary index and the edge index are hash indexes, but they are automatically created. Own hash indexes can be defined to speed up the performance of the queries. 

______________________________________________________________________

Filter

Filter is used to match the attributes based on the logical conditions. The condition must evaluate to True or False. Logical operators such as AND - && , OR - || and NOT ! operators. Multiple Filter statements can also be used.


FOR flight IN Flights 
	FILTER flight.TailNum == “N592ML”
	RETURN flight

When the Profile button is clicked, the steps taken to execute the query are shown. It also shows the number of times the query had to iterate through the Flights collection. For example, the iteration was done over 286,463 records, to fetch only 97 records to be filtered and returned.
Therefore, it makes sense to create an index over that variable.

For this use case, a Hash Index can be created for the variable by clicking on the indices tab in the Web UI. The tab selectivity estimate refers to the ranking, which index to use. The query runs in less time now, highlighting the use of indexing as a major factor in retrieval. 

GeoJSON
Native support based on S2 by Google. Full GeoJSON functionality. (Not focusing on this as this feature is not required for the project currently).

Joins
Joins are used to combine the data from different sources. They result from multiple collections. They are achieved by nested for loops.

For example, suppose we need to know the details of all the flights that land in Dallas

FOR airport in Airports
	FILTER airport.city == “Dallas”
	FOR flight in Flights
		FILTER flight._to == airport._id
		RETURN {
			“airport”:airport.name,
			“Flight”: flight.flightNum
		}

Performing Joins in a clustered environment may have certain bottlenecks and introduce further network hops and even proper indexing may not help 

Collect
COLLECT keyword is used for the aggregation and grouping tasks. The syntax is as follows:

COLLECT variableName = expression options 

Suppose we want to group the airports according to the state they are in and get the states that have the airports,

FOR airport in Airports
	COLLECT state = airport.state
	RETURN state

If we want to fetch the State and their corresponding count of the airports, then

FOR airport in Airports
	COLLECT state = airport.state WITH COUNT INTO total
SORT total DESC
	RETURN {
	    State: state,
	    "Total Airports": total
	    }

The above query does the purely grouping part.
Aggregate
Aggregate is a clause of the COLLECT. It must always follow the COLLECT function. 
MIN(Aggregate) keeps the minimum value and traverses throughout the database until it finds the smallest element. It only has a single element in store.
MAX(Aggregate) keeps the maximum value. 

FOR flight in Flights
	COLLECT AGGREGATE
	minDistance = MIN(flight.Distance),
	maxDistance = MAX(flight.DIstance)
RETURN {
	“Shortest Flight”: minDistance,
	“Largest Flight”: maxDistance
}

Without the aggregation, multiple For loops would have been used such that the entire database would have to be kept in the memory until the query executes.
Graphs
A Graph consists of Vertices and Edges. In ArangoDB, Vertices/Nodes are stored in normal document collections. As the example database explains, the airports are stored as normal document collections and the Flights are stored as edge collection. The edge collections are special collections of the JSON documents and it defines the relationships between the documents. 

The edge collections are not tied to the vertices collection. This allows the edge collection to be independent of the standard document collections.
Querying a graph
Querying can be done using a named graph or an anonymous graph. 

Named graphs are completely managed by the ArangoDB and give access to the complete spectrum of the features such as
Transactional Modifications#
Edge Consistency Checks
Edge Cleanup (When vertices are removed from the graph)
Graph Module (arangosh, drivers) (But the downside is that the AQL cannot be used)

Anonymous Graphs offer flexible access, no edge definitions / consistency and offer client-side consistency. 

During the named graph, the consistency check occurs on the database while with the anonymous graph, it happens elsewhere.

A good guide to the decision making would be the following questions:
Are integrity checks required?
Where should these checks occur? Client or the Server?
Is the readability improved when using a named graph query?

AQL Querying using Graphs

The AQL when using the Graph type queries would be using the Vertex, edge 

The special form of the FOR loop is to be used for the graph traversals. 

The for loop is generally represented by syntax 
FOR v, e, p IN 1..1 OUTBOUND
‘collection/documentName’
GRAPH ‘graphName’ # This refers to the edge collection
RETURN p

	where v represents the current vertex in traversal and the only required option
	e represents the edge, which would be the flight document
	p represents the path variable - it contains two arrays - array of Vertices and array of Edges.

IN 
IN defines the depth of the traversal IN min..max. If the min and the max numbers are the same, then only one number can be given as the input.This defines the depth of the traversal.

Specifying the Direction
The direction of the edge can be specified by the keywords INBOUND, OUTBOUND or ANY. This would be using the _to and _from keys. To search for the flights have the _from field as JFK, then the keyword OUTBOUND is used.

The start vertex forms
The Start vertex can be defined 
as an ID string - This defines the collection and the DocumentID in quotes. Example: ‘Airports/JFK’.
As a Document - Useful for more complex queries.

Filter with Graph Traversals
Suppose, the requirement is to find the flight that departs from San Francisco, reaches Hawaii and is a VIP flight.

	FOR airport in Airports
		FILTER airport.city == “San Fransisco” && airport.vip == true
		RETURN airport

OR One can also use multiple filter statements:

FOR airport in airports
		FILTER airport.city == “San Fransisco”
FILTER airport.vip == true
			FOR v, e, p in 1..1 OUTBOUND
			airport flights
			FILTER v._id == ‘airports/KOA’
			RETURN p

When the results are shown, the flights with the direct connection from San Francisco to Hawaii are shown. If multiple hops are then selected by modifying the min and the max, that would mean that the traversal has to be done from each airport from San Francisco, and then again from those airports to the KOA, which would mean a bigger computation time. 

Therefore, some optimizations would be required to make sure to decrease the computation.


