# Team 2 - arbetssammanfattning 2026-10-26

## Dagens mål

Fortsätta flytten av company-website till Comerits GCP-miljö och verifiera
att applikationen kan deployas säkert till Team 2:s nya K3s-kluster.

## Genomfört arbete

1. PR #45, som ersätter den tidigare Headscale-vägen för deployment med
   Workload Identity Federation (WIF) och en begränsad IAP-tunnel, och PR #46,
   som rättar kubeconfig-generatorn till `k3s kubectl`, verifierades som
   mergade till `main`.
2. K3s-noden `team2-primary` verifierades som `Ready`. Den nya miljön saknade
   tidigare GitHub-deployerns Kubernetes-resurser, Sigstore policy-controller,
   ingressklass och applikationens Kubernetes Secret.
3. `github-deployer`, dess token-secret och RBAC-resurser applicerades manuellt
   på klustret. En behörighetskontroll gav `yes` för att uppdatera deployments
   och `no` för att läsa Secrets. Efter Sigstore-installationen gav kontrollen
   `yes` för att uppdatera `ClusterImagePolicy`.
4. Sigstore policy-controller installerades manuellt via K3s Helm-controller,
   med chartversion `0.10.8`. Installationsjobbet slutfördes, CRD:n skapades
   och controllerns rollout lyckades. Policyn `verify-company-website`
   applicerades och rapporterade `Ready: True`.
5. En begränsad kubeconfig för `github-deployer` genererades och sparades som
   GitHub Actions-secreten `KUBECONFIG`. En ny sessionsnyckel skapades som
   Kubernetes Secret utan att nyckelvärdet dokumenterades.
6. Den manuella deploykörningen [40294327401](https://github.com/ITSX25-Chas/team2-company-website/actions/runs/40294327401)
   blev grön. Tester, image-build, CycloneDX-SBOM, Cosign-attestering och
   signering, WIF-autentisering, IAP-tunnel och Kubernetes-rollout lyckades.
7. Sigstore testades först i en separat test-namespace med server-side dry-run.
   Den signerade app-imagen godkändes; en BusyBox-image nekades eftersom den
   inte matchade någon policy. Det senare bevisar inte att imagen var osignerad.
   Därefter aktiverades enforcement i `default` och app-imagen godkändes även
   där vid dry-run.
8. App-podden verifierades som `1/1 Running`. Service-kontrollen `/healthz`
   svarade `healthy` med databasen `connected`.
9. Webbplatsen öppnades i webbläsaren via Kubernetes port-forward på primary
   och en SSH-tunnel genom IAP. Ingen publik webbport eller ny ingress-controller
   behövdes för detta test.

## Kvarstående uppföljning

- Inloggningen i den nya miljön är inte verifierad eftersom testkontots
  inloggningsuppgifter inte fanns tillgängliga under kontrollen.
- Ingress och tidigare intern DNS-adress har inte återställts; gruppen valde
  IAP-tunnel som tillräcklig åtkomst tills vidare.
- Kubernetes-RBAC, Sigstore HelmChart, policy, namespace-label och
  applikationssecret sattes upp manuellt. Beskriv en reproducerbar och säker
  återställningsrutin i rätt repo; dokumentera aldrig secret-värden.
- `team2-policy-test` skapades för den isolerade policykontrollen och behöver
  hanteras när teamet är klart med testerna.
- Den gamla Headscale-baserade deploykonfigurationens secrets bör inte tas bort
  förrän övergången och eventuell återkallning har kontrollerats.
- Actions gav en varning om Node.js-runtime för Google-actions. Varningen
  stoppade inte deployen men bör följas upp.

## Säkerhet

Sammanfattningen innehåller inga lösenord, tokens, kubeconfig-värden eller
sessionsnycklar. Närvaro för dagen är inte verifierad och har därför inte
angetts.
