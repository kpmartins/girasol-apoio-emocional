# Setembro Amarelo — Protótipo funcional

Site com 3 páginas (Home, Chats, Ajuda), tema visual de girassóis, chat em tempo
real por salas de assunto e localizador de UBS/CAPS por endereço.

## Estrutura de arquivos
```
setembro-amarelo/
├── index.html          Home
├── chats.html           Salas de conversa (chat em tempo real)
├── ajuda.html            CVV + localizador de UBS/CAPS
├── css/style.css        Identidade visual
├── server.js            Servidor: serve o site + guarda a chave da IA (/api/chat)
├── .env.example         Modelo do arquivo .env (copie para .env e coloque a chave)
└── js/
    ├── common.js          Navbar + faixa fixa do CVV
    ├── firebase-config.js Credenciais do Firebase (você precisa preencher)
    ├── chats.js            Lógica do chat em tempo real
    └── ajuda.js             Lógica do localizador de endereço
```

## Passo 1 — Rodar o site localmente
Como `chats.js` usa `import` (ES modules), não dá para abrir o `index.html`
direto no navegador (`file://`) — é preciso servir por http. A forma mais
simples:

```bash
cd setembro-amarelo
node server.js
```

Depois abra **http://localhost:8000** no navegador. Precisa do Node 18+
(nodejs.org); não há nada para instalar com `npm`.

⚠️ Use `node server.js` em vez de `python3 -m http.server`: o servidor simples
não tem a rota `/api/chat` que a IA usa.

## Passo 1.5 — Ativar a IA (Girassol) nas salas
A IA participa de cada sala de conversa e responde às mensagens. A chave da
API **nunca** vai para o navegador; ela fica só no servidor.

1. Crie uma chave em [console.anthropic.com](https://console.anthropic.com) (API Keys).
2. Copie `.env.example` para `.env` e troque `cole_sua_chave_aqui` pela chave.
3. Rode `node server.js` e abra a página Chats.

Sem Firebase preenchido, o chat entra em **modo local** (as mensagens ficam só
no seu navegador e só a IA responde) — ótimo para testar. Com Firebase, a IA
responde dentro da sala e todo mundo vê.

Pontos de atenção:
- **Não publique o `.env`** (já está no `.gitignore`).
- O servidor limita o uso por IP para proteger a sua chave de abuso.
- Para publicar o site, a hospedagem precisa rodar Node (Render, Railway, etc.);
  hospedagem só de arquivos estáticos (GitHub Pages) não executa o `server.js`.
- O comportamento da IA (tom, o que faz em sinais de risco) está em
  `SYSTEM_PROMPT`, no `server.js`.

## Passo 1.8 — Publicar o site (Render, gratuito)
1. Suba a pasta para um repositório no GitHub (o `.env` não vai junto, está no `.gitignore`).
2. Em [render.com](https://render.com): **New → Blueprint** e escolha o repositório. O `render.yaml` já configura tudo.
3. Quando o Render pedir, cole a `ANTHROPIC_API_KEY`.
4. Em Firebase, adicione o domínio do Render em **Authentication → Settings → Authorized domains** se usar login no futuro.

No plano gratuito o site "dorme" após alguns minutos sem acesso e a primeira visita demora cerca de 1 minuto.

## Passo 2 — Ativar o chat de verdade (Firebase, gratuito)
O Chats já está pronto para funcionar com mensagens reais e sincronizadas
entre pessoas, mas precisa de um projeto Firebase (plano gratuito é
suficiente):

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) e crie um projeto novo (gratuito).
2. No menu lateral, vá em **Build → Firestore Database → Criar banco de dados**.
   - Escolha **modo de teste** por enquanto (permite leitura/escrita livre por
     30 dias — ótimo para o protótipo da faculdade; antes de ir para produção
     de verdade, configure regras de segurança).
3. Vá em **Configurações do projeto (ícone de engrenagem) → Geral → Seus apps
   → Web (`</>`)** e registre um app. O Firebase vai te mostrar um objeto
   `firebaseConfig` com `apiKey`, `projectId`, etc.
4. Copie esses valores para dentro de `js/firebase-config.js`, substituindo os
   textos de exemplo.
5. Salve, recarregue a página `chats.html` — pronto, as mensagens agora são
   reais e em tempo real (teste abrindo em duas abas ou dois navegadores).

Sem esse passo, o Chats mostra um aviso explicando que as credenciais
precisam ser preenchidas — o resto do site (Home e Ajuda) funciona
normalmente sem o Firebase.

## Passo 3 — Localizador de UBS/CAPS
Já funciona de verdade: o campo de endereço usa o **Nominatim**
(OpenStreetMap), que é gratuito e não exige chave de API, para converter o
endereço digitado em coordenadas. A partir daí, o site calcula a distância
até uma lista de unidades de saúde.

⚠️ **Importante:** a lista de UBS/CAPS em `js/ajuda.js` (`MOCK_UNITS`) é uma
lista de **exemplo**, com algumas unidades reais de capitais brasileiras,
apenas para o protótipo funcionar de ponta a ponta. Para um TCC ou uso real,
troque por dados oficiais do **CNES/DataSUS** (Cadastro Nacional de
Estabelecimentos de Saúde), que lista todas as UBS e CAPS do Brasil com
endereço e coordenadas.

## O que eu expandiria primeiro
Por ordem de impacto para o projeto de faculdade:

1. **Moderação real do chat.** Hoje existe só um alerta automático por
   palavras-chave (`ALERT_KEYWORDS` em `chats.js`) — é um recurso simples e
   didático, não uma triagem clínica. Para algo mais robusto: um botão de
   "denunciar mensagem", um painel simples de moderação, ou um aviso mais
   claro de que humanos não estão monitorando a sala 24h.
2. **Dados reais de UBS/CAPS via CNES**, em vez da lista de exemplo — isso
   sozinho torna o "Ajuda" genuinamente útil para qualquer endereço do Brasil.
3. **Autenticação leve** (ex: Firebase Auth anônimo) para evitar spam nas
   salas sem exigir cadastro — mantém o anonimato mas dá uma trilha técnica
   mínima em caso de abuso.
4. **Regras de segurança do Firestore** (já escritas em `firestore.rules`: cole em Firestore → Regras → Publicar) antes de qualquer uso além do
   protótipo — o "modo de teste" do passo 2 é aberto demais para produção.
5. **Testes com a persona real** (e, se possível, pessoas reais dentro do
   público-alvo) para validar se o tom do texto e as cores realmente
   transmitem acolhimento — copy e microtexto costumam mudar bastante depois
   desse teste.

## Sobre o tema sensível deste projeto
Este é um protótipo acadêmico. Se ele for usado além da sala de aula —
publicado ou divulgado para pessoas reais —, vale conversar com seu
orientador/professor sobre acompanhamento de um profissional de saúde mental
por trás do chat, já que o tema (saúde mental, possível ideação suicida)
exige cuidado além do que uma ferramenta de software sozinha resolve.
