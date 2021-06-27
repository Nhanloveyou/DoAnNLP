# Trình đọc tài liệu

## Tiền xử lý dữ liệu

`preprocess.py` lấy một tập dữ liệu được định dạng SQuAD và xuất ra một tệp đã được tiền xử lý, sẵn sàng cho đào tạo. Đặc biệt, nó xử lý tokenize, ánh xạ các phần bù ký tự thành phần bù mã thông báo, và bất kỳ quá trình trang bị bổ sung nào như lemmazation, POS, và NER.

Để tiền xử lý SQuAD ( giả sử cả các tệp input và output trong `data/datasets`):

```
python scripts/reader/preprocess.py data/datasets data/datasets --split SQuAD-v1.1-train
```

```
python scripts/reader/preprocess.py data/datasets data/datasets --split SQuAD-v1.1-train
```

* bạn cần có SQuAD train-v1.1.json và dev-v1.1.json trong các data/datasets ( ở đây đã được đổi tên thành SQuAD-v1.1-<train/dev>.json)

## Đào tạo

`train.py` là tệp lệnh chính cho trình đọc tài liệu.

Để bắt đầu với việc huấn luyện một mô hình trên SQuAD với các siêu tham số tốt nhất của chúng tôi:

```
python scripts/reader/train.py --embedding-file glove.840B.300d.txt --tune-partial 1000
```

* Bạn cần có [glove embeddings](#note-on-word-embeddings) đã được tải đến data/embeddings/glove.840B.300d.txt.

* Bạn cần phải tiền xử lý trước.

Việc training có nhiều các tùy chọn mà bạn có thể chỉnh:

```
Environment:
--no-cuda           Đào tạo trên CPU, ngay cả khi GPUs có ẵn. (default: False)
--gpu               Chạy trên 1 GPU cụ thể (default: -1)
--data-workers      Số lượng quy trình con cho việc tải dữ liệu (default: 5)
--parallel          Sử dụng DataParallel trên tất cả các GPU có sẵn (default: False)
--random-seed       Hạt giống ngẫu nhiên cho tất cả các hoạt động numpy / torch / cuda (để tái tạo).
--num-epochs        Đào tạo lặp lại dữ liệu.
--batch-size        Batch size cho việc training.
--test-batch-size   Batch size trong quá trình validation/testing.

Filesystem:
--model-dir         Thư mục cho các models/checkpoints/logs đã được lưu (default: /tmp/drqa-models).
--model-name        Mã định dạng mô hình duy nhất (.mdl, .txt, .checkpoint) (default: <generated uuid>).
--data-dir          Thư mục của dữ liệu training/validation (default: data/datasets).
--train-file        Tệp train đã được tiền xử lý (default: SQuAD-v1.1-train-processed-corenlp.txt).
--dev-file          Tệp dev đã được tiền xử lý (default: SQuAD-v1.1-dev-processed-corenlp.txt).
--dev-json          tệp dev chưa được xử lý để chạy đánh giá trong khi đào tạo trên (được sử dụng để lấy văn bản gốc để nhận các span và trả lời văn bản) (default: SQuAD-v1.1-dev.json).
--embed-dir         Thư mục các tệp nhúng được đào tạo trước (default: data/embeddings).
--embedding-file    Tệp nhúng đã được đào tạo trước phân tách bởi dấu cách (default: None).

Saving/Loading:
--checkpoint        Lưu trạng thái model + optimizer sau mỗi epoch (default: False).
--pretrained        Đường dẫn đến mô hình được đào tạo trước để bắt đầu với (default: <empty>).
--expand-dictionary Mở rộng từ điển của mô hình đào tạo trước (--pretrained) để bao gồm các từ của data mới training/dev (default: False).

Preprocessing:
--uncased-question  Question words will be lower-cased (default: False).
--uncased-doc       Document words will be lower-cased (default: False).
--restrict-vocab    Only use pre-trained words in embedding_file (default: True).

General:
--official-eval     Validate with official SQuAD eval (default: True).
--valid-metric      The evaluation metric used for model selection (default: f1).
--display-iter      Log state after every <display_iter> epochs (default: 25).
--sort-by-len       Sort batches by length for speed (default: True).

DrQA Reader Model Architecture:
--model-type        Model architecture type (default: rnn).
--embedding-dim     Embedding size if embedding_file is not given (default: 300).
--hidden-size       Hidden size of RNN units (default: 128).
--doc-layers        Number of encoding layers for document (default: 3).
--question-layers   Number of encoding layers for question (default: 3).
--rnn-type          RNN type: LSTM, GRU, or RNN (default: lstm).

DrQA Reader Model Details:
--concat-rnn-layers Combine hidden states from each encoding layer (default: True).
--question-merge    The way of computing the question representation (default: self_attn).
--use-qemb          Whether to use weighted question embeddings (default: True).
--use-in-question   Whether to use in_question_* (cased, uncased, lemma) features (default: True).
--use-pos           Whether to use pos features (default: True).
--use-ner           Whether to use ner features (default: True).
--use-lemma         Whether to use lemma features (default: True).
--use-tf            Whether to use term frequency features (default: True).

DrQA Reader Optimization:
--dropout-emb           Dropout rate for word embeddings (default: 0.4).
--dropout-rnn           Dropout rate for RNN states (default: 0.4).
--dropout-rnn-output    Whether to dropout the RNN output (default: True).
--optimizer             Optimizer: sgd or adamax (default: adamax).
--learning-rate         Learning rate for SGD only (default: 0.1).
--grad-clipping         Gradient clipping (default: 10).
--weight-decay          Weight decay factor (default: 0).
--momentum              Momentum factor (default: 0).
--fix-embeddings        Keep word embeddings fixed (use pretrained) (default: True).
--tune-partial          Backprop through only the top N question words (default: 0).
--rnn-padding           Explicitly account for padding (and skip it) in RNN encoding (default: False).
--max-len MAX_LEN       The max span allowed during decoding (default: 15).
```

### Lưu ý trên các Word Embedding

Sử dụng các word embedding được đào tạo trước rất quan trọng cho hiệu suất. Các mô hình mà chúng tôi cung cấp đã được đào tạo với cách nhúng GloVe được đào tạo trên Common Crawl, tuy nhiên, chúng tôi cũng nhận thấy rằng các cách nhúng khác như FastText hoạt động khá tốt.

Chúng tôi khuyên bạn nên tải xuống các tệp nhúng và lưu trữ chúng dưới dạng `data/embeddings/<file>.txt` (đây là tệp mặc định cho `--embedding-dir`). Mã mong đợi các tệp văn bản thuần túy được phân tách bằng khoảng trắng (<token> <d1> ... <dN>).

- [GloVe: Common Crawl (cased)](http://nlp.stanford.edu/data/wordvecs/glove.840B.300d.zip)
- [FastText: Wikipedia (uncased)](https://dl.fbaipublicfiles.com/fasttext/vectors-english/wiki-news-300d-1M.vec.zip)

## Dự đoán

`predict.py` sử dụng mô hình Trình đọc tài liệu được đào tạo để đưa ra dự đoán cho tập dữ liệu đầu vào.

Các đối số được yêu cầu:

```
dataset               SQuAD-like dataset to evaluate on (format B).
```

Các đối số tùy chọn:

```
--model             Path to model to use.
--embedding-file    Expand dictionary to use all pretrained embeddings in this file.
--out-dir           Directory to write prediction file to (<dataset>-<model>.preds).
--tokenizer         String option specifying tokenizer type to use (e.g. 'corenlp').
--num-workers       Number of CPU processes (for tokenizing, etc).
--no-cuda           Use CPU only.
--gpu               Specify GPU device id to use.
--batch-size        Example batching size (Reduce in case of OOM).
--top-n             Store top N predicted spans per example.
--official          Only store single top span instead of top N list. (The SQuAD eval script takes a dict of qid: span).
```

*Lưu ý*: Chú thích CoreNLP NER không hoàn toàn xác định( dựa trên thứ tự các ví dụ được tiền xử lý). Các dự đoán có thể dao động rất nhẹ giữa các lần chạy nếu `num-workers` > 1 và mô hình được đào tạo với `use-ner`.

Đánh giá được thực hiện với tập lệnh Official_eval.py từ những người tạo SQuAD, có sẵn tại [đây](https://worksheets.codalab.org/rest/bundles/0xbcd57bee090b421c982906709c8c27e1/contents/blob/). Nó cũng có sẵn theo mặc định tại `scripts/reader/official_eval.py` sau khi chạy `./download.sh.`

```
python scripts/reader/official_eval.py /path/to/format/B/dataset.json /path/to/predictions/with/--official/flag/set.json
```

## Tương tác

Trình đọc tài liệu cũng có thể được sử dụng một cách tương tác( giống như [full pipeline](../../README.md#quick-start-demo)).

```bash
python scripts/reader/interactive.py --model /path/to/model
```

```
>>> text = "Mary had a little lamb, whose fleece was white as snow. And everywhere that Mary went the lamb was sure to go."
>>> question = "What color is Mary's lamb?"
>>> process(text, question)

+------+-------+---------+
| Rank |  Span |  Score  |
+------+-------+---------+
|  1   | white | 0.78002 |
+------+-------+---------+
```
