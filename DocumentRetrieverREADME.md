# Trình truy xuất tài liệu

## Lưu trữ các tài liệu

Để lưu trữ và truy cập tài liệu một cách hiệu quả, chúng tôi lưu trữ chúng trong cơ sở dữ liệu sqlite. Khóa là `doc_id` và giá trị là` văn bản`.

Để tạo một db sqlite từ một kho tài liệu, hãy chạy:

```bash
python build_db.py /path/to/data /path/to/saved/db.db
```

Các đối số tùy chọn:
```
--preprocess    File path to a python module that defines a `preprocess` function.
--num-workers   Number of CPU processes (for tokenizing, etc).
```

Đường dẫn dữ liệu có thể là đường dẫn đến một thư mục lồng nhau của các tệp( chẳng hạn như những gì mà tập lệnh [WikiExtractor](https://github.com/attardi/wikiextractor) xuất ra) hoặc một tệp duy nhất. Mỗi tệp phải bao gồm các tài liệu được mã hóa JSON có các trường `id` và` text`, mỗi trường một dòng: 

```python
{"id": "doc1", "text": "text of doc1"}
...
{"id": "docN", "text": "text of docN"}
```

`--preprocess /path/to/.py/file` là một đối số tùy chọn khác cho phép bạn cung cấp mô đun python xác định một hàm `preprocess(doc_object)` để lọc/xử lý tài liệu trước khi chúng được đưa vào db. Xem `prep_wikipedia.py` để biết ví dụ.

## Xây dựng TF-IDF N-grams

Để tạo ma trận word-doc có trọng số TF-IDF từ các tài liệu được lưu trữ trong sqlite db, hãy chạy: 

```bash
python build_tfidf.py /path/to/doc/db /path/to/output/dir
```

Các đối số tùy chọn:
```
--ngram        Sử dụng tối đa n-gam kích thước N (ví dụ: 2 = unigram + bigram). Theo mặc định, chỉ những ngrams không có từ dừng hoặc dấu chấm câu được giữ lại.
--hash-size     Số lượng bucket để sử dụng cho băm ngram.
--tokenizer    Tùy chọn chuỗi chỉ định loại tokenizer để sử dụng (ví dụ: 'corenlp').
--num-workers   Số lượng quy trình CPU (để mã hóa, v.v.).
```

Ma trận thưa thớt và metadata liên quan của nó sẽ được lưu vào thư mục đầu ra trong `<db-name>-tfidf-ngram=<N>-hash=<N>-tokenizer=<T>.npz`.

## Tương tác

Trình truy xuất tài liệu cũng có thể được sử dụng tương tác( như [full pipeline](../../README.md#quick-start-demo)).

```bash
python scripts/retriever/interactive.py --model /path/to/model
```

```
>>> process('question answering', k=5)

+------+-------------------------------+-----------+
| Rank |             Doc Id            | Doc Score |
+------+-------------------------------+-----------+
|  1   |       Question answering      |   327.89  |
|  2   |       Watson (computer)       |   217.26  |
|  3   |          Eric Nyberg          |   214.36  |
|  4   |   Social information seeking  |   212.63  |
|  5   | Language Computer Corporation |   184.64  |
+------+-------------------------------+-----------+
``` 
