# Guia de publicação do Sciverse

> Honest end-to-end guide: GitHub → GitHub Pages → (optional) Google Play → (optional) Apple App Store.

---

## Leia isto antes de começar — leitura obrigatória

Sciverse é um arquivo HTML único de ~35 MB. As lojas (Google Play, Apple App Store) **não aceitam HTML diretamente** — elas só aceitam apps Android (`.aab` ou `.apk`) e iOS (`.ipa`). Para publicar nas lojas você precisa **embrulhar** o HTML em um app nativo.

Vou ser direto com você sobre as três alternativas, do mais fácil ao mais difícil:

| Caminho | Custo | Tempo | Atualização | Recomendado para |
|---|---|---|---|---|
| **GitHub Pages (PWA instalável)** | Grátis | 10 min | Push e pronto | Começar — entrega 95% do valor |
| **Google Play (via TWA / Bubblewrap)** | USD 25 (1x) | 1-3 dias | Re-submeter app | Quem quer presença na loja Android |
| **Apple App Store (via Capacitor + Mac)** | USD 99/ano | 1-2 semanas | Re-submeter app | Só se for essencial — alto risco de rejeição |

**Minha recomendação honesta**: comece pelo GitHub Pages. Em 10 minutos seu app está no ar, funciona em qualquer celular ou desktop, e dá pra instalar como app (com ícone na tela inicial) tanto no Android quanto no iOS — sem loja, sem custo, sem espera. Se depois você ainda quiser as lojas, embrulha. Mas a maioria dos casos não precisa.

Riscos honestos das lojas:
- **Apple**: provavelmente vai rejeitar na primeira tentativa por "minimum functionality" (guideline 4.2) ou "low-quality content" (4.3), porque o app é um wrapper de website. Tem como passar — milhares de PWAs estão lá — mas exige polimento, screenshots bons, e às vezes argumentação por email com o reviewer.
- **Google Play**: muito mais permissivo, raramente rejeita TWAs/PWAs bem feitas. Mas exige verificação de identidade do desenvolvedor (pode pedir documento), pode levar dias.

---

## PARTE 1 — Subir no GitHub (passo a passo)

### Pré-requisitos
- Conta no GitHub. Se não tem, crie em https://github.com/signup (grátis).
- Saber usar um terminal **ou** Git Desktop (https://desktop.github.com).

### 1.1 — Criar o repositório

1. Vá em https://github.com/new
2. **Repository name**: `sciverse` (ou outro nome de sua preferência — use minúsculas, sem espaços)
3. **Description**: `Interactive Science Explorer — 33,603 topics, fully offline`
4. **Public** (precisa ser público para GitHub Pages grátis)
5. **Não** marque "Add a README" — você já tem um
6. Click **Create repository**

### 1.2 — Subir os arquivos

Você recebeu um zip `Sciverse-github.zip` com tudo dentro. Extraia em uma pasta.

**Opção A — via interface web (mais simples):**
1. Na página do repositório recém-criado, clique em **uploading an existing file**
2. Arraste TODOS os arquivos da pasta `sciverse-release/` (o conteúdo, não a pasta em si)
3. **IMPORTANTE**: o arquivo `Sciverse.html` tem 35 MB. O limite de upload por arquivo do GitHub via web é 25 MB — então via web **não vai funcionar para esse arquivo**. Você precisa usar Git desktop ou linha de comando.

**Opção B — via Git Desktop (recomendado):**
1. Instale https://desktop.github.com
2. **File → Clone repository** → escolha `sciverse` → clique Clone
3. Copie todos os arquivos da pasta `sciverse-release/` para dentro da pasta clonada local
4. No Git Desktop você verá as mudanças listadas. Escreva uma mensagem de commit ("Initial release"). Clique **Commit to main**
5. Clique **Push origin**
6. Pronto — vá ao GitHub e veja os arquivos lá

**Opção C — via terminal (se você é à vontade com Git):**
```bash
cd ~/Downloads/sciverse-release   # ou onde extraiu o zip
git init
git add .
git commit -m "Initial release: Sciverse 33,603 topics"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/sciverse.git
git push -u origin main
```

### 1.3 — Ativar GitHub Pages

1. No repositório, clique em **Settings**
2. No menu lateral esquerdo, clique em **Pages**
3. Em **Source**, escolha **Deploy from a branch**
4. Em **Branch**, escolha **main** e **/ (root)**
5. Clique **Save**
6. Em 1-2 minutos seu site estará no ar em:
   `https://SEU_USUARIO.github.io/sciverse/`

### 1.4 — Testar
- Abra a URL no celular
- Deve abrir o Sciverse normalmente
- **Android (Chrome)**: menu (⋮) → "Install app" ou "Add to home screen"
- **iOS (Safari)**: botão compartilhar → "Add to Home Screen"
- Verifique que o ícone aparece na tela inicial e o app abre standalone

**Se funcionou: parabéns, está publicado.** Compartilhe a URL. A maioria das pessoas vai estar feliz parando aqui.

---

## PARTE 2 — Google Play Store (opcional, ~USD 25)

Para o Google Play vamos usar **Bubblewrap** (oficial do Google) que cria um **TWA — Trusted Web Activity**. É a forma moderna recomendada para empacotar PWA → Android.

### 2.1 — Criar conta de desenvolvedor

1. Vá em https://play.google.com/console/signup
2. Pague **USD 25** (taxa única, vitalícia)
3. Preencha o cadastro como **conta pessoal** (a menos que você queira publicar em nome do IBM, o que NÃO recomendo — esse app não é um produto da IBM)
4. Google fará verificação de identidade — pode pedir foto de documento, comprovante de endereço. Leva de 1 a 3 dias.

### 2.2 — Preparar o assetlinks (necessário para TWA)

Antes de empacotar, você precisa publicar um arquivo de verificação no GitHub Pages:

1. Vamos pegar a "fingerprint" SHA-256 da assinatura do seu app — você só vai gerar essa chave no próximo passo. Volte aqui depois.

### 2.3 — Empacotar com Bubblewrap

**Pré-requisitos**: Node.js (https://nodejs.org), Java JDK 17 (https://adoptium.net).

```bash
# Instalar Bubblewrap globalmente
npm install -g @bubblewrap/cli

# Inicializar projeto
bubblewrap init --manifest=https://SEU_USUARIO.github.io/sciverse/manifest.json
```

Ele vai te fazer várias perguntas:
- **Domain**: `SEU_USUARIO.github.io`
- **URL path**: `/sciverse/`
- **Application name**: Sciverse
- **Short name**: Sciverse
- **App package name**: `io.github.SEU_USUARIO.sciverse` (use minúsculas, padrão Java)
- **Display mode**: standalone
- **Orientation**: any
- **Status bar color**: #001219
- **Splash screen color**: #001219
- **Icon URL**: aceite o padrão (puxa do manifest)
- **Maskable icon URL**: deixe vazio (ou aponte pro 512)
- **Monochrome icon URL**: deixe vazio
- **Include shortcuts**: No
- **Signing key**: cria uma nova (anote a senha e GUARDE o arquivo `.keystore` em local seguro — sem ela você nunca mais consegue atualizar o app)

```bash
# Construir o AAB (Android App Bundle, formato que Google Play exige)
bubblewrap build
```

Saída: `app-release-bundle.aab`

### 2.4 — Publicar o assetlinks.json (verificação de domínio)

Bubblewrap te dá uma string de SHA-256 fingerprint. Crie um arquivo `assetlinks.json` no seu repo, dentro de uma pasta `.well-known/`:

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "io.github.SEU_USUARIO.sciverse",
    "sha256_cert_fingerprints": ["AB:CD:EF:..."]
  }
}]
```

Substitua a fingerprint pela sua. Faça commit e push. O arquivo precisa estar acessível em:
`https://SEU_USUARIO.github.io/sciverse/.well-known/assetlinks.json`

**Sem isso o app abre com barra de URL do Chrome em cima (não fica "standalone")**, então é importante.

### 2.5 — Submeter ao Google Play

1. Em https://play.google.com/console, clique **Create app**
2. Nome do app: Sciverse
3. Default language: English (or Portuguese)
4. App or game: App
5. Free or paid: **Free**
6. Concorde com declarações
7. **Click Create**

Agora preencha as seções do checklist lateral:

- **App content**:
  - **Privacy policy**: você PRECISA hospedar uma. Crie uma página simples em `https://SEU_USUARIO.github.io/sciverse/privacy.html` dizendo que o app não coleta dados, é totalmente offline. Sem isso, Google rejeita.
  - **Data safety**: declare "No data collected" — é verdade
  - **Ads**: No
  - **Content rating**: faça o questionário (educativo, sem violência → vai dar Everyone / 3+)
  - **Target audience**: marque adultos OU "Designed for everyone" (cuidado: se marcar crianças <13, tem regras extras pesadas — provavelmente prefere deixar 13+)
  - **News app**: No
  - **COVID-19 contact tracing**: No
  - **Government app**: No

- **Store listing**:
  - **Short description** (80 chars): "Interactive science explorer — 33,603 topics, fully offline, no ads"
  - **Full description** (até 4000 chars): use o README + descreva as 11 categorias
  - **App icon**: 512x512 PNG (use o `icons/icon-512.png`)
  - **Feature graphic**: 1024x500 PNG — você precisa criar uma (banner promocional)
  - **Screenshots**: mínimo 2, recomendo 4-8 do app rodando no celular

- **Production track**:
  - **Create new release**
  - Faça upload do `app-release-bundle.aab`
  - Release notes: "Initial release"
  - **Review release** → **Start rollout to production**

### 2.6 — Espera

Review do Google leva tipicamente 1-7 dias. Você recebe email quando aprovado ou se houver problema. Se rejeitar, atendem ao motivo no email — geralmente é privacy policy ou content rating.

---

## PARTE 3 — Apple App Store (opcional, USD 99/ano, mais difícil)

**Aviso forte**: este é o caminho mais caro e mais lento, e tem o maior risco de rejeição. Pense bem se você precisa mesmo.

### 3.1 — Pré-requisitos

- **Apple Developer Program**: USD 99/ano, https://developer.apple.com/programs/enroll/
  - Pessoa física: pode demorar dias de verificação
  - Pessoa jurídica: precisa de DUNS number, mais burocracia
- **Um Mac** com macOS recente (não funciona em Linux/Windows)
- **Xcode** instalado (gratuito na Mac App Store, ~10 GB)
- **Conhecimento básico de Xcode** ou paciência para aprender

### 3.2 — Estratégia de empacotamento

A Apple não tem TWA equivalente. As opções são:

**Opção A — Capacitor (mais simples, recomendado)**:
```bash
npm install -g @ionic/cli
mkdir sciverse-ios && cd sciverse-ios
npm init -y
npm install @capacitor/core @capacitor/ios
npx cap init Sciverse com.SEU_USUARIO.sciverse --web-dir=www
mkdir www
cp -r ../sciverse-release/* www/
npx cap add ios
npx cap sync
npx cap open ios   # abre no Xcode
```

**Opção B — WKWebView nativa**: criar um projeto Swift do zero com um WKWebView apontando para o arquivo local. Mais trabalho, mais controle.

### 3.3 — Configurar no Xcode

Com o projeto aberto:

1. **Signing & Capabilities**: selecione seu time (Apple Developer account)
2. **Bundle Identifier**: `com.SEU_USUARIO.sciverse` (precisa ser único globalmente)
3. **Display Name**: Sciverse
4. **Deployment target**: iOS 14.0 ou superior
5. **Device orientation**: any
6. **App icons**: arraste o `icons/icon-512.png` no slot de App Icon e o Xcode gera as outras
7. **Info.plist**: adicione `NSAppTransportSecurity` permitindo conexões locais (file://)

### 3.4 — Testar

1. Conecte um iPhone via USB
2. No Xcode, selecione o dispositivo no topo
3. Clique no botão Play (▶)
4. Confie no certificado de desenvolvedor no iPhone: Ajustes → Geral → Gerenciamento → confiar
5. App abre no dispositivo

Faça vários testes — abrir, fechar, scroll, abrir tópicos, simulações, idiomas, mudança de tema. Use o Time Profiler do Xcode para confirmar que não está travando.

### 3.5 — Preparar para a App Store

1. **App Store Connect**: https://appstoreconnect.apple.com → My Apps → "+" → New App
2. **Name**: Sciverse
3. **Bundle ID**: o que você criou no Xcode
4. **SKU**: invente um (ex.: SCIVERSE001)
5. Preencha:
   - **App Privacy**: declare que NÃO coleta nada
   - **Pricing**: Free
   - **Availability**: todos os países (ou os que quiser)
   - **App Info**: categoria principal = Education; secundária = Reference
   - **App Description**: use o README expandido. Mínimo umas 200 palavras, recomendo 500+
   - **Keywords**: "science, physics, chemistry, biology, math, education, offline, study" (100 chars max, separados por vírgula)
   - **Support URL**: link do README no GitHub
   - **Marketing URL** (opcional): a URL do GitHub Pages
   - **Screenshots**: precisa de pelo menos 1 para iPhone 6.7" e 1 para iPhone 6.5". 2-10 screenshots cada. Tira do app rodando no Xcode Simulator
   - **App Icon**: 1024x1024 PNG sem transparência, sem cantos arredondados (Apple arredonda)
   - **Age Rating**: faça o questionário → 4+ provavelmente

### 3.6 — Build e submissão

No Xcode:
1. Selecione **Any iOS Device (arm64)** no topo (não um simulador)
2. **Product → Archive**
3. Quando terminar, abre a janela **Organizer**
4. Selecione seu archive → **Distribute App → App Store Connect → Upload**
5. Espere o upload (vários minutos por causa do tamanho)

No App Store Connect:
1. Vá em sua app → Build (vai aparecer em ~30 min)
2. Selecione o build
3. Em **App Review Information**: dê instruções claras ao reviewer. Sugestão: "This is an educational app with 33,603 science topics covering physics, chemistry, biology, math and more. All content works offline. There is no login, no ads, no in-app purchases. To test: tap any category, tap any topic to see its article and interactive simulation."
4. **Submit for Review**

### 3.7 — Review

A Apple revisa em 1-3 dias tipicamente. Possíveis resultados:
- **Approved** — vai para a loja em até 24h
- **Rejected** com motivo — você corrige e re-submete (sem custo adicional)

**Motivos comuns de rejeição para apps tipo Sciverse**:
- **Guideline 4.2 (Minimum Functionality)**: "this app appears to be a website wrapped in an iOS application" — argumente que ele tem 33k+ tópicos com simulações interativas, é genuinamente um app educacional, não só um site
- **Guideline 4.3 (Spam)**: se eles acharem que é conteúdo gerado em massa de baixa qualidade. Aqui o badge **Deep Dive** ajuda — você pode argumentar que 1.327 artigos são curados em profundidade
- **Guideline 5.1.1 (Privacy)**: privacy policy precisa estar acessível. Hospede no GitHub Pages, link no App Store Connect

Se rejeitarem, a Resolution Center deixa você responder. Seja educado, específico, e mostre como o app cumpre as regras.

---

## PARTE 4 — Checklist final antes de publicar em qualquer lugar

- [ ] Repositório no GitHub criado e arquivos subidos
- [ ] GitHub Pages ativado e URL funcionando
- [ ] App abre no celular (teste em Android E iOS se possível)
- [ ] Privacy policy hospedada (`privacy.html` ou similar)
- [ ] Ícones em todas as resoluções necessárias
- [ ] (Para lojas) Screenshots tirados do app em uso
- [ ] (Para lojas) Texto de descrição claro e específico
- [ ] (Para lojas) Conta de desenvolvedor verificada

---

## Suporte e atualizações

**Atualizar via GitHub Pages**: faça commit/push das mudanças, em 1-2 minutos está no ar. Os PWAs instalados atualizam automaticamente quando o service worker detecta versão nova.

**Atualizar Google Play**: re-empacote com Bubblewrap, suba novo `.aab`, espere review (mais rápido em updates — geralmente horas).

**Atualizar Apple**: re-archive no Xcode, upload, re-submit. Review de update geralmente 1-2 dias.

---

## Uma palavra final, honesta

Se um dia você decidir tirar das lojas, o conteúdo continua disponível no GitHub Pages sem nenhum problema. Hospedar lá é uma decisão sem caminho-sem-volta, ao contrário das lojas. É um motivo a mais para começar pelo GitHub Pages e só ir para as lojas se realmente fizer diferença para você.

Boa sorte com o lançamento.
