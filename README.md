# CNNs vs RNNs for Sentence Classification
## Models (WIP)
General idea is the same:
1) Encode source sentence to get single a vector sentence representation
2) Project this representation onto desired classes

## Results
| Model | SAI | SST1 | SST2 | TREC |
| --- | --- | ---| --- | --- |
| boe-baseline | 0.823 / 0.961 | x | x | x |  
| Elman-RNN-baseline | 0.785 / 0.937 | x | x | x |
| Elman-RNN-bi | 0.807 / 0.960 | x | x | x |  
| Elman-RNN-bi-avg | x | x | x | x | 
| GRU-bi-avg | 0.823 / 0.951 | x | x | x |
| LSTM-bi-avg | x | x | x | x |
| GRU-bi-avg-static | 0.757 / 0.939 | x | x | x |   
| GRU-bi-avg-rand | 0.817 / 0.953 | x | x | x |   
| CNN-rand |  | 0.814 / 0.952 | x | x |   
| CNN-static | 0.753 / 0.920 | x | x | x |   
| CNN-nonstatic | 0.822 / 0.959 | x | x | x |
| GRU-bi-avg-charseq | 0.824 / 0.953 | x | x | x |  
__Metric__: accuracy_top1 / accuracy_top2 <br>
__NB!__: you can find configs for all the models in experiments_configs folder (names align)

python3 -m run evaluate --archive_file experiments_models/boe-baseline/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/elman-rnn-baseline/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/elman-rnn-bi/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/elman-rnn-bi-avg/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/gru-bi-avg/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/lstm-bi-avg/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/gru-bi-avg-static/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/gru-bi-avg-rand/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/cnn-rand/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/cnn-static/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/cnn-nonstatic/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0
python3 -m run evaluate --archive_file experiments_models/gru-bi-avg-charseg/model.tar.gz --evaluation_data_file data_kaggle/preprocessed-dev.txt --cuda_device 0

Different approaches are different in a way we encode the sentence <br>
Models I experimented with: <br> 
- __boe-baseline__: just avarage over word embeddings 
- __Elman-RNN-baseline__: simple Elman RNN; starting point for future experiments; see "Hyperparameters and Training" section for details
- __Elman-RNN-bi__: the same as RNNs above, but bidirectional 
- __Elman-RNN-bi-avg__: use average over RNN hidden states instead of just last hidden state 
- __GRU-bi-avg__: use GRU cells
- __LSTM-bi-avg__: use LSTM cells
- __GRU-bi-avg-static__: embeddings are initialized with vectors from _glove_ and fixed  
- __GRU-bi-avg-rand__:  embeddings are randomly initialized
- __CNN-rand__:  word embeddings are randomly initialized
- __CNN-static__: word embeddings are initialized with vectors from _glove_ and fixed  
- __CNN-nonstatic__: same as CNN-static, but pre trained vectors are fine tuned
- __GRU-bi-avg-charseq__: word embeddings plus CNN based word characters embeddings  
Note: words embeddings are nonstatic unless otherwise noted

## Hyperparameters and Training
__experiments_configs__ folder containes hyperparameters for all the experiments (and models) (they are composed in a way to hold "all things being equal" property) 

## Spooky Author Identification using Neural Networks

Competition goal is to predict the author of excerpts from horror stories by Edgar Allan Poe, Mary Shelley, and HP Lovecraft

However, the goal of my experiments was not to get the best score possible, but to experience myself how several neural networks variants react to the sentence representation options. 


Competition link: [link](https://www.kaggle.com/c/spooky-author-identification) <br>
All the models are implemented using [allennlp](https://github.com/allenai/allennlp) lib

To reproduce experiments, fetch allennlp's commit specified at requirements.txt.   

[Competiton rules](https://www.kaggle.com/c/spooky-author-identification/rules) allow for use of the data for academic goals, 
so I attached prepared data to this repo (/data folder): `preprocessed-train.txt`, `preprocessed-dev.txt`, `tc-tok-test_public_X.txt`
## Steps to reproduce (Kaggle)
1) Download glove embeddings and put to to the data/glove folder: <br>
`https://s3-us-west-2.amazonaws.com/allennlp/datasets/glove/glove.6B.100d.txt.gz`
2) Train a model: <br>
`python3 -m run train experiments_configs/authors_classifier.json -s experiments_models/pilot`<br>
(you can choose another .json config to train another model, or choose another save folder)
3) [Optional] Compute metrics values for your model on dev set: <br>
`python3 -m run evaluate --archive_file experiments_models/pilot/model.tar.gz --evaluation_data_file data/preprocessed-dev.txt --cuda_device -1`
4) Convert test set to the json lines format: <br>
`python3 data_utils/predict_utils.py data/tc-tok-test_public_X.txt data/tc-tok-test_public_X.jsonl`
5) Translate the test set: <br>
`python3 -m run predict experiments_models/pilot/model.tar.gz data/tc-tok-test_public_X.jsonl --output-file data/submission-pilot.jsonl`
6) Convert test set to the Kaggle submision format: <br>
`python3 data_utils/predict_utils.py data/submission-pilot.jsonl data/submission-pilot.csv`  <br>
(assumes python3 points to python 3.6)
Note: you can reproduce for other datasets by changing dataset path in configs and following almost the same procedure as above

## Data preprocessing
The data is first split into dev (3000 points) and train sets (code is in the /data_utils folder), and then truecased and tokenized using [moses scripts](https://github.com/marian-nmt/moses-scripts)    