# ADR-002 — Design system

Status: APROVADO em 2026-10-22.
Origem: escrito pelo agente em 2026-10-19, por exceção pedida pelo
usuário à regra "Não escreva ADR" (rules/restrictions.md). As
decisões dos itens em aberto foram propostas pelo agente a pedido do
usuário. Preenche o item "Design system, tokens e identidade visual"
que o ADR-001 deixou em aberto, sem contrariar o ADR-001 nem o que já
está no front/ (branch `csp-api-exata`).

## 1. Fonte do design

| Item | Escolha |
|---|---|
| Fonte de verdade | Figma (ADR-001 §2) |
| Referência de trabalho | `vibing/layout.md`, enquanto ninguém do projeto tiver acesso ao Figma |
| Onde os valores ficam registrados | Neste ADR e em `front/src/index.css` |
| Base visual | Nocturne, só como referência; nem o CSS nem as classes dele entram no código |
| Conflito entre Figma e layout.md | Vale o Figma, quando houver acesso |

- **Fonte de verdade** — o ADR-001 já diz que o design vem do Figma; este ADR não muda isso.
- **Referência de trabalho** — sem acesso ao Figma, o layout.md é o único documento com valores; a landing (spec 001) já foi feita a partir dele.
- **Onde os valores ficam registrados** — o layout.md está fora do git; se ele sumir, os tokens continuam escritos aqui e no código.
- **Base visual** — o Nocturne traz a própria folha de estilos e classes (`.btn`, `.card`, `.dialog`), o que seria um segundo sistema de estilo ao lado do Tailwind.
- **Conflito entre Figma e layout.md** — o layout.md é derivado; quando o original estiver acessível, a cópia se ajusta a ele.

---

## 2. Estilização

| Item | Escolha |
|---|---|
| Estilização | Tailwind CSS v4 (ADR-001 §2) |
| Onde ficam os tokens | Bloco `@theme` de `front/src/index.css` |
| Valor fora da escala | Valor arbitrário do Tailwind (`text-[16.5px]`, `p-[18px]`) |
| Prop `style` do React | Só quando o valor não cabe em classe (gradiente, tamanho vindo de prop) |
| Componentes | shadcn/ui (ADR-001 §2), estilizados com os tokens deste ADR |
| Nomes de token do shadcn | Aliases que apontam para os nossos tokens |

- **Estilização** — já decidido no ADR-001; a landing usa Tailwind com os tokens do layout.md.
- **Onde ficam os tokens** — no Tailwind v4 o tema é CSS; um token declarado ali vira classe (`bg-bg`, `text-accent-500`) e variável CSS ao mesmo tempo.
- **Valor fora da escala** — o layout.md usa medidas exatas (16.5px, 18px, 292px); classe arbitrária mantém o valor no markup sem criar CSS próprio.
- **Prop `style` do React** — o React aplica o estilo pelo CSSOM, que a CSP sem `unsafe-inline` (ADR-001 §10) não bloqueia; atributo `style` escrito no HTML seria bloqueado. É o que a landing já faz no brilho radial e no tamanho da marca.
- **Componentes** — o shadcn copia o componente para o repositório, então a variante pode ser reescrita para o visual do layout.md (o botão primário é contorno, não preenchimento).
- **Nomes de token do shadcn** — os componentes do shadcn leem `--primary`, `--background`, `--border` etc.; apontar esses nomes para os nossos mantém os componentes sem edição e não obriga a renomear o que a landing já usa.

### Aliases do shadcn

Entram no `index.css` quando o shadcn for instalado (spec que precisar dele).

| Nome do shadcn | Aponta para |
|---|---|
| `--background` | `--color-bg` |
| `--foreground` | `--color-text` |
| `--card`, `--popover` | `--color-surface` |
| `--card-foreground`, `--popover-foreground` | `--color-text` |
| `--primary` | `--color-accent-500` |
| `--muted-foreground` | `--color-text` a 55% |
| `--border`, `--input` | `--color-divider` |
| `--ring` | `--color-accent-500` |
| `--destructive` | `--color-falha` |

O que a instalação do shadcn gerar no `index.css` é ajustado para
apontar para os tokens deste ADR; não substitui nenhum deles.

---

## 3. Tokens

### Cor base

| Token | Valor |
|---|---|
| `--color-bg` | `#161826` |
| `--color-surface` | `#232532` |
| `--color-text` | `#e9e9ed` |
| `--color-divider` | `color-mix(in srgb, #e9e9ed 16%, transparent)` |
| `--color-neutral-900` | `#292b31` |

### Acento

| Token | Valor |
|---|---|
| `--color-accent-100` | `#e6faf7` |
| `--color-accent-200` | `#c2f1eb` |
| `--color-accent-300` | `#93e3da` |
| `--color-accent-400` | `#5fcfc4` |
| `--color-accent-500` | `#2fb8ac` (base) |
| `--color-accent-600` | `#26978e` |
| `--color-accent-700` | `#1d746d` |
| `--color-accent-800` | `#15534e` |
| `--color-accent-900` | `#0f3733` |

### Situação

Mesmo padrão do `falha` que já existe: `--color-<estado>` para o
texto, `-fundo` e `-borda`.

| Estado | `--color-<estado>` | `-fundo` | `-borda` |
|---|---|---|---|
| `disponivel` (disponível, em dia) | `#5fcfc4` | `rgb(47 184 172 / 0.12)` | `rgb(47 184 172 / 0.40)` |
| `emprestado` (emprestado, a vencer) | `#c9a25f` | `rgb(201 162 95 / 0.12)` | `rgb(201 162 95 / 0.38)` |
| `falha` (em atraso, erro) — já existe | `#e88b8b` | `rgb(200 90 90 / 0.14)` | `rgb(200 90 90 / 0.40)` |
| `manutencao` (em manutenção, inativo) | `#9aa0ad` | `rgb(154 160 173 / 0.10)` | `rgb(154 160 173 / 0.30)` |
| `bloqueio` (banner de atraso) | `#e88b8b` | `rgb(122 59 59 / 0.14)` | `#7a3b3b` |

### Raio

| Token | Valor | Uso |
|---|---|---|
| `--radius-sm` | `4px` | — |
| `--radius-md` | `8px` | botão, input, linha de lista |
| `--radius-lg` | `14px` | card, painel, modal |

- **Cor** — valores do layout.md §1, os mesmos que já estão no `index.css`.
- **Situação** — o layout.md já agrupa "em atraso" e "erro" na mesma cor, então o `falha` existente cobre os dois; os nomes em português seguem o padrão que o código já tem.
- **Raio** — layout.md §3; já está no `index.css`.

---

## 4. Tipografia

| Item | Escolha |
|---|---|
| Família | Raleway, única, em título e corpo |
| Pesos carregados | 400, 500, 600, 700 |
| Origem do arquivo | Google Fonts |
| Peso | Título 600, corpo 400, marca 700 |
| Altura de linha | Título 1.12, corpo 1.55 |
| Escala | layout.md §2 |

- **Família** — layout.md §2; já aplicada no `index.css`.
- **Origem do arquivo** — já aprovada no plano da spec 001 e liberada na CSP do `vercel.json`; não adiciona dependência ao projeto.
- **Escala** — os tamanhos são exatos por elemento (44px no H1 do login, 16.5px no título de card); viram valor arbitrário do Tailwind (§2).

---

## 5. Tema

| Item | Escolha |
|---|---|
| Tema | Escuro, único |
| Variante clara | Nenhuma |

- **Tema** — é o único tema que o layout.md define; um tema claro seria um segundo conjunto de tokens que ninguém desenhou.
- **Variante clara** — sem variante, nenhum componente precisa de classe `dark:`.

---

## 6. Ícones

| Item | Escolha |
|---|---|
| Biblioteca | lucide-react |
| Quando entra | Junto com o shadcn/ui, na spec que precisar dele |

- **Biblioteca** — é a que os componentes do shadcn já importam; o Phosphor, citado no layout.md §4, seria uma segunda biblioteca de ícones ou exigiria reescrever os componentes. O layout.md só pede ícone "quando houver".
- **Quando entra** — a landing não usa ícone; instalar antes não tem uso.

---

## 7. Acessibilidade

| Item | Escolha |
|---|---|
| Contraste de texto | WCAG 2.2 AA (4.5:1 para texto normal) |
| Opacidade mínima de texto secundário | 55% de `--color-text` |
| Exceção | Elemento desabilitado (`opacity .45`) |
| Foco | `outline: 2px solid` acento, `outline-offset: 2px` |

- **Contraste de texto** — boa parte do texto é pequeno (11 a 13px), justamente onde contraste baixo mais atrapalha a leitura.
- **Opacidade mínima** — pela conta do agente, 55% sobre `--color-bg` dá cerca de 5,2:1 e sobre `--color-surface` cerca de 4,8:1; 45% dá cerca de 3,9:1 e 40% cerca de 3,3:1, abaixo do AA. Os níveis 45% e 40% do layout.md passam para 55%. A conta deve ser conferida com ferramenta na tarefa que aplicar.
- **Exceção** — o WCAG não exige contraste de componente desabilitado.
- **Foco** — layout.md §4; já está no `index.css`.

---

## 8. Movimento

| Item | Escolha |
|---|---|
| Animação | Só `fadeUp` (opacidade e 6px de deslocamento vertical) |
| Duração | Card .25s, modal .18s, toast .2s |
| Proibido | Escala, sombra pulsante, parallax, animação em número, tabela ou navegação |

- **Animação** — layout.md §8; o `@keyframes fadeUp` já está no `index.css`.

---

## 9. O que do layout.md não é adotado

| Trecho do layout.md | O que vale |
|---|---|
| §10: folha única `styles.css` do Nocturne | Tailwind com os tokens deste ADR (§2) |
| §4 e §10: classes do Nocturne, estilo inline sem classe própria | Classes do Tailwind e componentes do shadcn (§2) |
| §4: ícones Phosphor | lucide-react (§6) |
| §1: texto secundário a 45% e 40% | Mínimo de 55% (§7) |
| §6 e §10: troca de tela com estado em memória (`view`, `role`, `items[]`) | Uma rota por tela no React Router; estado de servidor no TanStack Query e de cliente no Zustand (ADR-001 §2) |
| §10: controles de protótipo (`startView`, `demoState`) | Não entram no app |

- **Troca de tela e estado em memória** — descrevem o protótipo, não o app; o ADR-001 já decidiu roteamento e estado.

---

## Alternativas descartadas

| Alternativa | Motivo do descarte |
|---|---|
| Folha `styles.css` do Nocturne ao lado do Tailwind | Dois sistemas de estilo no mesmo app; contraria a escolha de Tailwind do ADR-001 |
| Renomear os nossos tokens para os nomes do shadcn | Obriga a mexer na landing e afasta os nomes do layout.md, sem ganho |
| Phosphor Icons | Segunda biblioteca de ícones ao lado da que o shadcn já traz |
| Raleway hospedada no próprio front (`@fontsource/raleway` ou arquivos commitados) | Dependência nova e mudança no que a spec 001 já aprovou; fica como alternativa se o ponto de LGPD pesar |
| Tema claro, ou claro e escuro | Ninguém desenhou o tema claro |
| Manter 40% e 45% de opacidade no texto | Abaixo do contraste AA nos tamanhos usados |

---

## Consequências

### Fica mais fácil

- Componente do shadcn entra sem edição de cor: os aliases já apontam para os tokens.
- Um valor visual tem um lugar só no código: o `@theme` do `index.css`.
- Os tokens continuam registrados mesmo se o layout.md, que está fora do git, se perder.

### Fica mais difícil

- A landing já publicada usa texto a 45% e 40%; precisa de uma tarefa no front/ para subir para 55%, e o resultado fica um pouco mais claro do que o layout.md.
- Cada variante do shadcn que preenche com a cor primária (o Button padrão, por exemplo) precisa ser reescrita para contorno.
- Sem acesso ao Figma, divergência entre ele e o layout.md só aparece quando alguém abrir o Figma.
- Toda visita faz uma requisição ao Google Fonts com o IP de quem acessa.

### O que isso impede de fazer depois sem custo

- Tema claro: exige desenhar e declarar um segundo conjunto de tokens e revisar cada componente.
- Trocar a biblioteca de ícones: os componentes do shadcn importam lucide-react diretamente.

---

## O que este ADR não decide

- Contraste de elementos que não são texto (bordas de input, divisores a 16%), critério 1.4.11 do WCAG.
- Comportamento responsivo e largura mínima; o layout.md só tem larguras de referência de 1360 a 1440px.
- Se o login oferece "Entrar como Operações": o papel vem da membership e do JWT (ADR-001 §6 e §7), e a pergunta é do PRD.
- Textos e copy das telas, além do tom descrito no layout.md §10.
- Acesso ao Figma e quem mantém o layout.md em dia com ele.
