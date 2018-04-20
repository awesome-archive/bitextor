# Bitextor

## DESCRIPTION

Bitextor is a tool for automatically harvesting bitexts from multilingual websites. The user must provide a URL, a list of URLs in a file (one per line), or the path to a directory containing a crawled website. It is also necessary to specify the two languages on which the user is interested by setting the language IDs following the ISO 639-1. The tool works following a sequence of steps:
  1. Downloads a website by using the tool httrack: see module bitextor-downloadweb (optional step);
  2. The files in the website are analysed, cleaned and standardised: see module bitextor-webdir2ett;
  3. The language of every web page is detected: see module bitextor-ett2lett;
  4. The HTML structure is analysed to create a representation which is used to compare the different web pages: see module bitextor-lett2lettr;
  5. The a preliminary list of document-alignment candidates is obtained by computing bag-of-word-overlapping measures: see module bitextor-idx2ridx;
  6. The candidates are checked by using the HTML structure: see module bitextor-distancefilter;
  7. The documents are aligned: see module bitextor-align-documents;
  8. A set of aligned documents is obtained from the aligned documents: see modules bitextor-align-segments and bitextor-cleantextalign;
  9. The aligned segments are formatted into TMX standard format: see module bitextor-buildTMX (optional step).
It is worth noting that Each of these steps can be run separately.


## REQUIREMENTS

**Autotools** are necessary for building and installing the project. Tools from **JDK** as javac and jar are needed for building Java dependences, and the virtual machine of Java is needed for running them. In addition, a c++ compiler is required for compiling, and **cmake** for 'clustercat' project. Most of the scripts in bitextor are written in Python. Because of this, it is necessary to also install Python 2. All these tools are available in most Unix-based operating systems repositories.

Some external Python libraries should also be installed before starting the installation of bitextor:

- **python-Levenshtein**: Python library for computing the Levenshtein edit-distance.
- **LangID.py**: Python library for plain text language detection.
- **regex**: Python package for regular expressions in Python.
- **NLTK**: Python package with natural language processing utilities.
- **numpy**: Python package for scientific computing with Python.
- **keras**: Python package for implementing neural networks for deep learning.
- **h5py**: Pythonic interface to the HDF5 binary data format.
- **python-magic**: Python interface for the magic library, used to detect files' format (install from apt or source code in https://github.com/threatstack/libmagic/tree/master/python, not from pip: it has a different interface).
- **iso-639**: Python package to convert between language names and ISO-639 codes

The easiest way to install these Python libraries is using the tool pip (https://pypi.python.org/pypi/pip). For installing the three libraries at the same time, you can simply run:

`user@pc:~$ sudo pip install langid python-Levenshtein regex nltk numpy h5py keras tensorflow iso-639`

For system libraries and tools we used apt because we are in a Debian-like environment. In case you have another package manager, just run the equivalent installation with it, but we cannot ensure that the versions and interfaces match the Debian ones, or even exist. In case of any problem, just search how to install automake, gawk, cmake, Java JDK (Oracle or OpenJDK), pip (with get_pip.py) and libmagic with Python interface (https://github.com/threatstack/libmagic/tree/master/) in your distribution or from source code.

Most of these pip packages are also available in the repositories of many Unix-based systems.

In addition to these Python libraries, the tool Apertium (http://www.apertium.org/) may be necessary if you plan to use lemmatisation with bitextor crawl websites containing texts in highly inflective languages. If you do not need this functionally, just use the option "--without-apertium" when running the configuration script.


## INSTALLING BITEXTOR

To install bitextor you will first need to run the script 'configure', which will identify the location of the external tools used. Then the code will be compiled and installed by means of the command 'make':

```
user@pc:~$ ./configure
user@pc:~$ make
user@pc:~$ sudo make install
```

In case you do not have sudoer privileges, it is possible to install the tool locally by specifying a different installation directory when running the script 'configure':

```
user@pc:~$ ./configure --prefix=LOCALDIR
user@pc:~$ make
user@pc:~$ make install
```

where LOCALDIR can be any directory where you have writing permission, such as ~/local. In both examples, HTTRack is a requirement and an error will be prompted to the user if this tool is not installed when running configure. If you do not want to use this tool (and, therefore, you do not plan to use the script bitextor-downloadsite to download websites) you can run configure with the option --without-httrack.

Some more tools are included in the bitextor package and will be installed together with bitextor:
- hunalign: a software for sentence alignment (<http://mokk.bme.hu/resources/hunalign/>)
- mgiza: machine translation package, here used for building probabilistic bilingual dictionaries (<https://github.com/moses-smt/mgiza>)
- clustercat: Fast Word Clustering program, parallelised alternative to mkcls (<https://github.com/jonsafari/clustercat>)
- apache tika: a tool for HTML files normalisation (<http://tika.apache.org/>)
- boilerpipe: a tool for cleaning HTML files to remove useless information such as menus, banners, etc. (<https://code.google.com/p/boilerpipe/>)


## RUNNING BITEXTOR

There are three ways to call bitextor. Two of them include the first step (downloading the websites) and are:
```
bitextor [OPTIONS] -v LEXICON -u  URL LANG1 LANG2
bitextor [OPTIONS] -v LEXICON -U FILE LANG1 LANG2
```
In the first case, bitextor downloads the URL specified. In the second case, the file specified with the option -U should be a tab-separated file containing, in each line, a URL to be crawled and its destination ETT file. In all cases, it is mandatory to specify the lexicon to be used and the target languages to be crawled. One more way to run bitextor is available, using option *-e* to specify an ETT file containing a previously crawled website (this step starts in the second step described in the previous section): 
```
bitextor [OPTIONS] -v LEXICON -e ETT LANG1 LANG2
```
Options -u and -e can be combined to specify the file where the documents downloaded from the URL will be stored for future processing.

Several options can be set using the command line options:

- -U FILE   sets the path to a file containing a list of URLs and their destination ETT file to be crawled and processed (one per line).
- -e FILE   sets the path to the ETT file containing the documents crawled from a website.
- -L DIRECTORY   Target path where the log files of each module will be stored (by default, they are stored in a temporal directory which is removed at the end of the run)
- -I DIRECTORY   Target path where the intermediate files of each module will be stored (by default, they are stored in a temporal directory which is removed at the end of the run)
- -b NUM    when this option is enabled, only the first NUM candidates from the RINDEX candidate list are taken into account when computing the bidirectional document alignment.
- -v FILE   path to the dictionary used by the script bitextor-lettr2idx.
- -m NUM    if the number of wrong alignments in a pair of documents processed by bitextor-align-segments is higher than NUM, the pair of documents is discarded (5 by default).
- -q NUM    if the confidence score for a pair of segments aligned by hunalign in bitextor-align-segments is lower than NUM it is discarded (0 by default).
- -T DIRECTORY     alternative tmp directory (/tmp by default)
- -O FILE     target file containing the result of the crawling (either plain text or a TMX translation memory)
- -x        if this option is enabled, the output of bitextor will be formatted as a standard TMX translation memory (this option adds at the end of the pipeline the script bitextor-buildTMX).
- -a        if this option is enabled, bitextor will print aligned documents instead of aligned segments; the output is tab-separated, with the paths to the two aligned files and a general score provided by hunalign to the document en each line.
- -M      morphological analyser in the Apertium platform for source language that will allow to apply word matching directly on lemmas; this is an important tool for agglutinant languages in order to obtain a good coverage with the bilingual lexicon approach.
- -N      morphological analyser in the Apertium platform for target language that will allow to apply word matching directly on lemmas; this is an important tool for agglutinant languages in order to obtain a good coverage with the bilingual lexicon approach.
- -O FILE      if this option is enabled, the output of bitextor will be redirected to file FILE, if not it is redirected to the standard output.
- -d THRESHOLD     threshold for the parallel-document confidence score. This threshold can take real values in [0,1], being 0 equivalent to not setting any threshold and 1 the highest threshold possible.
- -s SIZE     size limit for the crawling process; if this option is set, the crawling process will stop after the amount of data specified has been crawled. This option must be an amount of Kilobytes (K), Megabytes (M) or Gigabytes (G) for example: '50M' for 50 Megabytes.
- -t TIME     time limit for the crawling process; if this option is set, the crawling process will stop after the amount of time specified. This option must be an amount of time and a time unit (h for hours, m for minutes and s for seconds), for example: '35m' for 35 minutes.

More options using -h command.

It is worth noting that a bilingual lexicon relating the languages of the parallel corpus that will be built is required. Some dictionaries are provided already, but customised dictionaries can easily be built from parallel corpora as explained in the next section.

## BUILDING BILINGUAL DICTIONARIES FROM PARALLEL CORPORA

To create a parallel corpus, it is necessary to have a bilingual dictionary containing translations of words. These dictionaries are formatted as follows:
```
LANGUAGE1_CODE	LANGUAGE2_CODE
word1_in_language1	word1_in_language2
word2_in_language1	word2_in_language2
word3_in_language1	word3_in_language2
...	...
```
For example, a valid dictionary could be:
```
en	es
car	coche
and	y
letter	carta
...	...
```
Some dictionaries are available in https://sourceforge.net/projects/bitextor/files/bitextor/bitextor-4.0/dictionaries/ . However, customised dictionaries can be automatically built from parallel corpora. This package includes the script bitextor-builddics to ease the creation of these dictionaries. The script uses the tool GIZA++ (http://code.google.com/p/giza-pp/) to build probabilistic dictionaries, which are filtered to keep only those pairs of words fitting the following two criteria:
- both words must occur at least 10 times in the corpus; and
- the harmonic mean of translating the word from lang1 to lang2 and from lang2 to lang1 must be equal or higher than 0.2.

To obtain a dictionary, it is only needed to have a parallel corpus in the following format:
 - the corpus must be composed by two files, one containing the segments in lang1, and the other containing the segments in lang2; and
 - the segments appearing in the same line in both files must be parallel.
For a pair of files FILE1 and FILE2 containing a parallel corpus, the script would be used as follows:
```
bitextor-builddics LANG1 LANG2 FILE1 FILE2 OUTPUT
```
with OUTPUT being the path to the file which will contain the resulting dictionary.





