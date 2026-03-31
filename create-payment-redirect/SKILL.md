## Metadata

name: Lina Pay Next.js Skill
description: Guia para integrar o SDK @lina-openx/web-lina-pay-sdk em páginas web de e-commerce com Next.js (última versão) e ShadCN-UI.
language: pt-BR
tags:

- lina-pay
- sdk
- nextjs
- shadcn-ui
- ecommerce
- pagamentos

## Objetivo

Esta Skill orienta a criação de páginas de e-commerce web com integração do SDK `@lina-openx/web-lina-pay-sdk`, seguindo boas práticas de arquitetura no Next.js e padrões de UI com ShadCN-UI.

## Stack Obrigatória

- Framework principal: **Next.js (última versão estável)** com App Router.
- UI: **ShadCN-UI** com Tailwind CSS.
- SDK de pagamento: **@lina-openx/web-lina-pay-sdk**.
- Linguagem recomendada: **TypeScript**.

## Quando Aplicar

Use esta Skill quando o usuário pedir:

- integração de checkout/pagamento no front-end com Lina Pay;
- criação de páginas de produto, carrinho e checkout em Next.js;
- padronização de componentes visuais com ShadCN-UI;
- arquitetura escalável para e-commerce web.

## Diretrizes Arquiteturais (Next.js)

1. Usar App Router (`app/`) com separação clara entre:
   - Server Components para dados e composição de página;
   - Client Components apenas para interações necessárias (SDK, estado de UI, formulários).
2. Criar uma camada de domínio para pagamento:
   - `src/features/checkout/` para casos de uso, tipos e integrações;
   - `src/lib/lina-pay/` para wrapper do SDK e helpers.
3. Evitar acoplamento direto do SDK em múltiplos componentes:
   - encapsular o SDK em serviço/facade;
   - expor API interna estável para o restante da aplicação.
4. Manter segredos no servidor:
   - nunca expor chaves sensíveis no client;
   - usar `process.env` e validação de variáveis.
5. Tratar estados de pagamento explicitamente:
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
    ui/                  # shadcn-ui
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
  iamBaseUrl: "https://iam.prod.linaob.com.br",
  apiBaseUrl: "https://embedded-payment-manager.prod.linaob.com.br",
});

// Configurar para homologação (padrão)
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
  iamBaseUrl: "https://iam.prod.linaob.com.br",
  apiBaseUrl: "https://embedded-payment-manager.prod.linaob.com.br",
});
```

---

### 2. `createConsent(credentials, payload)`

Cria um consentimento de pagamento Open Finance.

**Parâmetros:**

- `credentials`: Objeto com credenciais do subtenant:
  - `subtenantId` (string): ID do subtenant
  - `subtenantSecret` (string): Secret do subtenant
- `payload`: Objeto `CreateConsentRequest` com:
  - `organisationId` (string): ID da organização
  - `authorisationServerId` (string): ID do servidor de autorização
  - `payment` (objeto): Detalhes do pagamento
  - `redirectUri` (string, opcional): URI de redirecionamento
  - `platform` (string, opcional): Plataforma ('APP' | 'WEB')

**Retorna:** `Promise<CreateConsentResponse>` com `consentId` e `redirectUrl`

**Exemplo:**

```typescript
const consent = await createConsent(
  {
    subtenantId: 'seu-subtenant-id',
    subtenantSecret: 'seu-subtenant-secret'
  },
  {
    organisationId: 'c8f0bf49-4744-4933-8960-7add6e590841',
    authorisationServerId: 'c8f0bf49-4744-4933-8960-7add6e590841',
    payment: {
      redirectUri: 'https://exemplo.com/redirect',
      value: 1500.50,
      creditor: {
        name: 'João Silva',
        personType: 'PESSOA_NATURAL',
        cpfCnpj: '12345678901',
        accountNumber: '12345-6',
        accountIssuer: '0001',
        accountPixKey: 'joao@exemplo.com',
        accountIspb: '12345678',
        accountType: 'CACC'
      }
    },
    platform: 'WEB'
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
- Usar cache e revalidação no lado servidor para dados de catálogo.

## Prompt Base para Uso da Skill

Quando esta Skill for acionada, seguir este fluxo:

1. Confirmar requisitos do checkout (campos, meios de pagamento, regras).
2. Propor arquitetura em Next.js App Router com separação server/client.
3. Implementar integração via adapter do `@lina-openx/web-lina-pay-sdk`.
4. Construir interface com ShadCN-UI.
5. Garantir tratamento de estado, erro, loading e sucesso.
6. Sugerir testes e checklist de validação final.

## Referência

- Documentação do pacote:
  [https://www.npmjs.com/package/@lina-openx/web-lina-pay-sdk](https://www.npmjs.com/package/@lina-openx/web-lina-pay-sdk)
```
