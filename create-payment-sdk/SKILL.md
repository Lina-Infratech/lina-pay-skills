## Metadata

name: create-payment-sdk
description: Guia para integrar o SDK @lina-openx/web-lina-pay-sdk em páginas web
language: pt-BR
tags:

- lina-pay
- sdk
- pagamentos

## Objetivo

Esta Skill orienta a criação de páginas de pagamento open-finance web com integração do SDK `@lina-openx/web-lina-pay-sdk`, seguindo boas práticas de desenvolviment web.

## Stack Obrigatória

- SDK de pagamento: **@lina-openx/web-lina-pay-sdk**.
- Linguagem recomendada: **TypeScript**.

## Quando Aplicar

Use esta Skill quando o usuário pedir:

- integração de checkout/pagamento no front-end com Lina Pay;
- criação de páginas de produto, carrinho e checkout ou outras páginas de pagamento open-finance;
- padronização de componentes visuais usando framework de UI solicitado pelo usuário.

## Diretrizes Arquiteturais

1. Criar uma camada de domínio para pagamento:
   - `src/features/checkout/` para casos de uso, tipos e integrações;
   - `src/lib/lina-pay/` para wrapper do SDK e helpers.
2. Evitar acoplamento direto do SDK em múltiplos componentes:
   - encapsular o SDK em serviço;
   - expor API interna estável para o restante da aplicação.
3. Manter segredos no servidor:
   - nunca expor chaves sensíveis no client;
   - usar `process.env` e validação de variáveis.
4. Tratar estados de pagamento explicitamente:
   - `idle`, `loading`, `success`, `error`, `retry`.

## Estrutura Recomendada de Pastas

```txt
src/
  app/
    (store)/
      products/
      cart/
      checkout/
  components/
    ui/
    checkout/
  features/
    checkout/
      application/
      domain/
      infrastructure/
  lib/
    lina-pay/
      client.ts
      mapper.ts
      types.ts
```

## Boas Práticas de Integração do SDK

1. Instalar dependências:
   - `npm i @lina-openx/web-lina-pay-sdk`
2. Criar um adapter do SDK em `src/lib/lina-pay/client.ts`.
3. Inicializar o SDK apenas em ambiente client quando necessário.
4. Centralizar tratamento de erros do SDK e mapear para mensagens de UI.
5. Criar testes para fluxos críticos:
   - criação da intenção de pagamento;
   - confirmação e retorno de erro;
   - retry e feedback visual.

### Configuração Inicial do SDK

Antes de usar o SDK, você pode configurar as URLs base para diferentes ambientes (homologação ou produção):

```typescript
import { configure } from "@lina-openx/web-lina-pay-sdk";

// Configurar para produção
configure({
  iamBaseUrl: "https://iam.linaob.com.br",
  apiBaseUrl: "https://embedded-payment-manager.linaob.com.br",
});
```

## Métodos do SDK para integração no momento do redirecionamento após selecionar o meio de pagamento e o produto

### 1. `configure(config: Partial<LinaPayConfig>)`

Configura as URLs base do SDK para diferentes ambientes.

**Parâmetros:**

- `config`: Objeto parcial com as configurações:
  - `iamBaseUrl` (string, opcional): URL base do serviço de IAM
  - `apiBaseUrl` (string, opcional): URL base da API de pagamentos

**Exemplo:**

```typescript
configure({
  iamBaseUrl: "https://iam.linaob.com.br",
  apiBaseUrl: "https://embedded-payment-manager.linaob.com.br",
});
```

---

### 2. `createPaymentRequest(credentials, payload)`

Cria uma solicitação de pagamento e redireciona para o portal de inicialização do pagamento.

**Parâmetros:**

- `credentials`: Objeto com credenciais do subtenant:
  - `subtenantId` (string): ID do subtenant
  - `subtenantSecret` (string): Secret do subtenant
- `payload`: Objeto `CreatePaymentRequest` com:
  - `details` (string): Detalhes do pagamento
  - `txId` (string): ID da transação (opcional)
  - `redirectUri` (string): URI de redirecionamento
  - `cpfCnpj` (string): CPF ou CNPJ do cliente
  - `value` (number): Valor do pagamento
  - `creditor` (objeto): Detalhes do crédito
    - `personType` (string): Tipo de pessoa ('PESSOA_JURIDICA' | 'PESSOA_NATURAL')
    - `cpfCnpj` (string): CPF ou CNPJ do cliente
    - `name` (string): Nome do cliente
    - `accountNumber` (string): Número da conta
    - `accountIssuer` (string): Emissor da conta
    - `accountIspb` (string): ISPB da conta
    - `accountType` (string): Tipo de conta ('CACC' | 'SVGS')

**Retorna:** `Promise<CreatePaymentResponse>` com `id` e `redirectUrl`

**Exemplo:**

```typescript
const consent = await createPaymentRequest(
  {
    subtenantId: 'seu-subtenant-id',
    subtenantSecret: 'seu-subtenant-secret'
  },
  {
    "details": "Detalhes do pagamento",
    "redirectUri": "https://pix-portal.linaopenx.com.br/pagamentos/callback",
    "cpfCnpj": "CPF_DO_CLIENTE", // CPF ou CNPJ do cliente
    "value": 0.01, // Valor do pagamento indicado pelo cliente
    "creditor": {
      "personType": "PESSOA_JURIDICA",
      "cpfCnpj": "20640035000100",
      "name": "Instituto Gioia de Esporte"
      "accountNumber": "12920",
      "accountIssuer": "0297",
      "accountIspb": "60746948",
      "accountType": "CACC",
    }
  }
)
- Usar o retorno redirectUrl para redirecionar o usuário para o portal de inicialização do pagamento.

## Boas Práticas de UI/UX (ShadCN-UI)

- Usar componentes acessíveis (`Button`, `Input`, `Form`, `Dialog`, `Alert`).
- Garantir feedback visual de carregamento no checkout (skeleton/spinner).
- Exibir erros de pagamento de forma clara e acionável.
- Preservar consistência visual com tokens e tema centralizado.
- Evitar bloqueio da jornada: sempre oferecer opção de tentar novamente.

## Segurança e Confiabilidade

- Validar payloads com schema (ex.: Zod) antes de enviar ao SDK.
- Sanitizar e validar entradas de usuário.
- Implementar logs de erro sem expor dados sensíveis.
- Preferir idempotência em chamadas de backend relacionadas ao pagamento.

## Performance

- Minimizar bundle de checkout com importação sob demanda.
- Evitar renderização desnecessária de componentes client.

## Prompt Base para Uso da Skill

Quando esta Skill for acionada, seguir este fluxo:

1. Confirmar requisitos do checkout (campos, meios de pagamento, regras).
2. Propor arquitetura usando framework de UI solicitado pelo usuário.
3. Implementar integração via adapter do `@lina-openx/web-lina-pay-sdk`.
4. Construir interface com framework de UI solicitado pelo usuário.
5. Garantir tratamento de estado, erro, loading e sucesso.
6. Sugerir testes e checklist de validação final.

## Referência

- Documentação do pacote:
  [https://www.npmjs.com/package/@lina-openx/web-lina-pay-sdk](https://www.npmjs.com/package/@lina-openx/web-lina-pay-sdk)
```
