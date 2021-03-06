# fastText rewritten in fsharp
It is a port of original [fastText](https://github.com/facebookresearch/fastText) library to the .NET Framework. 


**This repository is on hold just to keep fsharp version as close to original as possible.**

**Main work moved to [this](https://github.com/hodzanassredin/NFastText) repository.**

# fastText

fastText is a library for efficient learning of word representations and sentence classification.

## Requirements

**fastText** builds everywhere.
It has no external dependencies.
For the word-similarity evaluation script you will need:

* python 2.6 or newer
* numpy & scipy

## Building fastText

In order to build `fastText`, use the following:

```
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
$ echo "deb http://download.mono-project.com/repo/debian wheezy main" | sudo tee /etc/apt/sources.list.d/mono-xamarin.list
$ sudo apt-get update
$ sudo apt-get install mono-devel fsharp
$ sudo cert-sync /etc/ssl/certs/ca-certificates.crt
$ git clone https://github.com/hodzanassredin/fastText.git
$ cd fastText
$ chmod +x build.sh
$ cd .paket
$ wget https://github.com/fsprojects/Paket/releases/download/3.19.6/paket.bootstrapper.exe
$ cd ..
$ ./buid.sh
```

This will produce object files for all the classes as well as the main binary `fasttext`.
If you do not plan on using the default system-wide compiler, update the two macros defined at the beginning of the Makefile (CC and INCLUDES).

## Example use cases

This library has two main use cases: word representation learning and text classification.
These were described in the two papers [1](#enriching-word-vectors-with-subword-information) and [2](#bag-of-tricks-for-efficient-text-classification).

### Word representation learning

In order to learn word vectors, as described in [1](#enriching-word-vectors-with-subword-information), do:

```
$ mono ./build/FastText.exe skipgram -input data.txt -output model
```

where `data.txt` is a training file containing `utf-8` encoded text.
By default the word vectors will take into account character n-grams from 3 to 6 characters.
At the end of optimization the program will save two files: `model.bin` and `model.vec`.
`model.vec` is a text file containing the word vectors, one per line.
`model.bin` is a binary file containing the parameters of the model along with the dictionary and all hyper parameters.
The binary file can be used later to compute word vectors or to restart the optimization.

### Obtaining word vectors for out-of-vocabulary words

The previously trained model can be used to compute word vectors for out-of-vocabulary words.
Provided you have a text file `queries.txt` containing words for which you want to compute vectors, use the following command:

```
$ mono ./build/FastText.exe print-vectors model.bin < queries.txt
```

This will output word vectors to the standard output, one vector per line.
This can also be used with pipes:

```
$ cat queries.txt | mono ./build/FastText.exe print-vectors model.bin
```

See the provided scripts for an example. For instance, running:

```
$ ./word-vector-example.sh
```

will compile the code, download data, compute word vectors and evaluate them on the rare words similarity dataset RW [Thang et al. 2013].

### Text classification

This library can also be used to train supervised text classifiers, for instance for sentiment analysis.
In order to train a text classifier using the method described in [2](#bag-of-tricks-for-efficient-text-classification), use:

```
$ mono ./build/FastText.exe supervised -input train.txt -output model
```

where `train.txt` is a text file containing a training sentence per line along with the labels.
By default, we assume that labels are words that are prefixed by the string `__label__`.
This will output two files: `model.bin` and `model.vec`.
Once the model was trained, you can evaluate it by computing the precision and recall at k (P@k and R@k) on a test set using:

```
$ mono ./build/FastText.exe test model.bin test.txt k
```

The argument `k` is optional, and is equal to `1` by default.

In order to obtain the k most likely labels for a piece of text, use:

```
$ mono ./build/FastText.exe predict model.bin test.txt k
```

where `test.txt` contains a piece of text to classify per line.
Doing so will print to the standard output the k most likely labels for each line.
The argument `k` is optional, and equal to `1` by default.
See `classification-example.sh` for an example use case.
In order to reproduce results from the paper [2](#bag-of-tricks-for-efficient-text-classification), run `classification-results.sh`, this will download all the datasets and reproduce the results from Table 1.

If you want to compute vector representations of sentences or paragraphs, please use:

```
$ mono ./build/FastText.exe print-vectors model.bin < text.txt
```

This assumes that the `text.txt` file contains the paragraphs that you want to get vectors for.
The program will output one vector representation per line in the file.

## Full documentation

Invoke a command without arguments to list available arguments and their default values:

```
$ mono ./build/FastText.exe supervised
Empty input or output path.

The following arguments are mandatory:
  -input      training file path
  -output     output file path

The following arguments are optional:
  -lr         learning rate [0.05]
  -dim        size of word vectors [100]
  -ws         size of the context window [5]
  -epoch      number of epochs [5]
  -minCount   minimal number of word occurences [1]
  -neg        number of negatives sampled [5]
  -wordNgrams max length of word ngram [1]
  -loss       loss function {ns, hs, softmax} [ns]
  -bucket     number of buckets [2000000]
  -minn       min length of char ngram [3]
  -maxn       max length of char ngram [6]
  -thread     number of threads [12]
  -verbose    how often to print to stdout [10000]
  -t          sampling threshold [0.0001]
  -label      labels prefix [__label__]
```

Defaults may vary by mode. (Word-representation modes `skipgram` and `cbow` use a default `-minCount` of 5.)

## References

Please cite [1](#enriching-word-vectors-with-subword-information) if using this code for learning word representations or [2](#bag-of-tricks-for-efficient-text-classification) if using for text classification.

### Enriching Word Vectors with Subword Information

[1] P. Bojanowski\*, E. Grave\*, A. Joulin, T. Mikolov, [*Enriching Word Vectors with Subword Information*](https://arxiv.org/pdf/1607.04606v1.pdf)

```
@article{bojanowski2016enriching,
  title={Enriching Word Vectors with Subword Information},
  author={Bojanowski, Piotr and Grave, Edouard and Joulin, Armand and Mikolov, Tomas},
  journal={arXiv preprint arXiv:1607.04606},
  year={2016}
}
```

### Bag of Tricks for Efficient Text Classification

[2] A. Joulin, E. Grave, P. Bojanowski, T. Mikolov, [*Bag of Tricks for Efficient Text Classification*](https://arxiv.org/pdf/1607.01759v2.pdf)

```
@article{joulin2016bag,
  title={Bag of Tricks for Efficient Text Classification},
  author={Joulin, Armand and Grave, Edouard and Bojanowski, Piotr and Mikolov, Tomas},
  journal={arXiv preprint arXiv:1607.01759},
  year={2016}
}
```

(\* These authors contributed equally.)

## Join the fastText community

* Facebook page: https://www.facebook.com/groups/1174547215919768
* Google group: https://groups.google.com/forum/#!forum/fasttext-library
* Contact: [egrave@fb.com](mailto:egrave@fb.com), [bojanowski@fb.com](mailto:bojanowski@fb.com), [ajoulin@fb.com](mailto:ajoulin@fb.com), [tmikolov@fb.com](mailto:tmikolov@fb.com)

## License

fastText is BSD-licensed. We also provide an additional patent grant.
