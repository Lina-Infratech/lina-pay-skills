---
name: web-lina-pay-sdk
description: Documentação e integração do pacote npm @lina-openx/web-lina-pay-sdk (Lina OpenX / Open Finance). Use este skill quando o usuário pedir ajuda com o SDK, exemplos de chamadas aos controllers, payloads, tipos TypeScript, fluxos de consentimento/pagamento/enrollment, participantes, configuração de ambiente, tratamento de erros ou organização do repositório web-lina-pay-sdk. Use também quando mencionar LinaPay, embedded payment manager, IAM Lina, subtenant, createConsent, createPayment, createPaymentRequest, enrollment ou FIDO no contexto deste projeto.
---

# Web Lina Pay SDK — skill de documentação

Skill para orientar assistentes e desenvolvedores com base na documentação oficial do SDK e nas tipagens de referência.

## Fonte da verdade (ordem de consulta)

1. **`README.md`** na raiz do repositório — descrição, instalação, métodos, exemplos, estrutura de pastas, execução local, erros e tipos principais.
2. **`reference.md`** (nesta pasta) — catálogo consolidado de tipagens alinhado ao README e ao que o pacote exporta em `src/types/index.ts` e `src/index.ts`.
3. **Código em `src/`** — quando o README não cobrir um detalhe ou houver divergência; prefira citar arquivos concretos (`controllers/`, `services/`, `types/`).

## Mapa do README (seções → conteúdo)

| Seção README | O que cobre |
|--------------|-------------|
| Descrição | Escopo do SDK (consentimento, payment request, pagamento, enrollment, participantes, FIDO/WebAuthn). |
| Instalação | `npm` / `yarn` / `pnpm` para `@lina-openx/web-lina-pay-sdk`. |
| Como usar → Importação | Funções e `LinaPayError` exportados pelo pacote. |
| Configuração inicial | `configure({ iamBaseUrl, apiBaseUrl })` — HML vs produção. |
| Métodos 1–10 | API pública dos controllers (ver tabela abaixo). |
| Organização de pastas | `src/config`, `controllers`, `services`, `types`, `utils`, `index.ts`. |
| Como rodar localmente | Node 18+, scripts `dev`, `test`, `format`, `build`, `ci`, clone, pasta `example/`. |
| Tratamento de erros | `try/catch`, `instanceof LinaPayError`, `statusCode`. |
| Tipos principais | Resumo de interfaces usadas nos exemplos. |
| Licença / Suporte | MIT, contatos e issues GitLab. |

## Métodos públicos (ordem do README)

| # | Função | Resumo |
|---|--------|--------|
| 1 | `configure` | URLs base IAM e API (`LinaPayConfig`). |
| 2 | `createConsent` | Consentimento de pagamento (`CreateConsentRequest` → `CreateConsentResponse`). |
| 3 | `createPayment` | Pagamento pós-OAuth (`CreatePaymentRequest`: state, code, idToken, tenantId → `CreatePaymentResponse`). |
| 4 | `createPaymentRequest` | Solicitação de pagamento distinta de `createPayment` (`CreatePaymentRequestDTO` → `PaymentRequestData`); validação Zod; erro `Invalid payload`. |
| 5 | `createEnrollment` | Enrollment FIDO (`CreateEnrollmentRequest` → `CreateEnrollmentResponse`). |
| 6 | `registerDevice` | Callback pós-enrollment (`RegisterDeviceRequest` → tipo exportado como `Enrollment`). |
| 7 | `getEnrollmentList` | Lista por CPF (`EnrollmentList`). |
| 8 | `revokeEnrollment` | Revoga por ID (`RevokeEnrollmentResponse`). |
| 9 | `createPaymentWithEnrollment` | Pagamento com enrollment (`PaymentWithEnrollmentRequest` → `RetornoJsrPaymentDto`). |
| 10 | `getParticipants` | Participantes (`Participant[]`). |

## Distinções importantes (evitar confusão)

- **`createPayment`** vs **`createPaymentRequest`**: o primeiro finaliza fluxo com tokens OAuth; o segundo envia dados do pagamento (valor, credor, redirect, schedule etc.) e retorna `id` + `redirectUri` da solicitação.
- **`CreatePaymentRequest`** (OAuth) vs **`CreatePaymentRequestDTO`** (solicitação de pagamento): nomes parecidos, payloads completamente diferentes — ver `reference.md`.
- Credenciais: sempre `LinaPayCredentials` (`subtenantId`, `subtenantSecret`); o SDK obtém o access token internamente.

## Como responder perguntas sobre o SDK

- Preferir exemplos em TypeScript alinhados ao README (nomes de campos e enums: `PESSOA_NATURAL`, `CACC`, dias da semana em português com `_FEIRA`, etc.).
- Mencionar `configure` quando o ambiente não for o padrão.
- Para payloads de **`createPaymentRequest`**, lembrar que a validação runtime exige `schedule` presente (pode ser `{}`) conforme documentação do README.
- Erros: descrever `LinaPayError` e uso de `statusCode` quando aplicável.

## Progressive disclosure

- Para **assinaturas completas, unions e interfaces aninhadas**, abrir **`reference.md`** nesta pasta em vez de duplicar tudo no corpo deste skill.
- Para **comportamento HTTP ou endpoints**, seguir `src/config/environment.ts` (`ENDPOINTS`) e serviços em `src/services/`.

## Idioma

- Responder em **português** quando o usuário do projeto utilizar português (alinhado ao README e ao repositório).
