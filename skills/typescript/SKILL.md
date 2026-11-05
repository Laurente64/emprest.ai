---
name: typescript
description: Regras de código limpo em TypeScript para o EMPREST.AI, adaptadas do clean-code-typescript (labs42io). Use ao escrever, alterar ou revisar qualquer arquivo .ts ou .tsx em back/ (NestJS) ou front/ (React + Vite).
---

# TypeScript limpo
> leitor: agente

Fonte: https://github.com/labs42io/clean-code-typescript (README, MIT).
Os conflitos já conhecidos com ADR, tsconfig e lint estão resolvidos em
"Adaptações ao EMPREST.AI", abaixo, que tem precedência sobre o resto
deste arquivo. Conflito com ADR ou spec que não esteja lá: PARE e avise
(rules/restrictions.md).

Os exemplos usam o vocabulário do PRD só como ilustração: não são o
schema, e regra de negócio só vale como estiver na spec aprovada.

## Adaptações ao EMPREST.AI

- **`enum`**: o front/ tem `erasableSyntaxOnly: true`, que proíbe
  `enum`, `namespace` e parameter properties. No front/, use array
  `as const` com union derivada. No back/, se o valor atravessa a API,
  use `z.enum`: é o schema Zod que gera o OpenAPI (ADR-001 §3, §4).

  ```ts
  // front/
  const SITUACOES = ['disponivel', 'emprestado', 'manutencao'] as const;
  type Situacao = (typeof SITUACOES)[number];

  // back/
  export const situacaoSchema = z.enum(['disponivel', 'emprestado', 'manutencao']);
  export type Situacao = z.infer<typeof situacaoSchema>;
  ```

- **Parameter properties** (`constructor(private readonly x: X) {}`):
  é o idioma de injeção do Nest e vale no back/. No front/ não compila:
  declare o campo e atribua no construtor.
- **Contexto do request**: no back/, `tenant_id` e `user_id` vêm do
  `AsyncLocalStorage`, nunca de parâmetro nem do body (ADR-001 §3, §6).
  `listarEmAberto()`, não `listarEmAberto(tenantId, colaboradorId)`.
- **Erros no back/**: a resposta HTTP de erro segue RFC 9457 Problem
  Details (ADR-001 §4). Não devolva string solta nem formato próprio.
- **Validação**: entrada e configuração são validadas com Zod
  (`nestjs-zod` no back/, `zodResolver` no front/). Não escreva
  validação manual de formato nem use `class-validator`.
- **Camadas**: no back/, Controller → Service → Repository, por domínio
  em `src/modules/<dominio>/` (ADR-001 §3). O Repository isola o Prisma
  do Service.
- **Cliente da API no front/**: gerado pelo orval. Nunca edite
  `src/api/generated` nem escreva tipo de resposta à mão.
- **Imports**: o README pede linha em branco entre os grupos; o código
  atual não usa, então não use. Aliases: só os que já existem no
  tsconfig; `paths` novo é alteração de config: proponha e espere.
- **Formatação**: ESLint + Prettier já configurados. Não discuta formato
  nem reformate arquivo fora do escopo; rode `npm run lint`.
- **Idioma dos nomes**: o código mistura português (`ARQUIVO`,
  `EstadoDaApi`) e inglês (`HealthService`, `loadEnv`). Siga o idioma
  do módulo que você está tocando. Em módulo novo, pergunte.
- **Testes**: back/ usa Jest e supertest; front/ usa Playwright. Teste
  de banco usa Testcontainers, nunca o Supabase (rules/checks.md).

## Variáveis

- Nomes com significado e pronunciáveis; nada de número mágico: dê nome
  à constante.
- O mesmo conceito tem o mesmo nome em todo lugar.
- Variáveis explicativas em vez de índice de array ou expressão longa.
- Sem mapeamento mental: `emprestimo`, não `e`, em callback com mais de
  uma linha.
- Sem contexto repetido: em `Equipamento`, o campo é `nome`, não
  `equipamentoNome`.
- Parâmetro com default em vez de `||`, que também troca `0` e `''`.

```ts
// Ruim: dias = 0 vira 14
function calcularPrazo(retirada: Date, dias?: number): Date {
  const prazoEmDias = dias || 14;
}

// Bom
const PRAZO_PADRAO_EM_DIAS = 14;
function calcularPrazo(retirada: Date, dias = PRAZO_PADRAO_EM_DIAS): Date {}
```

## Funções

- Dois argumentos ou menos. Passou disso, receba um objeto tipado e
  desestruture.
- Uma função faz uma coisa, num só nível de abstração.
- O nome diz o que a função faz: `adicionarDias(data, 14)`, não
  `adicionarAData(data, 14)`.
- Remova duplicação, mas sem abstração forçada: coisas parecidas com
  motivos diferentes de mudar podem ficar separadas.
- Defaults de objeto por desestruturação. Se o valor vem de fora, o
  default vai no schema Zod (`.default()`).
- Sem parâmetro booleano de modo: faça duas funções.
- Sem efeito colateral escondido: não mute argumento, devolva cópia.
  Efeito real (banco, serviço externo) fica num lugar só: no back/, o
  Repository.
- Não altere global nem protótipo de tipo nativo.
- Encapsule condicional com nome e prefira a forma positiva.
- `switch` repetido sobre o mesmo tipo vira `Record<Union, ...>` ou
  polimorfismo.
- Não compense tipo fraco com `typeof` em runtime: tipe direito e
  valide na fronteira com Zod.
- Não otimize antes de medir. Remova código morto: o git guarda.
- Sequência grande ou paginada: iterator/generator. Respeite o teto de
  60s por request (ADR-001 §9).

```ts
// Ruim: filtra, formata e monta texto no mesmo laço, e lê o relógio
function relatorioDeAtrasos(emprestimos: Emprestimo[]): string {
  let texto = '';
  for (const e of emprestimos) {
    if (!e.devolvidoEm && e.prazo < new Date()) {
      texto += `${e.colaborador.nome}: ${e.equipamento.nome}\n`;
    }
  }
  return texto;
}

// Bom: uma coisa por função, condição com nome, `hoje` vem de fora
function relatorioDeAtrasos(emprestimos: readonly Emprestimo[], hoje: Date): string {
  return emprestimos
    .filter((emprestimo) => estaAtrasado(emprestimo, hoje))
    .map(linhaDoRelatorio)
    .join('\n');
}

function estaAtrasado(emprestimo: Emprestimo, hoje: Date): boolean {
  return emprestimo.devolvidoEm === null && emprestimo.prazo < hoje;
}
```

```ts
// Ruim: muta o array de quem chamou
function selecionar(selecionados: Equipamento[], equipamento: Equipamento): void {
  selecionados.push(equipamento);
}

// Bom: devolve cópia; no React, é o único jeito de re-renderizar
setSelecionados((atuais) => [...atuais, equipamento]);
```

```ts
// Bom: situação nova sem rótulo não compila, ao contrário do switch
const ROTULO_DA_SITUACAO: Record<Situacao, string> = {
  disponivel: 'Disponível',
  emprestado: 'Emprestado',
  manutencao: 'Em manutenção',
};
```

## Objetos e estruturas de dados

- `private`, `readonly` e getter quando há regra para ler ou escrever;
  não para embrulhar campo à toa.
- Imutabilidade: `readonly` em campo, `readonly T[]` em parâmetro,
  `as const` em literal, `toSorted` em vez de `sort` no array alheio.
- `type` para union e intersecção; `interface` para `extends` e
  `implements`. No resto, siga o módulo.

## Classes

- Classe pequena, uma responsabilidade (medida por responsabilidade,
  não por linha).
- Alta coesão, baixo acoplamento.
- Composição antes de herança. Estado é dado, não subclasse.
- Method chaining quando a biblioteca oferece (Zod, Prisma); não
  invente builder.

```ts
// Ruim: ir para manutenção exigiria trocar o objeto de classe, e
// emprestar() lança no filho, quebrando o contrato do pai (LSP)
class NotebookEmManutencao extends Notebook {
  emprestar(): void {
    throw new Error('em manutenção');
  }
}

// Bom
type Equipamento = { id: string; nome: string; categoria: Categoria; situacao: Situacao };
```

## SOLID

- **SRP**: um motivo para mudar.
- **OCP**: caso novo é implementação nova, não mais um `if`.
- **LSP**: subtipo substitui o tipo base sem quebrar o contrato.
- **ISP**: interfaces pequenas; ninguém depende de método que não usa.
- **DIP**: receba a dependência de fora; no back/, via injeção do Nest.
  Antes de instanciar o client do banco, leia rules/secrets.md.

```ts
// Ruim: o Service cria a dependência e fala direto com o Prisma
@Injectable()
export class EmprestimosService {
  private readonly prisma = new PrismaClient();
}

// Bom: o Service orquestra, o Repository fala com o banco, a regra é pura
@Injectable()
export class EmprestimosService {
  constructor(
    private readonly equipamentos: EquipamentosRepository,
    private readonly emprestimos: EmprestimosRepository,
  ) {}

  async solicitar(equipamentoId: string): Promise<Emprestimo> {
    const equipamento = await this.equipamentos.buscarPorId(equipamentoId);
    const emAberto = await this.emprestimos.listarEmAberto();
    garantirQuePodeEmprestar(equipamento, emAberto, new Date());
    return this.emprestimos.criar(equipamento.id);
  }
}
```

## Testes

- Rápido, independente, repetível, auto-verificável e escrito junto com
  o código (F.I.R.S.T.).
- Um conceito por teste, com nome que diz a intenção.

```ts
// Ruim: it('funciona') com vários expect e new Date(): o resultado
// muda conforme o dia em que roda

// Bom
describe('estaAtrasado', () => {
  const hoje = new Date('2026-10-01T12:00:00Z');

  it('é falso quando o item já foi devolvido, mesmo depois do prazo', () => {
    const emprestimo = umEmprestimo({
      prazo: new Date('2026-09-30'),
      devolvidoEm: new Date('2026-09-29'),
    });
    expect(estaAtrasado(emprestimo, hoje)).toBe(false);
  });
});
```

## Concorrência

- `async`/`await` em vez de callback ou cadeia de `.then`.
- Operações independentes em `Promise.all`. No back/, isso não acelera
  consultas do Prisma: com `connection_limit=1` (ADR-001 §5) elas entram
  em fila na mesma conexão.

## Tratamento de erro

- Lance e rejeite sempre `Error` ou subclasse. Subclasse quando o
  chamador precisa distinguir o erro.
- Não engula erro: trate de verdade ou relance com contexto.
- Toda promise é aguardada ou tratada. Na Vercel, a function pode
  congelar assim que a resposta sai, e o que não foi aguardado pode não
  terminar.

```ts
// Ruim
throw 'equipamento indisponível';
try { await this.emprestimos.marcarDevolvido(id); } catch (erro) { console.log(erro); }

// Bom: campo declarado à parte (não parameter property) para compilar
// também no front/
export class EquipamentoIndisponivelError extends Error {
  readonly equipamentoId: string;

  constructor(equipamentoId: string) {
    super(`O equipamento ${equipamentoId} não está disponível.`);
    this.name = 'EquipamentoIndisponivelError';
    this.equipamentoId = equipamentoId;
  }
}

try {
  await this.emprestimos.marcarDevolvido(id);
} catch (erro) {
  throw new Error(`Falha ao registrar a devolução ${id}.`, { cause: erro });
}
```

## Formatação

- `PascalCase` para classe, interface, type e componente React;
  `camelCase` para variável, função e membro; `SNAKE_CASE` maiúsculo
  para constante de módulo (`API_PREFIX`).
- Quem chama fica acima de quem é chamado.
- Imports na ordem: polyfills, builtins do Node, pacotes externos, pai
  (`../`), mesmo diretório (`./`), como em `back/src/openapi/emit.ts`.
  Sem import não usado. Import só de tipo leva `type`
  (`import { type Env } from './config/env'`); no front/, com
  `verbatimModuleSyntax`, é obrigatório.

## Comentários

- O código se explica; o comentário diz o porquê e, quando couber, cita
  o ADR, como em `back/src/config/env.ts`.
- Sem código comentado, comentário de diário ou marcador posicional.
- `// TODO:` para melhoria futura conhecida, nunca como desculpa.
