# Referência de Tipos - @lina-openx/web-lina-pay-sdk

## Credenciais

```typescript
interface LinaPayCredentials {
  subtenantId: string;
  subtenantSecret: string;
}
```

## Configuração

```typescript
interface LinaPayConfig {
  iamBaseUrl: string;
  apiBaseUrl: string;
}

function configure(config: Partial<LinaPayConfig>): void;
```

**URLs padrão (homologação):**
- IAM: `https://iam.linaob.com.br`
- API: `https://embedded-payment-manager.linaob.com.br`

**URLs de produção:**
- IAM: `https://iam.prod.linaob.com.br`
- API: `https://embedded-payment-manager.prod.linaob.com.br`

## Consent

```typescript
interface CreateConsentRequest {
  organisationId: string;
  authorisationServerId: string;
  payment: Payment;
  redirectUri?: string;
  platform?: "APP" | "WEB";
}

interface Payment {
  redirectUri: string;
  value: number;
  creditor: Creditor;
  schedule?: PaymentSchedule;
}

interface Creditor {
  name: string;
  personType: "PESSOA_NATURAL" | "PESSOA_JURIDICA";
  cpfCnpj: string;
  accountNumber: string;
  accountIssuer: string;
  accountIspb: string;
  accountType: "CACC" | "SVGS";
  accountPixKey?: string;
}

interface PaymentSchedule {
  monthly?: {
    dayOfMonth: number;
    startDate: string; // formato ISO: 'YYYY-MM-DD'
    quantity: number;
  };
}

interface CreateConsentResponse {
  consentId: string;
  redirectUrl: string;
}

function createConsent(
  credentials: LinaPayCredentials,
  payload: CreateConsentRequest
): Promise<CreateConsentResponse>;
```

## Payment

```typescript
interface CreatePaymentRequest {
  state: string;
  code: string;
  idToken: string;
  tenantId: string;
}

interface CreatePaymentResponse {
  id: string;
  status: string;
  value: number;
}

function createPayment(
  credentials: LinaPayCredentials,
  payload: CreatePaymentRequest
): Promise<CreatePaymentResponse>;
```

## Enrollment

```typescript
interface CreateEnrollmentRequest {
  organisationId: string;
  authorisationServerId: string;
  enrollment: {
    deviceName: string;
    document: string;
    debtor: {
      accountIspb: string;
      accountIssuer: string;
      accountNumber: string;
      accountType: "CACC" | "SVGS";
    };
  };
  redirectUri: string;
  paymentId?: string;
}

interface CreateEnrollmentResponse {
  consentId: string;
  redirectUrl: string;
}

function createEnrollment(
  credentials: LinaPayCredentials,
  payload: CreateEnrollmentRequest
): Promise<CreateEnrollmentResponse>;
```

## Register Device

```typescript
type DevicePlatform = "ANDROID" | "IOS" | "WEB" | "BROWSER" | "CROSS_PLATFORM";

interface RegisterDeviceRequest {
  state: string;
  code: string;
  idToken: string;
  tenantId: string;
  platform: DevicePlatform;
  rp?: string;
}

interface RegisterDeviceResponse {
  id: string;
  deviceName: string;
}

function registerDevice(
  credentials: LinaPayCredentials,
  payload: RegisterDeviceRequest
): Promise<RegisterDeviceResponse>;
```

## Enrollment List

```typescript
interface Enrollment {
  id: string;
  deviceName: string;
}

function getEnrollmentList(
  credentials: LinaPayCredentials,
  cpf: string
): Promise<Enrollment[]>;
```

## Revoke Enrollment

```typescript
interface RevokeEnrollmentResponse {
  status: string;
}

function revokeEnrollment(
  credentials: LinaPayCredentials,
  enrollmentId: string
): Promise<RevokeEnrollmentResponse>;
```

## Payment with Enrollment (FIDO2)

```typescript
interface PaymentWithEnrollmentRequest {
  enrollmentId: string;
  organisationId: string;
  authorisationServerId: string;
  payment: Payment;
  fidoSignOptions: {
    rp: string;
    platform: DevicePlatform;
  };
  paymentId?: string;
}

function createPaymentWithEnrollment(
  credentials: LinaPayCredentials,
  payload: PaymentWithEnrollmentRequest
): Promise<CreatePaymentResponse>;
```

## Participants

```typescript
interface Participant {
  name: string;
  ispb: string;
}

function getParticipants(
  credentials: LinaPayCredentials
): Promise<Participant[]>;
```

## Tratamento de Erros

```typescript
class LinaPayError extends Error {
  message: string;
  statusCode: number;
}
```

Todos os métodos do SDK lançam `LinaPayError` para erros de validação (payload inválido, credenciais ausentes) e erros de comunicação HTTP.
