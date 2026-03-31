---
name: lina-payment-sdk
description: Guia completo para integrar o SDK @lina-openx/web-lina-pay-sdk em aplicações web. Cobre configuração, criação de consentimentos e pagamentos Open Finance, gerenciamento de enrollments FIDO2/WebAuthn, consulta de participantes e tratamento de erros. Use quando o usuário pedir integração de pagamentos Lina Pay, checkout Open Finance, pagamentos recorrentes com FIDO2, ou gerenciamento de dispositivos de autenticação.
---

# Lina Payment SDK

## Objetivo

Orientar a integração completa do SDK `@lina-openx/web-lina-pay-sdk` em aplicações web TypeScript, cobrindo todos os fluxos de pagamento Open Finance, enrollment de dispositivos FIDO2 e boas práticas de segurança.

## Quando Aplicar

- Integração de checkout/pagamento Open Finance com Lina Pay
- Criação de consentimentos e pagamentos via Open Finance
- Gerenciamento de enrollments e pagamentos recorrentes com FIDO2/WebAuthn
- Consulta de participantes (bancos) da plataforma
- Qualquer uso do pacote `@lina-openx/web-lina-pay-sdk`

## Instalação

```bash
npm install @lina-openx/web-lina-pay-sdk
# ou
pnpm add @lina-openx/web-lina-pay-sdk
# ou
yarn add @lina-openx/web-lina-pay-sdk
```

## Importação

```typescript
import {
  configure,
  createConsent,
  createPayment,
  createEnrollment,
  getEnrollmentList,
  revokeEnrollment,
  registerDevice,
  createPaymentWithEnrollment,
  getParticipants,
  LinaPayError,
} from "@lina-openx/web-lina-pay-sdk";
```

## Segurança de Credenciais e Segredos

**Regras obrigatórias para `subtenantId` e `subtenantSecret`:**

1. **Nunca expor no client-side**: credenciais JAMAIS devem aparecer em código front-end, bundles JS ou repositórios.
2. **Usar variáveis de ambiente no servidor**:
   ```bash
   # .env.local (NUNCA commitar este arquivo)
   LINA_PAY_SUBTENANT_ID=seu-subtenant-id
   LINA_PAY_SUBTENANT_SECRET=seu-subtenant-secret
   ```
3. **Adicionar ao `.gitignore`**:
   ```gitignore
   .env
   .env.local
   .env.*.local
   ```
4. **Proxy via API backend**: criar uma rota de API server-side que recebe o pedido do front-end e chama o SDK com as credenciais do ambiente servidor. O front-end nunca acessa as credenciais diretamente.
5. **Rotação de segredos**: implementar processo de rotação periódica das credenciais de subtenant.
6. **Ambientes separados**: usar credenciais distintas para homologação e produção.
7. **Validação de variáveis na inicialização**: falhar imediatamente se as variáveis obrigatórias não estiverem definidas.

```typescript
function getCredentials(): LinaPayCredentials {
  const subtenantId = process.env.LINA_PAY_SUBTENANT_ID;
  const subtenantSecret = process.env.LINA_PAY_SUBTENANT_SECRET;

  if (!subtenantId || !subtenantSecret) {
    throw new Error("Credenciais Lina Pay não configuradas no ambiente");
  }

  return { subtenantId, subtenantSecret };
}
```

**Dados sensíveis no payload:**
- CPF/CNPJ são dados pessoais protegidos pela LGPD. Transmitir apenas via HTTPS.
- Nunca logar payloads completos que contenham CPF/CNPJ em texto claro.
- Sanitizar logs removendo campos sensíveis antes de persistir.

## Configuração do SDK

### `configure(config)`

Define as URLs base para o ambiente desejado. Chamar antes de qualquer outro método.

```typescript
// Produção
configure({
  iamBaseUrl: "https://iam.prod.linaob.com.br",
  apiBaseUrl: "https://embedded-payment-manager.prod.linaob.com.br",
});

// Homologação (padrão)
configure({
  iamBaseUrl: "https://iam.linaob.com.br",
  apiBaseUrl: "https://embedded-payment-manager.linaob.com.br",
});
```

## Fluxo de Pagamento Padrão (Consent + Payment)

### Passo 1: `createConsent(credentials, payload)`

Cria um consentimento de pagamento Open Finance. Retorna `consentId` e `redirectUrl` para redirecionar o usuário ao banco.

```typescript
const consent = await createConsent(credentials, {
  organisationId: "org-uuid",
  authorisationServerId: "auth-server-uuid",
  payment: {
    redirectUri: "https://meusite.com/callback",
    value: 150.0,
    creditor: {
      name: "Nome Credor",
      personType: "PESSOA_NATURAL",
      cpfCnpj: "12345678901",
      accountNumber: "12345-6",
      accountIssuer: "0001",
      accountIspb: "12345678",
      accountType: "CACC",
    },
  },
  platform: "WEB",
});

// Redirecionar o usuário
window.location.href = consent.redirectUrl;
```

**Campos do `payment.creditor`:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `name` | string | Nome do credor |
| `personType` | `'PESSOA_NATURAL'` \| `'PESSOA_JURIDICA'` | Tipo de pessoa |
| `cpfCnpj` | string | CPF ou CNPJ sem formatação |
| `accountNumber` | string | Número da conta |
| `accountIssuer` | string | Código do emissor |
| `accountIspb` | string | Código ISPB do banco |
| `accountType` | `'CACC'` \| `'SVGS'` | Tipo de conta |
| `accountPixKey` | string (opcional) | Chave Pix |

**Agendamento de pagamentos recorrentes:**

```typescript
const consent = await createConsent(credentials, {
  organisationId: "org-uuid",
  authorisationServerId: "auth-server-uuid",
  payment: {
    redirectUri: "https://meusite.com/callback",
    value: 100.0,
    creditor: { /* ... */ },
    schedule: {
      monthly: {
        dayOfMonth: 15,
        startDate: "2026-04-15",
        quantity: 12,
      },
    },
  },
});
```

### Passo 2: `createPayment(credentials, payload)`

Após o redirecionamento de volta (callback), cria o pagamento com os dados do OAuth.

```typescript
const payment = await createPayment(credentials, {
  state: "oauth-state",
  code: "authorization-code",
  idToken: "id-token-jwt",
  tenantId: "tenant-id",
});

console.log(payment.id, payment.status, payment.value);
```

## Fluxo de Enrollment FIDO2 (Pagamentos Recorrentes)

### Passo 1: `createEnrollment(credentials, payload)`

Cadastra um dispositivo para pagamentos recorrentes via FIDO2/WebAuthn.

```typescript
const enrollment = await createEnrollment(credentials, {
  organisationId: "org-uuid",
  authorisationServerId: "auth-server-uuid",
  enrollment: {
    deviceName: "Notebook do Usuário",
    document: "12345678901",
    debtor: {
      accountIspb: "00000000",
      accountIssuer: "2558",
      accountNumber: "5271110",
      accountType: "CACC",
    },
  },
  redirectUri: "https://meusite.com/enrollment-callback",
});

window.location.href = enrollment.redirectUrl;
```

### Passo 2: `registerDevice(credentials, payload)`

Após o callback do enrollment, registra o dispositivo com autenticação FIDO2.

```typescript
const device = await registerDevice(credentials, {
  state: "oauth-state",
  code: "authorization-code",
  idToken: "id-token-jwt",
  tenantId: "tenant-id",
  platform: "WEB", // 'ANDROID' | 'IOS' | 'WEB' | 'BROWSER' | 'CROSS_PLATFORM'
  rp: "relying-party-id", // opcional
});

console.log(device.id, device.deviceName);
```

### Consultar enrollments: `getEnrollmentList(credentials, cpf)`

```typescript
const list = await getEnrollmentList(credentials, "12345678901");
list.forEach((e) => console.log(e.id, e.deviceName));
```

### Revogar enrollment: `revokeEnrollment(credentials, enrollmentId)`

```typescript
await revokeEnrollment(credentials, "enrollment-id-123");
```

### Pagar com enrollment: `createPaymentWithEnrollment(credentials, payload)`

```typescript
const payment = await createPaymentWithEnrollment(credentials, {
  enrollmentId: "enrollment-id-123",
  organisationId: "org-uuid",
  authorisationServerId: "auth-server-uuid",
  payment: {
    redirectUri: "https://meusite.com/callback",
    value: 250.0,
    creditor: {
      name: "Credor",
      personType: "PESSOA_NATURAL",
      cpfCnpj: "12345678901",
      accountNumber: "12345-6",
      accountIssuer: "0001",
      accountIspb: "12345678",
      accountType: "CACC",
    },
  },
  fidoSignOptions: {
    rp: "relying-party-id",
    platform: "WEB",
  },
});
```

## Consulta de Participantes

### `getParticipants(credentials)`

Retorna a lista de bancos/participantes registrados na plataforma Lina Open Finance.

```typescript
const participants = await getParticipants(credentials);
participants.forEach((p) => console.log(p.name, p.ispb));
```

## Tratamento de Erros

O SDK lança `LinaPayError` para erros de validação e comunicação. Sempre usar `try/catch`:

```typescript
import { LinaPayError } from "@lina-openx/web-lina-pay-sdk";

try {
  const consent = await createConsent(credentials, payload);
} catch (error) {
  if (error instanceof LinaPayError) {
    console.error("Erro Lina Pay:", error.message);
    console.error("Status:", error.statusCode);
    // Mapear para mensagem amigável na UI
  } else {
    console.error("Erro inesperado:", error);
  }
}
```

## Diretrizes Arquiteturais

1. **Encapsular o SDK** em um serviço/facade (`src/lib/lina-pay/client.ts`) para não acoplar múltiplos componentes diretamente.
2. **Separar server e client**: credenciais e chamadas ao SDK devem viver no server-side. O front-end consome via API interna.
3. **Validar payloads** com Zod antes de enviar ao SDK.
4. **Gerenciar estados de pagamento explicitamente**: `idle` | `loading` | `success` | `error` | `retry`.
5. **Testes unitários** para os fluxos críticos: consent, payment, enrollment, erro e retry.

## Estrutura Recomendada

```
src/
  lib/
    lina-pay/
      client.ts        # Facade do SDK (inicialização + métodos encapsulados)
      credentials.ts   # Obtenção segura de credenciais via env vars
      types.ts         # Re-exportação de tipos relevantes
      errors.ts        # Mapeamento de LinaPayError para mensagens de UI
  features/
    checkout/
      application/     # Casos de uso
      domain/          # Entidades e interfaces
      infrastructure/  # Implementações (chamadas ao facade)
```

## Checklist de Integração

- [ ] Variáveis de ambiente configuradas e validadas na inicialização
- [ ] `.env*` no `.gitignore`
- [ ] `configure()` chamado com URLs corretas do ambiente
- [ ] Credenciais acessadas apenas no server-side
- [ ] `createConsent` → redirecionamento → `createPayment` funcional
- [ ] Tratamento de `LinaPayError` com mensagens amigáveis
- [ ] Fluxo de enrollment FIDO2 (se aplicável)
- [ ] Testes cobrindo caminhos de sucesso e erro
- [ ] Logs não expõem CPF/CNPJ ou credenciais

## Referência

- Documentação do pacote: [npm @lina-openx/web-lina-pay-sdk](https://www.npmjs.com/package/@lina-openx/web-lina-pay-sdk)
- Para tipos TypeScript detalhados, consulte [reference.md](reference.md)
