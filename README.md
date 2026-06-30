# ⚽ Bolão da Alegria

> Aplicativo web de bolão para jogos da Copa do Mundo, com apostas de placar, pagamentos via PIX, comprovantes digitais e gestão financeira integrada.

![Status](https://img.shields.io/badge/status-ativo-brightgreen)
![Firebase](https://img.shields.io/badge/backend-Firebase-orange)
![Licença](https://img.shields.io/badge/licença-MIT-blue)

---

## 📋 Sumário

- [Visão Geral](#-visão-geral)
- [Funcionalidades](#-funcionalidades)
- [Tecnologias](#-tecnologias)
- [Pré-requisitos](#-pré-requisitos)
- [Configuração do Firebase](#-configuração-do-firebase)
- [Como Usar](#-como-usar)
- [Estrutura do Banco de Dados](#-estrutura-do-banco-de-dados)
- [Regras do Firestore](#-regras-do-firestore)
- [Deploy (GitHub Pages)](#-deploy-github-pages)
- [Fluxo do Usuário](#-fluxo-do-usuário)
- [Papéis e Permissões](#-papéis-e-permissões)
- [Capturas de Tela](#-capturas-de-tela)

---

## 🎯 Visão Geral

O **Bolão da Alegria** é uma aplicação web single-page (SPA) para organizar bolões de futebol entre amigos. Os participantes apostam no placar de jogos da Copa do Mundo, pagam via PIX e enviam comprovantes digitais. O sistema gerencia tudo: lances, pagamentos, resultados e distribuição do prêmio.

Funciona direto no navegador — sem instalação, compatível com iOS e Android como Progressive Web App (PWA).

---

## ✅ Funcionalidades

### 👤 Autenticação e Perfil
- Cadastro e login com e-mail e senha (Firebase Auth)
- Perfil editável: nome, telefone e **chave PIX para receber prêmios**
- Perfil acessível como overlay em qualquer tela do app

### 🎟️ Criação de Bolões
- Nome do bolão, times, data e horário do jogo
- Valor por lance e chave PIX para recebimento dos pagamentos
- Convidar participantes por e-mail
- Opção de bolão com **lances únicos pagos** (cada placar só pode ser apostado uma vez)
- Link de convite compartilhável

### 📨 Sistema de Convites e Notificações
- Convites enviados por e-mail são cruzados com usuários cadastrados
- Notificações em **tempo real** via `onSnapshot` — usuários já logados recebem o convite imediatamente sem precisar sair e colar o link
- Preview do convite para usuários não logados com banner na tela de auth
- Após login/cadastro, o bolão convidado abre automaticamente

### ⚽ Apostas
- Cada participante pode fazer múltiplos lances
- Apostas encerram automaticamente **30 minutos antes do jogo**
- Encerramento manual disponível para o criador
- Validação de lances duplicados (quando ativado pelo criador)
- Lances só contam no prêmio se **pagos com comprovante**

### 💳 Pagamentos via PIX
- Chave PIX exibida com botão de cópia rápida
- Upload de comprovante de pagamento (imagem ou PDF, comprimido no client)
- Anexar comprovante confirma o pagamento automaticamente
- Opção de marcar como pago sem comprovante (não conta no prêmio)

### 📊 Painel de Apostas (tempo real)
- Apostas confirmadas visíveis para todos os participantes em tempo real
- Estatísticas: apostadores, lances totais, pagos com comprovante, pot total
- Lista de participantes com chips visuais

### 💼 Administrador Financeiro
- O criador pode delegar a administração financeira a qualquer participante
- Link de convite disponível para criador e administrador
- Acesso a todos os comprovantes de pagamento das apostas
- Gestão do pagamento do prêmio:
  - Visualizar PIX de cada ganhador
  - Enviar alerta in-app se PIX não cadastrado
  - Ver dados de contato do ganhador (nome, e-mail, telefone)
  - Anexar comprovante de transferência do prêmio
  - Marcar prêmio como pago

### 📋 Painel do Criador
- Visualização de todos os lances pagos com comprovantes
- Acompanhamento de status de pagamento de cada apostador
- Acesso somente leitura ao status do prêmio

### 🏆 Resultado e Premiação
- Administrador informa o placar final
- Sistema calcula ganhadores automaticamente (apenas apostas com comprovante)
- Em caso de empate: prêmio dividido igualmente
- Sem acertadores: valor acumulado para o próximo bolão
- Notificação de parabéns para o(s) ganhador(es) com valor do prêmio
- Alerta automático se ganhador não tem chave PIX cadastrada
- Comprovante do prêmio visível para ganhador, criador e administrador

---

## 🛠 Tecnologias

| Camada | Tecnologia |
|--------|-----------|
| Frontend | HTML5, CSS3, JavaScript (ES Modules) |
| Backend | Firebase (BaaS) |
| Autenticação | Firebase Authentication |
| Banco de dados | Cloud Firestore |
| Armazenamento | Firestore (base64 — sem Firebase Storage, evita CORS) |
| Notificações | Firestore `onSnapshot` (tempo real) |
| Fontes | Google Fonts — Bebas Neue + Inter |
| Deploy | GitHub Pages |

---

## 📦 Pré-requisitos

- Conta no [Firebase Console](https://console.firebase.google.com)
- Conta no [GitHub](https://github.com) para deploy via GitHub Pages
- Nenhuma dependência de build — o projeto é puro HTML/CSS/JS

---

## 🔥 Configuração do Firebase

### 1. Criar o projeto

1. Acesse [console.firebase.google.com](https://console.firebase.google.com)
2. Clique em **"Adicionar projeto"** e siga os passos
3. Desative o Google Analytics se não precisar

### 2. Ativar serviços

No menu lateral do Console Firebase:

**Authentication**
- Clique em **Authentication → Começar**
- Aba **"Sign-in method"** → ative **"E-mail/senha"**

**Firestore Database**
- Clique em **Firestore Database → Criar banco de dados**
- Escolha **modo de produção** (as regras serão configuradas adiante)
- Selecione a região mais próxima (ex: `southamerica-east1`)

### 3. Obter as credenciais

1. Vá em **Configurações do Projeto** (ícone ⚙️)
2. Aba **"Geral"** → seção **"Seus apps"**
3. Clique em **"Adicionar app"** → ícone Web (`</>`)
4. Registre o app e copie o objeto `firebaseConfig`

### 4. Inserir as credenciais no código

Abra `index.html` (ou `bolao-alegria.html`) e substitua o bloco:

```javascript
const app = initializeApp({
  apiKey: "SUA_API_KEY",
  authDomain: "SEU_PROJETO.firebaseapp.com",
  projectId: "SEU_PROJETO_ID",
  storageBucket: "SEU_PROJETO.appspot.com",
  messagingSenderId: "SEU_SENDER_ID",
  appId: "SEU_APP_ID"
});
```

---

## 📐 Estrutura do Banco de Dados

```
firestore/
├── users/
│   └── {userId}
│       ├── name: string
│       ├── email: string
│       ├── phone: string
│       ├── pixRecebimento: string
│       └── createdAt: timestamp
│
├── boloes/
│   └── {bolaoId}
│       ├── name: string
│       ├── team1: string
│       ├── team2: string
│       ├── gameDateTime: string (ISO)
│       ├── valorLance: number
│       ├── pixKey: string
│       ├── lanceUnico: boolean
│       ├── invitedEmails: string[]
│       ├── creatorId: string
│       ├── creatorName: string
│       ├── adminId: string
│       ├── adminName: string
│       ├── status: "open" | "closed" | "finished"
│       ├── acumulado: number
│       ├── resultado: { gol1, gol2 } | null
│       ├── participants: [
│       │   { userId, userName, userEmail, pixRecebimento, joinedAt }
│       │   ]
│       ├── lances: [
│       │   {
│       │     id, userId, userName, userEmail,
│       │     gol1, gol2, pago, concluido,
│       │     comprovante: string (base64) | null,
│       │     timestamp
│       │   }
│       │   ]
│       └── ganhadores: [
│           {
│             userId, name, email, valor,
│             pixRecebimento, premioPago,
│             comprovantePremio: string (base64) | null,
│             dataPagamento
│           }
│           ]
│
└── notificacoes/
    └── {notifId}
        ├── tipo: "convite" | "premio" | "pagamento_recebido" | "alerta_pix"
        ├── userId: string
        ├── bolaoId: string
        ├── bolaoName: string
        ├── lida: boolean
        └── createdAt: timestamp
```

---

## 🔒 Regras do Firestore

Cole as regras abaixo em **Firestore Database → Regras** (no console do Firebase) e clique em **Publicar**:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Usuários: só o próprio usuário edita seu perfil
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }

    // Bolões: qualquer usuário autenticado pode ler e criar
    // Atualização permitida para participantes (lances, comprovantes)
    match /boloes/{bolaoId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if request.auth != null;
    }

    // Notificações: usuário lê/atualiza apenas as próprias
    // Qualquer usuário autenticado pode criar (para notificar outros ao convidar/premiar)
    match /notificacoes/{notifId} {
      allow read, update: if request.auth != null &&
        resource.data.userId == request.auth.uid;
      allow create: if request.auth != null;
    }
  }
}
```

> ⚠️ **Erro comum: "Missing or insufficient permissions" no sininho de notificações**
>
> Se o app exibir esse erro no console ou a mensagem "Erro ao carregar notificações", significa que as regras acima **ainda não foram publicadas** no projeto Firebase, ou que existe uma regra antiga/mais restritiva ativa. Para corrigir:
> 1. Acesse [console.firebase.google.com](https://console.firebase.google.com) → seu projeto → **Firestore Database** → aba **Regras**
> 2. Apague o conteúdo atual e cole o bloco de regras acima
> 3. Clique em **Publicar** (a propagação pode levar até 1 minuto)
> 4. Recarregue o app — o sininho deve voltar a funcionar

---

## 🚀 Deploy (GitHub Pages)

### Opção 1 — Repositório simples

1. Renomeie o arquivo para `index.html`
2. Crie um repositório no GitHub e faça o push:

```bash
git init
git add index.html
git commit -m "feat: Bolão da Alegria v1.0"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/SEU_REPO.git
git push -u origin main
```

3. No GitHub: **Settings → Pages → Branch: main → / (root) → Save**
4. O app estará disponível em: `https://SEU_USUARIO.github.io/SEU_REPO/`

### Opção 2 — Domínio personalizado

Adicione um arquivo `CNAME` na raiz com seu domínio:

```
meubolao.com.br
```

---

## 🔄 Fluxo do Usuário

```
Usuário recebe link de convite
         │
         ▼
  Está logado? ──Não──▶ Banner de convite na tela de login/cadastro
         │                        │
        Sim                  Login/Cadastro
         │                        │
         ▼                        ▼
  Notificação in-app ◀────────────┘
  (modal de convite)
         │
         ▼
  "Participar do Bolão"
         │
         ▼
  Tela do Bolão
  ┌─────────────────────────────┐
  │  VS Hero (times + data)      │
  │  ✅ Apostas Confirmadas      │ ← aberto por padrão
  │  📋 Informações do Bolão    │ ← minimizado
  │  ⚽ Lances                  │ ← minimizado
  │  💼 Painel Admin/Criador    │ ← minimizado
  └─────────────────────────────┘
         │
         ▼
  Faz Lance → Paga PIX → Anexa Comprovante
         │
         ▼
  Admin informa resultado
         │
         ▼
  Ganhador(es) notificados → Admin transfere prêmio → Anexa comprovante
```

---

## 👥 Papéis e Permissões

| Ação | Participante | Criador | Admin Financeiro |
|------|:---:|:---:|:---:|
| Fazer lances | ✅ | ✅ | ✅ |
| Ver apostas confirmadas | ✅ | ✅ | ✅ |
| Ver link de convite | ❌ | ✅ | ✅ |
| Encerrar bolão | ❌ | ✅ | ❌ |
| Ver comprovantes das apostas | ❌ | ✅ | ✅ |
| Informar resultado final | ❌ | ❌ | ✅ |
| Pagar prêmio / anexar comprovante | ❌ | ❌ | ✅ |
| Trocar administrador financeiro | ❌ | ✅ | ❌ |
| Enviar alerta de PIX não cadastrado | ❌ | ❌ | ✅ |

---

## 📱 Compatibilidade Mobile

O app foi desenvolvido mobile-first com suporte nativo a iOS e Android:

- `apple-mobile-web-app-capable` — instalável como PWA no iOS
- `safe-area-inset` — respeita notch e barra de navegação
- `font-size: 16px` nos inputs — evita zoom automático no iOS
- Touch targets mínimos de 44px (Apple Human Interface Guidelines)
- `-webkit-overflow-scrolling: touch` — scroll fluido no iOS
- Sem dependências externas além do Firebase SDK e Google Fonts

Para instalar como app no iPhone: **Safari → Compartilhar → Adicionar à Tela de Início**

---

## 📁 Estrutura do Projeto

```
bolao-da-alegria/
├── index.html          # Aplicação completa (single file)
└── README.md           # Este arquivo
```

> O projeto é intencionalmente um único arquivo HTML para facilitar o deploy
> em GitHub Pages e compartilhamento direto sem necessidade de build tools.

---

## 🤝 Contribuindo

1. Fork o projeto
2. Crie uma branch: `git checkout -b feat/minha-feature`
3. Commit suas mudanças: `git commit -m 'feat: adiciona minha feature'`
4. Push para a branch: `git push origin feat/minha-feature`
5. Abra um Pull Request

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## 👨‍💻 Autor

Desenvolvido com ⚽ e ☕ para unir a galera na Copa!

---

*Última atualização: Junho de 2026*
