# 🚀 ProcedereMQ - SDK Documentation

Seja bem-vindo à documentação oficial dos SDKs do **ProcedereMQ**. Este guia fornece tudo o que você precisa para integrar suas aplicações utilizando nossas bibliotecas em **Python**, **Go** e **Node.js**.

---

## 📖 1. Conceitos Fundamentais

O ProcedereMQ é um sistema de mensageria focado em simplicidade e performance.
- **Queue (Fila):** Onde as mensagens (Jobs) vivem até serem consumidas.
- **Job:** A unidade de trabalho (Payload JSON).
- **Visibility Timeout:** O tempo que um job fica "escondido" de outros consumidores após ser lido.
- **DLQ (Dead Letter Queue):** Fila automática para jobs que falharam após X tentativas.

---

## 🔑 2. Autenticação

Todos os SDKs suportam as seguintes formas de autenticação:
1. **Basic Auth:** Usuário e Senha (recomendado para uso interno/servidores).
2. **API Key:** Token de portador (Bearer Token).

---

## 🐍 3. Python SDK

### Instalação
```bash
pip install procederemq-sdk 
```

### Uso Básico
```python
from python_sdk.client import LiteMQClient

# Inicialização
client = LiteMQClient(
    base_url="http://localhost:65090",
    username="admin",
    password="password"
)

# 📢 Produtor: Enviando mensagens
producer = client.producer("minha.fila")
producer.publish({"status": "check", "value": 100})

# 🕒 Job Agendado (Delayed)
from datetime import datetime, timedelta
run_at = datetime.utcnow() + timedelta(minutes=5)
producer.publish({"tipo": "agendado"}, run_at=run_at)

# 🎧 Consumidor: Processando mensagens
consumer = client.consumer("minha.fila")
try:
    job = consumer.receive()
    print(f"Processando: {job['payload']}")
    consumer.ack(job['id']) # Sucesso!
except Exception as e:
    consumer.fail(job['id']) # Falha: Re-enfileira ou vai para DLQ
```

---

## 🐹 4. Go SDK

### Instalação
```bash
go get github.com/jeffersontadeuleite/procedereMQ/pkg/sdk
```

### Uso Básico
```go
import "github.com/jeffersontadeuleite/procedereMQ/pkg/sdk"

client := sdk.NewClientWithOptions("http://localhost:65090", sdk.ClientOptions{
    Username: "admin",
    Password: "password",
})

ctx := context.Background()

// 📢 Produtor
job, err := client.Enqueue(ctx, sdk.EnqueueRequest{
    Queue:   "reports.gen",
    Payload: map[string]string{"id": "123"},
})

// 🎧 Consumidor
received, err := client.Dequeue(ctx, "reports.gen")
if err == nil {
    // Processamento...
    client.Ack(ctx, received.ID)
}
```

---

## 🟢 5. Node.js SDK

### Instalação
```bash
npm install procederemq-sdk 
```

### Uso Básico
```javascript
const { LiteMQClient } = require('./node_sdk');

const client = new LiteMQClient('http://localhost:65090', {
  username: 'admin',
  password: 'password'
});

// 📢 Produtor
const job = await client.enqueue('emails', { to: 'user@example.com' });

// 🎧 Consumidor
const consumer = client.consumer('emails');
const work = await consumer.receive();
await consumer.ack(work.id);
```

---

## 🛠 6. Recursos Avançados (Enterprise)

### Purge (Limpeza Total)
Remove todas as mensagens de uma fila instantaneamente.
- **Python:** `client.purge("fila")`
- **Go:** `client.Purge(ctx, "fila")`
- **Node:** `await client.purge("fila")`

### Redrive (Recuperação de DLQ)
Move mensagens da DLQ de volta para a fila principal para reprocessamento.
- **Go:** `count, err := client.Redrive(ctx, "fila")`
- **Python:** `client.redrive("fila")`

### Delete Job
Exclui um job específico pelo ID, esteja ele pendente ou em processamento.
- **Node:** `await client.deleteJob("job-id")`

---

## 📊 7. Observabilidade

O ProcedereMQ fornece um Dashboard embutido pronto para uso:
- **URL:** `http://localhost:65090/ui`
- **Recursos:**
    - Monitoramento de métricas em tempo real (Goroutines, Memória, CPU).
    - Inspector de Jobs (Ver payload e erros).
    - Ações manuais de Purge, Ack e Fail.

---
© 2026 ProcedereMQ Enterprise. Documentação gerada pela Antigravity AI.
