# Company Website - ITSX25 Team 2

Detta repository innehåller Team 2:s applikation och CI/CD-flöde för Kurs 6,
Workshop 3 och Workshop 4 Blue Team. Applikationen är byggd med Flask,
paketeras som en container i GitHub Container Registry (GHCR) och driftsätts
kontrollerat till teamets K3s-kluster.

## Aktuell status

- PR #32 är mergad och driftsatt på `main`. Den aktuella hardeningen kör
  applikationen som användare `10001`, använder skrivskyddat rotfilsystem och
  förbereder SQLite-volymens rättigheter i en initcontainer.
- Deploymentkörning `37451241461` efter PR #32 är grön. En manuell kontroll
  2026-10-06 med ett tilldelat testkonto i utbildarens kursmiljö bekräftade
  profilflöde, omladdning och ny inloggning i drift. Riktade
  behörighetstester och den fullständiga testsviten är också gröna.
- SQLite använder en `ReadWriteOnce`-volym. Deploymenten använder därför
  `Recreate` för att undvika volymkonflikter, vilket kan ge ett kort avbrott
  vid rollout.
- Workshop 4 mergades genom PR #21 och dokumentationens efterkontroll genom
  PR #22 (`64bb55f`).
- Deploymentkörning `36435042346`, försök 2, är grön med 19 tester,
  CycloneDX-SBOM, Cosign-attestering, signering och verifierad K3s-rollout.
- Den deployade imagen är låst till digest `sha256:bf1b2a60...` och podden är
  `1/1 Ready`.
- Kubernetes-secreten `company-website-secrets` är provisionerad på
  `team2-primary` utan att värdet har lagts i Git.
- GitHub-repot och GHCR-paketet är publika.
- GitHub Actions bygger och publicerar containerimagen.
- Den tidigare Headscale-vägen används inte i Comerits miljö. Migreringen
  använder i stället WIF, en begränsad IAP-tunnel till `team2-primary:6443`
  och Kubernetes-ServiceAccounten `github-deployer`.
- Deployment, PersistentVolumeClaim och Service är applicerade i K3s.
- Podden är verifierad som `Running` och `/healthz` rapporterar en frisk
  applikation med databasanslutning.
- `main` skyddas med branch protection och krav på två godkännanden.
- `ingress-nginx` är installerat och routar det interna namnet
  `company-website.team2.arpa` till applikationens Service.
- Åtkomst via MagicDNS och Ingress är verifierad med HTTP-status `200`.
- Cosign-enforcement är aktivt i namespace `default`. En osignerad testimage
  nekades medan Team 2:s signerade digest godkändes.
- Rollback till en tidigare signerad digest och återställning till den senaste
  versionen är verifierade med redo pod och HTTP-status `200`.
- Efterkontrollen gav frisk databasanslutning, HTTP `200` via intern DNS samt
  externt verifierad Cosign-signatur och CycloneDX-attestering.
- Deployment `36565389383` efter PR #22 lyckades. Den deployade imagen
  är låst till digest `sha256:bf1b2a60...`. E-postsignaturens tillåtna
  och otillåtna platshållare verifierades dessutom manuellt i live-miljön
  utan att några profiländringar sparades.

Den tekniska setupen för Workshop 3 och Workshop 4 är verifierad. Defensiv
analys av applikationen fortsätter och observationer dokumenteras utan
credentials eller flaggvärden. Åtgärdsförslag hanteras via Issues, backlog,
branch och pull request.

## Viktiga filer

- [.github/workflows/deploy.yml](.github/workflows/deploy.yml): bygger imagen,
  använder WIF och en begränsad IAP-tunnel och driftsätter till K3s.
- [k8s/deployment.yaml](k8s/deployment.yaml): applikationens Deployment.
- [k8s/pvc.yaml](k8s/pvc.yaml): persistent lagring för SQLite-data.
- [k8s/service.yaml](k8s/service.yaml): intern Kubernetes Service.
- [k8s/ingress.yaml](k8s/ingress.yaml): hostbaserad routing för
  `company-website.team2.arpa`.
- [k8s/github-permissions.yaml](k8s/github-permissions.yaml): RBAC för
  GitHub-deployern; appliceras manuellt som engångskonfiguration.
- [k8s/image-policy.yaml](k8s/image-policy.yaml): Cosign-policy för images
  signerade av Team 2:s deploy-workflow på `main`.
- [scripts/generate-kubeconfig.sh](scripts/generate-kubeconfig.sh): genererar
  en begränsad kubeconfig för CI/CD.
- [docs/](docs/): backlog, status, guider och gemensamma sammanfattningar.
- [docs/comerit_app_platform_migration_plan.md](docs/comerit_app_platform_migration_plan.md):
  status och kontrollerad övergång till Comerits IAP-baserade deployväg.
- [members/](members/): personliga anteckningar och arbetssammanfattningar.

## Dokumentation

- [Dokumentationsöversikt](docs/README.md)
- [Produktbacklog](docs/product_backlog.md)
- [Workshop 3 - setup och verifiering](docs/workshop3_setup_status.md)
- [MITRE ATT&CK, Threat Intelligence och TLP](docs/mitre_ti_tlp_summary_2026-10-05.md)
- [Teamsammanfattning 2026-10-06](docs/team_work_summary_2026-10-06.md)
- [Teamsammanfattning 2026-09-21](docs/team_work_summary_2026-09-21.md)
- [Teamsammanfattning 2026-09-24](docs/team_work_summary_2026-09-24.md)
- [Teamsammanfattning 2026-09-28](docs/team_work_summary_2026-09-28.md)
- [Teamsammanfattning 2026-09-29](docs/team_work_summary_2026-09-29.md)

## Relaterad infrastruktur

Applikationens GCP-, nätverks- och K3s-infrastruktur hanteras i det
separata [infra-repot](https://github.com/ITSX25-Chas/kurs6-team2-infra).

- [Gemensam anslutningsguide](https://github.com/ITSX25-Chas/kurs6-team2-infra/blob/main/docs/gemensam_anslutningsguide.md)
- [Infra-repots README](https://github.com/ITSX25-Chas/kurs6-team2-infra/blob/main/README.md)
- [Infra-backlog](https://github.com/ITSX25-Chas/kurs6-team2-infra/blob/main/docs/product_backlog.md)

Personlig åtkomst till VM:erna sker via Google IAP och OS Login. GitHub Actions
använder en separat, begränsad IAP-tunnel för deploy till Kubernetes API.

## Arbetsflöde

1. Skapa eller välj ett GitHub Issue.
2. Arbeta från en egen branch, exempelvis `member/itzmejonny92` eller en kort
   feature-branch.
3. Gör en liten och tydligt avgränsad ändring.
4. Kör relevanta tester lokalt.
5. Pusha branchen och skapa en pull request mot `main`.
6. Kontrollera CI-resultatet.
7. Låt minst två medlemmar granska och godkänna.
8. Merga till `main` och verifiera deploymenten.
9. Uppdatera Issue och backlog efter verifierad merge.

Backlogfilen synkroniseras inte automatiskt med GitHub Issues. Den som ändrar
ett Issue ansvarar därför för att bedöma om även backloggen ska uppdateras.

## Lokal utveckling

Skapa en virtuell miljö och installera beroenden:

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Starta applikationen:

```bash
cp .env.example .env
# Ersätt exempelvärdet för SECRET_KEY i .env, till exempel med: openssl rand -hex 32
python wsgi.py
```

Alternativt med Flask CLI:

```bash
export PYTHONPATH=src
flask --app company_website run --port 7000
```

## Docker

```bash
docker compose up --build
```

SQLite-databasen sparas i Docker-volymen `app-data`. I K3s används i stället
en PersistentVolumeClaim monterad på `/app/data`.

## Tester

```bash
python -m pip install -r requirements-dev.txt
python -m pytest -q
```

Workflowen `Application Checks` kör samma tester och en Docker-build på
feature-branches och pull requests utan att publicera eller deploya imagen.

## Deployment

En manuell `workflow_dispatch` startar pipelinen under migrationen:

1. Koden checkas ut.
2. Metadata för `latest` och aktuell commit-SHA skapas.
3. Containerimagen byggs, publiceras i GHCR och identifieras med digest.
4. En CycloneDX-SBOM skapas och attesteras mot samma image-digest.
5. Imagen signeras nyckellöst med Cosign och GitHub OIDC.
6. GitHub OIDC autentiserar mot WIF med det begränsade deploykontot.
7. GitHub-runnern öppnar en IAP-tunnel enbart till `team2-primary:6443`.
8. IAP-tunneln och Kubernetes API verifieras.
9. Kubernetes-manifesten, inklusive Ingress, appliceras.
10. Rollout till den signerade imagen verifieras.

Workflowet använder GitHub Secrets och Variables. Hemliga värden får aldrig
skrivas i dokumentation, Issues, loggar eller commits.

WIF används utan en långlivad GCP Service Account Key. `CICD_SERVICE_ACCOUNT`
är läsande för GCP-kontroller och `DEPLOY_SERVICE_ACCOUNT` är begränsad till
IAP-tunneln. Kubernetes-behörigheten kommer från den scoped kubeconfigen.

Applikationens sessionsnyckel ligger i Kubernetes-secreten
`company-website-secrets`. Den skapas en gång av en behörig administratör före
första deploymenten:

```bash
kubectl create secret generic company-website-secrets \
  --from-literal=secret-key="$(openssl rand -hex 32)"
```

Kommandot behöver inte köras igen vid varje deployment. GitHub-runnerns begränsade
RBAC får medvetet inte läsa eller ändra Secrets.

## Projektstruktur

```text
.
|-- .github/workflows/
|-- docs/
|-- k8s/
|-- members/
|-- scripts/
|-- src/company_website/
|-- tests/
|-- Dockerfile
|-- docker-compose.yml
|-- requirements.txt
`-- wsgi.py
```

## Säkerhet

- Använd aldrig riktiga credentials i kod eller dokumentation.
- Kontrollera `git status` och diffen före varje commit.
- Kubeconfigen lagras endast som GitHub Secret och innehåller ett scoped
  ServiceAccount-token, aldrig cluster-admin.
- Deploykontot får endast öppna IAP-tunnel till Kubernetes API.
- Images taggas för spårbarhet, deployas med digest och signeras nyckellöst
  med Cosign.
- Policy-enforcement aktiveras först efter verifierad signering för att
  undvika att blockera en giltig rollout.
- Kursens analys genomförs endast i de miljöer som utbildaren har godkänt.
- Flaggar dokumenteras inte öppet i gemensamma filer.
