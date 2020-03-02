Markup
======

Most s-expression "markup languages" aren't really markup languages. Markup should mean the annotation of text content with metadata, not the embedding of text content within data.

#(sexup-html)
(html @(id thing)
  (h1 This is a Document)

  (p This is a paragraph.)

  (p
    Here is more text.

    Here is an inserted $(value).
    Or maybe \ this / is?

    (ul @(attribute $(value))
      (li Here are -)
      (li - some -)
      (li - list items))

    Here is a (a @(href url) link).)
  ((p In this paragraph we can use (brackets).))
  (((p And in this one we can use ((double brackets )).)))
  (p But not here!))

