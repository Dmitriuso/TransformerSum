# TransformerExtSum
> A model to perform neural extractive summarization using machine learning transformer models and a tool to convert abstractive summarization datasets to the extractive task.

**Attributions:**

* Code heavily inspired by the following projects:
  * Adapting BERT for Extractive Summariation: [BertSum](https://github.com/nlpyang/BertSum)
  * Word/Sentence Embeddings: [sentence-transformers](https://github.com/UKPLab/sentence-transformers)
  * CNN/CM Dataset: [cnn-dailymail](https://github.com/artmatsak/cnn-dailymail)
  * PyTorch Lightning Classifier: [lightning-text-classification](https://github.com/ricardorei/lightning-text-classification)
* Important projects utilized:
  * PyTorch: [pytorch](https://github.com/pytorch/pytorch/)
  * Training code: [pytorch_lightning](https://github.com/PyTorchLightning/pytorch-lightning/)
  * Transformer Models: [huggingface/transformers](https://github.com/huggingface/transformers)

## Pre-trained Models

None yet. Please wait.

| Name                    | Info | Download   |
|-------------------------|------|------------|
| distilbert-base-ext-sum | None | Not yet... |
| bert-base-ext-sum       | None | Not yet... |
| albert-base-v2-ext-sum  | None | Not yet... |

## Install

Installation is made easy due to conda environments. Simply run this command from the root project directory: `conda env create --file environment.yml` and conda will create and environment called `transformerextsum` with all the required packages from [environment.yml](environment.yml). The spacy `en_core_web_sm` model is required for the [convert_to_extractive.py](convert_to_extractive.py) script to detect sentence boundaries.

### Step-by-Step Instructions

1. Clone this repository: `git clone https://github.com/HHousen/transformerextsum.git`.
2. Change to project directory: `cd transformerextsum`.
3. Run installation command: `conda env create --file environment.yml`.
4. **(Optional)** If using the [convert_to_extractive.py](convert_to_extractive.py) script then download the `en_core_web_sm` spacy model: `python -m spacy download en_core_web_sm`.

## Supported Datasets

Currently only the CNN/DM summarization dataset is supported. The original processing code is available at [abisee/cnn-dailymail](https://github.com/abisee/cnn-dailymail), but for this project the [artmatsak/cnn-dailymail](https://github.com/artmatsak/cnn-dailymail) processing code is used since it does not tokenize and writes the data to text file `train.source`, `train.target`, `val.source`, `val.target`, `test.source` and `test.target`, which is the format expected by [convert_to_extractive.py](convert_to_extractive.py). 

### CNN/DM

Download and unzip the stories directories from [here](https://cs.nyu.edu/~kcho/DMQA/) for both CNN and Daily Mail. The files can be downloaded from the terminal with `gdown`, which can be installed with `pip install gdown`.

```
pip install gdown
gdown https://drive.google.com/uc?id=0BwmD_VLjROrfTHk4NFg2SndKcjQ
gdown https://drive.google.com/uc?id=0BwmD_VLjROrfM1BxdkxVaTY2bWs
tar zxf cnn_stories.tgz
tar zxf dailymail_stories.tgz
```

**Note:** The above Google Drive links may be outdated depending on the time you are reading this. Check the [CNN/DM official website](https://cs.nyu.edu/~kcho/DMQA/) for the most up-to-date download links.

Next, run the processing code in the git submodule for [artmatsak/cnn-dailymail](https://github.com/artmatsak/cnn-dailymail) located in `cnn_dailymail_processor`. Run `python make_datafiles.py /path/to/cnn/stories /path/to/dailymail/stories`, replacing `/path/to/cnn/stories` with the path to where you saved the `cnn/stories` directory that you downloaded; similarly for `dailymail/stories`.

For each of the URL lists (`all_train.txt`, `all_val.txt` and `all_test.txt`) in `cnn_dailymail_processor/url_lists`, the corresponding stories are read from file and written to text files `train.source`, `train.target`, `val.source`, `val.target`, and `test.source` and `test.target`. These will be placed in the newly created `cnn_dm` directory.

## Convert Abstractive to Extractive Dataset

Simply run [convert_to_extractive.py](convert_to_extractive.py) with the path to the data. For example, with the CNN/DM dataset downloaded above: `python convert_to_extractive.py ./cnn_dailymail_processor/cnn_dm`. However, the recommended command is: `python convert_to_extractive.py ./cnn_dailymail_processor/cnn_dm --shard_interval 5000 --compression --add_target_to test`, the `--shard_interval` processes the file in chunks of `5000` and writes results to disk in chunks of `5000` (saves RAM) and the `--compression` compresses each output chunk with gzip (depending on the dataset reduces space usage requirement by about 1/2 to 1/3). The default output directory is the input directory that was specified, but the output directory can be changed with `--base_output_path` if desired.

The `--add_target_to` argument will save the abstractive target text to the splits (in `--split_names`) specified.

If your files are not `train`, `val`, and `test`, then the `--split_names` argument will let you specify the correct naming pattern. The `--source_ext` and `--target_ext` let you specify the file extension of the source and target files respectively. These must be different so the process can tell each section apart.

### Script Help

Output of `python convert_to_extractive.py --help`:

```
usage: convert_to_extractive.py [-h] [--base_output_path BASE_OUTPUT_PATH]
                                [--split_names {train,val,test} [{train,val,test} ...]]
                                [--add_target_to {train,val,test} [{train,val,test} ...]]
                                [--source_ext SOURCE_EXT]
                                [--target_ext TARGET_EXT]
                                [--oracle_mode {greedy,combination}]
                                [--shard_interval SHARD_INTERVAL]
                                [--n_process N_PROCESS]
                                [--batch_size BATCH_SIZE] [--compression]
                                [--resume]
                                [-l {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                                DIR

Convert an Abstractive Summarization Dataset to the Extractive Task

positional arguments:
  DIR                   path to data directory

optional arguments:
  -h, --help            show this help message and exit
  --base_output_path BASE_OUTPUT_PATH
                        path to output processed data (default is `base_path`)
  --split_names {train,val,test} [{train,val,test} ...]
                        which splits of dataset to process
  --add_target_to {train,val,test} [{train,val,test} ...]
                        add the abstractive target to these splits (useful for
                        calculating rouge scores)
  --source_ext SOURCE_EXT
                        extension of source files
  --target_ext TARGET_EXT
                        extension of target files
  --oracle_mode {greedy,combination}
                        method to convert abstractive summaries to extractive
                        summaries
  --shard_interval SHARD_INTERVAL
                        how many examples to include in each shard of the
                        dataset (default: no shards)
  --n_process N_PROCESS
                        number of processes for multithreading
  --batch_size BATCH_SIZE
                        number of batches for tokenization
  --compression         use gzip compression when saving data
  --resume              resume from last shard
  -l {DEBUG,INFO,WARNING,ERROR,CRITICAL}, --log {DEBUG,INFO,WARNING,ERROR,CRITICAL}
                        Set the logging level (default: 'Info').
```

## Training an Extractive Summarization Model

Once the dataset has been converted to the extractive task, it can be used as input to a SentencesProcessor, which has a `add_examples()` function too add sets of `(example, labels)` and a `get_features()` function that returns a TensorDataset of extracted features (`input_ids`, `attention_masks`, `labels`, `token_type_ids`, `sent_rep_token_ids`, `sent_rep_token_ids_masks`). Feature extraction runs in parallel and tokenizes text using the tokenizer appropriate for the model specified with `--model_name_or_path`. The tokenizer can be changed to another huggingface/transformers tokenizer with the `--tokenizer_name` option. 

Continuing with the CNN/CM dataset, to train a model for 50,000 steps on the data run: `python main.py --data_path ./cnn_dailymail_processor/cnn_dm --default_save_path ./trained_models --do_train --max_steps 50000`.

The `--do_train` argument runs the training process. Set `--do_test` to test after training.
The `--data_path` argument specifies where the extractive dataset json file are located.
The `--default_save_path` argument specifies where the logs and model weights should be stored.
If you prefer to measure training progress by epochs instead of steps, the `--max_epochs` and `--min_epochs` options exist just for you.

The batch sizes can be changed with the `--train_batch_size`, `--val_batch_size`, and `--test_batch_size` options.

If the extractive dataset json files are compressed using json, then they will be automatically decompressed during the data preprocessing step of training.

By default the model is saved after every epoch to the `--default_save_path`.

### Automatic Preprocessing

While the [convert_to_extractive.py](convert_to_extractive.py) script prepares a dataset for the extractive task, the data still needs to be processed for usage with a machine learning model. This preprocessing depends on the chosen model, and thus is implemented in the [model.py](model.py) file.

The actual ExtractiveSummarizer LightningModule (which is similar to an nn.Module but with a built-in training loop, more info at the [pytorch_lightning documentation](https://pytorch-lightning.readthedocs.io/en/latest/)) implements a `prepare_data()` function. This `prepare_data()` function is automatically called by `pytorch_lightning` to load and process the examples.

Memory Usage Note: If sharding was turned off during the `convert_to_extractive` process then `prepare_data()` will run once, loading the entire dataset into memory to process just like the [convert_to_extractive.py](convert_to_extractive.py) script.

There is a `--only_preprocess` argument available to only run this preprocess step and exit the script after all the examples have been written to disk. This option will force data to be preprocessed, even if it was already computed and is detected on disk, and any previous processed files will be overwritten.

Thus, the command to only preprocess data for use when training a model run: `python main.py --data_path ./cnn_dailymail_processor/cnn_dm --use_logger tensorboard --model_name_or_path bert-base-uncased --model_type bert --do_train --only_preprocess`

**Important Note:** If processed files are detected, they will automatically be loaded from disk. This includes any files that follow the pattern `[dataset_split_name].*.pt`, where `*` is any text of any length.

### Script Help

Output of `python main.py --help`:

```
usage: main.py [-h] --default_save_path DEFAULT_SAVE_PATH
               [--learning_rate LEARNING_RATE] [--min_epochs MIN_EPOCHS]
               [--max_epochs MAX_EPOCHS] [--min_steps MIN_STEPS]
               [--max_steps MAX_STEPS]
               [--accumulate_grad_batches ACCUMULATE_GRAD_BATCHES]
               [--check_val_every_n_epoch CHECK_VAL_EVERY_N_EPOCH]
               [--gpus GPUS] [--gradient_clip_val GRADIENT_CLIP_VAL]
               [--overfit_pct OVERFIT_PCT] [--amp_level AMP_LEVEL]
               [--precision PRECISION] [--profiler]
               [--progress_bar_refresh_rate PROGRESS_BAR_REFRESH_RATE]
               [--num_sanity_val_steps NUM_SANITY_VAL_STEPS]
               [--use_logger {tensorboard,wandb}] [--do_train] [--do_test]
               [--use_custom_checkpoint_callback]
               [-l {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
               [--model_name_or_path MODEL_NAME_OR_PATH]
               [--model_type MODEL_TYPE] [--tokenizer_name TOKENIZER_NAME]
               [--tokenizer_lowercase] [--max_seq_length MAX_SEQ_LENGTH]
               [--oracle_mode {none,greedy,combination}] --data_path DATA_PATH
               [--num_threads NUM_THREADS]
               [--processing_num_threads PROCESSING_NUM_THREADS]
               [--weight_decay WEIGHT_DECAY]
               [--pooling_mode {sent_rep_tokens,mean_tokens}]
               [--web_learning_rate WEB_LEARNING_RATE]
               [--adam_epsilon ADAM_EPSILON] [--optimizer_type OPTIMIZER_TYPE]
               [--ranger-k RANGER_K] [--warmup_steps WARMUP_STEPS]
               [--use_scheduler USE_SCHEDULER]
               [--num_frozen_steps NUM_FROZEN_STEPS]
               [--train_batch_size TRAIN_BATCH_SIZE]
               [--val_batch_size VAL_BATCH_SIZE]
               [--test_batch_size TEST_BATCH_SIZE]
               [--processor_no_bert_compatible_cls] [--only_preprocess]
               [--create_token_type_ids {binary,sequential}]
               [--no_use_token_type_ids] [--train_name TRAIN_NAME]
               [--val_name VAL_NAME] [--test_name TEST_NAME]
               [--test_id_method {greater_k,top_k}] [--test_k TEST_K]

optional arguments:
  -h, --help            show this help message and exit
  --default_save_path DEFAULT_SAVE_PATH
                        Default path for logs and weights
  --learning_rate LEARNING_RATE
                        The initial learning rate for the optimizer.
  --min_epochs MIN_EPOCHS
                        Limits training to a minimum number of epochs
  --max_epochs MAX_EPOCHS
                        Limits training to a max number number of epochs
  --min_steps MIN_STEPS
                        Limits training to a minimum number number of steps
  --max_steps MAX_STEPS
                        Limits training to a max number number of steps
  --accumulate_grad_batches ACCUMULATE_GRAD_BATCHES
                        Accumulates grads every k batches or as set up in the
                        dict.
  --check_val_every_n_epoch CHECK_VAL_EVERY_N_EPOCH
                        Check val every n train epochs.
  --gpus GPUS           Number of GPUs to train on or Which GPUs to train on.
                        (default: -1 (all gpus))
  --gradient_clip_val GRADIENT_CLIP_VAL
                        Gradient clipping value
  --overfit_pct OVERFIT_PCT
                        Uses this much data of all datasets (training,
                        validation, test). Useful for quickly debugging or
                        trying to overfit on purpose.
  --amp_level AMP_LEVEL
                        The optimization level to use (O1, O2, etc…) for
                        16-bit GPU precision (using NVIDIA apex under the
                        hood).
  --precision PRECISION
                        Full precision (32), half precision (16). Can be used
                        on CPU, GPU or TPUs.
  --profiler            To profile individual steps during training and assist
                        in identifying bottlenecks.
  --progress_bar_refresh_rate PROGRESS_BAR_REFRESH_RATE
                        How often to refresh progress bar (in steps). In
                        notebooks, faster refresh rates (lower number) is
                        known to crash them because of their screen refresh
                        rates, so raise it to 50 or more.
  --num_sanity_val_steps NUM_SANITY_VAL_STEPS
                        Sanity check runs n batches of val before starting the
                        training routine. This catches any bugs in your
                        validation without having to wait for the first
                        validation check.
  --use_logger {tensorboard,wandb}
                        Which program to use for logging. If
                        `--use_custom_checkpoint_callback` is specified and
                        `wandb` is chosen then model weights will
                        automatically be uploaded to wandb.ai.
  --do_train            Run the training procedure.
  --do_test             Run the testing procedure.
  --use_custom_checkpoint_callback
                        Use the custom checkpointing callback specified in
                        main() by `args.checkpoint_callback`. By default this
                        custom callback saves the model every epoch and never
                        deletes and saved weights files. Set this option and
                        `--use_logger` to `wandb` to automatically upload
                        model weights to wandb.ai.
  -l {DEBUG,INFO,WARNING,ERROR,CRITICAL}, --log {DEBUG,INFO,WARNING,ERROR,CRITICAL}
                        Set the logging level (default: 'Info').
  --model_name_or_path MODEL_NAME_OR_PATH
                        Path to pre-trained model or shortcut name selected in
                        the list: bert-base-uncased, bert-large-uncased, bert-
                        base-cased, bert-large-cased, bert-base-multilingual-
                        uncased, bert-base-multilingual-cased, bert-base-
                        chinese, bert-base-german-cased, bert-large-uncased-
                        whole-word-masking, bert-large-cased-whole-word-
                        masking, bert-large-uncased-whole-word-masking-
                        finetuned-squad, bert-large-cased-whole-word-masking-
                        finetuned-squad, bert-base-cased-finetuned-mrpc, bert-
                        base-german-dbmdz-cased, bert-base-german-dbmdz-
                        uncased, bert-base-japanese, bert-base-japanese-whole-
                        word-masking, bert-base-japanese-char, bert-base-
                        japanese-char-whole-word-masking, bert-base-finnish-
                        cased-v1, bert-base-finnish-uncased-v1, bert-base-
                        dutch-cased, bart-large, bart-large-mnli, bart-large-
                        cnn, bart-large-xsum, openai-gpt, transfo-xl-wt103,
                        gpt2, gpt2-medium, gpt2-large, gpt2-xl, distilgpt2,
                        ctrl, xlnet-base-cased, xlnet-large-cased, xlm-mlm-
                        en-2048, xlm-mlm-ende-1024, xlm-mlm-enfr-1024, xlm-
                        mlm-enro-1024, xlm-mlm-tlm-xnli15-1024, xlm-mlm-
                        xnli15-1024, xlm-clm-enfr-1024, xlm-clm-ende-1024,
                        xlm-mlm-17-1280, xlm-mlm-100-1280, roberta-base,
                        roberta-large, roberta-large-mnli, distilroberta-base,
                        roberta-base-openai-detector, roberta-large-openai-
                        detector, distilbert-base-uncased, distilbert-base-
                        uncased-distilled-squad, distilbert-base-cased,
                        distilbert-base-cased-distilled-squad, distilbert-
                        base-german-cased, distilbert-base-multilingual-cased,
                        distilbert-base-uncased-finetuned-sst-2-english,
                        albert-base-v1, albert-large-v1, albert-xlarge-v1,
                        albert-xxlarge-v1, albert-base-v2, albert-large-v2,
                        albert-xlarge-v2, albert-xxlarge-v2, camembert-base,
                        umberto-commoncrawl-cased-v1, umberto-wikipedia-
                        uncased-v1, t5-small, t5-base, t5-large, t5-3b,
                        t5-11b, flaubert-small-cased, flaubert-base-uncased,
                        flaubert-base-cased, flaubert-large-cased, xlm-
                        roberta-base, xlm-roberta-large, xlm-roberta-large-
                        finetuned-conll02-dutch, xlm-roberta-large-finetuned-
                        conll02-spanish, xlm-roberta-large-finetuned-
                        conll03-english, xlm-roberta-large-finetuned-
                        conll03-german, google/electra-small-generator,
                        google/electra-base-generator, google/electra-large-
                        generator, google/electra-small-discriminator,
                        google/electra-base-discriminator, google/electra-
                        large-discriminator
  --model_type MODEL_TYPE
                        Model type selected in the list: t5, distilbert,
                        albert, camembert, xlm-roberta, bart, roberta, bert,
                        openai-gpt, gpt2, transfo-xl, xlnet, flaubert, xlm,
                        ctrl, electra
  --tokenizer_name TOKENIZER_NAME
  --tokenizer_lowercase
  --max_seq_length MAX_SEQ_LENGTH
  --oracle_mode {none,greedy,combination}
  --data_path DATA_PATH
  --num_threads NUM_THREADS
  --processing_num_threads PROCESSING_NUM_THREADS
  --weight_decay WEIGHT_DECAY
  --pooling_mode {sent_rep_tokens,mean_tokens}
                        How word vectors should be converted to sentence
                        embeddings.
  --web_learning_rate WEB_LEARNING_RATE
                        Word embedding model specific learning rate.
  --adam_epsilon ADAM_EPSILON
                        Epsilon for Adam optimizer.
  --optimizer_type OPTIMIZER_TYPE
                        Which optimizer to use: 1. `ranger` optimizer
                        (combination of RAdam and LookAhead) 2. `adamw` 3.
                        `yellowfin`
  --ranger-k RANGER_K   Ranger (LookAhead) optimizer k value (default: 6).
                        LookAhead keeps a single extra copy of the weights,
                        then lets the internalized ‘faster’ optimizer (for
                        Ranger, that’s RAdam) explore for 5 or 6 batches. The
                        batch interval is specified via the k parameter.
  --warmup_steps WARMUP_STEPS
                        Linear warmup over warmup_steps. Only active if
                        `--use_scheduler` is set.
  --use_scheduler USE_SCHEDULER
                        Two options: 1. `linear`: Use a linear schedule that
                        inceases linearly over `--warmup_steps` to
                        `--learning_rate` then decreases linearly for the rest
                        of the training process. 2. `onecycle`: Use the one
                        cycle policy with a maximum learning rate of
                        `--learning_rate`. (default: False, don't use any
                        scheduler)
  --num_frozen_steps NUM_FROZEN_STEPS
                        Freeze (don't train) the word embedding model for this
                        many steps.
  --train_batch_size TRAIN_BATCH_SIZE
                        Batch size per GPU/CPU for training.
  --val_batch_size VAL_BATCH_SIZE
                        Batch size per GPU/CPU for evaluation.
  --test_batch_size TEST_BATCH_SIZE
                        Batch size per GPU/CPU for testing.
  --processor_no_bert_compatible_cls
                        If model uses bert compatible [CLS] tokens for
                        sentence representations.
  --only_preprocess     Only preprocess and write the data to disk. Don't
                        train model. This will force data to be preprocessed,
                        even if it was already computed and is detected on
                        disk, and any previous processed files will be
                        overwritten.
  --create_token_type_ids {binary,sequential}
                        Create token type ids during preprocessing.
  --no_use_token_type_ids
                        Set to not train with `token_type_ids` (don't pass
                        them into the model).
  --train_name TRAIN_NAME
                        name for set of training files on disk (for loading
                        and saving)
  --val_name VAL_NAME   name for set of validation files on disk (for loading
                        and saving)
  --test_name TEST_NAME
                        name for set of testing files on disk (for loading and
                        saving)
  --test_id_method {greater_k,top_k}
                        How to chose the top predictions from the model for
                        ROUGE scores.
  --test_k TEST_K       The `k` parameter for the `--test_id_method`. Must be
                        set if using `top_k` option. (default: 0.5)
```

All training arguments can be found in the [pytorch_lightning trainer documentation](https://pytorch-lightning.readthedocs.io/en/latest/trainer.html).

## Experiments

Please see [experiments/README.md](experiments/README.md).

## Meta

Hayden Housen – [haydenhousen.com](https://haydenhousen.com)

Distributed under the GNU General Public License v3.0. See the [LICENSE](LICENSE) for more information.

<https://github.com/HHousen>

## Contributing

1. Fork it (<https://github.com/HHousen/TransformerExtSum/fork>)
2. Create your feature branch (`git checkout -b feature/fooBar`)
3. Commit your changes (`git commit -am 'Add some fooBar'`)
4. Push to the branch (`git push origin feature/fooBar`)
5. Create a new Pull Request
