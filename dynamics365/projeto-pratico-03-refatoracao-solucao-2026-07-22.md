# Projeto Prático 03 – Refatoração de Solução e Correção do Solution Checker – 22/07/2026

**Ambiente:** NEX Vendas (Dataverse) | **Solução:** Solucao_CRM_Concessionaria | **Versão:** 1.0.0.1 (não gerenciada)

---

## Status

✅ Concluído — Solution Checker zerado (0 problemas em 22/07/2026, 18:29:18)

---

## Objetivo

Corrigir os 2 achados de severidade Média (`meta-avoid-managed-entity-assets`) identificados pelo Solution Checker na sessão de 15/07/2026, entendendo a causa raiz em vez de apenas aplicar a correção.

---

## Escopo

### 1. Diagnóstico — dissecação do pacote exportado

Antes de tocar na ferramenta, o `.zip` da solução gerenciada v1.0.0.1 foi descompactado e inspecionado.

**Estrutura interna do pacote:**

```
Solucao_CRM_Concessionaria_1_0_0_1_managed.zip   (312 KB)
├── solution.xml          112 KB  → manifesto: o que a solução declara possuir
├── customizations.xml    130 KB  → definição técnica de cada componente
├── [Content_Types].xml   258 B   → metadado de empacotamento
├── Workflows/                    → 2 Business Rules (.xaml)
└── duplicaterules/               → 1 Duplicate Detection Rule
```

**Aprendizado estrutural:** uma Business Rule não é um objeto próprio no Dynamics — é armazenada como *workflow* em formato XAML, o mesmo dos fluxos clássicos. A interface visual de arrastar blocos compila para esse XML.

**Causa raiz localizada no `RootComponents` do `solution.xml`:**

```xml
<RootComponent type="1"  schemaName="account"  behavior="0" />
<RootComponent type="1"  schemaName="contact"  behavior="0" />
<RootComponent type="60" id="{15915835-...}" />
<RootComponent type="60" id="{70ad8b15-...}" />
```

| Atributo | Significado |
|---|---|
| `type="1"` | Entidade/tabela |
| `type="60"` | Formulário (systemform) |
| `behavior="0"` | **Incluir todos os subcomponentes** ← causa do achado |

Valores possíveis de `behavior`: `0` = incluir tudo, `1` = só metadados, `2` = não incluir subcomponentes.

**Impacto medido:**

| Componente | Total no pacote | Efetivamente meus |
|---|---|---|
| Formulários | 8 | 2 |
| Atributos | 4 | 2 (`cr2a0_CPF`, `cr2a0_MarcadoVeiculo`) |
| Business Rules | 2 | 2 |
| Duplicate Rule | 1 | 1 |

Entre os formulários indevidos: `Portal Contact (Enhanced)`, `Profile Web Form (Enhanced)`, `Invite Web Form` e `Profile Web Form (Enhanced) – Japanese`. Nenhum criado ou alterado por mim. Também entrou `msdyn_segmentid`, campo do módulo de Marketing da Microsoft.

Contador da própria ferramenta: **19 de 950 componentes** para uma solução que deveria ter ~4.

### 2. Higienização — nomenclatura das Business Rules

Ambas as regras estavam com o nome padrão "Nova regra de negócios", na mesma tabela, impossíveis de distinguir.

Convenção adotada: `BR_<Tabela>_<Função>`

| Antes | Depois | Status |
|---|---|---|
| Nova regra de negócios | `BR_Conta_AvisoAprovacaoGerencia` | Ativado |
| Nova regra de negócios | `BR_Conta_VisibilidadeReceita_PorMarca` | Desativado (rascunho) |

**Detalhe operacional:** regra ativa fica em modo Somente Leitura. Para renomear foi preciso Desativar → renomear → Salvar → Ativar, restaurando o estado original.

**Lógica de cada regra:**

```
BR_Conta_AvisoAprovacaoGerencia (ativa)
SE    Telefone Principal contém dados
ENTÃO Mostrar mensagem "Este desconto exige aprovação da gerência"

BR_Conta_VisibilidadeReceita_PorMarca (rascunho)
SE    Marca do Veículo igual a "Ford"
ENTÃO Mostrar campo Receita em Aberto
SENÃO Ocultar campo Receita em Aberto
```

### 3. Correção — remoção e readição com seleção de componentes

| Etapa | Ação | Componentes |
|---|---|---|
| Inicial | — | 19 |
| Remover tabela Conta da solução | Remove from solution | 11 |
| Remover tabela Contato | Remove from solution | 2 |
| Readicionar Contato → Editar objetos → aba Colunas → só `cr2a0_CPF` | Select components | — |
| Readicionar Conta → Limpar todos os objetos → só as 2 Business Rules | Select components | — |
| Adicionar `cr2a0_MarcadoVeiculo` (dependência) | Select components | — |

**Armadilha encontrada:** ao adicionar a tabela, o Dynamics **pré-seleciona** subcomponentes automaticamente (no caso, "6 formulários selecionado(s)"). É obrigatório clicar em **"Limpar todos os objetos"** antes de "Editar objetos", senão o achado é reintroduzido.

**Distinção crítica:** *Remover da solução* ≠ *Excluir do ambiente*. A primeira tira o objeto do pacote; a segunda apaga a tabela do ambiente inteiro.

### 4. Achado secundário — troca de excesso por falta

Após a correção, o Checker apontou um achado **novo**:

```
meta-include-missingunmanageddependencies
Gravidade: Média | Categoria: Design | Tipo: Configuração
```

**Diagnóstico:** as Business Rules entraram na solução, mas o campo customizado `cr2a0_MarcadoVeiculo`, que a regra `BR_Conta_VisibilidadeReceita_PorMarca` referencia, não. Se a solução fosse importada num ambiente sem esse campo, a regra quebraria.

**Correção:** adicionar o campo à solução via Select Components.

**Este é o eixo do ALM:** a solução foi de *excesso* (componentes da Microsoft de carona) para *falta* (dependência ausente) até chegar ao ponto de ter tudo que precisa e nada além.

---

## Resultado

| Verificação | Data | Problemas |
|---|---|---|
| Antes | 15/07/2026 14:30 | 2 (Média) — `meta-avoid-managed-entity-assets` |
| Intermediária | 22/07/2026 17:59 | 1 (Média) — `meta-include-missingunmanageddependencies` |
| **Final** | **22/07/2026 18:29** | **0** ✅ |

---

## Próxima ação

1. Exportar como Gerenciada v1.0.0.2
2. Importar no ambiente `NEX SOlution (default)` — teste de ciclo ALM completo Dev → QA, que ficou pendente desde 15/07 (na ocasião a reimportação no mesmo ambiente foi corretamente bloqueada pelo sistema)

---

## Aprendizados

**Sobre solução e componentes**

- Uma solução deve conter **apenas o que foi criado ou alterado** — nunca o objeto pai inteiro quando só se mexeu em partes dele.
- `behavior="0"` no `RootComponent` é o que traduz "incluir todos os objetos" na interface. Saber ler isso no XML permite auditar qualquer solução recebida de terceiros.
- Tabelas nativas aparecem como `Gerenciado: Sim / Personalizado: Não`. Componentes próprios aparecem como `Gerenciado: Não / Personalizado: Sim`. A própria grade da ferramenta mostra o achado antes do Checker rodar.
- Business Rule é atada à tabela onde foi criada — não é possível movê-la para outra tabela, só recriar.
- O seletor de subcomponentes só lista o que ainda não pertence à solução, o que gera estados intermediários confusos após remoções. Refazer o caminho limpo resolve.

**Sobre o Solution Checker**

- Categoria **Design** significa risco estrutural futuro, não bug atual. Por isso severidade Média e não Crítica: nada está quebrado hoje, mas a construção causa problema no dia da atualização do fabricante.
- Corrigir um achado pode revelar outro. Isso não é retrocesso — é o processo funcionando.

**Sobre método de estudo**

- "Incluir tudo" é o caminho que funciona no primeiro dia e cobra a conta no terceiro mês. Muitos tutoriais ensinam assim porque garante que nada quebre no exercício inicial. Vale sempre perguntar, sobre qualquer orientação recebida: *isso é o jeito certo ou o jeito rápido?*
- Errar, diagnosticar e corrigir gera entendimento do **porquê** da regra. Acertar de primeira gera apenas conhecimento da regra.

---

## Dívida técnica registrada

| Item | Descrição | Prioridade |
|---|---|---|
| Coerência da `BR_Conta_AvisoAprovacaoGerencia` | Condição olha *Telefone Principal* mas a mensagem fala de *desconto*. Lógica incoerente — em ambiente real seria bug reportado no primeiro dia. | Alta |
| Publisher padrão | Solução usa `DefaultPublisherorg63d67040` com prefixo `new`. Boa prática: criar Publisher próprio com prefixo identificável, para rastrear autoria quando múltiplos fornecedores atuam no mesmo ambiente. | Média |
| Arquitetura dividida | Campo CPF e Duplicate Rule em **Contato**; Business Rules em **Conta**. Sem ligação funcional clara entre as duas partes. | Baixa |
| `BR_Conta_VisibilidadeReceita_PorMarca` em rascunho | Regra bem construída (com ramo SENÃO), mas nunca ativada. | Baixa |

---

## Critério de conclusão

- [x] Causa raiz identificada no XML do pacote, não apenas na interface
- [x] Business Rules renomeadas com convenção rastreável
- [x] Tabelas readicionadas via Select Components
- [x] Dependência ausente resolvida
- [x] Solution Checker com 0 problemas
- [ ] Exportação v1.0.0.2
- [ ] Teste de importação em segundo ambiente

---

## Próxima revisão

Sessão seguinte: exportação 1.0.0.2 + importação no `NEX SOlution (default)`.
