# 📦 Rastro Correios API - v2.0.0

![Version](https://img.shields.io/badge/version-2.0.0-blue)

![rastro-correios](https://i.postimg.cc/BZ7bxwgr/rastro-correios-api.png)

**Rastro Correios** é uma API REST pública e gratuita que permite rastrear objetos dos [Correios do Brasil](https://www.correios.com.br) de forma simples, rápida. Ideal para integrar em sistemas logísticos, e-commerces, bots ou qualquer aplicação que precise acompanhar o status de entregas em tempo real.

Este repositório serve como a **documentação oficial** para a utilização da API.

---

## 🚀 Como usar

A API é pública e não requer autenticação. Para consultar o status de uma encomenda, basta fazer uma requisição `GET` para o endpoint oficial, informando o código de rastreamento:

```http
GET https://rastro-correios-api.zeabur.app/api/v2/track/:trackingCode
```

**Exemplo prático:**

```http
GET https://rastro-correios-api.zeabur.app/api/v2/track/PO123456789BR
```

---

## ⚙️ O que há de novo na v2.0.0

### 🛡️ Rate Limiting

Para garantir a disponibilidade e proteger o serviço contra usos abusivos, a API conta com um sistema de **rate limiting** baseado em janela deslizante (_sliding window_):

- **Limite:** 10 requisições por minuto por IP.
- **Bloqueio:** Ao atingir o limite, o IP fica temporariamente bloqueado por **5 minutos**.
- Em cada requisição, a API retorna cabeçalhos úteis para o controle do seu lado:

| Cabeçalho               | Descrição                                         |
| ----------------------- | ------------------------------------------------- |
| `x-ratelimit-limit`     | Número máximo de requisições permitidas na janela |
| `x-ratelimit-remaining` | Requisições restantes na janela atual             |
| `x-ratelimit-reset`     | Timestamp (Unix) de quando a janela será resetada |

Caso o limite seja excedido, você receberá um erro `429 Too Many Requests` com um campo `retryAfter` indicando o tempo de espera necessário (em segundos).

---

### ⚡ Cache Dinâmico Inteligente

Para oferecer respostas em milissegundos e evitar bloqueios na fonte (Correios), a API implementa um sistema de **cache dinâmico**. O tempo de vida (TTL) das respostas em cache se ajusta automaticamente com base no status atual do objeto:

| Status da Encomenda     | TTL do Cache | Motivo                                       |
| ----------------------- | ------------ | -------------------------------------------- |
| `delivered`             | 30 dias      | Objeto entregue — o status não mudará mais.  |
| `in_transit` / `posted` | 1,5 hora     | Em trânsito — atualizações são moderadas.    |
| `delivery_route`        | 15 minutos   | Saiu para entrega — muda a qualquer momento. |
| Demais status           | 15 minutos   | Comportamento padrão conservador.            |

---

### 🏗️ Arquitetura e Engenharia

Por baixo dos panos, o serviço foi completamente refatorado utilizando princípios de **Clean Architecture** e **Domain-Driven Design (DDD)**.

A API é servida via **Node.js** com o framework **Fastify**, garantindo altíssima performance. A arquitetura modular separa claramente as regras de domínio das camadas de infraestrutura. A entrada de dados passa por uma validação estrita, enquanto a comunicação com serviços externos e o gerenciamento de estado (Cache e Rate Limit) são orquestrados de forma eficiente utilizando **Redis**.

---

## 🧪 Exemplo de Resposta

O formato de resposta segue uma estrutura limpa e padronizada em JSON:

```json
{
  "data": {
    "code": "PO123456789BR",
    "type": "ENCOMENDA PAC",
    "tracks": [
      {
        "description": "Objeto entregue ao destinatário",
        "status": "delivered",
        "origin": "Agência Dos Correios FORTALEZA, CE",
        "date": "25/02/25",
        "time": "10:02"
      },
      {
        "description": "Objeto saiu para entrega ao destinatário",
        "status": "delivery_route",
        "origin": "Agência Dos Correios FORTALEZA, CE",
        "date": "25/02/25",
        "time": "09:34"
      },
      {
        "description": "Objeto em transferência - por favor aguarde",
        "status": "in_transit",
        "origin": "Unidade de Tratamento FORTALEZA, CE",
        "destination": "Agência Dos Correios FORTALEZA, CE",
        "date": "18/03/2025",
        "time": "13:43:20"
      },
      {
        "description": "Objeto postado",
        "status": "posted",
        "origin": "Agência Dos Correios São Paulo, SP",
        "date": "21/02/25",
        "time": "11:06"
      }
    ]
  }
}
```

---

## ✨ Resumo das Funcionalidades

- 📮 Consulta de rastreamento por código oficial dos Correios.
- 🛡️ Proteção por Rate Limiting inteligente.
- ⚡ Respostas otimizadas através de um Cache dinâmico.
- 🔒 Retorno padronizado e seguro em JSON.
- ⚠️ Tratamento de erros e instabilidades na fonte.

---

## ⚖️ Termos de Uso

Ao utilizar esta API pública, você concorda com os seguintes termos:

- **Disponibilidade:** Este é um projeto pessoal fornecido "como está" (as is). Não há garantias de 100% de uptime, suporte contínuo ou disponibilidade permanente do serviço.
- **Limitações:** O serviço possui proteções de _rate limit_ (descritas acima). Tentativas de burlar esses limites abusivamente ou realizar ataques resultarão no bloqueio permanente do IP do requisitante.
- **Dados:** A API atua apenas como um intermediário, repassando e formatando as informações oficiais do sistema dos Correios. Não há responsabilidade sobre a exatidão, atualização ou indisponibilidade dos dados retornados pela fonte original.

---

## 💡 Autor

Desenvolvido por [José Vitor Soares](https://github.com/josevitorsoares).
