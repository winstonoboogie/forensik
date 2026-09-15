# Lathund – Fältnamn i Entra ID (Azure AD) Sign-in-logg

En parlör över de vanligaste fälten i en Sign-in Activity-logg (SignInLogs) från Microsoft Entra ID, t.ex. hämtad via Log Analytics/Sentinel.

## Övergripande fält

| Fält | Förklaring |
|---|---|
| `time` | Tidpunkt (UTC) då loggposten skapades/skickades till loggsystemet. |
| `resourceId` | Resurs-ID i Azure, oftast tenantens AAD-provider-sökväg. |
| `operationName` | Typ av händelse, t.ex. "Sign-in activity". |
| `operationVersion` | Version av loggschemat. |
| `category` | Loggkategori, t.ex. `SignInLogs` (interaktiva inloggningar). |
| `tenantId` | Unikt ID för Entra ID-tenanten (organisationen). |
| `resultType` | Felkod för inloggningsresultatet. `0` = lyckad. |
| `resultSignature` | Textbeskrivning av resultatet, t.ex. `SUCCESS`. |
| `durationMs` | Hur lång tid (ms) autentiseringen tog. |
| `callerIpAddress` | IP-adressen som gjorde anropet. |
| `correlationId` | Unikt ID som binder ihop relaterade loggposter för samma händelse. |
| `identity` | Visningsnamn på användaren/kontot som loggades in. |
| `level` | Allvarlighetsgrad i loggsystemet (t.ex. 4 = Informational). |
| `location` | Landskod (ISO) för inloggningens geografiska ursprung. |

## `properties`-objektet (detaljer om själva inloggningen)

| Fält | Förklaring |
|---|---|
| `id` | Unikt ID för just denna sign-in-händelse. |
| `createdDateTime` | Exakt tidpunkt (med tidszon) då inloggningen skedde. |
| `userDisplayName` | Användarens visningsnamn. |
| `userPrincipalName` | Användarens inloggningsnamn (UPN), t.ex. e-postliknande ID. |
| `userId` | Användarens unika objekt-ID (GUID) i Entra ID. |
| `agent.agentType` | Om inloggningen gjordes av en AI-agent eller inte (`notAgentic` = vanlig mänsklig inloggning). |
| `agent.agentSubjectType` | Typ av agent-subjekt, relaterat till agentType. |
| `appId` | Applikations-ID (GUID) för appen som användes vid inloggningen. |
| `appDisplayName` | Namnet på appen, t.ex. "Microsoft 365 Copilot extension". |
| `ipAddress` | IP-adress som användes vid inloggningen. |
| `ipAddressFromResourceProvider` | Ev. IP rapporterad av resursleverantören (ofta tom). |
| `status.errorCode` | Felkod, `0` = ingen fel/lyckad. |
| `status.additionalDetails` | Extra information om hur autentiseringen godkändes. |
| `clientAppUsed` | Typ av klient, t.ex. `Browser`, `Mobile Apps and Desktop clients`. |
| `userAgent` | Webbläsarens/klientens user agent-sträng. |

### `deviceDetail` (info om enheten)

| Fält | Förklaring |
|---|---|
| `deviceId` | Unikt ID för enheten i Entra ID. |
| `displayName` | Enhetens namn/hostname. |
| `operatingSystem` | Operativsystem på enheten. |
| `browser` | Webbläsare och version. |
| `isCompliant` | Om enheten uppfyller Intune/complience-policy (true/false). |
| `trustType` | Enhetens registreringstyp, t.ex. "Azure AD joined", "Hybrid Azure AD joined". |

### `location` (geografisk info, inuti `properties`)

| Fält | Förklaring |
|---|---|
| `city` | Stad enligt IP-geolokalisering. |
| `state` | Region/delstat. |
| `countryOrRegion` | Landskod. |
| `geoCoordinates` | Latitud/longitud för platsen. |

## Villkorsstyrd åtkomst (Conditional Access)

| Fält | Förklaring |
|---|---|
| `conditionalAccessStatus` | Övergripande status, t.ex. `success` = alla policyer godkända/inte blockerade. |
| `appliedConditionalAccessPolicies` | Lista över CA-policyer som utvärderades. |
| &nbsp;&nbsp;`id` / `displayName` | Policyns ID och namn. |
| &nbsp;&nbsp;`enforcedGrantControls` | Krav som policyn ställer, t.ex. `Mfa` (kräver multifaktorautentisering). |
| &nbsp;&nbsp;`enforcedSessionControls` | Sessionsrelaterade krav, t.ex. `SignInFrequency` (hur ofta omautentisering krävs). |
| &nbsp;&nbsp;`result` | Utfall för just denna policy: `success`, `notApplied` (villkoren matchade inte), `failure` m.fl. |
| &nbsp;&nbsp;`conditionsSatisfied` / `conditionsNotSatisfied` | Antal villkor som uppfylldes respektive inte uppfylldes. |

## Autentiseringsdetaljer

| Fält | Förklaring |
|---|---|
| `authenticationContextClassReferences` | Ev. auth context-klasser kopplade till känsligare åtkomstkrav. |
| `originalRequestId` | Ursprungligt begäran-ID (ofta samma som `id`). |
| `tokenIssuerName` / `tokenIssuerType` | Vem som utfärdade token, t.ex. `AzureAD`. |
| `authenticationProcessingDetails` | Tekniska detaljer om autentiseringsprocessen, t.ex.: |
| &nbsp;&nbsp;"Legacy TLS" | Om äldre, osäker TLS-version användes. |
| &nbsp;&nbsp;"Token Binding Attempted" | Om token-bindning till enheten försöktes. |
| &nbsp;&nbsp;"Oauth Scope Info" | Vilka OAuth-behörigheter (scopes) som begärdes. |
| &nbsp;&nbsp;"Is CAE Token" | Om token stödjer Continuous Access Evaluation (realtidsindragning av åtkomst). |
| `clientCredentialType` | Typ av klientautentisering (t.ex. `none` för interaktiv användarinloggning). |
| `processingTimeInMilliseconds` | Bearbetningstid i Entra ID, i millisekunder. |
| `authenticationDetails` | Steg-för-steg-lista över hur autentiseringen genomfördes, inkl. metod och resultat per steg. |
| `authenticationRequirementPolicies` | Vilken typ av policymotor som krävde autentiseringen (t.ex. Conditional Access). |
| `sessionLifetimePolicies` | Policyer som styr sessionens livslängd, t.ex. periodisk omautentisering. |
| `authenticationRequirement` | Sammanfattat krav som uppfylldes, t.ex. `multiFactorAuthentication`. |

## Risk (Entra ID Protection)

| Fält | Förklaring |
|---|---|
| `riskDetail` | Detaljerad riskorsak (ofta `hidden` om man saknar rätt licens/rättighet att se den). |
| `riskLevelAggregated` | Sammanvägd riskbedömning för användaren. |
| `riskLevelDuringSignIn` | Riskbedömning specifikt vid detta inloggningstillfälle. |
| `riskState` | Status för risken, t.ex. `remediated` = åtgärdad/neutraliserad. |
| `riskEventTypes` / `riskEventTypes_v2` | Vilken typ av risksignal som utlösts, t.ex. `anonymizedIPAddress` (VPN/proxy/Tor). |

## Övriga identitets- och tenant-fält

| Fält | Förklaring |
|---|---|
| `resourceDisplayName` | Namn på resursen som anropades, t.ex. "Office 365 Exchange Microservices". |
| `resourceTenantId` / `homeTenantId` | Tenant-ID för resursen respektive användarens hemtenant. |
| `authenticationDetails[].authenticationMethod` | Metod som användes i autentiseringssteget, t.ex. "Previously satisfied" (redan uppfyllt via befintlig session/token). |
| `alternateSignInName` | Alternativt inloggningsnamn, om använt. |
| `signInIdentifier` / `signInIdentifierType` | Identifierare som angavs vid inloggning och dess typ. |
| `userType` | Kontotyp, t.ex. `Member` (intern anställd) eller `Guest`. |
| `flaggedForReview` | Om händelsen manuellt/automatiskt flaggats för granskning. |
| `isTenantRestricted` | Om tenant-restriktion (Tenant Restrictions) tillämpats. |
| `autonomousSystemNumber` | ASN – identifierar internetleverantör/nätverk för IP-adressen. |
| `crossTenantAccessType` | Typ av cross-tenant-åtkomst, `none` om ej tillämpligt. |
| `privateLinkDetails` | Info om Azure Private Link användes för anropet. |
| `servicePrincipalName` / `servicePrincipalId` | Namn/ID för tjänsteprincipalen (appen) om relevant. |
| `federatedCredentialId` | ID för ev. federerat inloggningsuppgift. |
| `signInEventTypes` | Typ av inloggningshändelse, t.ex. `interactiveUser`. |
| `incomingTokenType` | Typ av inkommande token (om något). |
| `authenticationProtocol` | Protokoll som användes för autentisering. |
| `signInTokenProtectionStatus` / `tokenProtectionStatusDetails` | Status för token-skydd (Token Protection), skyddar mot stulna tokens. |
| `originalTransferMethod` | Hur token ev. överfördes mellan enheter/sessioner. |
| `isThroughGlobalSecureAccess` | Om trafiken gick via Microsoft Global Secure Access. |
| `conditionalAccessAudiences` | Vilka resurser (audience) CA-policyerna utvärderades mot. |
| `sessionId` / `clientSessionId` | ID för användarens session respektive klientsession. |
| `appOwnerTenantId` | Tenant som äger appen (kan skilja sig från egen tenant vid multi-tenant-appar). |
| `resourceOwnerTenantId` | Tenant som äger den anropade resursen. |
| `sourceAppClientId` | Klient-ID för en ev. ursprungsapp i en anropskedja. |
| `redirectUrl` | Ev. redirect-URL använd i autentiseringsflödet. |

---

**Tips:** De mest användbara fälten för snabb incident-triage är ofta `userPrincipalName`, `callerIpAddress`, `location`, `deviceDetail.isCompliant`, `conditionalAccessStatus`, `riskState` och `resultSignature`.
