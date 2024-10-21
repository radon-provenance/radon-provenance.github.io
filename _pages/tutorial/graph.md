---
title: Radon Data Management System
permalink: /tutorial/graph.html
layout: default
sidebar_menu: Tutorial
sidebar_sub: Working with Graph
---

# Working with graphs

Radon graph capabilities are managed by {% include tooltip.html name="dse" %}. The
easiest way to experiment with graphs is to use Datastax Studio which is an
interactive developer tool for {% include tooltip.html name="cql" %}, Spark SQL
and DSE Graph.

Graphs can be manipulated using the [Gremlin graph traversal language](https://tinkerpop.apache.org/).
Studio is a tool accessible in a web browser, it provides a way to create 
notebooks consisting of different cells. Each cell contains Gremlin queries which
can be executed on the Radon server. Additionally, user can use Python scripts
to send the same queries to the server, this won't be covered by this tutorial
as this part is still under development.


# Tutorial


## Creating a simple notebook in DataStax Studio

1. Open Studio by entering ```htpp://Radon_URL:9091``` in your web browser

![Studio](/assets/images/main_studio.png){:width="100%"}


2. Click the plus (+) to add a notebook. 

The **Create Notebook**` dialog displays.

3. Enter the notebook name

4. Select an existing connection or click **+ Add New Connection**

For a localhost connection with dse running as a docker image the Host/IP field
should be set to _**dse**_.

5. Select an existing graph or create a new one.


## Executing Gremlin queries


1. In the default created cell, change the drop-down language to **Gremlin**.

2. Type a Gremlin query in the cell (for instance `system.graphs()`)

![Cell Query](/assets/images/cell_query.png){:width="100%"}

3. Select **Run Cell** to execute the query (Shift+Enter).


## Create a graph schema

1. Create the schema part for the vertices:

```
schema.vertexLabel('log').
  ifNotExists().
  partitionBy('log_id', Text).
  property('description', Text).
  create()

schema.vertexLabel('obj').
  ifNotExists().
  partitionBy('obj_id', Text).
  create()
```

2. Create the schema part for the edges between vertices:

```
schema.edgeLabel('child').
  ifNotExists().
  from('log').to('obj').
  create()

schema.edgeLabel('next').
  ifNotExists().
  from('log').to('log').
  create()
```

![Schema Query](/assets/images/schema_query.png){:width="100%"}

3. Once created, the schema can be visualised from the top right menu **_ Schema _**

![Schema Visualisation](/assets/images/schema_visu.png){:style="display:block; margin-left:auto; margin-right:auto"}


## Create Vertices and Edges

1. Create some vertices

```
g.addV('obj').
  property('obj_id', 'obj_001').
  iterate();
g.addV('obj').
   property('obj_id', 'obj_002').
  iterate();
g.addV('obj').
   property('obj_id', 'obj_003').
  iterate();

g.addV('log').
   property('log_id', 'log_001').
   property('description', 'operation 1').
  iterate();
g.addV('log').
   property('log_id', 'log_002').
   property('description', 'operation 2').
  iterate();
g.addV('log').
   property('log_id', 'log_003').
   property('description', 'operation 3').
  iterate();
g.addV('log').
   property('log_id', 'log_004').
   property('description', 'operation 4').
  iterate();
```


2. Create some edges

```
g.V('dseg:/log/log_001').as('a').
  V('dseg:/obj/obj_001').as('b').
  addE('child').from('a').to('b').
  iterate();
g.V('dseg:/log/log_002').as('a').
  V('dseg:/obj/obj_001').as('b').
  addE('child').from('a').to('b').
  iterate();
g.V('dseg:/log/log_003').as('a').
  V('dseg:/obj/obj_002').as('b').
  addE('child').from('a').to('b').
  iterate();
g.V('dseg:/log/log_004').as('a').
  V('dseg:/obj/obj_003').as('b').
  addE('child').from('a').to('b').
  iterate();
g.V('dseg:/log/log_001').as('a').
  V('dseg:/log/log_002').as('b').
  addE('next').from('a').to('b').
  iterate();
g.V('dseg:/log/log_002').as('a').
  V('dseg:/log/log_003').as('b').
  addE('next').from('a').to('b').
  iterate();
g.V('dseg:/log/log_003').as('a').
  V('dseg:/log/log_004').as('b').
  addE('next').from('a').to('b').
  iterate();
```

![Schema Elements](/assets/images/schema_elements.png){:width="100%"}


## Visualise Graph

1. Visualise the full graph

```
g.V()
```

2. Select the tool _**View results as graph**_


![Graph Visualisation](/assets/images/graph_visu.png){:width="100%"}

3. Modify the graph settings with the top right icon. **_Style_** can be used to 
define the aspect of the vertices and the edges. Displayed names can also be 
modified to make the graph more readable.


![Graph Configuration](/assets/images/graph_config.png){:width="100%"}


## Experiment with Gremlin queries


1. Select a specific vertex

```
g.with("label-warning", false).with('allow-filtering').V().has("log_id", "log_002").bothE()
```

2. Select a vertex and navigate through edges

```
g.with("label-warning", false).with('allow-filtering').V().has("obj_id", "obj_001").in("child").in("next").bothE()
```


