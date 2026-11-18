# Agentes de IA com MongoDB: ferramentas e memória

Projetos desenvolvidos no **AI Builder Day Brasília** (MongoDB, 18/11/2026). São dois labs práticos sobre como construir agentes de IA usando o MongoDB como base de dados e de memória.

| Notebook | Lab | Tema |
|---|---|---|
| [`ai-agents-lab.ipynb`](ai-agents-lab.ipynb) | The A to Z of Building AI Agents | Agente com ferramentas, aprovação humana e memória (LangGraph) |
| [`agent-memory-lab.ipynb`](agent-memory-lab.ipynb) | Designing Memory Systems for AI Agents | Sistema de memória de curto e longo prazo, sem framework |

---

## 1. Agente de viagens com LangGraph (`ai-agents-lab.ipynb`)

Um **agente de viagens** que conversa com o usuário, consulta voos e hospedagens em um banco MongoDB, faz reservas (pedindo aprovação antes) e lembra das preferências de cada usuário entre conversas.

- **Consulta o banco sozinho:** lista as coleções, lê o esquema e monta as consultas (MongoDB Database Toolkit do LangChain).
- **Cria reservas** com uma ferramenta própria (`create_booking`), que grava na coleção `bookings`.
- **Human-in-the-loop:** antes de gravar qualquer reserva, o agente pausa (`interrupt`) e pede aprovação do usuário.
- **Memória de curto prazo:** guarda o histórico de cada conversa (thread) no MongoDB com o `MongoDBSaver`.
- **Memória de longo prazo:** guarda preferências por usuário com o `MongoDBStore`, usando embeddings da Voyage AI e busca vetorial. As memórias não vazam entre usuários.
- **MCP:** mostra como obter ferramentas do MongoDB MCP Server.

```
START → agent ──(sem ferramentas)──→ END
          │
          ├──(consulta)──→ tools ──→ agent
          │
          └──(reserva)──→ confirm ──→ tools ──→ agent
```

> Este notebook foi remontado a partir do código do lab, por isso está sem as saídas das células.

## 2. Sistema de memória para um assistente de programação (`agent-memory-lab.ipynb`)

Um **assistente de programação** com três tipos de memória, construído "na mão" (sem LangGraph), para mostrar como cada peça funciona.

| Tipo de memória | O que guarda | Onde fica | Como é buscada |
|---|---|---|---|
| **Curto prazo** | Histórico da conversa atual | Coleção `chats` (por `session_id`) | `find()` ordenado por horário |
| **Semântica** (longo prazo) | Fatos e preferências do usuário | Coleção `preferences` (por `user_id`) | `find()` por usuário, mais recentes primeiro |
| **Procedural** (longo prazo) | Passo a passo de tarefas já resolvidas | Coleção `procedures`, com embeddings | `$vectorSearch` (busca vetorial) |

Pontos principais:
- **Índices:** índice em `session_id`, **TTL** (os documentos expiram sozinhos depois de 1 ano) e índice vetorial de 1024 dimensões (similaridade cosseno).
- **Bloco de notas:** durante a tarefa, o agente anota os passos num arquivo (`write_notes`). No fim, uma IA transforma as notas num **procedimento** reutilizável (`save_procedure`), com saída em JSON Schema, e salva com embedding da Voyage AI.
- **Prompt dinâmico:** o prompt de sistema mostra **quanto da janela de contexto já foi usado** e manda salvar um procedimento quando passar de 70%.
- **Loop do agente:** a IA decide quais ferramentas chamar, o código executa e devolve o resultado, até a resposta final.
- **Teste entre sessões:** numa sessão nova, o histórico da conversa começa vazio, mas o agente ainda lembra das preferências salvas (MongoDB e Voyage AI).

---

## Tecnologias

Python, MongoDB Atlas (incluindo Vector Search e índices TTL), LangChain, LangGraph, Voyage AI (`voyage-4`), MongoDB MCP Server.

## Observações

- Os notebooks foram feitos para o ambiente do workshop: leem `MONGODB_URI`, `PASSKEY` e `PROXY_ENDPOINT` de variáveis de ambiente e usam um módulo `utils` fornecido pelo lab, que não está neste repositório.
- Para rodar fora do workshop, use a string de conexão do seu cluster Atlas, sua própria chave da Voyage AI e o modelo de IA de sua preferência.
- Nenhuma senha ou chave está salva no código.

## Autor

Igor Carvalho de Oliveira, estudante de Ciência da Computação (UDF).
