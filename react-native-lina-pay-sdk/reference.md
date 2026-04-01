# Referência de tipagens — react-native-lina-pay-sdk

Fonte primária: `src/types/*.ts`, `src/config/environment.ts`, `src/utils/http.utils.ts` e exportações em `src/index.tsx`.  
Este documento consolida as tipagens citadas no [README.md](../../README.md) e as definições reais do código.

## Índice

1. [Nomes exportados pelo pacote](#nomes-exportados-pelo-pacote)
2. [Credenciais e configuração](#credenciais-e-configuração)
3. [Participantes](#participantes)
4. [Consentimento (`createConsent`)](#consentimento-createconsent)
5. [Pagamento (`createPayment`)](#pagamento-createpayment)
6. [Enrollment](#enrollment)
7. [Pagamento com enrollment (`createPaymentWithEnrollment`)](#pagamento-com-enrollment-createpaymentwithenrollment)
8. [Erros](#erros)
9. [Tipos internos relevantes (não reexportados em `index.tsx`)](#tipos-internos-relevantes-não-reexportados-em-indextsx)

---

## Nomes exportados pelo pacote

Funções (default module exports nomeados em `src/index.tsx`):

`configure`, `getParticipants`, `createConsent`, `createPayment`, `createEnrollment`, `registerDevice`, `getEnrollmentList`, `revokeEnrollment`, `createPaymentWithEnrollment`

Tipos e classe exportados em `src/index.tsx`:

| Nome exportado | Origem / observação |
|----------------|---------------------|
| `PaymentWithEnrollmentResponse` | Alias de `RetornoJsrPaymentDto` |
| `PaymentWithEnrollmentRequest` | `enrollment.types.ts` |
| `RevokeEnrollmentResponse` | `enrollment.types.ts` |
| `CreateEnrollmentResponse` | `enrollment.types.ts` |
| `CreateEnrollmentRequest` | `enrollment.types.ts` |
| `RegisterDeviceRequest` | `enrollment.types.ts` |
| `CreatePaymentResponse` | `payment.types.ts` |
| `CreateConsentResponse` | `consent.types.ts` (resposta do fluxo de consentimento de pagamento) |
| `CreateConsentRequest` | `consent.types.ts` |
| `CreatePaymentRequest` | `payment.types.ts` |
| `PlatformEnrollment` | `enrollment.types.ts` |
| `LinaPayCredentials` | `auth.types.ts` |
| `PaymentCreditor` | `payment.types.ts` |
| `PaymentDebitor` | `payment.types.ts` |
| `EnrollmentList` | `enrollment.types.ts` |
| `LinaPayConfig` | `environment.ts` |
| `PaymentStatus` | `payment.types.ts` |
| `PaymentItem` | `payment.types.ts` |
| `PaymentType` | `payment.types.ts` |
| `Participant` | `participants.types.ts` |
| `AccountType` | `consent.types.ts` |
| `PersonType` | `consent.types.ts` |
| `Enrollment` | Alias público de `EnrollmentUserResponseDto` (elemento de `EnrollmentList.enrollments`). **Não confundir** com o retorno estrutural de `registerDevice` — ver [Colisão de nome `Enrollment`](#colisão-de-nome-enrollment). |
| `Schedule` | `consent.types.ts` |
| `Platform` | `consent.types.ts` |
| `Creditor` | `consent.types.ts` |
| `Payment` | `consent.types.ts` (objeto de pagamento dentro de `CreateConsentRequest`) |
| `Debitor` | `consent.types.ts` |
| `LinaPayError` | classe em `http.utils.ts` |

Tipos definidos em `src/types/index.ts` mas **não** listados em `src/index.tsx` (não fazem parte da API pública atual do entrypoint): por exemplo `SingleSchedule`, `DailySchedule`, `WeeklySchedule`, `MonthlySchedule`, `CustomSchedule`, `DayOfWeek`, `PaymentItemStatus`, `EnrollmentListResponse`, `RetornoJsrPaymentDto` (use o alias `PaymentWithEnrollmentResponse`), `CreatePaymentWithEnrollmentResponse` (wrapper com `data`, `message`, `type` em `enrollment.types.ts`).

### Colisão de nome `Enrollment`

- O pacote exporta o tipo `Enrollment` como **alias de `EnrollmentUserResponseDto`** (itens retornados em `getEnrollmentList`).
- A função **`registerDevice`** tipa o retorno como a interface **`Enrollment`** definida em `enrollment.types.ts` (campos `enrollmentId`, `creationDateTime`, `status` discriminado, `loggedUser`, etc.). Em tipagem estrutural TypeScript isso pode compatibilizar em alguns casos, mas semanticamente são modelos diferentes; use a interface completa na secção [Enrollment — interface de registro](#enrollment--interface-de-registro-registerdevice) ao documentar o callback FIDO.

---

## Credenciais e configuração

### `LinaPayCredentials` (`auth.types.ts`)

```typescript
export interface LinaPayCredentials {
  subtenantId: string;
  subtenantSecret: string;
}
```

### `LinaPayConfig` (`environment.ts`)

```typescript
export interface LinaPayConfig {
  iamBaseUrl: string;
  apiBaseUrl: string;
}
```

A função pública `configure` aceita `Partial<LinaPayConfig>` (ver `config.controller.ts` / `environment.ts`).

**Defaults no código:** `DEFAULT_CONFIG` em `environment.ts` usa `https://iam.linaob.com.br` e `https://embedded-payment-manager.linaob.com.br`. O README exemplifica URLs de **homologação** (`*.hml.linaob.com.br`) e **produção** (`*.prod.linaob.com.br`); alinhe com o ambiente contratado.

---

## Participantes

### `Participant` (`participants.types.ts`)

```typescript
export interface Participant {
  organisationId: string;
  organisationName: string;
  legalEntityName: string;
  registrationNumber: string;
  authorisationServerId: string;
  customerFriendlyName: string;
  customerFriendlyLogoUri: string;
}
```

**Assinatura:** `getParticipants(credentials: LinaPayCredentials): Promise<Participant[]>`

---

## Consentimento (`createConsent`)

### Tipos literais

```typescript
export type PersonType = 'PESSOA_NATURAL' | 'PESSOA_JURIDICA';
export type AccountType = 'CACC' | 'SLRY' | 'SVGS' | 'TRAN';
export type DayOfWeek =
  | 'SEGUNDA_FEIRA'
  | 'TERCA_FEIRA'
  | 'QUARTA_FEIRA'
  | 'QUINTA_FEIRA'
  | 'SEXTA_FEIRA'
  | 'SABADO'
  | 'DOMINGO';
export type Platform = 'APP' | 'WEB';
```

### `Creditor`

```typescript
export interface Creditor {
  name: string;
  personType: PersonType;
  cpfCnpj: string;
  accountNumber: string;
  accountIssuer: string;
  accountPixKey: string;
  accountIspb: string;
  accountType: AccountType;
}
```

### `Debitor`

```typescript
export interface Debitor {
  accountNumber?: string;
  accountIssuer?: string;
  accountIspb?: string;
  accountType?: AccountType;
}
```

### Agendamento (`Schedule` e variantes — `consent.types.ts`)

```typescript
export interface SingleSchedule {
  date: string; // YYYY-MM-DD
}

export interface DailySchedule {
  startDate: string;
  quantity: number;
}

export interface WeeklySchedule {
  dayOfWeek: DayOfWeek;
  startDate: string;
  quantity: number;
}

export interface MonthlySchedule {
  dayOfMonth: number; // 1-31
  startDate: string;
  quantity: number;
}

export interface CustomSchedule {
  dates: string[];
  additionalInformation?: string;
}

export interface Schedule {
  single?: SingleSchedule;
  daily?: DailySchedule;
  weekly?: WeeklySchedule;
  monthly?: MonthlySchedule;
  custom?: CustomSchedule;
}
```

### `Payment` (dentro de `CreateConsentRequest`)

```typescript
export interface Payment {
  details?: string;
  externalId?: string;
  redirectUri: string;
  cpfCnpj?: string;
  value: number;
  creditor: Creditor;
  debitor?: Debitor;
  originalRecurringPaymentId?: string;
  schedule?: Schedule;
  txId?: string | string[];
}
```

### `CreateConsentRequest`

```typescript
export interface CreateConsentRequest {
  organisationId: string;
  authorisationServerId: string;
  payment: Payment;
  redirectUri?: string;
  platform?: Platform;
}
```

### `CreateConsentResponse`

```typescript
export interface CreateConsentResponse {
  consentId: string;
  redirectUrl: string;
  id: string;
}
```

**Assinatura:** `createConsent(credentials: LinaPayCredentials, payload: CreateConsentRequest): Promise<CreateConsentResponse>`

---

## Pagamento (`createPayment`)

### Tipos literais

```typescript
export type PaymentType =
  | 'NOW'
  | 'SINGLE'
  | 'DAILY'
  | 'WEEKLY'
  | 'MONTHLY'
  | 'CUSTOM';

export type PaymentStatus =
  | 'PENDENTE'
  | 'EM_PROCESSAMENTO'
  | 'CONSUMIDO'
  | 'EXPIRADO'
  | 'CANCELADO'
  | 'ERRO_NA_DETENTORA'
  | 'ERRO';

export type PaymentItemStatus =
  | 'PENDENTE'
  | 'REJEITADO'
  | 'EM_PROCESSAMENTO'
  | 'PAGO'
  | 'EXPIRADO'
  | 'CANCELADO'
  | 'ERRO_NA_DETENTORA'
  | 'ERRO'
  | 'PROGRAMADO'
  | 'AGUARDANDO_VALOR'
  | 'PRONTO_PARA_ENVIO';
```

### `PaymentItem`

```typescript
export interface PaymentItem {
  endToEndId: string;
  id: string;
  dueDate: string;
  externalComment?: string;
  txId: string | null;
  status: PaymentItemStatus;
  externalPaymentId: string;
}
```

### `PaymentCreditor` / `PaymentDebitor`

```typescript
export interface PaymentCreditor {
  name: string;
  personType: 'PESSOA_NATURAL' | 'PESSOA_JURIDICA';
  cpfCnpj: string;
  accountIspb: string;
  accountIssuer: string;
  accountNumber: string;
  accountPixKey: string | null;
  accountType: 'CACC' | 'SLRY' | 'SVGS' | 'TRAN';
}

export interface PaymentDebitor {
  accountIspb: string;
  accountIssuer: string;
  accountNumber: string;
  accountType: 'CACC' | 'SLRY' | 'SVGS' | 'TRAN';
}
```

### `CreatePaymentRequest`

```typescript
export interface CreatePaymentRequest {
  state: string;
  code: string;
  idToken: string;
  tenantId: string;
}
```

### `CreatePaymentResponse`

```typescript
export interface CreatePaymentResponse {
  id: string;
  type: PaymentType;
  createDateTime: string;
  externalId?: string;
  externalComment?: string;
  lastChangedDateTime: string;
  status: PaymentStatus;
  tenantId: string;
  cpfCnpj: string;
  redirectUri: string;
  value: string;
  consentId: string;
  creditor: PaymentCreditor;
  debitor?: PaymentDebitor;
  payments: PaymentItem[];
}
```

**Assinatura:** `createPayment(credentials: LinaPayCredentials, payload: CreatePaymentRequest): Promise<CreatePaymentResponse>`

**Nota (README):** o README menciona status como `PAGO` em exemplo; o tipo `PaymentStatus` no código não inclui `PAGO` — pode ser documentação desatualizada em relação à API. Priorize o union acima para typechecking.

---

## Enrollment

### `PlatformEnrollment`

```typescript
export type PlatformEnrollment =
  | 'ANDROID'
  | 'IOS'
  | 'WEB'
  | 'BROWSER'
  | 'CROSS_PLATFORM';
```

### `PaymentDebitorDto` (conta no enrollment)

```typescript
export interface PaymentDebitorDto {
  accountNumber: string;
  accountIssuer: string;
  accountIspb: string;
  accountType: AccountType;
}
```

### `EnrollmentDto`

```typescript
export interface EnrollmentDto {
  externalId?: string;
  document?: string;
  deviceName?: string;
  debtor?: PaymentDebitorDto;
}
```

### `CreateEnrollmentRequest`

```typescript
export interface CreateEnrollmentRequest {
  paymentId?: string;
  organisationId: string;
  authorisationServerId: string;
  enrollment: EnrollmentDto;
  riskSignals?: RiskSignalsDto;
  redirectUri: string;
}
```

(`RiskSignalsDto` e correlatos estão em `enrollment.types.ts` — usados para sinais de risco opcionais.)

### `CreateEnrollmentResponse`

```typescript
export interface CreateEnrollmentResponse {
  consentId?: string | null;
  id: string;
  redirectUrl?: string;
}
```

**Assinatura:** `createEnrollment(credentials, payload): Promise<CreateEnrollmentResponse>`

### `RegisterDeviceRequest`

```typescript
export interface RegisterDeviceRequest {
  state: string;
  code: string;
  idToken: string;
  tenantId: string;
  platform: PlatformEnrollment;
  rp?: string;
}
```

### Enrollment — interface de registro (`registerDevice`)

Interface `Enrollment` em `enrollment.types.ts` (retorno de `registerDevice` no service/controller):

```typescript
export interface Enrollment {
  enrollmentId: string;
  creationDateTime: string;
  status:
    | 'AWAITING_RISK_SIGNALS'
    | 'AWAITING_ACCOUNT_HOLDER_VALIDATION'
    | 'AWAITING_ENROLLMENT'
    | 'AUTHORISED'
    | 'REJECTED'
    | 'REVOKED';
  statusUpdateDateTime: string;
  permissions: 'PAYMENTS_INITIATE';
  expirationDateTime: string | null;
  loggedUser: LoggedUser;
  businessEntity: BusinessEntity | null;
  debtorAccount: AccountEntity | null;
  enrollmentName: string | null;
}
```

(`LoggedUser`, `BusinessEntity`, `AccountEntity` são interfaces internas no mesmo arquivo.)

**Assinatura:** `registerDevice(credentials, payload): Promise<Enrollment>` (tipo completo acima).

### `EnrollmentUserResponseDto` (exportado como nome `Enrollment` no `index.tsx`)

Itens dentro de `EnrollmentList.enrollments`:

```typescript
export interface EnrollmentUserResponseDto {
  externalId: string;
  createDateTime: Date;
  lastChangedDateTime: Date;
  organisationId: string;
  authorisationServerId: string;
  enrollmentId: string;
  status: string;
  document: string;
  debtor: PaymentDebitorDto;
}
```

### `EnrollmentList`

```typescript
export interface EnrollmentList {
  id: string;
  username: string;
  subTenant: string;
  enrollments: EnrollmentUserResponseDto[];
}
```

**Assinatura:** `getEnrollmentList(credentials, cpf: string): Promise<EnrollmentList>`

### `RevokeEnrollmentResponse`

```typescript
export interface RevokeEnrollmentResponse {
  data: string;
  message: string;
  type: 'success' | 'error';
  statusCode: number;
}
```

**Assinatura:** `revokeEnrollment(credentials, enrollmentId: string): Promise<RevokeEnrollmentResponse>`

---

## Pagamento com enrollment (`createPaymentWithEnrollment`)

### `FidoSignOptionsDto` (interno ao módulo; forma usada em `PaymentWithEnrollmentRequest`)

```typescript
interface FidoSignOptionsDto {
  rp?: string;
  platform: 'ANDROID' | 'BROWSER' | 'CROSS_PLATFORM' | 'IOS';
}
```

### `PaymentWithEnrollmentRequest`

```typescript
export interface PaymentWithEnrollmentRequest {
  fidoSignOptions: FidoSignOptionsDto;
  enrollmentId: string;
  organisationId: string;
  authorisationServerId: string;
  paymentId?: string;
  payment?: PaymentDto; // interno: ver enrollment.types.ts
}
```

O objeto `payment` interno (`PaymentDto`) espelha campos do pagamento (detalhes, credor, `txId` como `string[]`, agendamento via `PaymentSchedulerDto`, etc.) — ver `enrollment.types.ts` linhas ~246–312.

### `RetornoJsrPaymentDto` (exportado como `PaymentWithEnrollmentResponse`)

```typescript
export interface RetornoJsrPaymentDto {
  consentId: string;
  id: string;
  fidoSignOptions: FidoSignOptionsResponseDto;
}
```

`FidoSignOptionsResponseDto` contém `data` (challenge, timeout, rpId, allowCredentials, userVerification, extensions), `links`, `meta`, `passKeyId?`, `transports?` — ver `enrollment.types.ts`.

**Assinatura:** `createPaymentWithEnrollment(credentials, payload): Promise<RetornoJsrPaymentDto>`

### Wrapper API (não exportado no `index.tsx`)

```typescript
export interface CreateConsentResponse {
  data: RetornoJsrPaymentDto;
  message: string;
  type: 'success' | 'error';
}
```

**Nota:** em `enrollment.types.ts` este tipo também se chama `CreateConsentResponse`, mas é **distinto** de `CreateConsentResponse` de `consent.types.ts`. O barrel `src/types/index.ts` exporta o da enrollment como `CreatePaymentWithEnrollmentResponse`.

---

## Erros

### `LinaPayError` (`http.utils.ts`)

```typescript
export class LinaPayError extends Error {
  constructor(
    message: string,
    public statusCode?: number,
    public originalError?: unknown
  ) {
    super(message);
    this.name = 'LinaPayError';
  }
}
```

---

## Tipos internos relevantes (não reexportados em `index.tsx`)

### Autenticação (`auth.types.ts`)

```typescript
export interface TokenResponse {
  access_token: string;
  token_type: string;
  expires_in: number;
}

export interface CachedToken {
  accessToken: string;
  expiresAt: number;
}
```

### FIDO (`fido.types.ts`)

```typescript
export interface FIDOCredential { /* ... */ }
export interface FIDOOptions { /* ... */ }
export interface FidoSignOptions { /* ... */ }
export interface FIDOAssertion {
  id: string;
  rawId: string;
  response: {
    authenticatorData: string;
    clientDataJSON: string;
    signature: string;
    userHandle: string;
  };
  type: string;
  clientExtensionResults: {};
  authenticatorAttachment: string;
}
```

### Participantes — resposta bruta da API (`participants.types.ts`)

`ParticipantFull`, `AuthorisationServer`, `ApiResource`, `ParticipantsApiResponse` — usados internamente no service; não exportados no `index.tsx`.

### `AuthorisePaymentRequestDto` (`enrollment.types.ts`)

Usado no fluxo interno de autorização com assertion FIDO.

---

## Enum relacionado a agendamento no enrollment

```typescript
export enum DayWeekType {
  SEGUNDA_FEIRA = 'SEGUNDA_FEIRA',
  TERCA_FEIRA = 'TERCA_FEIRA',
  QUARTA_FEIRA = 'QUARTA_FEIRA',
  QUINTA_FEIRA = 'QUINTA_FEIRA',
  SEXTA_FEIRA = 'SEXTA_FEIRA',
  SABADO = 'SABADO',
  DOMINGO = 'DOMINGO',
}
```

---

*Última sincronização sugerida com o código-fonte ao alterar `src/types` ou `src/index.tsx`.*
