Repro for the bug described here https://vespatalk.slack.com/archives/C01QNBPPNT1/p1700581153179079?thread_ts=1700570336.591659&cid=C01QNBPPNT1

## 1. Start vespa
```
docker run --detach --name vespa-repro --hostname vespa-container \
  --publish 8080:8080 --publish 19071:19071 \
  vespaengine/vespa
```
## 2. Deploy `vespa deploy --wait 300 .`
## 3. Add a document
```
curl -H 'Content-Type: application/json' --data '{"fields": {"id": "1", "chunks": ["a"], "embeddings": {"1": [1]}}}'   http://localhost:8080//document/v1/repro/repro/docid/1
```
## 4. Querying fails
```
~/c/vespa-repro ❯❯❯ vespa query 'select * from repro where true'
Error: invalid query: 400 Bad Request
{
    "root": {
        "id": "toplevel",
        "relevance": 1.0,
        "fields": {
            "totalCount": 0
        },
        "errors": [
            {
                "code": 4,
                "summary": "Invalid query parameter",
                "source": "repro",
                "message": "'closest(embeddings)' must be of type tensor(), not tensor<bfloat16>(i{})"
            }
        ]
    }
}
```
## 5. No error
The query works as expected if you do either of these:
- Change the cross-encoder re-ranking to `second-phase` instead of `global-phase`
- Add `match-features: best_input`
```
~/c/vespa-repro ❯❯❯ vespa query 'select * from repro where true'
{
    "root": {
        "id": "toplevel",
        "relevance": 1.0,
        "fields": {
            "totalCount": 1
        },
        "coverage": {
            "coverage": 100,
            "documents": 1,
            "full": true,
            "nodes": 1,
            "results": 1,
            "resultsFull": 1
        },
        "children": [
            {
                "id": "id:repro:repro::1",
                "relevance": -3.4893136024475098,
                "source": "repro",
....
```
