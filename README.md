<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [piisa](#piisa)
  - [Specification](#specification)
  - [Who are we](#who-are-we)
  - [Contributing](#contributing)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# piisa

PIISA stands for a *Personal Identifiable Information Standard*. This standard
allows seamless interoperability between various PII detection frameworks.


## Rationale

Our mission statement stems from these facts:
 * Proper PII management is hard, and has many facets.
 * There are solutions available for PII processing, both open source and
   commercial
 * We might want to _combine_ several solutions to achieve better results, or
   to adapt to specific use cases
 * However there is no practical way of achieving such combination, or of
   customizing solutions

Our approach has been _let's define an architecture that decomposes the PII
problem into blocks, and let's define interfaces between those blocks_

Therefore the PIISA specification, and its reference implementation, tries to
follow the approach of _independent components that pass data between them to
compose a full solution_


## Specification

[Click here for the latest specification document](docs/specs.md).


## Usage

We are developing a reference software of this specification, delivered as a
set of Python packages that implement each block in the architecture.


## Who are we

We are a team of privacy enthusiasts who are interested in improving PII management across multiple domains. 
- [Read our blog posts](https://privacyprotection.substack.com/).
- Current contributors to this codebase:
  - [@paulovn](https://github.com/paulovn)
  - [@ontocord](https://github.com/ontocord)
  - [@omri374](https://github.com/omri374)
  - [@piskvorky](https://github.com/piskvorky)
  - [@piesauce](https://github.com/piesauce)
  - [@edugp](https://github.com/edugp)
  - [@shamikbose](https://github.com/shamikbose)
  - [@ianyu93](https://github.com/ianyu93)


## Contributing

We are happy to accept contributions from anyone interested in shaping out PIISA. 
To contribute:
-  Make sure you have a [GitHub account](https://github.com/signup/free).
-  Check if a [Github issue](https://github.com/piisa/piisa/issues) already exists. If not, create one.
-  Clearly describe the issue.
-  Fork the repository on GitHub.
-  If your contribution contains code, please make sure you have unit tests added.
-  *Optional but recommended*: Run `flake8` and `black` on your code prior to publishing your pull request.
-  Run all tests locally before publishing your pull request.
-  Push your changes to a topic branch in your fork of the repository.
-  Submit a pull request to the repository.

## License
* The PIISA specification is licensed under a [Creative Commons
Attribution-NoDerivatives 4.0 International License].
* The PIISA reference implementation is licensed under an Apache license

[Creative Commons Attribution-NoDerivatives 4.0 International License]: http://creativecommons.org/licenses/by-nd/4.0/
