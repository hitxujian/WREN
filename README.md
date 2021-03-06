# WREN

## Table of contents ##

* [Introduction](#introduction)
  * [LODIE_WI](#lodie_wi)
  * [REX](#rex)
* [Resources](#resources)
* [Workflow](#workflow)
  * [Input](#input)
  * [Page pre-processing](#page-pre-processing)
  * [Identifying extractors for each concept attribute](#identifying-extractors-for-each-concept-attribute)

### Introduction ###

This project is a joint effort to combine technologies from [LODIE_WI](#lodie_wi) and [REX](#rex).

##### LODIE_WI
LODIE_WI is the Wrapper Induction component from the [LODIE](http://oak.dcs.shef.ac.uk/lodie/) project.
LODIE_WI provides methods to learn website wrappers.
The assumptions are:
- we have a given Knowledge base K (e.g. DBpedia)
- we want to extract instances of a particular concept C and its attributes (as defined in K)
- there is a fixed domain D (e.g. Book) D ={d<sub>1</sub> … d<sub>n</sub>} where each d<sub>i</sub> is a set of homogeneous *entity-centric* webpages i.e.:
  - each webpage in d<sub>i</sub> belongs to the same website (and share a common template)
  - each webpage in d<sub>i</sub> describes one entity e of type C

The method takes as input a set of homogeneous *entity-centric* webpages d<sub>i</sub> describing entities of type C; for each attribute to extract the method takes as input a gazetteer with possible values for the attribute, obtained from K.
Each gazetteer can be constructed with varying degrees of complexity, from simple SPARQL query, to more complex ones, can be cleaned with outlier detection strategies etc. In this project the gazetteers are assumed given; facilities to generate gazetteers will be released separately.
The method generates:
- a set of xpath extractors (the cardinality of the set can be 0, 1 or multiple)
- results of the extraction performed on d<sub>i</sub> applying the xpath extractors.

A very short presentation can be found [here](http://www.slideshare.net/AnnaGentile/mining-entities-from-the-web) 

Relevant papers:
- **AI Magazine 2015**. Anna Lisa Gentile, Ziqi Zhang and Fabio Ciravegna (2015). [Early Steps Towards Web Scale Information Extraction with LODIE](http://www.aaai.org/ojs/index.php/aimagazine/article/view/2567). AI Magazine, 36(1), 55--64.

- **KCAP 2013**. Anna Lisa Gentile, Ziqi Zhang, Isabelle Augenstein and Fabio Ciravegna (2013). [Unsupervised wrapper induction using linked data](http://dl.acm.org/citation.cfm?doid=2479832.2479845). Proceedings of the seventh international conference on Knowledge capture, 41--48. Banff, Canada: ACM 

- **TSD 2014**. Anna Lisa Gentile, Ziqi Zhang and Fabio Ciravegna (2014). [Self Training Wrapper Induction with Linked Data](http://link.springer.com/chapter/10.1007%2F978-3-319-10816-2_35). Text, Speech and Dialogue - 17th International Conference, {TSD} 2014, Brno, Czech Republic, September 8-12, 2014. Proceedings, 285--292. [Paper PREPRINT](http://www.tsdconference.org/tsd2014/download/preprints/681.pdf)

- **ISWC 2014**. Anna Lisa Gentile and Suvodeep Mazumdar (2014). [User driven Information Extraction with LODIE](http://ceur-ws.org/Vol-1272/paper_112.pdf). Proceedings of the ISWC 2014 Posters & Demonstrations Track a track within the 13th International Semantic Web Conference (ISWC 2014), 385-388.

You can also view a less than two minutes [demo video](http://staffwww.dcs.shef.ac.uk/people/A.L.Gentile/demo/iswc2014.html).

##### REX


### Resources ###
The folder [resources](./resources) contains:
- [gazetteers](./resources/gazetteers) that are used to seed the annotation phase. These gazetteers have been automatically generated, but are given as static resource here for reproducibility. Relevant gazetteers are provided for all domain-attributes tackled in the the evaluation [datasets](./resources/datasets)
- evaluation [datasets](./resources/datasets) with the relative groundtruth
- the [temp](./temp) folder is the default location where the method creates intermediate representations of pages.

The folder [experimentResults](./experimentResults) is the default location where the method saves experimental results.

### Workflow ###

##### Input
Input is provided as follows:
- a folder D which represents the domain and contains subfolders d<sub>i</sub> containing a set of homogeneous *entity-centric* webpages; each webpage is a single html file
- *for test purposes* we provide example files for the book [book](./resources/datasets/swde-17477/testset/book) domain;
the subfolders [book-booksamillion-2000](./resources/datasets/swde-17477/testset/book/book-booksamillion-2000) and [book-christianbook-2000](./resources/datasets/swde-17477/testset/book/book-christianbook-2000) contain 2000 pages each, describing books, respectively from http://www.booksamillion.com/ and http://www.christianbook.com/ websites.

##### Page pre-processing

The original webpages are transformed in an internal xpath-value representation where:
- we extract all text nodes for each page
- we save each page as the collection of its text nodes, as pairs of *xpath expression to reach the node* - *text content of the node*
- the internal xpath-value representation of pages can be obtained using methods provided in the class [ReducePagesToXpath](./src/uk/ac/shef/dcs/oak/xpath/processors/ReducePagesToXpath.java)
- *for test purposes* the main method in [ReducePagesToXpath](./src/uk/ac/shef/dcs/oak/xpath/processors/ReducePagesToXpath.java) will produce the xpath-value representation of [book](./resources/datasets/swde-17477/testset/book) and save it in the [temp](./temp) folder.

##### Identifying extractors for each concept attribute

###### Candidate patterns for entity attributes
Given:
- a set of homogeneous *entity-centric* webpages d<sub>i</sub> in xpath-value representation
- the attribute p<sub>j</sub> to extract and its relevant gazetter

The method reduces each webpage to a set of xpath-value pairs, which are the candidate xpath extractors as found on the page.
To create such set, the method matches all the values in the page against the provided gazetteers, and retains only the xpath-value pairs where the value is a strict match.

###### Boilerplate removal
Given:
- the set of candidate xpath-value pairs obtained in previous step, this method implements heuristics to remove spourions xpaths.

This method is optionally applied.

###### Pattern ranking
Given:
- the candidate set of xpath-value pairs for all the pages from d<sub>i</sub>

The method produces a ranke list of xpaths which are the extractors for attribute p<sub>j</sub> from pages in d<sub>i</sub>

