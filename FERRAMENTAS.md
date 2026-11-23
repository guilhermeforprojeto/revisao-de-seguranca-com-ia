# Ferramentas

Ferramentas de linha de comando e skills para Claude Code que a revisão usa, e quando usar cada uma. Todas foram conferidas nos repositórios oficiais; mesmo assim, **leia o `SKILL.md` e os scripts antes de instalar** qualquer skill: ela roda com as permissões do seu usuário.

## Ferramentas de linha de comando

| Ferramenta | Fase | O que faz | Instalar |
| --- | --- | --- | --- |
| [Gitleaks](https://github.com/gitleaks/gitleaks) | 2 | Acha chaves, tokens e senhas no código e no histórico do git | `brew install gitleaks`, `winget install gitleaks`, ou binário da página de releases |
| [OSV-Scanner](https://github.com/google/osv-scanner) | 3 | Vulnerabilidades conhecidas nas dependências de quase todos os ecossistemas | `go install github.com/google/osv-scanner/v2/cmd/osv-scanner@latest` ou binário |
| [Semgrep](https://github.com/semgrep/semgrep) | 4 | Análise estática com regras prontas (OWASP, por linguagem) | `pip install semgrep` ou `brew install semgrep` |
| [Trivy](https://github.com/aquasecurity/trivy) | 6 | Dockerfile, Kubernetes, Terraform, imagens e dependências | `brew install trivy` ou binário |
| Auditor do gerenciador | 3 | `npm audit`, `pip-audit`, `composer audit`, `cargo audit`, `govulncheck` | Já vem ou um `install` |

## Skills para Claude Code

| Skill / repositório | Foco | Quando usar | Cuidado |
| --- | --- | --- | --- |
| [trailofbits/skills](https://github.com/trailofbits/skills) | Auditoria avançada: `static-analysis` (Semgrep, CodeQL, SARIF), `audit-context-building`, `fp-check`, `insecure-defaults`, `sharp-edges`, `supply-chain-risk-auditor`, `differential-review`, `agentic-actions-auditor` | A melhor base para revisão de código. Instale só os plugins que vai usar | Licença CC BY-SA 4.0. A Snyk deu risco médio a configurações que clonam e carregam skills externas automaticamente: instale manualmente |
| [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) — Security Reviewer | DevSecOps: orquestra scanners e gera relatório com severidade e plano de correção | Quando quiser um segundo revisor com outro formato de relatório | Vem dentro do plugin `fullstack-dev-skills`, que traz muitas outras skills; instale e desative as que não usar |
| [BagelHole/DevOps-Security-Agent-Skills](https://github.com/BagelHole/DevOps-Security-Agent-Skills) — `linux-hardening` | Servidor Linux: CIS Benchmarks, SSH, sysctl, usuários, firewall | Só com VPS ou servidor próprio. Em hospedagem compartilhada (cPanel) você não tem root para aplicar | Mudanças em SSH e firewall podem te trancar fora do servidor: aplique com outra sessão aberta |
| [Masriyan/Claude-Code-CyberSecurity-Skill](https://github.com/Masriyan/Claude-Code-CyberSecurity-Skill) | 22 skills de segurança: varredura de vulnerabilidades, web, logs/SIEM, resposta a incidentes, cadeia de suprimentos | Use só as **defensivas** (02 Vulnerability Scanner, 09 Web Security, 12 Log Analysis & SIEM, 07 Incident Response, 20 Supply Chain) | Traz também skills **ofensivas** (exploit, red team) e scripts Python; antivírus pode acusar. Não instale a coleção inteira globalmente |
| [FlorianBruniaux/claude-code-ultimate-guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) | Guia de Claude Code com seção de hardening e um prompt de auditoria de permissões (`tools/permissions-audit-prompt.md`) | Para revisar a configuração do próprio Claude Code (permissões, hooks, MCPs) | Não achamos nele um comando "quick audit" de código; a triagem rápida deste repositório (fases 2, 3 e injeção) cumpre esse papel |

### Instalação dos plugins da Trail of Bits

No Claude Code:

```text
/plugin marketplace add trailofbits/skills
/plugin menu
```

E escolha só os plugins da tabela acima.

## No CI

Exemplos prontos em [`exemplos/github-actions/`](exemplos/github-actions): Gitleaks em todo push/PR e Semgrep em PRs. Copie para `.github/workflows/` do seu projeto.
