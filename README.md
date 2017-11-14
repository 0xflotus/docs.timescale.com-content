# README #
for version 0.4.0

This is the source for content for docs.timescaledb.com.
The docs site uses this repo as a submodule and converts the files directly into
pages using a bash script and markdown parser.

All files are written in standard markdown.

### A note on anchors

If you want to link to a specific part of the page from the docs sidebar, you
need to place an anchor `<a id="anchor_name"></a>`.  Do not use `name` in place
of `id` or it will disrupt the javascript scrolling method that has been set up
in the docs.

**Your anchor name must be unique** in order for the highlight scrolling to work properly.

### A note on code blocks
When showing commands being entered from a command line, do not include a
character for the prompt.  Do this:

```bash
some_command
```

instead of this:
```bash
$ some_command
```

or this:
```bash
> some_command
```

Otherwise the code highlighter may be disrupted.

### Special rules
There are some custom modifications to the markdown parser to allow for special
formatting within the docs.

+ Adding 'sss ' to the start of every list item in an ordered list will result in
  a switch to "steps" formatting which is used to denote instructional steps, as
  for a tutorial.
+ Adding '>ttt ' to the start of a blockquote (using '>') will create a "tip" callout.
+ Adding '>vvv ' to the start of a blockquote (using '>') will create a "warning" callout.
+ Adding '>toplist ' (no spaces) as the first line of a blockquote (using '>') will create a fixed right-oriented box, useful for a table of contents or list of functions, etc.  See the FAQ page for an example.
    - The first headline in the toplist will act as the title and will be separated from the remainder of the content stylewise (on the FAQ page, it's the headline "Questions").
    - Everything else acts as a normal blockquote does.
+ Adding 'fff ' to the start of a paragraph(line) will format it as a "footer link".
+ Adding 'ddd ' to the start of a link will append a 'download link' icon to the end of the link inline.
+ Adding 'x.y.z' anywhere in the text will be replaced by the version number of the branch.  Ex. `look at file foo-x.y.z` >> `look at file foo-0.4.2`.

_Make sure to include the space after the formatting command!_

**Warning**: Note the single space required in the special formats before adding
normal text. Adding 'ttt' or 'vvv' to the start of any standard paragraph will
result in non-optimal html.  The characters will end up on the outside of the
paragraph tag.  This is due to the way that the markdown parser interprets
blockquotes with the new modifications.
This will be fixed in future versions if it becomes a big issue, but we don't
anticipate that.
