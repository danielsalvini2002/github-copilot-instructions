# github-copilot-instructions

Meus instructions do GitHub Copilot para Melhorar e padronizar as respostas com base nas tecnologias utilizadas.

## O que estas instruções fazem

Estas instruções fornecem contexto detalhado para o modelo de IA sobre as tecnologias e arquitetura específicas do projeto (como Next.js 16, Chakra UI v3, Prisma v7 e TypeScript rigoroso). O objetivo é garantir que toda sugestão de código e refatoração siga os padrões de Clean Code, SOLID, e as regras de governança de componentes, evitando divergências arquiteturais e agilizando o desenvolvimento.

> ⚠️ **Aviso Importante:** Estes arquivos de instrução foram criados e otimizados para uma stack muito específica (Next.js 16, Chakra UI v3, Prisma v7, BetterAuth, etc.). Caso você vá trabalhar com um conjunto diferente de tecnologias, é estritamente necessário modificar as instruções, alterando principalmente o arquivo central `.github/copilot-instructions.md` para que o Copilot não sugira padrões incorretos para o seu projeto.

## Como aplicar as instruções

1. **Localização Padrão:** Mantenha o arquivo `copilot-instructions.md` e a pasta `instructions/` dentro do diretório `.github/` na raiz do seu workspace (ex: `.github/copilot-instructions.md`).
2. **Integração no VS Code:** O GitHub Copilot Chat (e a geração de código) reconhece nativamente os arquivos de instrução posicionados na pasta `.github`.
3. **Configuração Explícita (Opcional):** Nas configurações do usuário ou do workspace (`.vscode/settings.json`), você pode adicionar o caminho do arquivo para garantir sua aplicação:

   ```json
   {
     "github.copilot.chat.codeGeneration.instructions": [
       {
         "file": ".github/copilot-instructions.md"
       }
     ]
   }
   ```

4. **Uso no Chat:** Ao solicitar a criação de código via Copilot Chat, a extensão processará automaticamente essas regras. Você também pode anexá-las ativamente como contexto no chat para forçar a validação das regras.

## Como criar as suas próprias instruções

Criar boas instruções é fundamental para extrair o melhor do Copilot. Aqui estão algumas dicas:

* **Seja Específico:** Declare as versões exatas das bibliotecas (ex: Prisma v7, Next.js 16). O Copilot pode sugerir códigos usando APIs defasadas se isso não estiver explícito.
* **Forneça Contexto Arquitetural:** Explique onde os componentes devem ser colocados e se há regras de server/client borders.
* **Defina Limites:** Diga explicitamente o que *não* fazer (ex: "não use `any` no TypeScript", "não exporte dados sensíveis em Server Actions").
* **Divida e Conquiste:** Para projetos complexos, divida as instruções em múltiplos arquivos menores e mais focados (ex: `prisma.instructions.md`, `react.instructions.md`) e os referencie num arquivo principal.

### Modelo de Instrução (Template)

Aqui está um modelo básico que você pode usar para começar novos arquivos de instruções:

```markdown
# [Nome da Tecnologia/Contexto] Instructions

Você é um engenheiro de software sênior escrevendo código focado em [Tecnologia].
Siga estritamente as diretrizes abaixo:

## 1. Bibliotecas e Versões
- Framework principal: [ex: Next.js 14]
- Estilização: [ex: Tailwind CSS]
- Gerenciamento de estado: [ex: Zustand]

## 2. Padrões de Código (Code Style)
- Utilize [ex: TypeScript estrito]. Nunca utilize `any`.
- Prefira [ex: funções curtas e princípios SOLID].
- [Outra regra específica, ex: Nomeie os arquivos usando kebab-case].

## 3. Arquitetura e Estrutura de Pastas
- Coloque componentes reutilizáveis em `src/components/ui`.
- Lógica de banco de dados deve ficar em `src/lib/db`.

## 4. O que NÃO Fazer (Anti-patterns)
- Evite [ex: criar componentes gigantes no arquivo page.tsx].
- Não [ex: faça chamadas de API diretamente do lado do cliente sem usar React Query].
```
