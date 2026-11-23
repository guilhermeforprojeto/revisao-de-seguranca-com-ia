# AGENTS.md — revisão de segurança com IA

Você (IA) recebeu o link deste repositório com um pedido parecido com **"revise a segurança seguindo este repositório"**. Faça uma revisão de segurança do projeto aberto seguindo as fases abaixo, na ordem, e entregue o relatório no formato de [`templates/RELATORIO.md`](templates/RELATORIO.md).

Responda no idioma do usuário.

## Regras (valem para a revisão inteira)

1. **Somente leitura por padrão.** Não edite código, não faça commit, não mude configuração. Correções só depois do relatório e com o "pode corrigir" do usuário (fase 9).
2. **Nada de ataque real.** Não rode exploit, fuzzing, scanner de rede ou teste de carga contra servidores em produção ou de terceiros. Para provar uma falha, use teste local, teste unitário ou leitura de código.
3. **Nunca mostre segredos.** Ao achar uma chave, token ou senha, mostre só o arquivo, a linha e os 4 primeiros caracteres (`sk_l****`). Não copie o valor para o relatório, para o chat ou para nenhum arquivo.
4. **Pergunte antes de instalar.** Liste a ferramenta, o comando e o motivo, e espere o OK. Se o usuário não quiser instalar nada, faça a fase com leitura de código e diga o que ficou sem ferramenta.
5. **Skills de terceiros:** só as de [`FERRAMENTAS.md`](FERRAMENTAS.md), só as partes defensivas, e leia o `SKILL.md` e os scripts antes de usar.
6. **Evidência ou não conta.** Todo achado tem arquivo e linha (ou comando e saída). Suspeita sem evidência vai na seção "A investigar", não na lista de falhas.
7. **Projeto grande:** priorize o que fica exposto na internet (rotas, APIs, uploads, login) e diga o que não foi coberto.

## Fases

### 0. Combinar o escopo

Pergunte uma vez, tudo junto, e siga com padrões razoáveis se o usuário não souber:

- O que entra: o repositório inteiro, uma pasta, ou só as mudanças de uma branch/PR.
- Onde roda: hospedagem compartilhada (cPanel), VPS, container, serverless, app desktop/mobile.
- Tem usuários de vários clientes no mesmo banco (multi-tenant)?
- Pode instalar ferramentas? (Gitleaks, Semgrep, OSV-Scanner, Trivy)

Se o usuário pediu uma revisão rápida antes de commit ou deploy, pule para o modo rápido: fases 2, 3 e o item "Injeção" da fase 5.

### 1. Mapear

Antes de procurar falhas, entenda o sistema (o mesmo princípio do `audit-context-building` da Trail of Bits):

- Linguagens, frameworks, versões, gerenciadores de pacote.
- **Pontos de entrada:** rotas HTTP, APIs, webhooks, filas, CLIs, uploads, jobs agendados, extensões, agentes.
- **Fronteiras de confiança:** o que vem do usuário, de terceiros, de IA/LLM, do servidor para um cliente instalado.
- Autenticação, sessões, papéis e onde as permissões são checadas.
- Onde ficam dados sensíveis (senhas, tokens, dados pessoais, pagamentos).

Escreva esse mapa em 10 a 20 linhas no topo do relatório.

### 2. Segredos

- Com ferramenta: `gitleaks git . --redact` (varre o histórico do git) e `gitleaks dir . --redact` (arquivos no disco, inclusive fora do git). Em versões antigas do Gitleaks: `gitleaks detect --source . --redact`.
- Sem ferramenta: procure `.env`, `*.pem`, `*.key`, `id_rsa`, `credentials`, `AKIA`, `sk_live`, `ghp_`, `xox`, `-----BEGIN`, `password =`, `secret =`, `token =`, e confira o `.gitignore`.
- Segredo que já foi commitado é **vazado**, mesmo se apagado depois: a correção é **trocar a chave**, não só remover o arquivo.

### 3. Dependências e cadeia de suprimentos

- `osv-scanner scan source -r .` cobre quase todos os ecossistemas. Alternativas: `npm audit`, `pnpm audit`, `pip-audit`, `composer audit`, `cargo audit`, `govulncheck ./...`, `bundle audit`.
- Para npm, PyPI e Go, se a skill estiver disponível: `supply-chain-risk-auditor` da Trail of Bits (pacotes abandonados, mantenedor único, scripts de instalação).
- Confira: lockfile commitado, versões fixadas, scripts `postinstall` estranhos, pacotes com nome parecido com pacotes famosos (typosquatting).

### 4. Análise estática

- `semgrep scan --config auto --error` (ou `--config p/default`, `p/owasp-top-ten`, e as regras da linguagem: `p/php`, `p/javascript`, `p/python`...).
- Se houver a skill `static-analysis` da Trail of Bits, use-a para Semgrep/CodeQL e leitura de SARIF.
- **Verifique cada achado antes de reportar** (o princípio do `fp-check`): o dado chega mesmo do usuário? Existe validação antes? O código roda em produção? Achado de ferramenta sem confirmação vai para "A investigar".

### 5. Revisão manual

Siga o [`CHECKLIST.md`](CHECKLIST.md). Os itens que mais aparecem em projetos feitos com IA:

- **Controle de acesso:** cada rota checa se *este* usuário pode ver/alterar *este* registro (IDOR). Em multi-tenant, toda consulta filtra pelo tenant/workspace.
- **Injeção:** SQL concatenado, `eval`, `exec`/`system`/`shell=True`, template sem escape, caminhos de arquivo montados com entrada do usuário (path traversal), deserialização insegura.
- **Autenticação e sessão:** hash de senha (bcrypt/argon2), limite de tentativas, recuperação de senha, cookies `HttpOnly`/`Secure`/`SameSite`, JWT sem `alg: none` e com expiração.
- **Uploads:** tipo e tamanho checados no servidor, nome gerado pelo servidor, fora da pasta pública ou sem execução.
- **SSRF:** o servidor busca URLs que o usuário informa?
- **Padrões inseguros** (o princípio do `insecure-defaults`): debug ligado, CORS `*` com credenciais, chave padrão, rota de admin aberta, "se falhar, libera".
- **IA e agentes:** prompt injection a partir de conteúdo externo, ferramentas com mais permissão que o necessário, agente que executa comando vindo de fora, chave de API no front-end.

### 6. Infraestrutura e CI

- **Container:** `trivy config .` e `trivy fs .`; Dockerfile rodando como root, imagem sem versão, segredo em `ENV`/`ARG`.
- **GitHub Actions:** `pull_request_target` com checkout do código do PR, `permissions` amplas, actions sem versão fixa, segredos em logs, workflows com agentes de IA (skill `agentic-actions-auditor` da Trail of Bits).
- **Servidor:** só se for VPS/servidor próprio e o usuário pedir. Use a skill `linux-hardening` (BagelHole) como checklist e **não altere SSH, firewall ou usuários** sem confirmação, porque dá para se trancar fora do servidor.
- **Cabeçalhos HTTP:** HSTS, CSP, `X-Content-Type-Options`, `frame-ancestors`.

### 7. Classificar

Para cada falha confirmada:

- **Severidade** pela CVSS 3.1 (Crítica 9.0–10, Alta 7.0–8.9, Média 4.0–6.9, Baixa 0.1–3.9). Mostre o vetor (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`) e explique em uma frase por que escolheu.
- **Explorabilidade** em linguagem simples: quem consegue, de onde, precisa estar logado?
- **Correção** concreta, com o trecho de código ou o comando.

### 8. Relatório

Use [`templates/RELATORIO.md`](templates/RELATORIO.md). Ordene por severidade. Comece com um resumo de 3 a 5 linhas que alguém não técnico entenda. Salve como `RELATORIO-SEGURANCA.md` na raiz do projeto (pergunte antes se já existir) e **confira que nenhum segredo foi parar nele**.

### 9. Corrigir (só com OK)

Pergunte o que corrigir. Corrija em ordem de severidade, uma falha por commit, com teste quando possível, e rode as ferramentas de novo no fim para mostrar que o achado sumiu. Segredo vazado: oriente a **rotação da chave** no provedor; isso o usuário faz.
