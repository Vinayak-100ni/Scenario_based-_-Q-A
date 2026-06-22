# Stage 1 - Build dependencies
# Stage 2 - Runtime

```
FROM python:3.12-slim AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

FROM python:3.12-slim
WORKDIR /app

COPY --from=builder /usr/local /usr/local
COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
