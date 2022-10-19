# Proposed future enhancements/modifications to the standard

These are ongoing conversations, which may or may not translate into future
revisions of the specification:

* There is the concept of Table layout which is 2D, where the "adjacent"
  paragraphs are actually in 2 dimensions. In hypertext, adjacent documents
  might be a graph. But this type of information is probably in a context of
  the chunk. There is implicity adjacency information from say the sequence of
  iteration itself. Or are we saying that iteration might not be guaranteed to
  be linear in the structural interation case?

  We have 3 abstractions, and some documents will not feet neatly into
  one of them. The tree abstraction would be the closest one as a
  document. But hypertext links are defined (mostly) across documents, and
  this type of context is something we haven't touched on yet,

* As context elements, before and after definitely works. But there could be
  left and right as well. In anycase, linked data would be hard to capture
  here and might just be supported by a user defined context json blob that
  the user can pass along and use as a callback? Say, a json blob could have a

        {'version':0.1, 'type':, 'my content type', 'date': ..., 
	     'content1': ...}

  Future more specific document types might have additional context elements,
  apart from the "simple" before/after chunks. In fact the table document has
  one, the "column" context, which kind of a "hypertexty jump" since it points
  to the column name (in the header row)

* We don't talk about security, but one thing to note is that since we are
  serializing, the data will be stored in some medium. It would be best if it
  was the same medium as the original document (if from a secured database,
  the serializaiton is put back to the secured database). However, if the
  document was purely in memory, but the serialization puts it in disk, there
  is a greater security risk.
  
  Security could be an aspect to be dealt with later, or in a parallel/wrapper
  standard?

* It should be noted that the actual location of the serialization of the YAML
  data doesn't have to be a "file". It could be a database, or any datastore
  for storing the data securely and for later retrieval.
