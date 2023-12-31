# MDict

- MDict is a multi-platform open dictionary
  - the dictionary definitions, i.e. (key word, explanation) are stored in `MDX` file
  - the dictionary reference data, e.g. images, pronunciations, stylesheets in `MDD` file.
  - Although holding different contents, these two file formats share the same structure.
- download dicts from:
  - https://mdx.mdict.org/
- convert mdx to text:
  - https://github.com/csarron/mdict-analysis/tree/master (archived)
  - https://bitbucket.org/xwang/mdict-analysis/src/master/
  - usage: `python readmdict.py -x example.mdx`
