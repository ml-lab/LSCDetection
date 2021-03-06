# LSCDetection
Data Sets and Models for Evaluation of Lexical Semantic Change Detection.

If you use this software for academic research, please [cite](#bibtex) this paper:

- Dominik Schlechtweg, Anna Hätty, Marco del Tredici, and Sabine Schulte im Walde. 2019. [A Wind of Change: Detecting and Evaluating Lexical Semantic Change across Times and Domains](https://www.aclweb.org/anthology/papers/P/P19/P19-1072/). In Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics, pages 732-746, Florence, Italy. ACL.

Also make sure you give appropriate credit to the below-mentioned software this repository depends on.

Parts of the code rely on [DISSECT](https://github.com/composes-toolkit/dissect), [gensim](https://github.com/rare-technologies/gensim), [numpy](https://pypi.org/project/numpy/), [scikit-learn](https://pypi.org/project/scikit-learn/), [scipy](https://pypi.org/project/scipy/), [VecMap](https://github.com/artetxem/vecmap).

### Usage

The scripts should be run directly from the main directory. If you wish to do otherwise, you may have to change the path you add to the path attribute in `sys.path.append('./modules/')` in the scripts. All scripts can be run directly from the command line:

	python3 representations/count.py <corpDir> <outPath> <windowSize>

e.g.

	python3 representations/count.py corpora/test/corpus1/ test_matrix1 1

The usage of each script can be understood by running it with help option `-h`, e.g.:

	python3 representations/count.py -h

We recommend you to run the scripts within a [virtual environment](https://pypi.org/project/virtualenv/) with Python 3.7.4. Install the required packages running `pip install -r requirements.txt`. (See also [error sources](#errorsources).)

### Models

A standard model of LSC detection executes three consecutive steps:

1. learn semantic representations from corpora (`representations/`)
2. align representations (`alignment/`)
3. measure change (`measures/`)

As an example, consider a very simple model (CNT+CI+CD) going through these steps:

1. learn count vectors from each corpus to compare (`representations/count.py`)
2. align them by intersecting their columns (`alignment/ci_align.py`)
3. measure change with cosine distance (`measures/cd.py`)

You can apply this model to the testing data using the following commands:

        python3 representations/count.py corpora/test/corpus1/ test_matrix1 1
        python3 representations/count.py corpora/test/corpus2/ test_matrix2 1

        python3 alignment/ci_align.py test_matrix1 test_matrix2 test_matrix1_aligned test_matrix2_aligned

        python3 measures/cd.py -s testsets/test/targets.tsv test_matrix1_aligned test_matrix2_aligned test_results.tsv

__Input Format__: All the scripts in this repository can handle two types of matrix input formats:

- sparse scipy matrices stored in [npz format](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.save_npz.html)
- dense matrices stored in [word2vec plain text format](https://tedboy.github.io/nlps/generated/generated/gensim.models.Word2Vec.save_word2vec_format.html)
 
To learn more about how matrices are loaded and stored check out `modules/utils_.py`.

The scripts assume a corpus format of one sentence per line in UTF-8 encoded (optionally zipped) text files. You can specify either a file path or a folder. In the latter case the scripts will iterate over all files in the folder.

#### Semantic Representations

|Name | Code | Type | Comment |
| --- | --- | --- | --- |
| Count | `representations/count.py` | VSM | |
| PPMI | `representations/ppmi.py` | VSM | |
| SVD | `representations/svd.py` | VSM | |
| RI | `representations/ri.py` | VSM | - use `-a` for good performance |
| SGNS | `representations/sgns.py` | VSM | |
| SCAN | [repository](https://github.com/ColiLea/scan) | TPM | - different corpus input format |

Table: VSM=Vector Space Model, TPM=Topic Model

#### Alignment

|Name | Code | Applicability | Comment |
| --- | --- | --- | --- |
| CI | `alignment/ci_align.py` | Count, PPMI | |
| SRV | `alignment/srv_align.py` | RI | - use `-a` for good performance <br> - consider using the efficient and more powerful [TRIPY](https://github.com/Garrafao/TRIPY) |
| OP | `alignment/map_embeddings.py` | SVD, RI, SGNS | - drawn from [VecMap](https://github.com/artetxem/vecmap) <br> - for OP- and OP+ see `scripts/` |
| VI | `alignment/sgns_vi.py` | SGNS | - bug fixes 27/12/19 (see script for details) |
| WI | `alignment/wi.py` | Count, PPMI, SVD, RI, SGNS | - consider using the more advanced [Temporal Referencing](https://github.com/Garrafao/TemporalReferencing) |

#### Measures

|Name | Code | Applicability | Comment |
| --- | --- | --- | --- |
| CD | `measures/cd.py` | Count, PPMI, SVD, RI, SGNS | |
| LND | `measures/lnd.py` | Count, PPMI, SVD, RI, SGNS | |
| JSD | - | SCAN | |
| FD | `measures/freq.py` | from corpus | - log-transform with `measures/trsf.py` <br> - get difference with `measures/diff.py` |
| TD | `measures/typs.py` | Count | as above |
| HD | `measures/entropy.py` | Count | as above |

### Parameter Settings

Find detailed notes on model performances and optimal parameter settings in [these papers](#bibtex).

### Evaluation

The evaluation framework of this repository is based on the comparison of a set of target words across two corpora. Hence, models can be evaluated on a triple (dataset, corpus1, corpus2), where the dataset provides gold values for the change of target words between corpus1 and corpus2.

| Dataset | Corpus 1 | Corpus 2 | Download | Comment |
| --- | --- | --- | --- | --- |
| DURel | DTA18 | DTA19  | [Dataset](https://www.ims.uni-stuttgart.de/forschung/ressourcen/experiment-daten/durel.html), [Corpora](https://www.ims.uni-stuttgart.de/forschung/ressourcen/korpora/wocc.html) | - version from Schlechtweg et al. (2019) at `testsets/durel/` |
| SURel | SDEWAC | COOK | [Dataset](https://www.ims.uni-stuttgart.de/forschung/ressourcen/experiment-daten/surel.html), [Corpora](https://www.ims.uni-stuttgart.de/forschung/ressourcen/korpora/wocc.html) | - version from Schlechtweg et al. (2019) at `testsets/surel/` |
| SemCor LSC | SEMCOR1 | SEMCOR2 | [Dataset](https://www.ims.uni-stuttgart.de/data/lsc-simul), [Corpora](https://www.ims.uni-stuttgart.de/data/lsc-simul) | |

We provide several evaluation pipelines, downloading the corpora and evaluating the models on the above-mentioned datasets, see [pipeline](#pipeline).

#### Metrics

|Name | Code | Applicability | Comment |
| --- | --- | --- | --- |
| Spearman correlation | `evaluation/spr.py` | DURel, SURel, SemCor LSC | - outputs rho (column 3) and p-value (column 4) |
| Average Precision | `evaluation/ap.py` | SemCor LSC | - outputs AP (column 3) and random baseline (column 4) |

Consider uploading your results for DURel as a submission to the shared task [Lexical Semantic Change Detection in German](https://codalab.lri.fr/competitions/560).

#### Pipeline

Under `scripts/` you find an example of a full evaluation pipeline for the models on two small test corpora. Assuming you are working on a UNIX-based system, first make the scripts executable with

	chmod 755 scripts/*.sh

Then run

	bash -e scripts/run_test.sh

The script first reads the two gzipped test corpora `corpora/test/corpus1/` and `corpora/test/corpus2/`. Then it produces model predictions for the targets in `testsets/test/targets.tsv` and writes them under `results/`. It finally writes the Spearman correlation between each model's predictions and the gold rank (`testsets/test/gold.tsv`) under the respective folder in `results/`. Note that the gold values for the test data are meaningless, as they were randomly assigned.

We also provide a script for each dataset running all the models on it including necessary downloads. For this run either of 

	bash -e scripts/run_durel.sh
	bash -e scripts/run_surel.sh
	bash -e scripts/run_semcor.sh

As is the scripts will reproduce the results from Schlechtweg et al. (2019) and Schlechtweg & Schulte im Walde (2020). You may want to change the parameters in `scripts/parameters_durel.sh`, etc. (e.g. vector dimensionality, iterations), as running the scripts on the full parameter set may take several days and require a large amount of disk space.

### Important Changes

September 1, 2019: Python scripts were updated from Python 2 to Python 3.
December 27, 2019: bug fixes in `alignment/sgns_vi.py` (see script for details)

### Error Sources

- if you are on a Windows system and get error messages like `[bash] $'\r': command not found`, consider removing trailing '\r' characters with `sed -i 's/\r$//' scripts/*.sh`

BibTex
--------

```
@inproceedings{Schlechtwegetal19,
	title = {{A Wind of Change: Detecting and Evaluating Lexical Semantic Change across Times and Domains}},
	author = {Dominik Schlechtweg and Anna H\"{a}tty and Marco del Tredici and Sabine {Schulte im Walde}},
    booktitle = {Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics},
	year =  {2019},
	address =  {Florence, Italy},
	publisher =  {Association for Computational Linguistics},
	pages     = {732--746}
}
```
```	
@inproceedings{SchlechtwegWalde20,
	author = {Dominik Schlechtweg and Sabine {Schulte im Walde}},
	booktitle = {{The Evolution of Language: Proceedings of the 13th International Conference (EVOLANGXIII)}},
	editor = {C. Cuskley and M. Flaherty and H. Little and Luke McCrohon and A. Ravignani and T. Verhoef},
	title = {{Simulating Lexical Semantic Change from Sense-Annotated Data}},
	year = {2020}
}
	
```

