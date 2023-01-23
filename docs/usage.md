# Usage of the reference implementation

## TL;DR

The fastest procedure to execute the implementation is to install all the
required packages and then execute the end-to-end processing script. Asuming
we have an active Python virtualenv (3.8 or later), it would be:

```
pip install wheel
pip install pii-preprocess pii-extract-plg-regex pii-extract-plg-presidio pii-transform
python -m spacy download en_core_web_lg
```

and then execute

```
pii-process <input-document> <output-document.yml> --lang en --default-policy label
```

... where:

 * `<input-document>` is a text, Word o CSV file (the ones currently 
   supported by pii-preprocess), or a YAML dump of an already-parsed document
 * `<output-document.yml>` is a YAML representation of the document with
   all found PII entities changed to a label that indicates the type of PII
 * `--lang en` indicates the language to use (using the ISO 639-1 two-letter
   code). This is required because some PII Detectors are customized per
   language (but if the document metadata already contains a language tag, then
   it will be used from there, and this command-line option is not needed).
 * `label` is the name of the policy to apply to modify the PII occurrences;
   current choices are `passthrough`, `redact`, `hash`, `label`,
   `placeholder` or `annotate`.  Future versions might define additional
   policies
   
Additionally:

 * Output document can also be a JSON or text file (just change the file
   extension), or a _compressed_ file (e.g. use a `name.yml.gz` filename).
 * The argument `--save-pii <output>` will save in a JSON file the extracted
   PII entities, as a collection.
 * To get a list of the currently installed capabilities in terms of PII
   detection tasks, execute `pii-task-info list-tasks`
 * To get a list of all languages for which there is at least one available
   detector task, execute `pii-task-info list-languages`


## Full process

The whole workflow is structured around a set of [Python libraries], which
coordinate to perform the whole process. Here we comment some of the stages.

### Detect

* The minimum package installation requirement for PII detection is 
  `pii-extract-base` (which will also install `pii-data`). 
* However this package does not contain any detectors. Installing a plugin
  will include detectors. Two plugins are available:
    - `pii-extract-plg-regex` will add [a plugin that includes some
	  regex-based detectors] for PII instances in several languages/countries.
    - `pii-extract-plg-presidio` will add [a plugin that uses Microsoft
	  Presidio] to perform PII instance dectection. Note that Presidio needs
	  an NLP engine for its model-based recognizers (the default is to use
	  spaCy)

The base detection package installs a `pii-detect` command-line script. The
script can *only* process documents in serialized SourceDocument format (a
YAML o JSON format containing the document split in chunks). It will output
a PiiCollection: a JSON file containing all PII instances detected.

The package also installs a `pii-task-info` script that can be used to query
the currently installed capabilities, in terms of locally available plugins,
languages and tasks.


### Preprocess

In order to process other types of documents, install the `pii-preprocess`
package. This will add a `pii-preprocess` command-line script that can read
documents in other formats, and convert them to YAML Source Documents, hence
allowing its processing by `pii-detect`.

The current supported formats are: plain text files (with different options on
how to split the document in chunks), Microsoft Word files and CSV
files. Future packages, or plugins, will add more formats.


### Transform

The `pii-transform` package can read a PiiCollection and use it to _modify_
a SourceDocument, replacing PII occurrences with a different string,
according to a set of possible substitution policies.

Thos package provides also a `pii-process` wrapper command-line script that
works as a combined processing pipeline, including preprocessing, detection and
PII transformation in a single execution. This is the one shown in the above
end-to-end section.


## Programmatic API

In addition to command-line operation, the packages also provide a Python API
that can be used to integrate processing into other workflows. Some examples
are:

 * the `pii-preprocess` package contains a [DocumentLoader] class to read
   files and convert them to [Source Documents]
 * the `pii-extract-base` package contains a [Python API for PII Detection],
   at various level of detail.
 * the `pii-transform` package contains (check its [api document])
     - an API for PII transformation
     - wrapper APIs for end-to-end processing


[a plugin that includes some regex-based detectors]: https://github.com/piisa/pii-extract-plg-regex
[a plugin that uses Microsoft Presidio]: https://github.com/piisa/pii-extract-plg-presidio
[Python libraries]: libraries.md
[DocumentLoader]: https://github.com/piisa/pii-preprocess/tree/main/doc/loader.md
[Source Documents]: libraries.md#source-document
[Python API for PII Detection]: https://github.com/piisa/pii-extract-base/tree/main/doc/usage.md
[api document]: https://github.com/piisa/pii-transform/tree/main/doc/api.md
