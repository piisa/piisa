## 1. PIISA configurations

All steps in the processing chain in a PIISA framework are meant to be
configurable. Such configuration can be provided by three means:

 * command-line tools can provide modifiers as arguments
 * object constructors can also accept some arguments as modifiers
 * configuration files can integrate most of the configuration capabilities

Those configuration files can be written in either YAML or JSON, and have
three sources:

 * packages contain their own local configurations as resource files; those act
   as default configurations
 * package-level configuration files can be supplied at object construction time
 * global configuration files (containing aggregated information for several
   packages) can also be supplied at object construction time


## 2. Configuration formats

The base syntax for those files is either YAML or JSON. The contents of one
configuration file is a dictionary, whose fields depend on the specific
configuration.

There are however, two standardized fields:

 * `format`: this is a *compulsory* field, whose value is a string that
   indicates the type of configuration held by the dictionary (i.e. the
   configuration section; it is typically a package + module identifier).
 * `name`: a string giving a name to this configuration. This is optional;
   if it is not present and the configuration is loaded from a file, the
   framework will automatically use the filename as configuration name.


### 2.1. Package level

A _package-level configuration_ file has the structure of a dictionary. The
`format` field for such a file has the general shape 
`piisa:config:<module>:<section>:v1`, where

 * `piisa:config` is a fixed prefix
 * `<module>` identifies which module this configuration is for
 * `<section>` identifies the section in the module that is to be configured
 * `v1` is a version format string.

   
### 2.2. Full file

A _global configuration file_ contains simply a `config` field with a list
of package configurations, i.e. it is a list of dictionaries, each one with its
`format` key. It carries configuration for all (or many) PIISA modules from
different packages. This makes possible to encapsulate in a single file
a configuration for the whole PIISA toolchain.

A full file contains also a global `format` key, whose value is
`piisa:config:full:v1`

Note that in a full configuration it is possible to have more than one
configuration section with the same `format` tag; they will be combined (later
fields with the same name will override/update previous fields).

The general shape is thus as follows:


```JSON
 {
   "format": "piisa:config:full:v1",
   "config": [
     {
       "format": "piisa:config:<module1>:<name1>:v1",
       ...config for module1/name1
     },
	 {
       "format": "piisa:config:<module1>:<name2>:v1",
       ...config for module1/name2
     },
	 {
       "format": "piisa:config:<module2>:<name>:v1",
       ...config for module2/name
     }
   ]
 }
```



## 3. Default configurations

Some examples of provided default configuration files are:

* The [loader.json] file in the `pii-preprocess` package maps file extensions
  to file types, and for each type defines a loader to read that document type
* A [placeholder.json] file in the `pii-transform` package defines the dummy
  substitution values for the _placeholder_ policy.
* The [pii-extract-plg-transformers] plugin contains a configuration file to
  define the models to load and the mappping of model entities to PIISA entities
* The [pii-extract-plg-presidio] plugin contains a configuration file to map
  Presidio entities to PIISA entities


## 4. Custom configurations

The default files can be replaced at execution time by custom configurations.
Additionally other aspects of the processing flow can be also modified with
additional configurationss:

* A task configuration file can be used to define additional PII detection tasks,
  perhaps coming from custom external code (defining the external tasks by
  specifying their class paths). There is a [small example] available.
* A `plugins.json` file can be defined to define the plugins to load, and
  provide custom arguments to the loader (by default the PIISA system loads all
  the plugins it can detect)


## 5. Dynamic configurations

When using the APIs provided by the PIISA packages, many objects take a
`config` argument in the constructor. This argument can be a single config
element or a list of elements (which will be merged). Each config element, in
turn, can be
 * a path to a configuration file
 * a dictionary containing a live configuration object, created on the fly

This live, dynamic configuration is a dictionary, indexed by PIISA section:
 * The key is the PIISA module package the configuration is for, as a
   `<module>:<name>:v1` string (i.e. the same string as the `format` field
   in configuration files, without the `piisa:config:` prefix)
 * The value contains a standard configuration for that PIISA module

This is an example, in this case containing a custom configuration for the
`pii-transform` package:

```Python

from pii_data.types import PiiEnum
from pii_transform.defs import FMT_CONFIG_TRANSFORM

config = {
    FMT_CONFIG_TRANSFORM: {
        "default": "annotate",
        "policy": {
            PiiEnum.CREDIT_CARD.name: "synthetic",
            PiiEnum.GOV_ID.name: "label"
        }
    }
}
```

If used in an object constructor, a custom config will be merged with the default
config for the module (if it exists), overriding any matching fields and adding
the new ones.


[loader.json]: https://github.com/piisa/pii-preprocess/blob/main/src/pii_preprocess/resources/doc-loader.json
[placeholder.json]: https://github.com/piisa/pii-transform/blob/main/src/pii_transform/resources/placeholder.json
[small example]: https://github.com/piisa/pii-extract-base/blob/main/test/data/tasklist-example.json
[pii-extract-plg-presidio]: https://github.com/piisa/pii-extract-plg-presidio/blob/main/src/pii_extract_plg_presidio/resources/plugin-config.json
[pii-extract-plg-transformers]: https://github.com/piisa/pii-extract-plg-transformers/blob/main/src/pii_extract_plg_transformers/resources/plugin-config.json
