# Ghi Chu De Hieu Luong Chay Project

File nay giai thich bang tieng Viet:
- de bai yeu cau gi
- `main.py` chay nhu nao
- khi nao data duoc embed vao store
- khi nhap cau hoi tu CLI thi code di vao dau
- OpenAI API key duoc dung o dau
- tung file chinh va tung ham chinh lam gi

---

## 1. De bai yeu cau gi

De bai nam o:
- [README.md](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/README.md)
- [exercises.md](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/exercises.md)

Ban can lam 2 nhom viec:

### 1.1 Phan code

Hoan thanh cac TODO trong:
- [chunking.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/chunking.py)
- [store.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/store.py)
- [agent.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/agent.py)

Muc tieu:
- `pytest tests/ -v` pass

### 1.2 Phan data va report

Ban can:
- chuan bi data trong `data/`
- gan metadata
- tao benchmark queries
- dien [REPORT.md](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/report/REPORT.md)

Domain hien tai:
- `Vinlex -> chatbot cho sinh vien tra cuu chinh sach VinUni`

---

## 2. Ban can hieu RAG o project nay thanh 2 pha

Day la phan ban dang hoi, va no rat quan trong.

Project nay co 2 pha:

### Pha A. Nap data vao vector store

Muc dich:
- doc tai lieu
- embed tai lieu
- luu vector vao store

### Pha B. Hoi dap

Muc dich:
- nhan cau hoi
- embed cau hoi
- tim tai lieu/chunk lien quan
- dua context cho agent
- sinh cau tra loi

Tom gon:

```text
Pha A: data -> embedding -> store
Pha B: question -> embedding -> search -> context -> answer
```

---

## 3. `main.py` co tu dong embed data vao store khong

Co.

Moi lan ban chay:

```bash
python3 main.py
```

hoac:

```bash
python3 main.py "Cau hoi cua ban"
```

thi `main.py` se:

1. doc cac file trong `data/`
2. tao `Document`
3. chon embedder
4. tao `EmbeddingStore`
5. goi `store.add_documents(docs)`
6. luc nay data duoc embed va luu vao store
7. sau do moi search/query
8. roi agent moi answer

Dieu quan trong:
- store hien tai la in-memory
- nghia la no chi song trong luc chuong trinh dang chay
- moi lan chay `main.py`, no nap lai va embed lai tu dau

No khong phai dang:
- save vector xuong database that tren dia
- roi lan sau mo len dung tiep

Hien tai no dang:
- doc file
- embed
- luu vao RAM
- query
- ket thuc chuong trinh thi du lieu trong store mat

---

## 4. `main.py` chay tu dau den dau nhu nao

File chinh la:
- [main.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/main.py)

Luong chay day du:

```text
python3 main.py "question"
-> main()
-> run_manual_demo(question)
-> load_documents_from_files(files)
-> tao list Document
-> chon embedder
-> EmbeddingStore(...)
-> store.add_documents(docs)
-> store.search(query)
-> KnowledgeBaseAgent(store, demo_llm)
-> agent.answer(query)
-> in ket qua ra terminal
```

Hay tach no ra thanh 2 nua:

### 4.1 Nua dau: nap data

```text
run_manual_demo()
-> load_documents_from_files()
-> docs
-> store = EmbeddingStore(...)
-> store.add_documents(docs)
```

### 4.2 Nua sau: hoi dap

```text
run_manual_demo()
-> query
-> store.search(query)
-> agent = KnowledgeBaseAgent(...)
-> agent.answer(query)
```

---

## 5. Khi nhap cau hoi tu CLI thi code chay vao dau

Vi du ban chay:

```bash
python3 main.py "Sinh vien co the phuc khao diem trong bao lau?"
```

thi code di theo duong nay:

### Buoc 1. `main()` nhan cau hoi tu command line

Trong [main.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/main.py):

```python
question = " ".join(sys.argv[1:]).strip() if len(sys.argv) > 1 else None
```

Nghia la:
- lay toan bo phan sau `python3 main.py`
- ghep thanh 1 chuoi
- dua vao bien `question`

Sau do:

```python
return run_manual_demo(question=question)
```

### Buoc 2. `run_manual_demo()` nhan cau hoi

No dat:

```python
query = question or "Summarize the key information from the loaded files."
```

Neu ban co truyen cau hoi:
- dung cau hoi do

Neu khong truyen:
- no dung cau hoi mac dinh

### Buoc 3. Search truc tiep trong store

Trong `run_manual_demo()`:

```python
search_results = store.search(query, top_k=3)
```

Luc nay code di vao:
- [store.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/store.py)
- ham `EmbeddingStore.search()`

Ben trong `search()`:
- query duoc embed
- query vector duoc so voi vector cua cac document da luu
- lay top 3 ket qua cao nhat

### Buoc 4. Agent tra loi

Sau khi search xong, `main.py` tao agent:

```python
agent = KnowledgeBaseAgent(store=store, llm_fn=demo_llm)
```

Sau do:

```python
agent.answer(query, top_k=3)
```

Luc nay code di vao:
- [agent.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/agent.py)
- ham `KnowledgeBaseAgent.answer()`

Ben trong `answer()`:
- no lai goi `self.store.search(question, top_k=top_k)`
- lay cac chunk lien quan
- ghep thanh `context`
- tao `prompt`
- dua `prompt` vao `llm_fn`

O project hien tai:
- `llm_fn` dang la `demo_llm`
- nghia la cau tra loi agent hien tai la cau tra loi demo, khong phai goi LLM that

---

## 6. OpenAI API key dang duoc dung o dau

Ban noi ban co dung OpenAI API key, vay can phan biet ro:

### 6.1 OpenAI key hien tai co the dung cho embedding

Trong [embeddings.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/embeddings.py) co:
- `OpenAIEmbedder`

Neu ban dat env:

```bash
export EMBEDDING_PROVIDER=openai
export OPENAI_API_KEY=your_key
```

thi trong `main.py` doan nay se chay:

```python
provider = os.getenv(EMBEDDING_PROVIDER_ENV, "mock").strip().lower()
```

Neu `provider == "openai"`:
- tao `OpenAIEmbedder(...)`
- dung OpenAI API de embed document
- dung OpenAI API de embed query

### 6.2 Nhung `main.py` hien tai CHUA dung OpenAI de sinh cau tra loi

Day la diem rat de nham.

Trong `main.py`, agent dang duoc tao bang:

```python
agent = KnowledgeBaseAgent(store=store, llm_fn=demo_llm)
```

Tuc la:
- embedding co the dung OpenAI
- nhung answer cua agent van dang dung `demo_llm`

No khong dang:
- goi Chat Completions
- goi Responses API
- goi GPT that de tao answer

Nghia la:

```text
OpenAI key hien tai chi dung cho embedding neu ban bat provider=openai.
Cau tra loi cuoi cung cua agent van la demo_llm.
```

---

## 7. Giai thich bang so do cuc de

### 7.1 Khi chay `main.py`

```text
python3 main.py "query"
    |
    v
main()
    |
    v
run_manual_demo()
    |
    +--> load_documents_from_files()
    |       |
    |       v
    |    list[Document]
    |
    +--> chon embedder
    |
    +--> store = EmbeddingStore(...)
    |
    +--> store.add_documents(docs)
    |       |
    |       v
    |    document duoc embed va luu vao RAM
    |
    +--> store.search(query)
    |       |
    |       v
    |    lay top-k ket qua
    |
    +--> agent = KnowledgeBaseAgent(store, demo_llm)
    |
    +--> agent.answer(query)
            |
            v
         store.search(query)
            |
            v
         ghep context
            |
            v
         demo_llm(prompt)
            |
            v
         in answer
```

### 7.2 Hai pha RAG trong code hien tai

```text
Pha 1: Index
Document -> embedding -> self._store

Pha 2: Query
question -> embedding -> similarity search -> top-k -> prompt -> answer
```

---

## 8. Tung file chinh va cac ham chinh

## 8.1 [main.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/main.py)

### `load_documents_from_files(file_paths)`

Lam gi:
- doc file `.md` va `.txt`
- tao `Document`

Output:
- danh sach `Document`

### `demo_llm(prompt)`

Lam gi:
- tao cau tra loi demo
- khong goi model that

Dung de:
- test luong agent ma khong can API LLM that

### `run_manual_demo(question=None, sample_files=None)`

Lam gi:
- day la ham chay end-to-end
- no vua nap data, vua search, vua goi agent

Day la ham quan trong nhat cua `main.py`.

### `main()`

Lam gi:
- lay query tu CLI
- goi `run_manual_demo()`

---

## 8.2 [models.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/models.py)

### `Document`

Lam gi:
- luu 1 tai lieu

Gom:
- `id`
- `content`
- `metadata`

---

## 8.3 [embeddings.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/embeddings.py)

### `MockEmbedder`

Lam gi:
- embed gia de test

### `LocalEmbedder`

Lam gi:
- embed bang model local

### `OpenAIEmbedder`

Lam gi:
- embed bang OpenAI embeddings API

Quan trong:
- file nay chi lo phan embedding
- khong lo phan sinh answer cua chatbot

---

## 8.4 [store.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/store.py)

Day la noi dong vai tro "vector db" trong project hien tai.

### `EmbeddingStore.__init__(...)`

Lam gi:
- khoi tao store
- nhan `embedding_fn`
- tao `self._store`

Quan trong:
- `self._store` la noi dang luu vector trong RAM

### `EmbeddingStore._make_record(doc)`

Lam gi:
- chuyen `Document` thanh 1 record luu trong store

Record gom:
- `id`
- `content`
- `metadata`
- `embedding`

### `EmbeddingStore.add_documents(docs)`

Lam gi:
- embed tung document
- luu tung record vao `self._store`

Day chinh la buoc "nap data vao vector store".

### `EmbeddingStore._search_records(query, records, top_k)`

Lam gi:
- embed query
- tinh score query voi tung record
- sort
- lay top-k

### `EmbeddingStore.search(query, top_k=5)`

Lam gi:
- ham search chinh
- goi logic tim cac record lien quan nhat

Day chinh la buoc "retrieve".

### `EmbeddingStore.search_with_filter(...)`

Lam gi:
- filter metadata truoc
- roi moi search

### `EmbeddingStore.get_collection_size()`

Lam gi:
- dem so record trong store

### `EmbeddingStore.delete_document(doc_id)`

Lam gi:
- xoa record theo `doc_id`

---

## 8.5 [agent.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/src/agent.py)

### `KnowledgeBaseAgent.__init__(store, llm_fn)`

Lam gi:
- nhan store
- nhan ham sinh answer

### `KnowledgeBaseAgent.answer(question, top_k=3)`

Lam gi:
- search lai trong store
- lay top-k context
- ghep prompt
- goi `llm_fn(prompt)`
- tra ve answer

Quan trong:
- day la phan "AG" trong "RAG"
- "R" la retrieval tu store
- "AG" la dung context de tra loi

---

## 8.6 [test_solution.py](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/tests/test_solution.py)

File nay lam gi:
- test code trong `src`

No khong nap `main.py`.
No se:
- import `src`
- goi tung class / ham
- check ket qua

Nghia la:
- `pytest` la de cham solution
- `main.py` la de demo luong chay

---

## 9. Minh da lam gi roi

Da lam:
- chuyen 4 file PDF sang `.md`
- xoa `data5` vi bi trung
- tao [metadata.json](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/data/metadata.json)
- tao [benchmark_queries.md](/home/hung/code/AI_CODE_VIN/lab/day7/2A202600465-DongManhHung-Day07/data/benchmark_queries.md)
- implement xong TODO trong `src/`
- chay test pass

Ket qua test:

```text
42 passed
```

---

## 10. Tinh trang that cua project hien tai

Hay nho dung 3 y nay:

### Y 1. `main.py` moi lan chay deu nap data lai

Nghia la:
- no khong doc vector da luu san
- no doc file text lai tu dau
- embed lai tu dau
- luu lai vao RAM

### Y 2. Store hien tai la in-memory

Nghia la:
- vector dang luu trong bien `self._store`
- chua phai vector database persistent that

### Y 3. OpenAI key hien tai chi anh huong toi embedding neu ban bat provider=openai

Nhung:
- answer cuoi cung cua agent van dang dung `demo_llm`
- chua phai GPT that

---

## 11. Lenh can nho

### Chay test

```bash
python3 -m pytest -q tests
```

### Chay demo

```bash
python3 main.py
```

### Chay demo voi query

```bash
python3 main.py "Sinh vien co the phuc khao diem trong bao lau?"
```

### Dung OpenAI cho embedding

```bash
export OPENAI_API_KEY=your_key
export EMBEDDING_PROVIDER=openai
python3 main.py "Sinh vien co the phuc khao diem trong bao lau?"
```

---

## 12. Tom tat cuc ngan

Neu chi nho 1 dieu:

```text
main.py moi lan chay se:
doc data -> embed data -> luu vao store trong RAM -> nhan query -> embed query -> search -> agent answer
```

Va hien tai:

```text
OpenAI co the dang duoc dung cho embedding.
Answer cua agent van la demo_llm, chua phai LLM that.
```
