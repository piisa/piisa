## 1. Software architecture

Our reference implementation for the [PIISA specification] has been developed
as a set of Python packages:

 * [pii-data] is the foundational library, containing base data structures. It
   defines our abstraction for a document to be analyzed (Source Document) and
   the elements defining a PII instance.
 * [pii-preprocess] is the package in charge of _reading_ different document
   formats into a Source Document, the standard representation defined by 
   [pii-data]; as such it implements the _Preprocess_ block in the PIISA
   architecture
 * [pii-extract-base] is the base package for the PII _Detect_ block. It
   provides the infrastructure needed to extract PII instances from Source
   Documents. However, it does not implement any PII Detector itself, delegating
   that task to external libraries (attached via plugins or configuration files)
 * [pii-extract-plg-regex] is a pii-extract plugin that implements some PII
   Detectors for a number of tasks and languages, based on regular expressions
   (plus optional context validation and/or checksums)
 * [pii-extract-plg-transformers] is a pii-extract plugin that implements some PII
   Detectors by creating NER model pipelines running on the [Hugging Face
   Transformers] library.
 * [pii-extract-plg-presidio] is a pii-extract plugin that implements some PII
   Detectors by calling the [Microsoft Presidio] library.
 * pii-decide (will) implement the _Decision_ block
 * [pii-transform] implements the PII _Transform_ block of the architecture:
   it takes a PII Collection created by pii-extract (and confirmed by 
   pii-decide), and replaces/modifies the PII strings in the original Source
   Document. It also provides some end-to-end API objects and command-line
   scripts.

For a brief initial tutorial for the packages, check out the [usage document].


## 2. Source Document

The abstraction we use to manage original data to be processed is called
[Source Document]. It is a simple representation of a document that contains:

 * a document header, containing some document-level metadata
 * a list of document _chunks_, each one containing a text block extracted
   from the document, plus some additional metadata
   
On top of this representation a few variants have been defined to carry 
some particular document structures: _sequence_, _tree_, _table_

Downstream tools (such as the ones in the [pii-extract-base] or [pii-transform] packages)
can iterate over Source Documents, processing them chunk by chunk (they can also process
single-chunk documents, for the case in which there is no structure to be used).


[PIISA specification]: specs.md
[pii-data]: https://github.com/piisa/pii-data
[pii-preprocess]: https://github.com/piisa/pii-preprocess
[pii-extract-base]: https://github.com/piisa/pii-extract-base
[pii-extract-plg-regex]: https://github.com/piisa/pii-extract-plg-regex
[pii-extract-plg-presidio]: https://github.com/piisa/pii-extract-plg-presidio
[pii-extract-plg-transformers]: https://github.com/piisa/pii-extract-plg-transformers
[pii-transform]: https://github.com/piisa/pii-transform
[pii-decide]: https://github.com/piisa/pii-decide
[source document]: https://github.com/piisa/pii-data/tree/main/doc/srcdocument.md
[usage document]: usage.md
[Microsoft Presidio]: https://microsoft.github.io/presidio/
[Hugging Face Transformers]: https://huggingface.co/docs/transformers/main/en/index
