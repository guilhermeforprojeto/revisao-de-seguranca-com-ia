# Checklist da revisão manual

Marque `OK`, `FALHA` (vai para o relatório com evidência), `N/A` ou `A INVESTIGAR`. Baseado no OWASP Top 10 (2021) e no OWASP ASVS, com itens extras para projetos feitos com IA.

## 1. Controle de acesso

- [ ] Toda rota que lê ou altera dados exige login, menos as públicas de propósito.
- [ ] A permissão é checada **no servidor**, não só escondendo o botão no front-end.
- [ ] Ao buscar um registro por ID, o código confere que ele pertence ao usuário (sem IDOR).
- [ ] Multi-tenant: toda consulta filtra pelo tenant/workspace; não dá para trocar o ID na URL e ver outro cliente.
- [ ] Rotas de admin exigem papel de admin.
- [ ] Banco com RLS (Supabase/Postgres): RLS ligado em todas as tabelas expostas, com políticas testadas.

## 2. Autenticação e sessão

- [ ] Senhas com bcrypt, scrypt ou argon2. Nunca MD5/SHA1/texto puro.
- [ ] Limite de tentativas de login e de recuperação de senha.
- [ ] Token de recuperação de senha de uso único, com expiração curta.
- [ ] Cookies de sessão `HttpOnly`, `Secure`, `SameSite=Lax` ou `Strict`.
- [ ] JWT: algoritmo fixado no servidor, expiração, segredo forte fora do código.
- [ ] Logout invalida a sessão no servidor.
- [ ] Tokens de API/aparelho guardados como hash no banco e revogáveis.

## 3. Injeção

- [ ] SQL só com parâmetros/ORM; nenhuma string concatenada com entrada do usuário.
- [ ] Nenhum `eval`, `new Function`, `exec`, `system`, `shell=True`, `child_process.exec` com dado externo.
- [ ] Caminhos de arquivo não são montados com entrada do usuário sem normalizar e limitar a uma pasta (path traversal).
- [ ] HTML gerado com escape (sem `dangerouslySetInnerHTML`, `v-html`, `{!! !!}` com dado do usuário).
- [ ] Deserialização só de formatos seguros (nada de `pickle`/`unserialize` de dado externo).
- [ ] Consultas NoSQL não aceitam objetos do usuário direto (`$where`, `$ne`).

## 4. Dados e criptografia

- [ ] HTTPS em tudo; HSTS ligado.
- [ ] Dados pessoais e segredos não vão para logs.
- [ ] Nenhuma chave de API privada no front-end, no app mobile ou no executável distribuído.
- [ ] Backups existem e não ficam em pasta pública.
- [ ] LGPD/GDPR: dá para exportar e apagar os dados de um usuário.

## 5. Uploads e arquivos

- [ ] Tipo e tamanho checados no servidor.
- [ ] Nome do arquivo gerado pelo servidor.
- [ ] Arquivos fora da pasta pública ou servidos sem permissão de execução.
- [ ] Imagens/PDFs processados com biblioteca atualizada.
- [ ] Upload em partes confere hash e tamanho no fim.

## 6. Requisições para fora (SSRF) e integrações

- [ ] Se o servidor busca URL informada pelo usuário, bloqueia IPs internos (`127.0.0.1`, `169.254.169.254`, `10.*`, `192.168.*`) e só aceita `http(s)`.
- [ ] Webhooks recebidos têm assinatura verificada.
- [ ] Chamadas a terceiros têm timeout.

## 7. Configuração e padrões inseguros

- [ ] Debug/stack trace desligado em produção.
- [ ] CORS sem `*` quando há cookies ou credenciais.
- [ ] Nenhuma senha, chave ou usuário padrão ativo.
- [ ] Falha de autorização ou de validação **bloqueia**, nunca libera.
- [ ] Cabeçalhos: CSP, `X-Content-Type-Options: nosniff`, `frame-ancestors`/`X-Frame-Options`, `Referrer-Policy`.
- [ ] Rate limit em login, cadastro, envio de e-mail/SMS e rotas caras (IA, upload).

## 8. IA, LLM e agentes

- [ ] Conteúdo externo (página, e-mail, arquivo, resposta de API) que vai para um LLM é tratado como dado, não como instrução; o modelo não consegue chamar ferramentas perigosas por causa dele.
- [ ] Ferramentas do agente com o mínimo de permissão; ações destrutivas pedem confirmação humana.
- [ ] Agente instalado em máquina de usuário só executa uma lista fechada de ações com parâmetros validados, nunca comando livre vindo do servidor.
- [ ] Chave do provedor de IA só no servidor; limite de gasto configurado.
- [ ] Saída do LLM tratada como entrada não confiável antes de virar HTML, SQL ou comando.
- [ ] Skills, MCPs e plugins de terceiros revisados antes de instalar.

## 9. Dependências e build

- [ ] Lockfile commitado.
- [ ] Nenhuma vulnerabilidade Crítica/Alta conhecida sem plano.
- [ ] Scripts de instalação (`postinstall`) de dependências revisados.
- [ ] Executável/instalador distribuído é assinado; atualização automática verifica assinatura.

## 10. CI/CD e repositório

- [ ] Workflows com `permissions` mínimas.
- [ ] Nenhum `pull_request_target` que faz checkout e roda código do PR.
- [ ] Actions de terceiros fixadas por versão (de preferência por SHA).
- [ ] Segredos só em GitHub Secrets/variáveis do provedor, nunca em arquivo.
- [ ] Branch principal protegida; varredura de segredos (Gitleaks) no CI.

## 11. Servidor (só VPS/servidor próprio)

- [ ] SSH sem senha (só chave), sem login de root.
- [ ] Firewall libera só as portas necessárias.
- [ ] Atualizações de segurança automáticas.
- [ ] Serviços rodando com usuário próprio, sem root.
- [ ] Banco de dados não exposto na internet.
