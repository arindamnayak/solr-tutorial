## Indexing concepts

Solr builds on another open source search technology: Lucene, a Java library that provides indexing and search technology, as well as spellchecking, hit highlighting and advanced analysis/tokenization capabilities. Both Solr and Lucene are managed by the Apache Software Foundation.

The fundamental premise of Solr is simple. You give it a lot of information, then later you can ask it questions and find the piece of information you want. The part where you feed in all the information is called indexing or updating. When you ask a question, it’s called a query.

### Analyzers

It analyzes text of fields and generated token stream. Analyzers are specified as a child of the `<fieldType>` element in the schema.xml configuration file (in the same conf/ directory as solrconfig.xml).

E.g. WhitespaceAnalyzer is responsible for analyzing the content of the named text field and emitting the corresponding tokens. There are different type of analyzers available for solr, listed [here](http://www.solr-start.com/info/analyzers/).

### Tokenizers

The job of a tokenizer is to break up a stream of text into tokens, where each token is (usually) a sub-sequence of the characters in the text. There are several type of tokenizers in solr, listed [here](https://lucene.apache.org/solr/guide/6_6/tokenizers.html#tokenizers). 

E.g. StandardTokenizer splits the text field to tokens treating whitespace and punctuation as delimiters. Delimiter characters are discarded, with the following exceptions:

- Periods (dots) that are not followed by whitespace are kept as part of the token, including Internet domain names.
- The "@" character is among the set of token-splitting punctuation, so email addresses are not preserved as single tokens.

E.g 
In: "Please, email john.doe@foo.com by 03-09, re: m37-xq."
Out: "Please", "email", "john.doe", "foo.com", "by", "03", "09", "re", "m37", "xq"

### Filters

Unlike tokenizers, a filter’s input is another TokenStream. The job of a filter is usually easier than that of a tokenizer since in most cases a filter looks at each token in the stream sequentially and decides whether to pass it along, replace it or discard it.

Because filters consume one TokenStream and produce a new TokenStream, they can be chained one after another indefinitely. Each filter in the chain in turn processes the tokens produced by its predecessor. The order in which you specify the filters is therefore significant. Typically, the most general filtering is done first, and later filtering stages are more specialized.

E.g. StandardFilterFactory removes dots from acronyms, and performs a few other common operations. All the tokens are then set to lowercase, which will facilitate case-insensitive matching at query time.

### Solr Document , Field

Solr’s basic unit of information is a document, which is a set of data that describes something. A recipe document would contain the ingredients, the instructions, the preparation time, the cooking time, the tools needed, and so on. A document about a person, for example, might contain the person’s name, biography, favorite color, and shoe size. A document about a book could contain the title, author, year of publication, number of pages, and so on.

In the Solr universe, documents are composed of fields, which are more specific pieces of information. Shoe size could be a field. First name and last name could be fields.

Fields can contain different kinds of data. A name field, for example, is text (character data). A shoe size field might be a floating point number so that it could contain values like 6 and 9.5. Obviously, the definition of fields is flexible (you could define a shoe size field as a text field rather than a floating point number, for example), but if you define your fields correctly, Solr will be able to interpret them correctly and your users will get better results when they perform a query.

### Schema design

Solr stores details about the field types and fields it is expected to understand in a schema file. The name and location of this file may vary depending on how you initially configured Solr or if you modified it later.

- `managed-schema` is the name for the schema file Solr uses by default to support making Schema changes at runtime via the Schema API, or Schemaless Mode features. You may explicitly configure the managed schema features to use an alternative filename if you choose, but the contents of the files are still updated automatically by Solr.
- `schema.xml` is the traditional name for a schema file which can be edited manually by users who use the ClassicIndexSchemaFactory.
- If you are using SolrCloud you may not be able to find any file by these names on the local filesystem. You will only be able to see the schema through the Schema API (if enabled) or through the Solr Admin UI’s Cloud Screens.


Solr stored document in inverted index format which helps to look up the query terms in fast and efficient manner. Solr uses vector space model to calculate relevancy/score of a document. 

### Inverted index 

It is designed to allow very fast full-text searches. An inverted index consists of a list of all the unique words that appear in any document, and for each word, a list of the documents in which it appears.

E.g. Say we have 2 documents.

Doc1: "This explains how cars work."
Doc2: "Cars can carry multiple persons."

Terms          Presence
This           doc1
explains       doc1
how            doc1
cars           doc1, doc2 
work           doc1
can            doc2
carry          doc2
multiple       doc2
persons        doc2

Advantage of Inverted Index are:

- Inverted index is to allow fast full text searches, at a cost of increased processing when a document is added to the database.
- It is easy to develop.
- It is the most popular data structure used in document retrieval systems, used on a large scale for example in search engines.

### Vector space model

Vector space model or term vector model is an algebraic model for representing text documents (and any objects, in general) as vectors of identifiers, such as, for example, index terms. It is used in information filtering, information retrieval, indexing and relevancy rankings. 