# Enno Text Annotator
A web-based visual text annotation tool, which hopefully is less painful to use...

Demo video of the first usable prototype (18.08.2017, 64a9ad5)

[![Demo 1](http://img.youtube.com/vi/17WjPtzdwnw/0.jpg)](http://www.youtube.com/watch?v=17WjPtzdwnw "Demo of the Enno Annotator (First prototype!)
")

Another demo: https://youtu.be/tDAIttzuxE8

## Usage

### Installation
```bash
pip install Flask, email
npm install
grunt dist
# or for dev (no minified js)
#grunt dev
grunt flask
```

### Running the example
To get an idea how this thing works, there is an example setup. The data for the example can be found in `data/example_eml` for a showcase on how Enno deals with email files and `data/example_txt` for simple txt files. The configuration file is `config-example.json`.

The two folders in data are referred to as "datasets". Within the config, you can setup multiple datasets ("datasources"). The name you use there has to be set in `enno/static/enno.html`.

Yes, not very elegant, but maybe dataset selection would be a nice pullrequest? Note, that after building with grunt, this is overridden. For permanent change, edit `enno/src/enno.html`.

Replace `<DATASOURCE>` with one of those in your config (the example offers `txt` and `eml` (see section below).

```js
enno.selectSource('<DATASOURCE>').then(function (d) {
    enno.selectSample(); //'train/jones-t_all_documents_10047.txt'
});
```

Start the backend:
```bash
python run.py config-example.json
```

Now go to your favourite browser and open http://localhost:5000/static/enno.html
When something doesn't work, use Chrome (Enno is tested with that...)

### Config File
Please read the previous section first.

Within the config file, you can configure multiple datasets. You can also write a single config for each dataset if you run the server for one thing only.

The structure in the example config is like that:
```
{
  "datasources": {
    "txt": {
      "wrapper": "text",
      "options": {
        "path": "data/example_txt",
        "defaultDenotationType": null,
        "defaultRelationType": null,
        "denotationTypes": [...],
        "relationTypes": {
          ...
      }
    },
    "eml": { 
      "wrapper": "enron",
      ...
    }
}
```
Here you see, that there are two datasets, `txt` and `eml`. Each dataset specifies a wrapper. The wrapper indicates the wrapper to use to handle your input files, in case you want to read from a database or use other data formats. Currently Enno implements plaintext (`text`) and eml-emails (`enron`, sorry, confusing name).

Under `options` you set the path relative to the working directory you start the server in. Ignore `defaultDenotationType` and `defaultRelationType`, that is for future features (maybe). 

Then there are `denotationTypes` (list of str) and `relationTypes` (dict of dicts).
Denotation types is the list of types you want to label selected character sequences with. Relation types are what you can link denotations with.

For example:
```
"Relation1": {
	"type": "symmetric-transitive",
	"from": [
		"Type1",
		"Type2"
	],
	"to": [
		"Type3"
	]
}
```
The relation type `Relation1` is a `symmetric-transitive` link between denotations of type `Type1` or `Type2` and `Type3`. You can also set the type to `directed` for directed links.

### Annotation files
Once you annotate something in the frontend, the existing wrappers automatically saves the file next to the original with the extension `.ann`.
This reflects the internal format of annotations (see `Annotation` in `utils/annotation.py`).
The format is the following:
```
{
  "wrapper": "plaintext",
  "text": "Lorem ipsum dolor sit amet, ...",
  "denotations": [...],
  "relations": [...],
  "meta": {},
  "id": "text_1.txt"
}
```
It is almost self-explanatory. It specifies the wrapper used, the id (in the existing wrappers, that is the relative path to the datasource path), a full copy of the entire text, some meta-data (the email (enron) wrapper stores the email meta-data here, and relations and denotations.

Each denotation is a labelled character sequence with `start` and `end` position, so you can find it again within the `text`, a copy of the text, the denotation `type` (as specified in the config), an `ID` (current wrapper just use an incremental count), and `meta`data (current wrappers don't use it).
```
{
  "id": 1,
  "start": 150,
  "end": 165,
  "text": "Fusce ultricies",
  "type": "Type2",
  "meta": null
},
```

Each relation is a link between two denotations.
It also has an `ID` and `meta`data and a `type` as specified in the config. Also there is an `origin` and `target`, which referrs to the IDs of denotations. 
```
{
  "id": 1,
  "origin": 1,
  "target": 3,
  "type": "Relation3",
  "meta": null
},
```
In case you use symmetric relations, you have to take care of that in your downstream application logic. Enno only checks for that during saving and rejects a user interaction that tries to do something, that is not allowed based on the config.

The meta fields may come in handy for future development, e.g. in a wrapper that stores stuff in databases or elsewhere.

## Future Improvements
You may find, that Enno is pretty bare bones. 
That is by intention! Who wants a huge system that does the same thing?
The goal is to use vanillaJS (yes, I cheated and used jsPlumb). Grunt might be dropped in the future, doesn't do much anyway.
In case you make a PR, please keep in mind not to add any new requirements (or have good reasons for why).

Ideas for improvements:
- Currently Enno is only really usable if you open the JS console in the browser. Would be nice to have small info boxes.
- Currenty Enno doesn't colour denotations by type. That would be something (either by setting colours in the config or calculating hashes from their type names).
- jsPlumb isn't very good. Implement some other way to draw arrows for relationship links.
- New wrappers are always welcome.
- Currently one has to set the dataset within the `enno.html`, which isn't really nice...

## Why that name?
I needed an annotation tool to annotate emails in the [Enron Corpus](https://www.cs.cmu.edu/~./enron/).

## Other tools
- [TextAE](https://github.com/pubannotation/textae), nice, but bloated codebase
- [brat](https://github.com/nlplab/brat), very common, but buggy and frustrating to use
- [Spreadsheet](https://docs.google.com/spreadsheets/d/14ionbRVYBQuD0cNLazKfRWYzrkax3qFCspm9SiaG5Aw/edit) of lots of annotation tools
