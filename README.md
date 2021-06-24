# Giới thiệu

DrQA là một hệ thống cho việc đọc hiểu được áp dụng để trả lời câu hỏi mã nguồn mở. Cụ thể, DrQA nhằm mục đích giải quyết nhiệm vụ đọc máy quy mô lớn. Trong phần cài đặt, chúng tôi tìm kiếm câu trả lời của một câu hỏi trong một kho tài liệu có khả năng cao của các tài liệu phi cấu trúc. Vì thế, hệ thống phải kết hợp cả thách thức trích xuất và hiểu máy một đoạn văn bản.

Thực nghiệm với DrQA tập trung vào việc trả lời các câu hỏi thực tế trong khi sử dụng Wikipedia như nguồn kiến thức duy nhất cho các tài liệu. Wikipedia là một nguồn phong phú, chi tiết, phù hợp quy mô lớn. Để trả lời bất kỳ câu hỏi nào, đầu tiên phải trích xuất các bài báo chung trong hơn 5 triệu bài, và sau đó quét chúng cẩn thẩn, để xác định câu trả lời.

Lưu ý rằng DrQA coi Wikipeadia như tập hợp các bài báo chung và không dựa vào cấu trúc đồ thị bên trong. Do đó, **DrQA có thể được áp dụng dễ dàng cho bất kỳ bộ sưu tập tài liệu nào**. Trong phần trích xuất tài liệu README có mô tả chi tiết.

Kho tài liệu này bao gồm code, dữ liệu và mô hình tiền huấn luyện cho tiền xử lý và truy vấn Wikipedia như được mô tả trong bài báo -- Xem Mô hình được train và dữ liệu. Chúng tôi cũng liệt kê một số bộ dữ liệu khác nhau cho việc đánh giá, xem phần các bộ dữ liệu QA. Lưu ý rằng công việc này là được cấu trúc lại và hiệu quả hơn phiên bản code gốc. Các con số sản phẩm rất giống nhau nhưng không hoàn toàn.

## Quick Start: Demo

[Install](#installing-drqa) DrQA và [download](#trained-models-and-data) các mô hình của chúng tôi để bắt đầu hỏi các câu hỏi miền mở!

Chạy `python scripts/pipeline/interactive.py` để tham gia một phiên tương tác. Đối với mỗi câu hỏi, khoảng trên cùng và đoạn Wikipedia được trả về.

```
>>> process('What is question answering?')

Top Predictions:
+------+----------------------------------------------------------------------------------------------------------+--------------------+--------------+-----------+
| Rank |                                                  Answer                                                  |        Doc         | Answer Score | Doc Score |
+------+----------------------------------------------------------------------------------------------------------+--------------------+--------------+-----------+
|  1   | a computer science discipline within the fields of information retrieval and natural language processing | Question answering |    1917.8    |   327.89  |
+------+----------------------------------------------------------------------------------------------------------+--------------------+--------------+-----------+

Contexts:
[ Doc = Question answering ]
Question Answering (QA) is a computer science discipline within the fields of
information retrieval and natural language processing (NLP), which is
concerned with building systems that automatically answer questions posed by
humans in a natural language.
```

```
>>> process('What is the answer to life, the universe, and everything?')

Top Predictions:
+------+--------+---------------------------------------------------+--------------+-----------+
| Rank | Answer |                        Doc                        | Answer Score | Doc Score |
+------+--------+---------------------------------------------------+--------------+-----------+
|  1   |   42   | Phrases from The Hitchhiker's Guide to the Galaxy |    47242     |   141.26  |
+------+--------+---------------------------------------------------+--------------+-----------+

Contexts:
[ Doc = Phrases from The Hitchhiker's Guide to the Galaxy ]
The number 42 and the phrase, "Life, the universe, and everything" have
attained cult status on the Internet. "Life, the universe, and everything" is
a common name for the off-topic section of an Internet forum and the phrase is
invoked in similar ways to mean "anything at all". Many chatbots, when asked
about the meaning of life, will answer "42". Several online calculators are
also programmed with the Question. Google Calculator will give the result to
"the answer to life the universe and everything" as 42, as will Wolfram's
Computational Knowledge Engine. Similarly, DuckDuckGo also gives the result of
"the answer to the ultimate question of life, the universe and everything" as
42. In the online community Second Life, there is a section on a sim called
43. "42nd Life." It is devoted to this concept in the book series, and several
attempts at recreating Milliways, the Restaurant at the End of the Universe, were made.
```

```
>>> process('Who was the winning pitcher in the 1956 World Series?')

Top Predictions:
+------+------------+------------------+--------------+-----------+
| Rank |   Answer   |       Doc        | Answer Score | Doc Score |
+------+------------+------------------+--------------+-----------+
|  1   | Don Larsen | New York Yankees |  4.5059e+06  |   278.06  |
+------+------------+------------------+--------------+-----------+

Contexts:
[ Doc = New York Yankees ]
In 1954, the Yankees won over 100 games, but the Indians took the pennant with
an AL record 111 wins; 1954 was famously referred to as "The Year the Yankees
Lost the Pennant". In , the Dodgers finally beat the Yankees in the World
Series, after five previous Series losses to them, but the Yankees came back
strong the next year. On October 8, 1956, in Game Five of the 1956 World
Series against the Dodgers, pitcher Don Larsen threw the only perfect game in
World Series history, which remains the only perfect game in postseason play
and was the only no-hitter of any kind to be pitched in postseason play until
Roy Halladay pitched a no-hitter on October 6, 2010.
```

Hãy tự mình thử nó. Tất nhiên, DrQA có thể cung cấp các dữ kiện thay thế, vì vậy hãy tận hưởng chuyến đi.

## Cài đặt drqa

DrQA yêu cầu sử dụng Linux/OSX và python 3.5 trở lên. Nó cũng yêu cầu PyTorch phiên bản 1.0. Nó dựa trên các thư viện khác được liệt kê trong file requirements.txt. CUDA là được yêu cầu mạnh mẽ cho tốc độ, nhưng không bắt buộc.

Chạy câu lệnh sau đây để clone reposity về và cài đặt DrQA:

```
git clone https://github.com/facebookresearch/DrQA.git
cd DrQA; pip install -r requirements.txt; python setup.py develop
```

Lưu ý: requirements.txt bao gồm một tập con tất cả các gói có thể được yêu cầu. 

Tùy thuộc vào cái bạn muốn chạy, bạn có thể cần cài đặt thêm các gói( ví dụ spacy)

Nếu bạn sử dụng CoreNLP Tokenizer hoặc SpacyTokenizer bạn sẽ cần cài đặt Stanford CoreNLP jars và mô hình spaCy en tương ứng. Nếu bạn dùng Stanford CoreNLP, có jars trong biến môi trường CLASSPATH trong java của bạn, hoặc đặt đường dẫn tự động với:

```
import drqa.tokenizers
drqa.tokenizers.set_default('corenlp_classpath', 'your/corenlp/classpath/*')
```

**Quan trọng: Tokenizer mặc định là CoreNLP nên bạn sẽ cần điều đó trong CLASSPATH để chạy các ví dụ ở phần README**

Vd: ```export CLASSPATH=$CLASSPATH:/path/to/corenlp/download/* ```

Nếu bạn chưa cài đặt CoreNLP thì có thể chạy:
```
./install_corenlp.sh
```

Xác nhận xem nó có chạy không bằng :

```
from drqa.tokenizers import CoreNLPTokenizer
tok = CoreNLPTokenizer()
tok.tokenize('hello world').words() # Should complete immediately
```

Để thuận tiện, mô hình trình đọc tài liệu, trích xuất và Pipeline sẽ cố gắng load các mô hình mặc định nếu không có đối số mô hình được truyền vào. Xem phía dưới cho việc cài đặt các mô hình này.

## Đào tạo mô hình và dữ liệu

Để tải tất cả các mô hình đã đào tạo được cung cấp và dữ liệu cho trả lời câu hỏi Wikipedia, chạy:

```
./download.sh
```
*Cảnh báo: this downloads a 7.5GB tarball (25GB untarred) và sẽ mất một khoảng thời gian.*

Cấu trúc, vị trí tải:



```
DrQA
├── data (or $DRQA_DATA)
    ├── datasets
    │   ├── SQuAD-v1.1-<train/dev>.<txt/json>
    │   ├── WebQuestions-<train/test>.txt
    │   ├── freebase-entities.txt
    │   ├── CuratedTrec-<train/test>.txt
    │   └── WikiMovies-<train/test/entities>.txt
    ├── reader
    │   ├── multitask.mdl
    │   └── single.mdl
    └── wikipedia
        ├── docs.db
        └── docs-tfidf-ngram=2-hash=16777216-tokenizer=simple.npz
```

Các đường dẫn mô hình mặc định cho các mô đun khác nhau cũng có thể được chỉnh sửa tự động trong code, ví dụ:

```
import drqa.reader
drqa.reader.set_default('model', '/path/to/model')
reader = drqa.reader.Predictor() # Mô hình mặc định được tải cho việc dự đoán
```

## Trình trích xuất tài liệu

Mô hình TF-IDF sử dụng Wikipedia( unigrams và bigrams, 2^24 bins, tokenization đơn giản), được đánh giá trên nhiều bộ dữ liệu( bộ test, bộ dev cho SQuAD):

| Model | SQuAD P@5 | CuratedTREC P@5 | WebQuestions P@5 | WikiMovies P@5 | Size |
| :---: | :-------: | :-------------: | :--------------: | :------------: | :---: |
| [TF-IDF model](https://dl.fbaipublicfiles.com/drqa/docs-tfidf-ngram%3D2-hash%3D16777216-tokenizer%3Dsimple.npz.gz) | 78.0 | 87.6 | 75.0 | 69.8 | ~13GB |

P@5 ở đây là được định nghĩa như % các câu hỏi cho các phần câu trả lời xuất hiện trong top5 tài liệu

## Wikipedia

Các thử nghiệm quy mô đầy đủ của chúng tôi được thực hiện trên Wikipedia tiếng Anh 2016-12-21. Kết xuất được xử lý bằng WikiExtractor và được lọc cho các trang nội bộ, danh sách, chỉ mục và phác thảo (các trang thường chỉ là liên kết). Chúng tôi lưu trữ các tài liệu trong cơ sở dữ liệu sqlite mà drqa.retriever.DocDB cung cấp một giao diện.

| Database | Num. Documents | Size |
| :------: | :------------: | :-----------------: |
| [Wikipedia](https://dl.fbaipublicfiles.com/drqa/docs.db.gz) | 5,075,182 | ~13GB |

## QA Datasets

Các bộ dữ liệu được sử dụng cho đào tạo và đánh giá DrQA có thể được tìm thấy ở đây:

- SQuAD: [train](https://rajpurkar.github.io/SQuAD-explorer/dataset/train-v1.1.json), [dev](https://rajpurkar.github.io/SQuAD-explorer/dataset/dev-v1.1.json)
- WebQuestions: [train](http://nlp.stanford.edu/static/software/sempre/release-emnlp2013/lib/data/webquestions/dataset_11/webquestions.examples.train.json.bz2), [test](http://nlp.stanford.edu/static/software/sempre/release-emnlp2013/lib/data/webquestions/dataset_11/webquestions.examples.test.json.bz2), [entities](https://dl.fbaipublicfiles.com/drqa/freebase-entities.txt.gz)
- WikiMovies: [train/test/entities](https://dl.fbaipublicfiles.com/drqa/WikiMovies.tar.gz)
(Rehosted in expected format from https://research.fb.com/downloads/babi/)
- CuratedTrec: [train/test](https://dl.fbaipublicfiles.com/drqa/CuratedTrec.tar.gz)
(Rehosted in expected format from https://github.com/brmson/dataset-factoid-curated)

**Định dạng A**

Các tập lệnh
    retriever/eval.py, pipeline/eval.py
và 
    distant/generate.py

mong đợi các bộ dữ liệu như file .txt mà mỗi dòng là một cặp QA được mã hóa JSON, giống như này:

```python
'{"question": "q1", "answer": ["a11", ..., "a1i"]}'
...
'{"question": "qN", "answer": ["aN1", ..., "aNi"]}'
```

Các tập lệnh để chuyển SQuAD và WebQuestion thành định dạng này là được bao gồm trong
    scripts/convert
Điều này tự động làm trong *download.sh*

**Định dạng B**

Các tập lệnh thư mục *reader* mong đợi các tập dữ liệu dưới dạng tệp .json, nơi dữ liệu được sắp xếp giống SQuAD:

```
file.json
├── "data"
│   └── [i]
│       ├── "paragraphs"
│       │   └── [j]
│       │       ├── "context": "paragraph text"
│       │       └── "qas"
│       │           └── [k]
│       │               ├── "answers"
│       │               │   └── [l]
│       │               │       ├── "answer_start": N
│       │               │       └── "text": "answer"
│       │               ├── "id": "<uuid>"
│       │               └── "question": "paragraph question?"
│       └── "title": "document id"
└── "version": 1.1
```

## Danh sách thực thể
Một số bộ dữ liệu có danh sách ứng viên (có thể lớn) để chọn câu trả lời. Ví dụ: câu trả lời của WikiMovies là các mục nhập OMDb trong khi WebQuestions dựa trên Freebase. Nếu chúng tôi biết các ứng cử viên, chúng tôi có thể áp đặt rằng tất cả các câu trả lời được dự đoán phải nằm trong danh sách này bằng cách loại bỏ bất kỳ khoảng(span) nào mà điểm cao hơn là không có.

# DrQA Components

## Document Retriever

DrQA không bị ràng buộc bởi bất kỳ loại hệ thống truy xuất cụ thể nào -- miễn là nó thu hẹp không gian tìm kiếm một cách hiệu quả và tập trung vào các tài liệu liên quan.

Theo các hệ thống QA cổ điển, chúng tôi bao gồm một hệ thống truy xuất tài liệu dựa trên các vector tú từ có trọng số TF-IDF thưa thớt( không dùng máy học). Chúng tôi dùng thêm các túi được băm n-grams( ở đây là unigrams và bigrams).

Để xem cách xây dựng mô hình như vậy của riêng bạn trên các tài liệu mới, hãy xem README của trình trích xuất.

Để truy vấn Wikipedia có thể tương tác:

```
python scripts/retriever/interactive.py --model /path/to/model
```

Nếu *model* đã "rời khỏi", mô hình mặc định của chúng tôi sẽ được sử dụng( giả sử nó đã được tải).

Để đánh giá tỉ lệ trích xuất( % phù hợp trong top 5) trên bộ dữ liệu:

```
python scripts/retriever/eval.py /path/to/format/A/dataset.txt --model /path/to/model
```

## Document Reader

Trình đọc tài liệu của DrQA là một mô hình hiểu máy mạng thần kinh tái phát nhiều lớp được đào tạo để trả lời câu hỏi khai thác. Đó là, mô hình cố gắng tìm câu trả lời cho bất kỳ câu hỏi nào dưới dạng khoảng văn bản trong một trong các tài liệu được trả về.


Trình đọc Tài liệu được lấy cảm hứng từ và được đào tạo chủ yếu về tập dữ liệu SQuAD. Nó cũng có thể được sử dụng độc lập trên các tác vụ giống SQuAD như vậy, trong đó ngữ cảnh cụ thể được cung cấp cùng với câu hỏi, câu trả lời được chứa trong ngữ cảnh.


Để xem cách đào tạo Trình đọc tài liệu trên SQuAD, hãy xem trình đọc README.


Để tương tác đặt câu hỏi về văn bản với một mô hình được đào tạo: 

```
python scripts/reader/interactive.py --model /path/to/model
```

Một lần nữa, *model* ở đây là tùy chọn; một **model mặc định(có link)** được dùng nếu nó không được chỉ định.

Để chạy mô hình dự đoán trên một bộ dữ liệu:

```
python scripts/reader/predict.py /path/to/format/B/dataset.json --model /path/to/model
```

## DrQA Pipeline

Toàn bộ hệ thống được liên kết cùng nhau trong
    drqa.pipeline.DrQA
Để đặt câu hỏi tương tác sử dụng DrQA đầy đủ:

```
python scripts/pipeline/interactive.py
```

Các đối số tùy chọn:

```
--reader-model    Path to trained Document Reader model.
--retriever-model Path to Document Retriever model (tfidf).
--doc-db          Path to Document DB.
--tokenizer      String option specifying tokenizer type to use (e.g. 'corenlp').
--candidate-file  List of candidates to restrict predictions to, one candidate per line.
--no-cuda         Use CPU only.
--gpu             Specify GPU device id to use.
```

Để chạy dự đoán trên bộ dữ liệu:

```
python scripts/pipeline/predict.py /path/to/format/A/dataset.txt
```

Các đối số tùy chọn:

```
--out-dir             Directory to write prediction file to (<dataset>-<model>-pipeline.preds).
--reader-model        Path to trained Document Reader model.
--retriever-model     Path to Document Retriever model (tfidf).
--doc-db              Path to Document DB.
--embedding-file      Expand dictionary to use all pretrained embeddings in this file (e.g. all glove vectors to minimize UNKs at test time).
--candidate-file      List of candidates to restrict predictions to, one candidate per line.
--n-docs              Number of docs to retrieve per query.
--top-n               Number of predictions to make per query.
--tokenizer           String option specifying tokenizer type to use (e.g. 'corenlp').
--no-cuda             Use CPU only.
--gpu                 Specify GPU device id to use.
--parallel            Use data parallel (split across GPU devices).
--num-workers         Number of CPU processes (for tokenizing, etc).
--batch-size          Document paragraph batching size (Reduce in case of GPU OOM).
--predict-batch-size  Question batching size (Reduce in case of CPU OOM).
```

## Giám sát từ xa (DS)

Hiệu suất của DrQA cải thiện đáng kể trong chế độ cài đặt đầy đủ khi được cung cấp dữ liệu giám sát từ xa từ các bộ dữ liệu bổ sung. Đưa ra các cặp câu hỏi-câu trả lời nhưng không có ngữ cảnh hỗ trợ, chúng ta có thể sử dụng phương pháp heuristics đối sánh chuỗi để tự động liên kết các đoạn văn với các ví dụ đào tạo này.

>Question: What U.S. state’s motto is “Live free or Die”?
>
>Answer: New Hampshire
>
>DS Document: Live Free or Die

Các tập lệnh
    scripts/distant
chứa mã để tạo và kiểm tra dữ liệu được giám sát từ xa như vậy. Chi tiết hơn có thể xem DS README.

## Tokenizers

Chúng tôi cung cấp một số tùy chọn tokenizer khác nhau cho sự thuận tiện. Mỗi cái có ưu nhược điểm dựa trên bao nhiêu thư viện được yêu cầu, chi phí để chạy, tốc độ và hiệu suất. Đối với các thử nghiệm được báo cáo của chúng tôi, chúng tôi đã sử dụng CoreNLP (nhưng tất cả các kết quả đều tương tự). 

Xem các danh sách tùy chọn trong class tokenizers.
