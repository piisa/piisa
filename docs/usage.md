## 1. TL;DR

The fastest procedure to execute the implementation is to install all the
required packages and then execute the end-to-end processing script. We will
need
  * An active Python virtualenv (3.8 or later)
  * [PyTorch installed] in that virtualenv (either for CPU or GPU, depending
    on hardware availability)

Then setting up the process for English PII processing would be:

```
pip install wheel
pip install pii-process[transformers]
```

and then we execute:

```
pii-process-doc <input-document> <output-document.yml> --lang en --default-policy label
```

... where:

 * `<input-document>` is a text, Word o CSV file (the formats currently supported by
   the `pii-preprocess` package), or a YAML dump of an already-parsed document.
 * `<output-document.yml>` is a YAML representation of the document with
   all found PII entities changed to a label that indicates the type of PII
 * `--lang en` indicates the language to use (using the ISO 639-1 two-letter
   code). This is required because some PII Detectors are customized per
   language (but if the document metadata already contains a language tag, then
   it will be used from there, and this command-line option is not needed).
 * `label` is the name of the policy to apply to modify the PII occurrences;
   current choices are `passthrough`, `redact`, `hash`, `label`,
   `placeholder`, `synthetic` or `annotate`.  Future versions might define
   additional policies
   
Additionally:

 * An alternative script that can process JSONL multi-documents is
   [pii-process-jsonl], see below.
 * Output document can also be a JSON or text file (just change the file
   extension), or an equivalent _compressed_ file (e.g. use a `name.yml.gz`
   filename). If the input document is a table (a CSV file), the output can
   also be a CSV file.
 * The argument `--save-pii <output>` will save in a JSON file the extracted
   PII entities, as a collection.
 * To get a list of the currently installed capabilities in terms of PII
   detection tasks, execute `pii-task-info list-tasks`
 * To get a list of all languages for which there is at least one available
   detector task, execute `pii-task-info list-languages`
 * For additional languages, there may be models available to detect some of the
   PII Entities. These models would need to be installed. Check the
   [Transformers plugin] docs for installation instructions.
 * In addition to the Transformers-based plugin, there is also another
   available plugin for model-based PII detection: [Presidio plugin], which
   uses Microsoft Presidio for detection. It can be used as an alternative, or
   in combination; check the [pii-process] package documentation for installation
   instructions.


### Multi-language processing for JSONL files

There is a variant, provided by the [pii-process-jsonl] script. This one
assumes that the format is in JSONL format (a series of lines, each one
containing a full JSON document), and that each document may be in a different
language. Provided the languages are supported by the packages, it can
generate an output JSONL file with the desired transformations on the PII
instances detected.


## 2. Full process

The whole workflow is structured around a set of [Python libraries], which
coordinate to perform the whole process. Here we comment briefly these processing
stages.


### 2.1 Preprocess

In order to process documents in different formats than YAML or JSON, we need the
`pii-preprocess` package. This will add a `pii-preprocess` command-line
script that can read documents in some other formats and convert them to YAML
Source Documents, hence allowing its processing by `pii-detect`.

The current supported formats are: plain text files (with different options on
how to split the document in chunks), Microsoft Word files and CSV
files. Future versions, or plugins, will add more formats.


### 2.2 Detect

* The minimum package installation requirement for PII detection is 
  [`pii-extract-base`] (which will also install `pii-data`). 
* However this package does not contain any detectors. Installing a plugin
  will include detectors. Three plugins are available:
    - `pii-extract-plg-regex` will add [a plugin that includes some
	  regex-based detectors] for PII instances in several languages/countries.
    - `pii-extract-plg-transformers` will add a [Transformers plugin], which
	  uses models built with the [Hugging Face Transformers] library to perform
	  PII instance dectection.
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


### 2.3 Decide

The [`pii-decide`] takes a PiiCollection and consolidates its contents, deciding
which PII instances to keep and which ones to discard.

Right now is a very simple package that only takes care of resolving PII
instance _overlaps_ (by choosing the longest instance). Future versions will
add improved capabilities.


### 2.4 Transform

The [`pii-transform`] package can read a PiiCollection and use it to _modify_
a SourceDocument, replacing PII occurrences with a different string, according
to a set of possible substitution policies.


### 2.5 Process wrapper

The [`pii-process`] package is a wrapper that provides both an API and
comand-line scripts to carry out the full end-to-end process, calling the APIs
of the other four packages as needed.

It provides two wrapper command-line scripts (as shown in the above end-to-end
section):

* `pii-process-doc` works as a combined processing pipeline, including
  preprocessing, detection and PII transformation of a document in a single
  execution.
* [pii-process-jsonl] does the same, but for JSONL files


## 3. Programmatic API

In addition to command-line operation, the packages also provide a Python API
that can be used to integrate processing into other workflows. Some examples
are:

 * the `pii-preprocess` package contains a [DocumentLoader] class to read
   files and convert them to [Source Documents]
 * the `pii-extract-base` package contains a [Python API for PII Detection],
   at various levels of detail.
 * the `pii-transform` package contains an API for PII transformation
 * the `pii-process` package contains wrapper APIs for end-to-end processing,
   for both single- and multi-language processing (check its [api document])


[a plugin that includes some regex-based detectors]: https://github.com/piisa/pii-extract-plg-regex
[a plugin that uses Microsoft Presidio]: https://github.com/piisa/pii-extract-plg-presidio
[Presidio plugin]: https://github.com/piisa/pii-extract-plg-presidio
[Transformers plugin]: https://github.com/piisa/pii-extract-plg-transformers
[Hugging Face Transformers]: https://huggingface.co/docs/transformers/main/en/index
[Python libraries]: libraries.md
[DocumentLoader]: https://github.com/piisa/pii-preprocess/tree/main/doc/loader.md
[Source Documents]: libraries.md#source-document
[Python API for PII Detection]: https://github.com/piisa/pii-extract-base/tree/main/doc/usage.md
[pii-process-jsonl]: https://github.com/piisa/pii-transform/tree/main/doc/jsonl.md
[api document]: https://github.com/piisa/pii-process/tree/main/doc/api.md
[`pii-extract-base`]: https://github.com/piisa/pii-extract-base
[`pii-decide`]: https://github.com/piisa/pii-decide
[`pii-transform`]: https://github.com/piisa/pii-transform
[`pii-process`]: https://github.com/piisa/pii-process

[PyTorch installed]: https://pytorch.org/get-started/locally/
