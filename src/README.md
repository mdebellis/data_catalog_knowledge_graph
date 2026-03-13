# How to Use add_embedding_notes_from_properties.py

The problem I'm trying to solve is that when building a graph RAG syste, the LLM does not have access to the information in the knowledge graph. My solution is to generate text strings from the knowledge graph and add those to a new annotation property called embedding_note. I was originally going to try something more sophisticated like using Graph2Vec or using a Python NLP library to generate english strings that capture the meaning of the knowledge graph. Thanks to the encouragement of a colleague who aked a very simple but good question: "Have you tried just basic pattern matching?" I realized I should try that. If nothing else, everything I would need to do with a sophisticated NLP approach, I would need to do with a basic pattern matching approach. Also, I had some practical work where it would have been good to have some kind of capability to do this but I didn't have time to do the more sophisticated approaches. 

To my surprise the simple approach works extremely well. The only additional thing I had to do is create an annotation property called embedding_label. The domain is a data catalog built on DCAT and PROV and many of their properties have simple names that don't translate into useful strings. E.g., dcat:catalog points from a data catalog to a sub-catalog. The string "StreamForge Data Catalog catalog StreamForge Data Products" doesn't say much. In order to add a better string without changing DCAT or PROV, I added that property and I define a string that yields better strings. E.g., for dcat:catalog the embedding_label is "has sub-catalog". So now the the dcat:catalog property example, the generated string is: "StreamForge Data Catalog has sub-catalog StreamForge Data Products"

There are two files: add_embedding_notes_from_properties.py and add_embedding_notes_from_classes.py, add_embedding_notes_from_properties.py takes as input the path to a knowledge graph and another file that has the QNames to use for each property  and parameters (--debug or --install) that determine if the function should generate a new turtle file or simply list the strings it would generate. Debug is the default.  In this example, the ontology file without the embedding notes is: stream_forge_data_catalog_3-12-26_no_embedding_notes.ttl and the properties are in the file: embedding_properties.txt. Each property is on a new line with no commas or any other characters besides a carriage return. For each property in that file the function does the following:
1. Find all the triples that match that property. E.g., in this example for the property dp:has_bounded_context, (note: A Bounded Context is a concept from Domain Driven Design see the book by Evans and the [Semantic Web Glossary](https://github.com/mdebellis/data_catalog_knowledge_graph/wiki/Semantic-Web-Glossary) on this repository's Wiki). The function will find all patterns of the form: ?subject dp:has_bounded_context ?object. 

2. For each property in the property file it finds all matching triples with that property as the predicate and generates text strings that describe that triple in a way the LLM can understand. This is especially useful
in this example for PROV data. E.g., we can now use the LLM to ask "Which files were used to generate Creator Engagement Report Q3 2025 and the LLM will be able to use the 
embedding_note strings: "Creator Engagement Report Q3 2025 uses data product: SF Video Metadata Product" and "Creator Engagement Report Q3 2025 uses data product: SF Creator Insights Product".

The other function is add_embedding_notes_from_classes.py This adds type info such as: "StreamForge data catalog is an instance of Data Catalog" It also adds subsumption info such as: "Data Product is a kind of Data Catalog Asset". Where "kind of" is used to document that one class is a subclass of another. The file embedding_classes.txt defines all the classes to use. Note: for classes that are subclasses of Thing and included in that file, I don't bother adding strings such as "Activity is a kind of Thing". 

This is very tentative. Limitations: 
1) I assume that each property has either a dp:embedding_label or an rdfs:label value. I also assume that all strings are langStrings. If there is no rdfs:label but a skos:altLabel or skos:prefLabel this causes an error now (just temporary, will be refactored).
2) The function is hard coded to generate an output file called: output_with_embedding_notes.ttl in the same directory as the function. 
This will be a turtle file with all the info from the input file and generated vector embedding notres.
3) Right now the package prefixes are hard coded. Near the bottom of the file look for the following:
 ```python
 # Bind prefixes (adjust if needed)
    g.bind("dp", Namespace("https://www.michaeldebellis.com/dp/"))
 ```
You will see several more statements that start with `g.bind(...` Those are all the prefixes I needed to use for the current example. If you use this you need to edit those and add any prefixes you need. If you have a 
QName in the properties file like `ex:my_property` then you need to have a g.bind statement for each. Also, never remove the binding for the rdfs namespace as that is always needed because the function falls back to `rdfs:label`
if it doesn't find a value for dp:embedding_label. Also, you need to keep the binding for the dp namespace as that is used both for the `dp:embedding_label` and `dp:embedding_note` annotation properties.

4) When I started this, I also planned to dig into axioms and use NLTK or Spacy to transform those into strings that further define each class. That would still be a nice to have and I think it depends on the domain but for this domain, there aren't that many axioms and they really aren't that important. Since it is a Data Catalog example, what is important is to trace provenance dependencies defined in PROV and DCAT (and my extensions which are in a namespace called: https://www.michaeldebellis.com/dp/).  Strings like: "Dataset1 uses Dataset2" "Dataset2 was generated from Creator Metadata Data Product". And this works very well for that.
  
5) I use the Python library RDFLib. You will need to load that with pip

6)  Have done limited testing. Only on the graph currently part of this repository. 
