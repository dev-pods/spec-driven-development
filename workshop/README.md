# Hands-on: Spec-Driven Development com GitHub Copilot

> Preparação do ambiente e criação do repositório: [README.md](../README.md).

## Objetivo do exercício

Neste hands-on, você vai usar artefatos de especificação como contexto direto para o GitHub Copilot. O foco não é produzir muito código, e sim mostrar como SDD melhora clareza, consistência e validação ao longo do desenvolvimento.

Ao final do exercício, você terá:

- refinado requisitos com apoio da IA;
- organizado a implementação em tasks pequenas e rastreáveis;
- criado testes completos antes da implementação;
- implementado uma API simples em FastAPI guiada por spec e testes;
- validado que todos os testes passam e o código segue o design.

## Ambiente

No terminal do Codespace, valide as ferramentas necessárias:

```bash
python3 --version
specify --help
```

Em seguida, inicialize o Spec Kit e a integração com o Copilot:

```bash
specify init . --script sh
specify integration install copilot
```

Esses comandos habilitam os comandos `/speckit.*` no Copilot Chat.
Durante essa etapa, é esperado que sejam criados artefatos como `.specify/`
(local), a pasta `.github/` (com prompts e agentes do Speckit, além do copilot-instructions.md).

Se quiser contextualizar melhor o agente para SDD, use o bloco abaixo.

<details>
<summary>Arquivo de instruções (opcional, recomendado):</summary><br/>

Inicialmente, o arquivo `.github/copilot-instructions.md` é criado com um placeholder básico. 
Opcionalmente, podemos atualizá-lo para refletir o contexto de SDD.

Sugestão de prompt:

```text
#codebase Atualize o arquivo #file:.github/copilot-instructions.md para refletir o uso de Spec-Driven Development. Mantenha simples, direto e acionável.
```
</details>

## Fonte de verdade do exercício

Até o Passo 1, os arquivos em [.specs](../.specs/) são a base do hands-on:

- [../.specs/spec.yaml](../.specs/spec.yaml): escopo e proposta do sistema.
- [../.specs/requirements.md](../.specs/requirements.md): requisitos funcionais e critérios de aceite.
- [../.specs/design.md](../.specs/design.md): design técnico da solução.
- [../.specs/tasks.md](../.specs/tasks.md): ordem de execução da implementação.

Após concluir o Passo 1 (`/speckit.clarify`), a **fonte de verdade** principal
passa a ser `specs/<feature-id>/spec.md`.

**Convenção de precedência (SDD):**

- [../.specs/](../.specs/): baseline de produto e contexto arquitetural.
- `specs/<feature-id>/`: execução da feature ativa (quando essa pasta existir).
- Em caso de conflito durante a implementação da feature, prevalecem os artefatos em `specs/<feature-id>/`.


## Fluxo do hands-on

### 1. Refinar requisitos

Revise [../.specs/requirements.md](../.specs/requirements.md) com base em [../.specs/spec.yaml](../.specs/spec.yaml) para identificar critérios de aceite faltantes, casos de borda e respostas de erro.

Sugestão de prompt:

```text
/speckit.clarify Identifique critérios de aceite faltando em #.specs/requirements.md e casos de borda para endpoints de tarefas. 
Se necessário crie a branch que atenda ao padrão de branch do Speckit.
```

> **Após executar:** Revise as perguntas geradas e responda com atenção — as respostas alimentam o `spec.md` da feature. Reserve um momento para validar se os critérios de aceite resultantes cobrem também os cenários de erro e os casos de borda que você já conhece do domínio.

---

### 2. Refinar tasks

Crie ou refine as tasks com base em `specs/<feature-id>/spec.md` e [../.specs/design.md](../.specs/design.md).
Para rodar `/speckit.tasks`, o `plan.md` da feature é obrigatório. Se ainda não existir, gere antes:

```text
/speckit.plan Crie um plano de implementação mínimo para #specs/<feature-id>/spec.md, usando FastAPI com armazenamento em memória.
```

Se houver falha no `/speckit.tasks` por ausência de `plan.md`, rode primeiro o comando acima e repita o `/speckit.tasks`.

> **Pré-check rápido — antes de continuar, confirme:**
> - existe `specs/<feature-id>/spec.md`;
> - existe `specs/<feature-id>/plan.md` (ou rode `/speckit.plan` para gerar);
> - `specs/<feature-id>/tasks.md` está atualizado com dependências e ordem.

Sugestão de prompt:

```text
/speckit.tasks Crie ou refine #specs/<feature-id>/tasks.md com base em #specs/<feature-id>/spec.md e #.specs/design.md. Quebre em tasks pequenas, com critério de pronto testável, dependências explícitas e ordem de implementação.
```

> **Após executar:** Leia `specs/<feature-id>/tasks.md` e valide se as tasks têm granularidade adequada (cada uma deve ser implementável de forma isolada), se as dependências fazem sentido e se toda task tem critério de pronto claro. Ajuste se necessário antes de avançar.

---

### 3. Criar testes antes da implementação (RED)

Neste passo do SDD, você vai **escrever os testes primeiro**, guiado pelos requisitos.

Crie testes em [../app/test_main.py](../app/test_main.py) cobrindo todos os cenários definidos em `specs/<feature-id>/spec.md`.

Sugestão de prompt:

```text
/speckit.implement Crie testes em #app/test_main.py cobrindo TODOS os critérios de aceite em #specs/<feature-id>/spec.md. Use pytest e inclua casos de sucesso, erro e borda.
```

Confirme que os testes falham (sem `app/main.py`, é o estado esperado):

```bash
python -m pytest app/test_main.py -v
```

> **Após executar:** Verifique se todos os testes estão em estado FAILED (não ERROR). Erros de coleta ou import indicam problema na estrutura do arquivo de testes — corrija antes de avançar. O objetivo aqui é garantir que os testes estão escritos corretamente, só não têm implementação ainda.

---

### 4. Implementar (GREEN)

Implemente as tasks usando `specs/<feature-id>/tasks.md`, `specs/<feature-id>/spec.md` e `../.specs/design.md` como fonte de verdade. O arquivo-alvo é `app/main.py`.

Como primeiro movimento do passo 4, crie um esqueleto mínimo de [../app/main.py](../app/main.py) para eliminar erro de import/coleta e, em seguida, repita o comando de teste do passo 3 durante a implementação.

Objetivo: passar de RED (testes falhando) para GREEN (testes passando).

**Sugestão de prompt de inicialização** (usar uma vez no início do passo 4):

```text
/speckit.implement Crie um esqueleto mínimo de #app/main.py para remover erro de coleta dos testes e iniciar a implementação incremental de #specs/<feature-id>/tasks.md.
```

Após executar o prompt de inicialização, rode os testes para confirmar que o erro de coleta/import foi resolvido:

```bash
python -m pytest app/test_main.py -v
```

**Sugestão de prompt de implementação** (usar em seguida, repetidamente, para cada task):

```text
/speckit.implement Implemente a próxima task pendente em #specs/<feature-id>/tasks.md usando #specs/<feature-id>/spec.md e #.specs/design.md como fonte de verdade. Os testes em #app/test_main.py devem passar ao final.
```

> **Após cada ciclo:** Rode os testes e confirme que os testes da task implementada passaram para GREEN. Repita o prompt para a próxima task. Idealmente, avance para a task seguinte com os testes da task atual em GREEN.

Se quiser validar visualmente os endpoints durante a implementação, use o Swagger abaixo.

<details>
<summary>Validação rápida via Swagger (opcional, recomendado):</summary><br/>

Suba a API localmente:

```bash
python -m uvicorn app.main:app --reload
```

Com a API em execução, valide a documentação e o contrato exposto:

- Swagger UI: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc
- OpenAPI JSON: http://127.0.0.1:8000/openapi.json

> **Após executar:** Confirme se os endpoints esperados aparecem no Swagger e se os schemas de request/response estão coerentes com `specs/<feature-id>/spec.md`.
</details>

---

### 5. Validar aderência à spec

Nesta etapa, você vai validar se implementação, spec, design e tasks continuam alinhados.

Sugestão de prompt:

```text
#codebase A implementação em #app/main.py corresponde ao design em #.specs/design.md e aos critérios em #specs/<feature-id>/spec.md? Liste divergências objetivas em formato de tabela.
```

Você também pode pedir uma revisão contra as tasks:

Sugestão de prompt:

```text
#codebase Compare #specs/<feature-id>/tasks.md com #app/main.py e liste quais tasks ainda estão pendentes.
```

> **Após executar:** Analise a tabela de divergências. Pequenas diferenças de nomenclatura são aceitáveis; divergências de comportamento ou endpoints ausentes devem ser corrigidas antes de considerar a feature concluída.

---

## Checkpoints sugeridos

- Os requisitos estão claros o suficiente para mais de uma pessoa implementar a mesma solução.
- As tasks seguem uma ordem que reduz retrabalho.
- Os testes cobrem 100% dos critérios de aceite em `specs/<feature-id>/spec.md`.
- Todos os testes falham inicialmente (RED), indicando que a spec foi traduzida corretamente para código de teste.
- O código em [../app/main.py](../app/main.py) reflete o design, não decisões improvisadas.
- Todos os testes passam ao final da implementação (GREEN).
- A validação final mostra aderência entre requisitos, design, tasks e implementação.

## Definition of Done (DoD)

- `python -m pytest app/test_main.py -v` executa com sucesso (GREEN).
- `app/main.py` está aderente a `specs/<feature-id>/spec.md` e [../.specs/design.md](../.specs/design.md).
- Não há tasks pendentes em `specs/<feature-id>/tasks.md`.

## Material de apoio

- [../README.md](../README.md)
