# Plan: flytta appplattformen till Comerits GCP-miljö

## Status

Planen beskriver vad som behöver vara på plats innan `company-website` kan
driftsättas i Team 2:s nya GCP-miljö, `itsx25-comerit`. Den är en
besluts- och arbetsplan, inte ett genomförande. Inga secrets, nycklar eller
produktionsändringar ingår i detta dokument.

## Verifierat nuläge

- Infra-repot har driftsatt `team2-primary` och `team2-jumphost` i Comerits
  projekt. IAP och OS Login är verifierade för båda VM:erna.
- `team2-primary` kör Debian 12 och K3s är installerat samt verifierat.
  Efter den manuella bootstrap-körningen `39395769638` den 2026-10-20 var
  tjänsten aktiv, noden `team2-primary` `Ready` och Kubernetes API:s
  readiness-kontroll svarade `ok`.
- Den nya jumphosten används för nätåtkomst från VM:erna. Headscale är inte
  en del av den nya plattformens verifierade baslinje.
- App-repots senaste gröna workflow-körning skedde 2026-10-13, före
  flytten av infrastrukturen till Comerits GCP-miljö.
- Nuvarande `deploy.yml` bygger, testar, SBOM-attesterar och Cosign-signerar
  imagen. Själva deployen förutsätter dessutom Headscale, K3s på
  `10.0.2.3:6443` och ett begränsat `KUBECONFIG` i GitHub Secrets.
- Ingress-manifestet använder det interna namnet
  `company-website.team2.arpa`.

Det betyder att applikationskoden och supply-chain-kontrollerna kan användas
vidare, men att den tidigare körplattformen inte automatiskt följer med till
de nya VM:erna.

## Beslutad CI/CD-modell

Comerit har godkänt en ny, begränsad deployväg. GitHub Actions autentiserar
med Team 2:s befintliga WIF-provider och tjänstkontot
`team2-deploy@itsx25-comerit.iam.gserviceaccount.com`. Det kontot får enbart
öppna en IAP-tunnel till `team2-primary:6443`; det får inte skapa eller ändra
infrastruktur.

Väl inne i klustret använder workflowet ServiceAccounten `github-deployer`.
Dess kubeconfig innehåller ett scoped token och ersätter tidigare
cluster-admin-åtkomst. Den läsande kontrollen i app-repot fortsätter använda
`team2-las@itsx25-comerit.iam.gserviceaccount.com` via
`CICD_SERVICE_ACCOUNT`.

Headscale används inte i den nya deploykedjan. Infra-repots brandvägg öppnar
redan IAP-intervallet `35.235.240.0/20` mot TCP 6443 på primary-noden.

## Genomförande efter godkänt beslut

### 1. Komplettera den nya applikationsplattformen i infra-repot

- Behåll den verifierade K3s-installationen på `team2-primary`.
- Verifiera lokal persistent lagring och installera en ingress-controller
  genom reproducerbar konfiguration.
- Återskapa endast nödvändiga RBAC-rättigheter för deploy-servicekontot.
- Installera och verifiera policy-stödet som krävs av
  `k8s/image-policy.yaml` innan policyn appliceras.
- Bekräfta att IAP-brandväggsregeln fortsatt begränsas till
  `35.235.240.0/20` mot TCP 6443 och primary-noden.

### 2. Förbered applikationsrepo och CI/CD

- Skapa en ny, begränsad kubeconfig för GitHub-deployern när den nya
  kluster-RBAC:en är verifierad.
- Applicera begränsad deployer-RBAC, skapa en scoped token och uppdatera
  GitHub-secreten `KUBECONFIG` med en kubeconfig för IAP-tunneln.
- Lägg `DEPLOY_SERVICE_ACCOUNT` som repo-variabel. Behåll
  `CICD_SERVICE_ACCOUNT` för den läsande GCP-kontrollen.
- Behåll de tidigare secretsen orörda tills den nya deploykedjan fungerar;
  återkalla dem först efter en kontrollerad övergång.
- Bekräfta att `TEAM_ID` och övriga icke-hemliga repo-variabler stämmer med
  den nya miljön.
- Uppdatera workflow och anslutningsdokument endast för den valda modellen.

### 3. Validera i en PR

- Kör `Application Checks` för kod, tester och container-build.
- Kör deploy-workflowet mot den nya miljön från en granskad PR eller en
  manuellt initierad körning enligt teamets rutin.
- Verifiera signerad image-digest, SBOM-attestering, policykontroll,
  Kubernetes-rollout, PVC och `/healthz`.
- Kontrollera att ingress och intern DNS fungerar enligt den nya
  åtkomstmodellen.

### 4. Avsluta migrationen

- Uppdatera README, anslutningsguide, backlog och arbetssammanfattning med
  verifierade resultat.
- Markera tidigare Headscale-, K3s- och proxyinstruktioner som historiska om
  de inte längre används.
- Ta bort eller rotera föregående CI/CD-uppgifter först när den nya kedjan är
  grön och dokumenterad.

## Acceptanskriterier

Migrationen är klar först när följande kan visas utan att hemliga värden
exponeras:

- `team2-primary` har ett fungerande K3s-kluster i Comerits miljö.
- GitHub Actions når enbart K3s API genom den godkända, begränsade
  CI/CD-anslutningen.
- Deploy-workflowet är grönt och använder signerad image med SBOM-attestering.
- Kubernetes-rollouten är redo och `/healthz` rapporterar en frisk
  applikation.
- Åtkomst, DNS/Ingress, secrets och ansvar är dokumenterade och granskade i
  en PR.

## Nästa konkreta steg

K3s-basen och den godkända IAP-modellen är klara. Nästa steg är att applicera
och verifiera deployer-RBAC, lägga in den scoped kubeconfigen som GitHub
Secret och därefter köra den manuella deployen. Policystöd, ingress och
applikationssecret ska verifieras före en godkänd rollout.
