**Note:** README is under construction to be updated wrt the current version of NERTools. 

# NERTools
This repo consists of scripts to test existing named-entity recognition tools (such as StanfordNER, Spacy, ...) 
in order to determine baseline models for EMW project in Koc University, Istanbul.

## Tool 1: StanfordNER

StanfordNER is a widely used extensive NLP library. This script tests a pretrained Stanford NER classifier to several news datasets,
namely, CONLL2003; Folia and ACE.

The script simply takes a **pretrained model** and an **input text** as input, gives a **scores** file as output. 

### Requirements

- Python3.x.x (my version is 3.6.3), 
- [Stanford's pretrained classifiers 3.9.1](https://nlp.stanford.edu/software/CRF-NER.html) [1], 
- [NLTK's Stanford NER library 3.2.4](https://www.nltk.org/_modules/nltk/tag/stanford.html) [2].

### Usage
The script allows two types of configurations: **default** and **custom** mode.

To run on default mode, type:

`python tagger.py`

To run on custom mode, you should specify the parameters in the order below:

`python tagger.py <ner-tool> <annotationformat> <test-file> <out-file>`

An example configuration (Paths are valid from within the project main folder "NERTools")

`python tagger.py stanford conll './conll-dataset/test-a.txt' './stanford-out-files/out-a.txt'`

If you choose Stanford as NER Tool, the program will want you to specify model and tagger paths.

As an example:

`'./stanford-ner-files/stanford-ner.jar' './stanford-ner-files/ner-model-english-conll-4class.ser.gz'`

### Parameters
- ner-tool: name of the nlp tool you want to use (stanford, spacy, etc.)

- annotation-format: annotation format of the input file (conll, folia, etc.)

- test-file: Test annotated data file (CONLL2003, ACE, your own annotated files, ...)

- out-file: Program output in which the scores of the pretrained model are reported.

- tagger-file: the path to the Stanford's tagger file you downloaded earlier on your local.

- model-file: the path to a pretrained model Stanford provided which you downloaded earlier on your local.


### Results

Please see [the Google Docs document](https://docs.google.com/document/d/1wKh2Hzld9ull8IR_dRrcGP6N4TBeJKMxeJllDPkvwGY/edit?usp=sharing) for the results.

### Code

Mostly benefitted from a [blog post](https://blog.sicara.com/train-ner-model-with-nltk-stanford-tagger-english-french-german-6d90573a9486) [4].

## Tool 2: NeuroNER

This package uses Spacy as the default tokenizer (a tokenizer is used only if input is of BRAT format). It contains a pretrained model, Conll data and a Glove word vector model.

### Requirements
- python, tensorflow, pycorenlp
  - My versions: python 3.6.3, tensorflow 1.8.0, pycorenlp 0.3.0
  
### Installation

- Follow the instructions on [the original page](https://github.com/Franck-Dernoncourt/NeuroNER#requirements)
  - If you already have tensorflow and python3.x, then you do not the script provided. Directly download and unzip NeuroNER:
  `wget https://github.com/Franck-Dernoncourt/NeuroNER/archive/master.zip
sudo apt-get install -y unzip # This line is for Ubuntu users only
unzip master.zip`
- Download the Glove word embeddings. 
`# Download some word embeddings
mkdir NeuroNER-master/data/word_vectors
cd NeuroNER-master/data/word_vectors
wget http://neuroner.com/data/word_vectors/glove.6B.100d.zip
unzip glove.6B.100d.zip`

### Usage

- Open terminal from inside ./src
- Run command:
`python main.py --train_model=False --use_pretrained_model=True --dataset_text_folder=../data/example_unannotated_texts --pretrained_model_folder=../trained_models/conll_2003_en`

### Possible Problems and Solutions

Bunch of problems you may encounter and the fixes:

- **ModuleNotFoundError: No module named '\_struct'**:
Ensure you are using the right pyhton on your system. For example in my case it was looking for the module '\_struct' under the wrong python (/usr/local/lib/python). Running the command with the right python (Anaconda's) fixed the issue.

- **ModuleNotFoundError: No module named 'pycorenlp'**:
It is missing in the requirements in the documentation but NeuroNER requires pycorenlp. To install:
`pip install pycorenlp`
Make sure you install it under the right python dist. For example in my case I needed to install it under Anaconda. So I had to run the pip of Anaconda which resides in /anaconda/bin.

  - version: pycorenlp-0.3.0

- **OSError: [E050] Can't find model 'en', ..., AttributeError: 'NeuroNER' object has no attribute 'sess'**:

I got this error with the example command in the **Usage** section above. Could't figure out why. Since we are not very interested in predicting the example_annotaated_texts, I changed the command to:

`python main.py --train_model=False --use_pretrained_model=True --dataset_text_folder=../data/conll_2003/en --pretrained_model_folder=../trained_models/conll_2003_en`

But then I got this error:

- **OSError: For prediction mode, either test set and deploy set must exist in the specified dataset folder: ../data/conll_2003/en**

Configure the arguments 'parameters.ini' with the exact values: 

`train_model = False
use_pretrained_model = True
pretrained_model_folder = ../trained_models/conll_2003_en
dataset_text_folder = ../data/conll2003/en`

Then run:

`python main.py`

A rather hacky solution but it worked.

### To run with Folia-annotated files:

#### Create a folder

Create a folder named 'folia' inside './data' of the NeuroNER directory.

#### Convert your Folia files to Conll format:

Open terminal on ./standalone_python_scripts. Run:

`python foliaHelper.py folia2conll foliafile outfile`

where

foliafile: path to a folder containing files OR a single file.
outfile: path to create a single file containing conll formatted version of folia content.

Preferably, set outfile path under the 'folia' folder you have just created. NeuroNER needs to have the outfile under that path.
outfile name needs to be ''

#### Configure './src/parameters.ini':

- train_model = False
- use_pretrained_model = True
- pretrained_model_folder = ../trained_models/conll_2003_en
- dataset_text_folder = ../data/folia

#### Modify train.py:

- Modify line 75.

from:

`assert(token == token_original and gold_label == gold_label_original)`

to:

`assert(token == token_original)`

#### Run NeuroNER

Open terminal on ./src. Run:

`python main.py`

Results are recorded to files named: '000_test.txt' and '000_test.txt_conll_evaluation.txt' in the 'output' folder.

#### Evaluating for raw tag:

- Omit initial letters from conll tags

`python ./standalone_python_scripts/utilFormat.py conll2raw original-outfile edited-outfile`

where:

'original-outfile' is the path to the NeuroNER output file named '000_test.txt'
'edited-outfile' is any path you want for the new file to be created.

- Run conlleval for raw tags:

`python conlleval -r < edited-outfile > resultfile`

where:

'edited-outfile' is the path to the output of the previous step.
'resultfile' is any path you want for the new file to be created.

## Tool 3: Spacy

### Requirements:
- Refer to [spacy's own requirements notes](https://github.com/explosion/spaCy/blob/master/requirements.txt). You do not need to worry about them. They are installed alongside.
### My versions:
- spacy 2.0.11
- python 3 .6.3

### Installation:
- Run:
`pip install spacy`
If you have multiple pips, please use the one under the python distribution you use. For anaconda pip is under `anaconda/bin`.
Spacy should now be under `anaconda/lib/python3.6/site-packages`.

- Install a pretrained model:
Normally `python -m spacy download xx_ent_wiki_sm` should work but for me this worked:

  - Download model tar.gz and unzip.
  
  The model I used is the default model pretrained with Wikipedia data. ([Wikipedia data paper](https://ac.els-cdn.com/S0004370212000276/1-s2.0-S0004370212000276-main.pdf?_tid=a4122aa4-585d-45b5-937d-071f529bb90f&acdnat=1531911108_e342997b556d6a38872a815d6ebaa858))
  
  Model name/ver: **xx-ent-wiki-sm-2.0.0**
  
  - Copy `xx-ent-wiki-sm-2.0.0/xx-ent-wiki-sm/xx-ent-wiki-sm-2.0.0/vocab` into `xx-ent-wiki-sm-2.0.0/xx-ent-wiki-sm/xx-ent-wiki-sm-2.0.0/ner`
  - Put `xx-ent-wiki-sm-2.0.0/xx-ent-wiki-sm` under `spacy/data`. **Note that you put there only the second level `xx-ent-wiki-sm` folder.**
  
 ### Usage:
 - Please refer to the [ipython notebook](https://github.com/emerging-welfare/NERTools/blob/master/spacyner.ipynb) 
 
 OR 
 
 - Run spacy under NERTools with the command:
 
 `python tagger spacy conll conll-testa.txt conll-testa-out.txt`

Then the program will want you to specify model and tagger paths.

As an example:

`'xx_ent_wiki_smr' '/home/berfu/anaconda/lib/python3.6/site-packages/spacy/data/xx_ent_wiki_sm/xx_ent_wiki_sm-2.0.0/ner'`

_Note that to use these paths you will need to install and do necessary modifications on spacy model folder first. Please refer to the **Installation** section._
 
 
## Results:

Please see [the Google Docs document](https://docs.google.com/document/d/1wKh2Hzld9ull8IR_dRrcGP6N4TBeJKMxeJllDPkvwGY/edit?usp=sharing) for the results.

IMPORTANT NOTE: NeuroNER source code is modified (one line) to be able to predict with Folia documents. Details in the google docs.

## Notes

The test data and stanford pretrained models used as default are available in the project.

## References

[1] [Stanford NER website](https://nlp.stanford.edu/software/CRF-NER.html)

[2] [NLTK’s Stanford NER Library](https://www.nltk.org/_modules/nltk/tag/stanford.html)

[3] Erik F. Tjong Kim Sang and Fien De Meulder. 2003. Introduction to the CoNLL-2003 shared task: Language-independent named entity recognition. In CoNLL-2003. (link)

[4] [Code blog](https://blog.sicara.com/train-ner-model-with-nltk-stanford-tagger-english-french-german-6d90573a9486)

[5] [NeuroNER Github Repository](https://github.com/Franck-Dernoncourt/NeuroNER)

