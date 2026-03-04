The only Python file currently in this directory is add_embedding_notes.py The inputs to this are an RDF or OWL knowledge graph in Turtle format, another text file with a list of properties and
parameters (--debug or --install) that determine if the function should generate a new turtle file or simply list the strings it would generate. Debug is the default. 
The function iterates over each property in the properties file. These are meant to be properties that contain useful information for a Large Language Model (LLM) that supports a graph RAG system.
An issue with such systems is that they don't have access to the significant information in the knowledge graph. This is a simple but fairly effective way to provide them with such information.
The properties in the text file should be listed one per line with no commas or other delimiters. For each property in that file the function does the following:
1. Find all the triples that match that property. E.g., in this example the first property is: dp:has_bounded_context. 
A Bounded Context is a concept from Domain Driven Design (see the book by Evans and the glossary). The function will find all patterns of the form: ?subject dp:has_bounded_context ?object. 
Then it will find the rdfs:label for the ?subject and ?object. For the property dp:has_bounded_context it will first see if the annotation property dp:embedding_label is defined. If it is the function uses that string.
If not then it falls back to the rdfs:label. This is required because I didn't want to change anything in DCAT but several of the properties in DCAT either don't have labels or the labels don't generate useful strings. 
E.g., the dcat:catalog property relates a catalog to its sub-catalog. An example triple for this property would be: sf:StreamForge_Catalog dcat:catalog sf:StreamForge_Data_Products. If we used the label for dcat:catalog
we would get the string: "StreamForge catalog catalog StreamForge Data Products" which woudn't result in a useful vector. In this example, the dp:embeding_label defined for dcat:catalog
is "has sub-catalog". So the embedding_note generated for this hypothetical triple would be: "StreamForge catalog has sub-catalog StreamForge Data Products".
2. For each property in the property file it finds all matching triples with that property as the predicate and generates text strings that describe that triple in a way the LLM can understand. This is especially useful
in this example for PROV data. E.g., we can now use the LLM to ask "Which files were used to generate Creator Engagement Report Q3 2025 and the LLM will be able to use the 
embedding_note strings: "Creator Engagement Report Q3 2025 uses data product: SF Video Metadata Product" and "Creator Engagement Report Q3 2025 uses data product: SF Creator Insights Product".

The turtle file: https://github.com/mdebellis/data_catalog_knowledge_graph/blob/main/data_product_catalog.ttl is a knowledge graph data catalog and has significant PROV sample data for the made-up company StreamForge. 
I'm currently working on generating embeddings and creating example questions and answers using both the contents of the documents themselves and the generated strings that describ PROV and other triples 
that describe Data Provenance and other issues in the Data Catalog. 

This is very tentative. Limitations: 
1) I assume that each property has either a dp:embedding_label or an rdfs:label value. I also assume that all strings are langStrings.
2) The function is hard coded to generate an output file called: output_with_embedding_notes.ttl in the same directory as the function. 
This will be a turtle file with all the info from the input file and generated vector embedding notres.
3) Have done limited testing. Only on the graph currently part of this repository. 
