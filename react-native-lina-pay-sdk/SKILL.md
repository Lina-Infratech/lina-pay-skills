---
name: react-native-lina-pay-sdk
description: Integração com o pacote react-native-lina-pay-sdk (Lina Open Finance) — pagamentos Open Finance, consentimento, createPayment, participantes, enrollment FIDO2/Passkeys, createPaymentWithEnrollment, configure, LinaPayError. Use este skill sempre que o usuário mencionar este SDK, Lina Pay, Lina OpenX, embedded payment, subtenant, consentimento de pagamento, enrollment de dispositivo, ou exemplos de código deste repositório; também quando precisar alinhar implementação à documentação do README ou às tipagens do SDK.
---

# React Native Lina Pay SDK

Skill para orientar implementação e respostas técnicas sobre o **react-native-lina-pay-sdk**, alinhada ao [README.md](../../README.md) e aos documentos referenciados por ele.

## Quando carregar material extra

- **Tipagens completas, unions e interfaces espelhando o código-fonte:** leia [reference.md](./reference.md) antes de afirmar formatos de payload ou campos opcionais/obrigatórios.
- **Fluxos detalhados e passo a passo de implementação:** [docs/USER_GUIDE.md](../../docs/USER_GUIDE.md).
- **Diagramas visuais do fluxo de pagamento:** [docs/FLOW_DIAGRAM.md](../../docs/FLOW_DIAGRAM.md).

## Mapa da documentação (README)

| Tópico no README | Onde está no repositório | Resumo |
|------------------|--------------------------|--------|
| Instalação (`npm` / `yarn`) | README — seção Instalação | Pacote `react-native-lina-pay-sdk`. |
| Requisitos (RN ≥ 0.83, Node ≥ 20) | README — Requisitos | Validar ambiente antes de sugerir versões. |
| Documentação estendida | `docs/USER_GUIDE.md`, `docs/FLOW_DIAGRAM.md` | Guia completo + diagramas. |
| Configuração de ambiente (IAM / API) | README — Uso → Configuração Inicial | `configure({ iamBaseUrl?, apiBaseUrl? })`; padrões descritos no README (HML) podem diferir do default em `src/config/environment.ts` — em dúvida, confira o código e [reference.md](./reference.md). |
| Listagem de participantes | README — Listagem de Participantes + API `getParticipants` | `getParticipants(credentials)` → `Participant[]`. |
| Consentimento de pagamento | README — Criação de Consentimento + API `createConsent` | `createConsent(credentials, CreateConsentRequest)` → `CreateConsentResponse`. |
| Pagamento pós-autorização | README — Criação de Pagamento + API `createPayment` | `createPayment(credentials, CreatePaymentRequest)` → `CreatePaymentResponse` (inclui `payments: PaymentItem[]`). |
| Enrollment (FIDO2/Passkeys) | README — Enrollment | `createEnrollment`, `registerDevice`, `getEnrollmentList`, `revokeEnrollment`. |
| Pagamento com enrollment | README — Pagamento com Enrollment + API `createPaymentWithEnrollment` | Sem redirect ao banco; biometria; requer enrollment `AUTHORISED`. |
| Exemplo React (lista) | README — Exemplo Completo com React | `FlatList` + `getParticipants`. |
| API Reference (parâmetros e retornos) | README — seção API Reference | Tabelas de parâmetros; cruzar com [reference.md](./reference.md) para precisão. |
| Autenticação OAuth2 | README — Autenticação | Token cacheado e renovação (~5 min antes do fim). |
| Tratamento de erros | README — Tratamento de Erros | `instanceof LinaPayError`; `statusCode` opcional. |
| Desenvolvimento / exemplo / testes | README — Desenvolvimento | `yarn`, `yarn prepare`, `yarn example android|ios`, `yarn test`, `yarn typecheck`, `yarn lint`. |
| Estrutura `src/` | README — Estrutura do Projeto | `controllers/`, `services/`, `types/`, `utils/`, `index.tsx`. |

## API pública exportada (funções)

Use imports nomeados a partir de `react-native-lina-pay-sdk` (a API pública está em `src/index.tsx`):

- `configure`
- `getParticipants`
- `createConsent`
- `createPayment`
- `createEnrollment`
- `registerDevice`
- `getEnrollmentList`
- `revokeEnrollment`
- `createPaymentWithEnrollment`

Exportações de tipo e classe: ver lista em [reference.md](./reference.md) (espelha `src/index.tsx` e `src/types/index.ts`).

**Nota sobre o README:** alguns trechos mostram `import LinaPaySdk from 'react-native-lina-pay-sdk'` e `LinaPaySdk.configure(...)`. O entrypoint atual exporta funções nomeadas (`import { configure, ... }`). Prefira o padrão nomeado; se o usuário copiar o README literalmente, avise que pode ser necessário ajustar o import.

## Fluxos principais (ordem lógica)

1. **Pagamento com redirect (fluxo clássico):** `configure` (se não for HML/padrão desejado) → `getParticipants` (opcional para UX) → `createConsent` → abrir `redirectUrl` → callback com `state`, `code`, `idToken` → `createPayment`.
2. **Enrollment:** `createEnrollment` → redirect → `registerDevice` com callback → `getEnrollmentList` / `revokeEnrollment` conforme necessário.
3. **Pagamento com enrollment:** enrollment `AUTHORISED` → `createPaymentWithEnrollment` com `fidoSignOptions` (inclui `rp` e `platform`).

## Boas práticas inferidas da documentação

- Sempre tratar `LinaPayError` para mensagens e `statusCode` HTTP quando existir.
- Credenciais: `subtenantId` e `subtenantSecret` em todos os métodos que recebem `LinaPayCredentials`.
- Valores monetários: no consent, `payment.value` é `number` (positivo); na resposta de pagamento, `value` vem como `string` na API (`CreatePaymentResponse`).
- Plataforma para enrollment/dispositivo: mapear `Platform.OS` para `'IOS' | 'ANDROID' | 'WEB'` (ou o conjunto completo em `PlatformEnrollment` no reference).

## Manutenção desta skill

- Se o README ou as exportações em `src/index.tsx` mudarem, atualize a tabela “Mapa da documentação” e o [reference.md](./reference.md).
- Mantenha o corpo deste arquivo enxuto; detalhes de tipos pertencem ao `reference.md`.
