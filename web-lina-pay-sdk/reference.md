# Referência de tipagens — @lina-openx/web-lina-pay-sdk

Documento de apoio ao **README.md** e ao skill `web-lina-pay-sdk`. Consolida tipos citados na documentação do SDK e os **exportados publicamente** via `src/types/index.ts` e `src/index.ts`. Ajustes finos podem existir no código-fonte (`src/types/*.ts`).

## Índice

1. [Configuração](#configuração)
2. [Autenticação e credenciais](#autenticação-e-credenciais)
3. [Erros](#erros)
4. [Consentimento (`consent.types`)](#consentimento-consenttypes)
5. [Pagamento — OAuth e resposta (`payment.types`)](#pagamento--oauth-e-resposta-paymenttypes)
6. [Solicitação de pagamento — DTO (`CreatePaymentRequestDTO`)](#solicitação-de-pagamento--dto-createpaymentrequestdto)
7. [Participantes (`participants.types`)](#participantes-participantstypes)
8. [Enrollment (`enrollment.types`)](#enrollment-enrollmenttypes)
9. [Aliases de export do pacote](#aliases-de-export-do-pacote)

---

## Configuração

### `LinaPayConfig`

```typescript
interface LinaPayConfig {
  iamBaseUrl: string
  apiBaseUrl: string
}
```

Usado por `configure(config: Partial<LinaPayConfig>)`.

---

## Autenticação e credenciais

### `LinaPayCredentials`

```typescript
interface LinaPayCredentials {
  subtenantId: string
  subtenantSecret: string
}
```

Parâmetro `credentials` da maioria dos métodos do SDK.

### Tipos internos (não exportados pelo `index` do pacote, úteis em diagnóstico)

```typescript
interface TokenResponse {
  access_token: string
  token_type: string
  expires_in: number
}

interface CachedToken {
  accessToken: string
  expiresAt: number
}
```

---

## Erros

### `LinaPayError` (exportado em `src/index.ts`)

```typescript
class LinaPayError extends Error {
  constructor(
    message: string,
    public statusCode?: number,
    public originalError?: unknown,
  )
  name: 'LinaPayError'
}
```

---

## Consentimento (`consent.types`)

### Tipos literais

```typescript
type PersonType = 'PESSOA_NATURAL' | 'PESSOA_JURIDICA'
type AccountType = 'CACC' | 'SLRY' | 'SVGS' | 'TRAN'
type DayOfWeek =
  | 'SEGUNDA_FEIRA'
  | 'TERCA_FEIRA'
  | 'QUARTA_FEIRA'
  | 'QUINTA_FEIRA'
  | 'SEXTA_FEIRA'
  | 'SABADO'
  | 'DOMINGO'
type Platform = 'APP' | 'WEB'
```

### `Creditor`

```typescript
interface Creditor {
  name: string
  personType: PersonType
  cpfCnpj: string
  accountNumber: string
  accountIssuer: string
  accountPixKey: string
  accountIspb: string
  accountType: AccountType
}
```

### `Debitor`

```typescript
interface Debitor {
  accountNumber?: string
  accountIssuer?: string
  accountIspb?: string
  accountType?: AccountType
}
```

### Agendas

```typescript
interface SingleSchedule {
  date: string // YYYY-MM-DD
}

interface DailySchedule {
  startDate: string
  quantity: number
}

interface WeeklySchedule {
  dayOfWeek: DayOfWeek
  startDate: string
  quantity: number
}

interface MonthlySchedule {
  dayOfMonth: number // 1–31
  startDate: string
  quantity: number
}

interface CustomSchedule {
  dates: string[]
  additionalInformation?: string
}

interface Schedule {
  single?: SingleSchedule
  daily?: DailySchedule
  weekly?: WeeklySchedule
  monthly?: MonthlySchedule
  custom?: CustomSchedule
}
```

### `Payment` (dentro do consentimento)

```typescript
interface Payment {
  details?: string
  externalId?: string
  redirectUri: string
  cpfCnpj?: string
  value: number
  creditor: Creditor
  debitor?: Debitor
  originalRecurringPaymentId?: string
  schedule?: Schedule
  txId?: string | string[]
}
```

### `CreateConsentRequest` / `CreateConsentResponse`

```typescript
interface CreateConsentRequest {
  organisationId: string
  authorisationServerId: string
  payment: Payment
  redirectUri?: string
  platform?: Platform
}

interface CreateConsentResponse {
  consentId: string
  redirectUrl: string
  id: string
}
```

---

## Pagamento — OAuth e resposta (`payment.types`)

### `CreatePayment` / `CreatePaymentRequest`

No código, a interface OAuth chama-se `CreatePayment`. O pacote exporta o alias:

```typescript
type CreatePaymentRequest = CreatePayment

interface CreatePayment {
  state: string
  code: string
  idToken: string
  tenantId: string
}
```

Payload do método **`createPayment`**.

### Tipos de domínio de pagamento

```typescript
type PaymentType = 'NOW' | 'SINGLE' | 'DAILY' | 'WEEKLY' | 'MONTHLY' | 'CUSTOM'

type PaymentStatus =
  | 'PENDENTE'
  | 'EM_PROCESSAMENTO'
  | 'CONSUMIDO'
  | 'EXPIRADO'
  | 'CANCELADO'
  | 'ERRO_NA_DETENTORA'
  | 'ERRO'

type PaymentItemStatus =
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
  | 'PRONTO_PARA_ENVIO'
```

### `PaymentItem`, `PaymentCreditor`, `PaymentDebitor`

```typescript
interface PaymentItem {
  endToEndId: string
  id: string
  dueDate: string
  externalComment?: string
  txId: string | null
  status: PaymentItemStatus
  externalPaymentId: string
}

interface PaymentCreditor {
  name: string
  personType: 'PESSOA_NATURAL' | 'PESSOA_JURIDICA'
  cpfCnpj: string
  accountIspb: string
  accountIssuer: string
  accountNumber: string
  accountPixKey: string | null
  accountType: 'CACC' | 'SLRY' | 'SVGS' | 'TRAN'
}

interface PaymentDebitor {
  accountIspb: string
  accountIssuer: string
  accountNumber: string
  accountType: 'CACC' | 'SLRY' | 'SVGS' | 'TRAN'
}
```

### `CreatePaymentResponse`

Retorno de **`createPayment`** (dados após unwrap da API):

```typescript
interface CreatePaymentResponse {
  id: string
  type: PaymentType
  createDateTime: string
  externalId?: string
  externalComment?: string
  lastChangedDateTime: string
  status: PaymentStatus
  tenantId: string
  cpfCnpj: string
  redirectUri: string
  value: string
  consentId: string
  creditor: PaymentCreditor
  debitor?: PaymentDebitor
  payments: PaymentItem[]
}
```

### `PaymentRequestData`

Retorno de **`createPaymentRequest`**:

```typescript
interface PaymentRequestData {
  redirectUri: string
  id: string
}
```

---

## Solicitação de pagamento — DTO (`CreatePaymentRequestDTO`)

Payload do método **`createPaymentRequest`**. O README descreve validação Zod em `src/utils/payment.utils.ts` (incluindo necessidade de chave `schedule`, podendo ser `{}`).

```typescript
interface CreatePaymentRequestDTO {
  details: string
  txId?: string | string[]
  externalId?: string
  redirectUri: string
  cpfCnpj: string
  value: number
  creditor: Creditor
  schedule?: {
    daily?: { startDate: string; quantity: string }
    monthly?: { startDate: string; quantity: string; dayOfMonth: string }
    weekly?: { startDate: string; quantity: string; dayOfWeek: string }
    single?: { date: string }
    custom?: { dates: string[]; additionalInformation: string }
  }
}
```

**Nota:** na validação Zod, `quantity` / `dayOfMonth` em schedules são tratados como **número**; a interface TypeScript acima pode divergir — em caso de dúvida, seguir o schema em `payment.utils.ts`.

---

## Participantes (`participants.types`)

### `Participant` (formato retornado pelo SDK)

```typescript
interface Participant {
  organisationId: string
  organisationName: string
  legalEntityName: string
  registrationNumber: string
  authorisationServerId: string
  customerFriendlyName: string
  customerFriendlyLogoUri: string
}
```

Tipos brutos da API (`ParticipantFull`, `AuthorisationServer`, etc.) existem no mesmo arquivo e **não** são reexportados pelo `src/types/index.ts`.

---

## Enrollment (`enrollment.types`)

### `PlatformEnrollment`

```typescript
type PlatformEnrollment =
  | 'ANDROID'
  | 'IOS'
  | 'WEB'
  | 'BROWSER'
  | 'CROSS_PLATFORM'
```

### `PaymentDebitorDto`

```typescript
interface PaymentDebitorDto {
  accountNumber: string
  accountIssuer: string
  accountIspb: string
  accountType: AccountType // de consent.types
}
```

### `EnrollmentDto`

```typescript
interface EnrollmentDto {
  externalId?: string
  document?: string
  deviceName?: string
  debtor?: PaymentDebitorDto
}
```

### `RiskSignalsDto` e `ScreenDimensionDto`

```typescript
interface ScreenDimensionDto {
  width: number
  height: number
}

interface RiskSignalsDto {
  deviceId: string
  deviceName?: string
  osVersion?: string
  userTimeZoneOffset: string
  language: string
  screenDimensions?: ScreenDimensionDto
  accountTenure: string
  elapsedTimeSinceBoot?: number | null
  screenBrightness?: number | null
  isRootedDevice?: boolean | null
}
```

### `CreateEnrollmentRequest`

```typescript
interface CreateEnrollmentRequest {
  paymentId?: string
  organisationId: string
  authorisationServerId: string
  enrollment: EnrollmentDto
  riskSignals?: RiskSignalsDto
  redirectUri: string
}
```

### `CreateEnrollmentResponse`

```typescript
interface CreateEnrollmentResponse {
  consentId?: string | null
  id: string
  redirectUrl?: string
}
```

### `RegisterDeviceRequest`

```typescript
interface RegisterDeviceRequest {
  state: string
  code: string
  idToken: string
  tenantId: string
  platform: PlatformEnrollment
  rp?: string
}
```

### `EnrollmentList` / item da lista

```typescript
interface EnrollmentList {
  id: string
  username: string
  subTenant: string
  enrollments: EnrollmentUserResponseDto[]
}

interface EnrollmentUserResponseDto {
  externalId: string
  createDateTime: Date
  lastChangedDateTime: Date
  organisationId: string
  authorisationServerId: string
  enrollmentId: string
  status: string
  document: string
  debtor: PaymentDebitorDto
}
```

O pacote exporta `EnrollmentUserResponseDto` com o nome **`Enrollment`**:

```typescript
type Enrollment = EnrollmentUserResponseDto
```

### `RevokeEnrollmentResponse`

```typescript
interface RevokeEnrollmentResponse {
  data: string
  message: string
  type: 'success' | 'error'
  statusCode: number
}
```

### `PaymentWithEnrollmentRequest`

Campos principais exportados (há tipos internos `PaymentDto`, agendadores e FIDO no mesmo arquivo fonte):

```typescript
interface PaymentWithEnrollmentRequest {
  fidoSignOptions: {
    rp?: string
    platform: 'ANDROID' | 'BROWSER' | 'CROSS_PLATFORM' | 'IOS'
  }
  enrollmentId: string
  organisationId: string
  authorisationServerId: string
  paymentId?: string
  payment?: {
    details: string
    externalId?: string
    redirectUri: string
    cpfCnpj: string
    value: number
    creditor: {
      name: string
      personType: PersonType
      cpfCnpj: string
      accountNumber: string
      accountIssuer: string
      accountPixKey: string | null
      accountIspb: string
      accountType: AccountType
    }
    debitor?: PaymentDebitorDto | null
    originalRecurringPaymentId?: string
    schedule?: {
      single?: { date: string } | null
      daily?: { startDate: string; quantity: number } | null
      weekly?: { dayOfWeek: DayWeekType; startDate: string; quantity: number } | null
      monthly?: { dayOfMonth: number; startDate: string; quantity: number } | null
      custom?: { dates: string[]; additionalInformation: string } | null
    } | null
    txId?: string[]
  }
}
```

`DayWeekType` é um `enum` espelhando os valores `SEGUNDA_FEIRA` … `DOMINGO`.

### `RetornoJsrPaymentDto`

Retorno típico de **`createPaymentWithEnrollment`** (estrutura pública relevante):

```typescript
interface RetornoJsrPaymentDto {
  consentId: string
  id: string
  fidoSignOptions: {
    data: {
      challenge: string
      timeout: number
      rpId: string
      allowCredentials?: { id: string; type: 'public-key' }[]
      userVerification?: 'required' | 'preferred' | 'discouraged'
      extensions?: { additionalProp1: Record<string, unknown> }
    }
    links: { self: string; next: string; last: string }
    meta: { totalRecords: number; totalPages: number }
    passKeyId?: string
    transports?: string[]
  }
}
```

### Alias `CreatePaymentWithEnrollmentResponse`

```typescript
type CreatePaymentWithEnrollmentResponse = CreateConsentResponse
```

(`CreateConsentResponse` neste arquivo de enrollment é o wrapper de API para o fluxo JSR; ver `enrollment.types.ts`.)

---

## Aliases de export do pacote

Conforme `src/types/index.ts`:

| Nome exportado | Origem |
|----------------|--------|
| `CreatePaymentWithEnrollmentResponse` | `CreateConsentResponse` (enrollment.types) |
| `Enrollment` | `EnrollmentUserResponseDto` |

---

## Manutenção

Ao atualizar o **README.md**, revise este arquivo e o **SKILL.md** para manter exemplos, nomes de tipos e fluxos alinhados.
