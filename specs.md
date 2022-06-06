# PII data specification v. 0.2.1


## Some rough initial ideas


# 1. Overall architecture

The general structure of a framework dealing with PII management could be visualized as the following diagram:



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


There are up to four processing blocks for such a framework:



1. **Preprocess**: block whose mission is to read a document in an arbitrary format (a Word Document, a Web page, a PDF file, etc) and produce a normalized version, retaining only a simplified version of the high-level structure and all the text data.
2. **Detect**: block in charge of processing input data (usually in text format) and performing detection of candidates to be assigned as PII data. This block uses as input:
    * **source document:** we will consider a normalized data format that conveys the raw text contents, together with some structural information. (which can provide useful hints to the PII Detection modules about the relations between text chunks)
    * **configuration information**: specification of contextual elements affecting detection (e.g. text language, applicable countries, etc)
    * **component information**: the set of available PII Detectors that can be used (assuming we take a modular approach, there might be a database of “pluggable modules” we can use for PII detection). Each Detector will define the type and parameters of PII that can detect.
3. **Decide**: block that takes a number of PII candidates, as produced by the Detection block, and consolidates that information, producing as final result the set of PII elements in the text that need to be addressed. In the process it might combine PII candidates, choose among overlapping PII candidates, reject others, etc. This block uses as input:
    * Candidate list: A list of detected PII candidates
    * Configuration information, as provided by the Decision block (language, countries, etc)
    * An optional purpose/application scenario, to guide the decisions
    * _Context information, as defined in its own configuration. This might include: requirements on PII specificity, sensitivity and scarcity, applicable regulations, etc_
4. **Transform**. This is the block that takes the decided PII entities, and acts upon them, depending on the intended purpose.

    There can be different Transformation blocks, all of them sharing the same interface but providing different outcomes. Some examples are:

* _Anonymization_: modify the text to eliminate decided PII entities. Depending on options they can be replaced by placeholders, dummy values, generated fake PII data, etc
* _Analytics_:  provide the capability to extract and visualize aggregated statistics on decided PII and their associated parameters
* _Visualization_: enable browsing the input data and highlight/examine PII decisions. There can be different types of visualizations (e.g.: redaction, reporting, etc)
* _Evaluation_: provide the capability to assess the performance of detection and/or decision, possibly by using a ground truth evaluation dataset, and estimate precision values.
* _Interpretability_: Provide the capability to interpret the decision process (e.g. why a certain span was decided to be detected as PII and additional metadata on the decision)


# 2. Specification interfaces

The main interfaces to be specified are those that act as boundaries between architecture blocks:



* interface between preprocessing and detection
* interface between detection and decision
* interface between decision and transformation

It might be possible to also define some interfaces internal to one block, so that the block can be decomposed into modular elements (e.g. for pluggable detectors inside the Decision block)


# 3. Specification type

At any given interface, we can envision three types of specification:



1. A **data specification**: syntax & semantics of the data structures that will be sent through one of the interfaces
2. A **program specification**: programmatic interfaces to let components call or be called across the interfaces. This would need to fix an initial default programming language (e.g. Python) to be able to instantiate such programmatic interface; additional languages might be defined later
3. An **API specification**, as a programming language-independent way of interchanging data information across interfaces. This would use a definition such as an [OpenAPI](https://spec.openapis.org/oas/latest.html) specification so that it can be applicable regardless of the programming language; to this aim the _data specification_ would be instantiated into a JSON schema or similar

Note that this is a nested structure: the _Data specification_ is the minimum required element; on top of that we can add the _Program specification_ (or a number of them, for different programming languages) and over it the _API specification_ (or we could also add the _API specification_ directly over the _Data specification_, and leave a programmatic specification undefined)


# 4. Description of possible use cases

_[An enumeration of possible transformation use cases and the requirements they would impose on the architecture blocks]_



* PII redaction prior to ML model training (e.g., prior to training LLMs)
* Pseudonymization of clinical notes for secondary analysis
* Realtime redaction of PII from system logs
* Redact textual PII from images, forms, PDFs etc.
* PII de-identification on semi-structured data, e.g., specific areas in a JSON file, XML, free text columns in tabular data.
* Semi-automated PII removal (human in the loop)
* Classify documents based on whether they contain PII


# 5. Data Specification

The data formats for the Detection block will be:



* as _input_, a source document, either generated directly or via the Preprocessing block
* as _output_, a PII collection


## 5.1 Source document

We need to balance two conflicting requirements



1. an easy format to work with: it needs to be machine-processable but also amenable to human editing and reading
2. an expressive format: able to reflect (at least to some level) the document structure, since that structure might be important to connect PII elements

There are quite sophisticated “Layout” formats for text documents: Word documents (Office Open XML aka OOXML), PDF files, RTF, ODF (Open Document), etc. They are very complex, with specifications compressing many pages, since they allow the complete and precise specification of all aspects of document layout, structure and presentation. They would, of course, offer the greatest nuance in determining the relationships between the text chunks they contain[^1], but would be too difficult to handle for our purpose of consolidating a data interchange format for PII processing that has “reasonable” complexity.

Instead, we are aiming at a simpler solution. As a very minimum, a document can be considered as _a collection of text chunks_. How those chunks are structured is what generates the model to be represented. In general terms, we will consider two main document models:



* a **hierarchical** model: a top-down structure relates text chunks one to another
* a **tabular** model: a 2-D structure (i.e. something that can be expressed as rows and columns, with the implicit assumption of some semantic linking across rows and columns)

There might be mixed documents, which contain both hierarchical and tabular sections, or other structures.


### 5.1.1 Hierarchical source document

In this document format we are trying to preserve two main structural relations between text chunks:



1. an “_is-contained-in_” relation: a text chunk can be considered as semantically contained within another chunk
2. an “_is-next-to_” relation: a text chunk has a relation of being after or before another text chunk

These two relationships can be nested and combined at will. They alone can be enough to describe many of the links that we could need to establish between text chunks (not all of them, but hopefully enough for PII determination).

The specification would be like this:



* A document is considered as a sequence of chunks
* Each chunk contains a small dictionary with 3 or 4 elements:
    1. **id**: an arbitrary string that should be unique per document. Its mission is to make it easier later on to map detected PII instances to the chunk they are part of
    2. **text**: a text section that contains the textual contents of the chunk. It will be a string containing UTF-8 raw text. It can contain newlines or blank lines, to be considered as part of the text structure, but no formatting or layout contents (it is assumed that exact formatting & layout is lost when creating the source document for PII processing – this is a price we pay for simplicity)
    3. **chunks**: (optional) if the current chunk contains subchunks below in the hierarchy, this element contains a sequence of them. This can be nested as needed.

A chunk position in the document hierarchy (its “level”, with 1 being a top-level chunk) could be deduced unambiguously from its location in the nested sequence of chunks.

Note that there is some inherent ambiguity when constructing a document with this model: for the same document, the decision on whether two blocks should be considered “next-to” or “included-in” is not always univocal, and in some cases the content of the text blocks is what gives the semantics away. It is hoped that these variants should not affect the result of any PII Detector processors significatively

This specification would be enough to roughly translate the overall structure of a Word document, a Web Page or a PDF file, assuming that structure can be mapped into this simple hierarchy (some documents are of course more complex than that). Also, a simple raw text document can be easily modeled as a single top-level text chunk.

As a support file format, we could consider YAML files. Those are easy to inspect visually and handle/edit manually, and also can be processed via automatic tools and packages. The YAML specification is actually somehow complex, but we would use only a subset of it: the part strictly needed to support the definitions above:



* sequences, for the sequences of chunks
* mappings, for each chunk
* literal block scalars, to hold the text contents of each chunk

YAML is therefore a good candidate for storage and for manual inspection or editing. For online APIs its equivalent representation as JSON might be more appropriate, though the result would be more involved, specially with the need to serialize the text chunks, including newlines and character escaping

Note: the Google Drive folder contains



* a [small example](https://drive.google.com/file/d/1jjMChdlPfYUx4pXiuhIY_NmtgcqO3zMU/view?usp=sharing) of such a Source Document
* a [small Python script](https://drive.google.com/file/d/1zI0PwSda3JmO6Q314lFT2Oz8V2i3pw5S/view?usp=sharing) developed as a simple exercise to read & write this format


### 5.1.2 Tabular source document

TBD


### 5.1.3 Free-form contextual source document

This is a lower-level document model in which the document is divided into independent chunks, each one with three elements:



* **id**: an arbitrary string that should be unique per document. Its mission is to make it easier later on to map detected PII instances to the chunk they are part of
* **text**: a text section that contains the textual contents of the chunk. It will be a string containing UTF-8 raw text. It can contain newlines or blank lines, to be considered as part of the text structure, but no formatting or layout contents (it is assumed that exact formatting & layout is lost when creating the source document for PII processing – this is a price we pay for simplicity)
* **context**: (optional) additional UTF-8 text that contains unspecified context for this chunk (i.e. it can be the chunks immediately before or after the document, the section title, etc)

The way in which the context is extracted, and therefore its semantics, is not part of the specification; processing modules must only assume that it is related somehow to the current chunk.


## 5.2 PII Collection

A PII collection is the result of running a set of PII detectors on a source document. This result takes the form of a header + a list of detected PII instances.


### 5.2.1 Header

The header contains generic metadata that affects all the PII instances in the collection. Elements of this metadata are:



* `date`: a timestamp on when the process was run
* `format`: a string indicating the format the data is in. E.g. `pii:pii-collection`
* `format_version`: a string used to be able to version PII Collection formats. E.g.: “1.0.0”
* `detectors`: a dictionary describing all detectors (i.e. subsystems or packages) employed to produce the list. Each entry has as key a `detectorId` (an arbitrary string), and as value a dictionary with fields
    * `name`: the name of the package
    * `version`: the package version
    * `source`: a string defining the origin of the package (e.g. a vendor or an organization name)
    * `url`: an address used as reference for the package (e.g. a website or a GitHub repository)
    * `method`: an optional string defining the process used for detection, e.g. `Regex`, `NerModel`, `Regex+Context`, `Checksum`, etc


### 5.2.1 PII instance

A PII instance describes one recognized PII entity. It can be considered as a dictionary containing three types of information:



* PII Description: set of fields characterizing the instance
    * `type`: a string denoting the broad class of PII this instance belongs to. Typically a set of PII types will be predefined so that it can be shared across systems.
    * `subtype`: certain PII classes may have optional subtypes, which help qualify its meaning. For instance, the `GOV_ID` type might have as subtypes “driving license”, “passport number”, etc
    * `value`: the text string from the document containing the PII Instance, as extracted by the detector
    * `lang`: the ISO 639-1 code of the language the document chunk (and possibly the PII instance) is in
    * `country`: the ISO 3166-1 code of the country that is relevant for the PII Instance, if any. E.g. a `CREDIT_CARD` number PII may have an associated country, while a `BITCOIN_ADDRESS` PII has not.
* PII Location: information used to place the PII instance inside the document it belongs to
    * `docid`: the id of the document (optional, used if the PII Collection refers to more than one document)
    * `chunkid`: the id of the document chunk the PII instance belongs to
    * `start`: position of the start of the PII instance inside the document chunk (measured as number of characters from the chunk start)
    * `end`: position corresponding to _one character beyond_ the end of the PII instance inside the document chunk (note that if the PII instance is right at the end of the chunk, this value will point beyond the chunk). The relation _end = start + length(value)_ always holds
* PII Detection: information characterizing the detection process (it can help later in the evaluation by the Decision module)
    * `detectorid`: the identifier for the detector that produced this PII instance, using the key defined in the Collection header
    * `score`: an optional floating point number between 0.0 and 1.0 that gives a measure of the confidence of the Detector on this PII instance. Each detector has its own way of assessing such confidence, so scores are not necessarily comparable across detectors.


### 5.2.2 File format

When formatting a PII Collection for storage or transmission, any format capable of preserving its structure can be used. For the ease of compatibility with most REST-type interfaces, two formats can be proposed:



* **full format**, for storage or local processing: contains the PII Collection as one single JSON object with two subobjects: `metadata` and `piiList`
* **streaming format**: it uses NDJSON (aka JSONL) to send the data as separate, newline-delimited chunks:
    * the first line contains the collection metadata, as a JSON object
    * the rest of the lines contain the list of PII instances, one per line, each one containing a JSON object 

Both formats carry the exact same information; they only differ in its structure

Two examples of the format are:

**Full format**


```
{
  "metadata": {
    "date": "2022-05-18T15:00:01+00",
    "format": "pii:pii-collection",
    "format_version": "0.0.1",
    "detectors": {
      "01": {
	"name": "pii-manager",
	"version": "0.6.0",
	"source": "PIISA",
	"url": "https://github.com/piisa"
      },
      "02": {
	"name": "presidio",
	"version": "1.2.2",
	"source": "Microsoft",
	"url": "https://microsoft.github.io/presidio/"
      }
    }
  }
  "pii_list": [
    {
      "type": "CREDIT_CARD",
      "value": "4273 9666 4581 5642",
      "lang": "en",
      "chunkid": "36",
      "pos": 25,
      "detectorid": "01",
      "score": 1.0
    },
    {
      "type": "GOV_ID",
      "subtype": "SSN",
      "value": "536-90-4399",
      "lang": "en",
      "country": "us",
      "chunkid": "12",
      "pos": 102,
      "detectorid": "02"
    },
    {
      "type": "GOV_ID",
      "subtype": "NIF",
      "value": "34657934-Q",
      "lang": "es",
      "country": "es",
      "chunkid": "1",
      "pos": 10,
      "detectorid": "02"
    }
  }
}
```


**Streaming format**

(note that this example shows additional newlines not present in the file itself)


```
{"date":"2022-05-18T15:00:01+00","format":"pii:pii-collection","formatVersion":"0.0.1","detectors":{"01":{"name":"pii-manager","version":"0.6.0","source": "PIISA", "url":"https://github.com/piisa"},"02":{"name":"presidio","version": "1.2.2","source":"Microsoft","url":"https://microsoft.github.io/presidio/"}}}
{"type":"CREDIT_CARD","value":"4273966645815642","lang":"en","chunkid":"36","pos":25,"detectorid":"01","score":1.0}
{"type":"GOV_ID","subtype":"SSN","value":"536-90-4399","lang":"en","country":"us","chunkid":"12","pos":102",detectorid":"02"}
{"type":"GOV_ID","subtype":"NIF","value":"34657934-Q","lang":"es","country":"es","chunkid":"1","pos":10",detectorid":"02"}

```

## Notes

[^1]:
    
    	Though the document formatting options can be used in many different ways, not all of them with semantic meaning