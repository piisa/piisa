# Usage of the reference implementation

## TL;DR

The fastest procedure to execute the implementation is to install all the
required packages and then execute the end-to-end processing script. Asuming
we have an active Python virtualenv (3.8 or later), it would be:

```
pip install wheel
pip install pii-preprocess pii-extract-plg-regex pii-transform


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

### Detect

* The minimum package installation requirement is `pii-extract-base` (which
  will also install `pii-data`). 
* However this package does not contain any detectors. Installing
  `pii-extract-plg-regex` will add a plugin that includes some regex-based
  detectors for PII instances in several languages/countries.

This detection package installs a `pii-detect` command-line script. The
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


#### Transform

The `pii-transform` package can read a PiiCollection and use it to _modify_
a SourceDocument, replacing PII occurrences with a different string,
according to a set of possible substitution policies.

It also provides a `pii-process` command-line script that works as a combined
processing pipeline, including preprocessing, detection and PII transformation
in a single execution. This is the one shown in the above end-to-end section.
