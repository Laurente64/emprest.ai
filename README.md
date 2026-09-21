# EMPREST.AI — vibing

Documentação e instruções de agente do EMPREST.AI. O código vive em
outros dois repositórios:

- API: https://github.com/Laurente64/emprest.ai-back
- SPA: https://github.com/Laurente64/emprest.ai-front

## Montando a pasta do projeto

Os três repositórios ficam lado a lado dentro de uma pasta emprest.ai/,
e o agente é sempre aberto nessa pasta, não dentro de um dos repos:

    emprest.ai/
    ├── CLAUDE.md   ← cópia de vibing/setup/CLAUDE.root.md, fora do git
    ├── vibing/     (este repositório)
    ├── back/
    └── front/

Depois de clonar os três, rode a partir de emprest.ai/ (funciona em
Git Bash, PowerShell, macOS e Linux):

    cp vibing/setup/CLAUDE.root.md CLAUDE.md

Sem esse arquivo, o agente abre sem nenhuma instrução do projeto.

Rode de novo sempre que vibing/setup/CLAUDE.root.md mudar. Não edite o
CLAUDE.md da raiz: edite a fonte, commite em vibing/ e copie.
