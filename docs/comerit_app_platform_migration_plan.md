# Plan: flytta appplattformen till Comerits GCP-miljö

## Status

Planen beskriver vad som behöver vara på plats innan `company-website` kan
driftsättas i Team 2:s nya GCP-miljö, `itsx25-comerit`. Den är en
besluts- och arbetsplan, inte ett genomförande. Inga secrets, nycklar eller
produktionsändringar ingår i detta dokument.

## Verifierat nuläge

- Infra-repot har driftsatt `team2-primary` och `team2-jumphost` i Comerits
  projekt. IAP och OS Login är verifierade för båda VM:erna.
- `team2-primary` kör Debian 12 men har ännu inte K3s installerat.
- Den nya jumphosten har ännu inte en dokumenterat återställd
  Headscale-installation.
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

## Beslut som krävs

Teamet behöver tillsammans med Comerit bekräfta vilken väg som är tillåten
för GitHub Actions till den nya K3s-miljön:

1. **Återskapa tidigare modell:** K3s på `team2-primary` och en begränsad
   Headscale-anslutning från GitHub-runnern via jumphosten.
2. **Godkänd alternativ modell:** exempelvis en särskilt tilldelad
   CI/CD-åtkomstväg genom Comerits plattform.

IAP är rätt väg för gruppmedlemmarnas interaktiva SSH-åtkomst, men är inte i
sig en ersättning för GitHub-runners automatiserade åtkomst. Vald modell ska
vara dokumenterad och granskad innan den byggs.

## Genomförande efter godkänt beslut

### 1. Bygg den nya applikationsplattformen i infra-repot

- Installera och starta K3s på `team2-primary` genom reproducerbar
  konfiguration.
- Verifiera Kubernetes API, lokal persistent lagring och ingress-controller.
- Återskapa endast nödvändiga RBAC-rättigheter för deploy-servicekontot.
- Installera och verifiera policy-stödet som krävs av
  `k8s/image-policy.yaml` innan policyn appliceras.
- Om Headscale-modellen väljs: installera och dokumentera Headscale på
  jumphosten, dess begränsade policy samt route mellan GitHub-runnern och
  K3s API. Inga administrativa nyckelvärden sparas i Git.

### 2. Förbered applikationsrepo och CI/CD

- Skapa en ny, begränsad kubeconfig för GitHub-deployern när den nya
  kluster-RBAC:en är verifierad.
- Skapa eller rotera `KUBECONFIG`, `HEADSCALE_URL` och
  `HEADSCALE_API_KEY` i GitHub Secrets först när den nya plattformen är klar.
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

Ta beslut om CI/CD-anslutningsmodellen innan någon K3s-, Headscale- eller
secret-ändring genomförs. Därefter skapas avgränsade infra-issues och PR:er
för plattform, åtkomst och slutlig applikationsdeploy.
