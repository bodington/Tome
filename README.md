# Tome: Temperature optima for microorganisms and enzymes
Tome (Temperature optima for microorganisms and enzymes) is an open source suite
for two fundamental applications:
  * predict the optimal growth temperature from proteome sequences
  * get enzymes that have a temperature optima in a specified range for a given EC number or CAZy family ID. If a protein sequence is provided, then it will select and report all homologous enzymes.       


## Citation
If you have used `tome getEnzymes` in Tome v1.0 or `tome predOGT`, please cite  
*Li G., et al., (2019). Machine learning applied to predicting microorganism growth temperatures and enzyme catalytic optima. ACS Synthetic Biology, 8, 1411–1420.*

If you have `tome getEnzymes` in Tome v2.0, please cite   
*Li G., et al., (2019). Performance of regression models as a function of experiment noise. arXiv:1912.08141 [q-bio.BM]*

## System
* macOS
* Linux

## Installation
##### (1). Download tome package
##### (2). Open your terminal
##### (3). Change directory to the tome package
```linux
cd [directory to tome, where setup.py is]
```
##### (4). Run following command
```linux
pip install -e .
```
##### (5) Now you can use 'tome' via command line.
There is a folder named 'test' in the package. One can use the instructions in
'Usage' section to test the package.

## Depedencies
* ncbi-blast-2.7.1+ (ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/)
(This is only mandatory for 'tome predOGT -seq')
* pandas
* Biopython
* numpy
* scikit-learn
* Python 2.7 or Python 3


## Usage:
Get help message of `tome` with 
```
tome -h
```
```
usage: tome [-h] {predOGT,getEnzymes} ...

Tome (Temperature optima for microorganisms and enzymes) is an open source suite for two purposes: (1) predict the
optimal growth temperature from proteome sequences (predOGT) (2) get homologue enzymes for a given class id (EC number or
CAZy family) with/without a sequence (getEnzymes) A detailed list of options can be obtained by calling 'tome predOGT
--help'for predOGT or 'tome getEnzymes --help' for getEnzymes

positional arguments:
  {predOGT,getEnzymes,predTopt}
    predOGT             Predict the optimal growth temperature from proteomes
    predTopt            Predict the optimal temperature of enzymes in a proteome
    getEnzymes          Get (homologue) enzymes for a given EC number of CAZy class with/without a sequence

optional arguments:
  -h, --help            show this help message and exit
```

### 1. Prediction of optimal growth temperature
Get help message of `tome predOGT` with 
```
tome predOGT -h
```

```
usage: tome predOGT [-h] [--fasta] [--indir] [--train] [-o] [-p]

optional arguments:
  -h, --help       show this help message and exit
  --fasta          a fasta file containing all protein sequence of a proteome.
  --indir          a directory that contains a list of fasta files. Each fasta file is a proteome. Required for the
                   prediction of OGT for a list of microorganisms. Important: Fasta file names much end with .fasta
  --train          train the model again
  -o , --out       out file name
  -p , --threads   number of threads used for feature extraction, default is 1. if set to 0, it will use all cpus
                   available
```

#### Case 1.1 Prediction of optimal growth temperature for one microorganism
```linux
tome predOGT --fasta test/proteomes/pyrococcus_horikoshii.fasta
```
Then you will get following results:<br/>
```
FileName	predOGT (C)
pyrococcus_horikoshii.fasta	94.0
```

#### Case 1.2 Predict optimal growth temperatures for a list of microorganisms. Fasta files must end with .fasta
```linux
tome predOGT --indir test/proteomes/ -o test/proteomes/predicted_ogt.tsv
```
Then you will get an tab-seperated output file predicted_ogt.tsv with following
contents:<br/>
```
FileName	predOGT (C)
succinivibrio_dextrinosolvens.fasta	38.27
pyrococcus_horikoshii.fasta	94.0
caldanaerobacter_subterraneus.fasta	70.0
```
#### train the model
In case there would be some warnings due to the versions of sklearn when loading
the model, one can use following command to train the model again:
```linux
tome predOGT --train
```
Expected output after training is
```
A new model has beed successfully trained.
Model performance:
        RMSE: 2.159489340036136
          r2: 0.9552614628185572
  Pearson r:(0.9775886084277753, 0.0)
  Spearman r:SpearmanrResult(correlation=0.93331975456613, pvalue=0.0)

Saving the new model to replace the original one...
Done!
```

### 2. Get enzyme sequences
The first time `tome getEnzymes --database brenda` used, a SQLite3 database ~3.5 GB will be downloaded.  
The first time `tome getEnzymes --database cazy` used, a SQLite3 database ~601 MB will be downloaded.  
Those two databases are deposited on Zenodo (https://zenodo.org/record/3578468#.XffgbpP0nOQ). These files contain the enzyme annotation data.

If automatic download failed, then one can manually download following two files from Zenodo and put them into `tome/external_data/`: 
BRENDA: https://zenodo.org/record/3578468/files/brenda.sql
CAZy  : https://zenodo.org/record/3578468/files/cazy.sql
  
Get help message of `tome getEnzymes` with
```
tome getEnzymes -h
```

```
usage: tome getEnzymes [-h] [--database] [--class_id] [--temp_range] [--data_type] [--seq] [--outdir] [--evalue] [-p]

optional arguments:
  -h, --help          show this help message and exit
  --database          the dataset should be used. Should be either brenda or cazy
  --class_id , --ec   EC number or CAZy family. 1.1.1.1 for BRENDA, GH1 for CAZy, for instance.
  --temp_range        the temperature range that target enzymes should be in. For example: 50,100. 50 is lower bound and
                      100 is upper bound of the temperature.
  --data_type         [OGT or Topt], If OGT, Tome will find enzymes whose OGT of its source organims fall into the
                      temperature range specified by --temp_range. If Topt, Tome will find enzymes whose Topt fall into
                      the temperature range specified by --temp_range. Default is Topt
  --seq               input fasta file which contains the sequence of the query enzyme. Optional
  --outdir            directory for ouput files. Default is current working folder.
  --evalue            evalue used in ncbi blastp. Default is 1e-10
  -p , --threads      number of threads used for blast, default is 1. if set to 0, it will use all cpus available
```


#### Brenda Case 2.1 Get enzymes for a given ec number.
For example, we want to get the enzymes with EC 3.2.1.1 with a temperature optima
higher 50 °C.
```linux
tome getEnzymes --ec 3.2.1.1 --temp_range 50,200 --data_type Topt --outdir test/enzyme_without_seq/
```

Two output files will be generated: test/enzyme_without_seq/3.2.1.1_all.fasta and
test/enzyme_without_seq/3.2.1.1_all.tsv
3.2.1.1_all.fasta contains all sequences for this EC number. This can be used for
mutisequence alignment with tools like Clustal Omega (https://www.ebi.ac.uk/Tools/msa/clustalo/)
enzyme_without_seq/3.2.1.1_all.tsv contains following columns:
* uniprot id
* domain: Domain information of source organism (Archaea/Bacteria/Eukaryote)
* organism: name of source organism
* ogt: optimal growth temperature of source organism
* ogt_source: if the growth temperature is predicted or experimentally determined
* topt: temperature optima of the enzyme
* topt_source: if topt is predicted or experimentally determined
* seqeunce: protein sequence

One can also use following command to find enzymes from organisms with a OGT fall
into the temperature range specified with --temp_range. The output files have the same
format as above described.
```linux
tome getEnzymes --ec 3.2.1.1 --temp_range 50,200 --data_type OGT --outdir test/enzyme_without_seq/
```

#### Brenda Case 2.2 Get homologous enzymes for an given ec number and sequence.
For example, we want to get all homologs of an enzyme with EC 3.2.1.1
from Photobacterium profundum (OGT = 13°C). We want those homologs with a temperature
optima higher 50 °C. The sequence for this enzyme is
```
>Q1Z0D7
MTSLFNTEYASTLSAPSVATNVILHAFDWPYSKVTENAKAIADNGYKAILVSPPLKSFHSKDGTQWWQRYQPQDYRVIDN
QLGNTNDFRTMVEILSLHDIDIYADIVFNHMANESHERSDLNYPNSNIISQYKDKREYFDSIKLFGDLSQPLFSKDDFLS
AFPIKDWKDPWQVQHGRISSGGSDPGLPTLKNNENVVKKQKLYLKALKKIGVKGFRIDAAKHMTLDHIQELCDEDITDGI
HIFGEIITDGGATKEEYELFLQPYLEKTTLGAYDFPLFHTVLDVFNKNASMASLINPYSLGSALENQRAITFAITHDIPN
NDVFLDQVMSEKNEQLAYCYILGRDGGVPLIYTDLDTSGIKNSRGKPRWCEAWNDPIMAKMIHFHNIMHCQPMVIIEQTL
DLLVFSRGHSGIVAINKGKTAVCYKLPAKYSEQDHTEIKEVINMEGVKLSPPSLSTEAGVILQLPAQSCAMLMV
```
There should be only one sequence in the fasta file. If more than 1 sequence is provided,
only the first sequence would be used.
```linux
tome getEnzymes --seq test/enzyme_with_seq/test.fasta --ec 3.2.1.1 --temp_range 50,200 --data_type Topt --outdir test/enzyme_with_seq/
```
Two output files will be created:
* Q1Z0D7_homologs.fasta: a fasta file which contains sequences for all homologs of query enzyme
* Q1Z0D7_homologs.tsv: a tab-seperated file with following columns:
  * uniprot id
  * domain: Domain information of source organism (Archaea/Bacteria/Eukaryote)
  * organism: name of source organism
  * ogt: optimal growth temperature of source organism
  * ogt_source: if the growth temperature is predicted or experimentally determined
  * topt: temperature optima of the enzyme
  * topt_source: if topt is predicted or experimentally determined
  * Identity(%) from blast
  * Coverage(%) from blast
  * seqeunce: protein sequence

In this test case, 44 homologs with a temperature optima higher than 50 °C were found.

One can also use following command to find homologous enzymes from organisms with a OGT fall
into the temperature range specified with --temp_range. The output files have the same
format as above described.
```linux
tome getEnzymes --seq test/enzyme_with_seq/test.fasta --ec 3.2.1.1 --temp_range 50,200 --data_type OGT --outdir test/enzyme_with_seq/
```
In this case, 13 homologs from organisms with a OGT higher than 50 °C were found

#### CAZy case 2.4 Get enzymes for a given CAZy family ID.
```
tome getEnzymes --database cazy --class_id GH1 --temp_range 50,60 --data_type Topt --outdir test/cazy_enzyme_without_seq/
```
Similar as for BRENDA, This will find all enzymes in CAZy database that (1) belongs to GH1 family, (2) have a predicted Topt between 50 and 60 °C.

#### CAZy case 2.4 Get homologous enzymes for a given CAZy family ID.
```
tome getEnzymes --database cazy --class_id GH1 --temp_range 50,60 --data_type Topt --outdir test/cazy_enzyme_with_seq/ --seq test/cazy_enzyme_with_seq/test.fasta
```
Similar as for BRENDA, This will find all enzymes in CAZy database that (1) belongs to GH1 family, (2) have a predicted Topt between 50 and 60 °C, (3) homologous to the given sequence in `test.fasta`

### 3. Prediction of optimal temperature for each enzyme
Get help message of `tome predTopt` with 
```
tome predTopt -h
```

```
usage: tome predTopt [-h] [--fasta] [--ogt] [-o]

optional arguments:
  -h, --help       show this help message and exit
  --fasta          a fasta file containing all protein sequence of a proteome.
  --ogt            a file containing a list of all proteins in a proteome and the OGT of the organism.
                   The first line/row of the text file must be a header and the OGT data must be in the last column.
                   This file can be generated using the --ogt option of predOGT
  -o , --out       out file name
```

#### Case 1.1 Prediction of optimal growth temperature for all enzymes in one microorganism
```linux
tome predTopt --fasta test/proteomes/pyrococcus_horikoshii.fasta --ogt test/proteomes/pyrococcus_horikoshii.ogt.list
```
This will output a file listing the predicted optimal temperature for each enzyme in the fasta file, with standard error based on the learning model as specified in the tomer section below. The second line includes the max, min, range, standard deviation and standard error of predicted optimal temperatures for all enzymes within the organism.<br/>
```
Filename	Topt Min	Topt Max	Topt Range Topt SD Topt SE
pyrococcus_horikoshii.fasta	56.685	96.955	40.269999999999996	4.503333564959797 0.503333564959797

                                                ID     Topt   Std err
 BAA29069 pep chromosome:ASM1110v1:Chromosome:1...     84.4   1.92934
 BAA29070 pep chromosome:ASM1110v1:Chromosome:6...    78.97   1.51987
 BAA29071 pep chromosome:ASM1110v1:Chromosome:1...   85.565    1.6083

```




**TOMER: Temperature Optima for Enzymes with Resampling**
------------------------------------------------------------

TOMER is a Python package for predicting the catalytic optimum temperature (Topt) of enzymes with machine learning. TOMER was trained with a bagging ensemble on a dataset of 2,917 proteins. To prevent large error on the prediction of higher temperature values, resampling strategies were applied to mitigate the effects of the imbalanced distribution of the dataset. Code for design of TOMER can be found `here <https://github.com/jafetgado/tomerdesign>`_.

Citation
----------
If you find TOMER useful, please cite:

* Gado, J.E., Beckham, G.T., and Payne, C.M (2020). Improving enzyme optimum temperature prediction with resampling strategies and ensemble learning. *J. Chem. Inf. Model.* 60(8), 4098-4107.


Installation
-------------
Install with pip

.. code:: shell-session

    pip install tomer

Or from source (preferred).

.. code:: shell-session

    git clone https://github.com/jafetgado/tomer.git
    cd tomer
    pip install -r requirements.txt
    python setup.py install



Prerequisites
----------------
(version used in this work)

1. Python 3 (3.6.6)
2. scikit-learn (0.21.2)
3. numpy (1.19.5)
4. pandas (0.24.1)


Usage
-----
There are two main functions in TOMER for predicting the enzyme optimum temperature: ``pred_seq_topt``, which predicts optimum temperature of a single protein sequence (in string format), and ``pred_fasta_topt``, which predicts the optimum temperatures of protein sequences in a fasta file. To use these functions, you need to specify the optimal growth temperature (OGT) of the source organism of the protein. If the OGT is not known, a prediction may be obtained using `TOME <https://github.com/EngqvistLab/Tome>`_.



Examples
----------
.. code:: python

    import tomer

    # Predict optimum temperature of a single sequence.
    sequence = '''MKKQVVEVLVEGGKATPGPPLGPAIGPLGLNVKQVVDKINEATKEFAGMQVPVKIIV
                  DPVTKQFEIEVGVPPTSQLIKKELGLEKGSGEPKHNIVGNLTMEQVIKIAKMKRSQML
                  ALTLKAAAKEVIGTALSMGVTVEGKDPRIVQREIDEGVYDELFEKAEKE'''
    ogt = 95
    y_pred, y_err = tomer.pred_seq_topt(sequence, ogt)

    print(y_pred)   # predicted optimum temperature
    84.4

    print(y_err)    # Standard error of prediction (from 100 base learners in ensemble)
    1.929

    # Predict optimum temperatures of sequences in fasta file
    fasta_file = 'test/sequences.fasta'
    ogt_file = 'test/ogts.txt'
    df = tomer.pred_fasta_topt(fasta_file, ogt_file) # returns Pandas dataframe

    print(df)
          ID     Topt    Std err
    0   P43408  79.345   1.53561
    1   Q97X08  81.705  0.442442
    2   F8A9V0   76.37   1.16195


