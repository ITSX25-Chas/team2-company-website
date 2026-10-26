# Company Website i Comerits miljö

## Status 2026-10-26

Applikationen har driftsatts på K3s på `team2-primary` i `itsx25-comerit`.
Detta dokument skiljer på verifierade resultat och återstående arbete. Inga
secret-värden eller tokens ska dokumenteras här.

## CI/CD och åtkomst

- GitHub Actions använder Team 2:s WIF-provider och det begränsade
  `team2-deploy`-kontot för en IAP-tunnel till primarys Kubernetes API.
- Kubernetes-åtkomsten sker med en scoped kubeconfig för ServiceAccounten
  `github-deployer`. En kontroll visade att kontot kan uppdatera deploymenten
  men inte läsa secrets. Policyns klusterresurs har en separat behörighet.
- Den läsande GCP-kontrollen använder `team2-las`, inte infra-kontot.
- [Deploy-körningen](https://github.com/ITSX25-Chas/team2-company-website/actions/runs/40294327401)
  var grön med tester, imagebygge, SBOM, signering och rollout. Den kördes
  innan enforcement aktiverades i `default`.
- Personlig webbåtkomst verifierades via IAP och port forwarding. Inloggning
  och tidigare profildata har inte verifierats i den nya miljön.

## Policystöd och drift

Sigstore policy-controller installerades manuellt genom en K3s HelmChart.
`verify-company-website` rapporterade `Ready`, och enforcement aktiverades
med en label på `default`. I en separat testnamespace godkändes appens image
med server-side dry-run, medan en BusyBox-image nekades eftersom ingen policy
matchade. Appens image godkändes även i en dry-run i `default`.

Applikationspodden var `1/1 Running` och `/healthz` rapporterade frisk
applikation och ansluten databas. Applikationssecreten, deployer-RBAC och
policykonfigurationen har satts upp manuellt, inte återskapats automatiskt
av infra-repots bootstrap.

Ingress-manifestet finns kvar och appliceras av workflowet, men en fungerande
Ingress-controller och den gamla DNS-vägen har inte verifierats i den nya
miljön. Teamet använder tills vidare IAP-tunnel för webbåtkomst.

## Återstår

1. Verifiera en full deploy och rollout efter att enforcement aktiverats, och
   kontrollera även hur återställning ska genomföras med gällande policy.
2. Dokumentera och besluta hur manuell RBAC, policy-controller, namespace-label
   och applikationssecret ska kunna återskapas vid en ny installation.
3. Följ upp säker lagring, rotation och återkallning av den scoped Kubernetes-
   token som används i GitHub Secrets. Publicera aldrig token eller kubeconfig.
4. Testa inloggning med ett tilldelat konto och verifiera eventuell data enligt
   teamets behörigheter. Hälsokontrollen bevisar inte att inloggning fungerar.
5. Bedöm om Ingress och DNS behövs framöver. Ta inte bort eller installera en
   controller utan ett separat beslut och verifiering.
6. Se över äldre uppgifter och secrets efter en kontrollerad övergång samt
   GitHub Actions varningar om äldre Node-versioner.

Tidigare Headscale-anslutning gäller den gamla miljön och används inte i den
nuvarande deploykedjan. Ändringar går genom teamets vanliga branch- och
PR-rutin.
