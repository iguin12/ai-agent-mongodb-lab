# Agente de IA com MongoDB e LangGraph

Projeto desenvolvido no **AI Builder Day Brasília** (MongoDB, 18/11/2026), no lab *"The A to Z of Building AI Agents"*.

Um **agente de viagens** que conversa com o usuário, consulta voos e hospedagens em um banco MongoDB, faz reservas (pedindo aprovação antes) e lembra das preferências de cada usuário entre conversas.

## O que o agente faz

- **Consulta o banco sozinho:** lista as coleções, lê o esquema e monta as consultas (MongoDB Database Toolkit do LangChain).
- **Cria reservas** com uma ferramenta própria (`create_booking`), que grava na coleção `bookings`.
- **Human-in-the-loop:** antes de gravar qualquer reserva, o agente pausa (`interrupt`) e pede aprovação do usuário.
- **Memória de curto prazo:** guarda o histórico de cada conversa (thread) no MongoDB com o `MongoDBSaver`.
- **Memória de longo prazo:** guarda preferências por usuário com o `MongoDBStore`, usando embeddings da Voyage AI e busca vetorial. As memórias não vazam entre usuários.
- **MCP:** mostra como obter ferramentas do MongoDB MCP Server.

## Arquitetura (grafo do LangGraph)

```
START → agent ──(sem ferramentas)──→ END
          │
          ├──(consulta)──→ tools ──→ agent
          │
          └──(reserva)──→ confirm ──→ tools ──→ agent
```

## Tecnologias

Python, MongoDB Atlas, LangChain, LangGraph, Voyage AI (embeddings `voyage-4`), MongoDB MCP Server.

## Observações

- O notebook foi feito para o ambiente do workshop: ele lê `MONGODB_URI` e `PASSKEY` de variáveis de ambiente e usa um módulo `utils` (com `set_env` e `get_llm`) fornecido pelo lab, que não está neste repositório.
- Para rodar fora do workshop, troque `get_llm` pelo modelo de sua preferência no LangChain, configure sua própria chave da Voyage AI e use a string de conexão do seu cluster Atlas.
- Nenhuma senha ou chave está salva no código.

## Autor

Igor Carvalho de Oliveira, estudante de Ciência da Computação (UDF).
