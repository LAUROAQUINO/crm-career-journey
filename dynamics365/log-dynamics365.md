# Dynamics 365 & Power Platform — Log de Prática

Documentação prática da minha jornada de aprendizado em Microsoft Dynamics 365 e Power Platform.

**Contexto:** profissional com experiência prática em GoHighLevel e Smartico (automação, integração, treinamento de equipes), em expansão de conhecimento para o ecossistema Microsoft.

---

## 📅 Log de Progresso

### 14/07/2026 — Governança de Dados: Regra de Detecção de Duplicidade

**Objetivo:** configurar prevenção de cadastros duplicados usando CPF como chave de identificação única.

**O que foi feito:**
1. Criação de campo customizado (`CPF`) na tabela Contato
2. Configuração de Business Rule
3. Criação de Duplicate Detection Rule (`DDR_CPF_Duplicado`)
   - Registro base: Contato
   - Registro correspondente: Contato
   - Critério: CPF, Correspondência Exata
4. Publicação da regra
5. Teste em ambiente real: criação de dois contatos com CPF idêntico

**Resultado:** sistema exibiu alerta "Registros duplicados encontrados" em tempo real, antes do salvamento, oferecendo opções de mesclar ou ignorar.

**Aprendizado / erro corrigido:**
Na primeira tentativa, usei "Marca do Veículo" como critério de correspondência — um erro conceitual, já que esse campo não é um identificador único (múltiplos clientes podem comprar a mesma marca). Corrigido para CPF, que garante unicidade real por registro.

**Conceitos aplicados:**
- Deduplicação por chave
- Regras de validação preventiva
- Diferença entre tabela Conta (pessoa jurídica) e Contato (pessoa física) para campos como CPF/CNPJ

---

## 🧠 Conceitos-chave documentados até aqui

| Conceito | Resumo |
|---|---|
| Business Unit (BU) | Estrutura hierárquica de contas — BU pai e BUs filhas |
| Security Role | Permissões de acesso dentro de uma BU |
| Business Rule | Regra técnica aplicada na tela, sem necessidade de código |
| Solução Não Gerenciada / Gerenciada | Ambiente editável (Dev) vs. pacote fechado read-only (Sandbox/Produção) |
| UAT | Teste de aceite do usuário de negócio antes de ir a produção |
| Duplicate Detection Rule | Prevenção automática de cadastros duplicados via campo-chave |

---

## 🎯 Próximos passos

- [ ] ALM na prática: criar Solução, exportar como gerenciada, importar em outro ambiente
- [ ] Integração real com Power Automate (Cloud Flow)
- [ ] Explorar Web API / Dataverse API
- [ ] Certificação PL-900 (Power Platform Fundamentals)
- [ ] Certificação PL-200 (Power Platform Functional Consultant)
- [ ] Certificação MB-210 (Dynamics 365 Sales Functional Consultant)
