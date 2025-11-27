# Pull Request Creation

Este comando finaliza o processo de desenvolvimento criando e submetendo um Pull Request para revisão.

## Configuração do Jira

As configurações do Jira estão no arquivo `ai.properties.md` na raiz do projeto:
- `jira_cloud_id`: ID do cloud Atlassian para chamadas MCP
- `jira_project_key`: Chave do projeto (ex: TID)
- `jira_project_id`: ID numérico do projeto
- `jira_base_url`: URL base do Jira

## Objetivo

Criar um Pull Request bem documentado e pronto para revisão humana, seguindo todos os padrões e processos estabelecidos.

## Processo de Criação do PR

### 1. Validações Finais

#### Execução de Testes

- Execute a suite completa de testes do projeto
- **Comando típico**: `npm test` ou conforme configuração do projeto
- **🛑 BLOQUEANTE**: Se algum teste falhar, corrija antes de prosseguir
- Confirme que novos testes foram adicionados para funcionalidade implementada

#### Verificação de Build

- Execute build do projeto para garantir que compila sem erros
- Verifique linting e formatação
- Confirme que não há warnings críticos

#### Limpeza do Código

- Remova código comentado, debug statements, TODOs temporários
- Confirme que não há referências a IA ou Cursor no código
- Valide que todos os commits são relevantes e bem descritos

### 2. Preparação dos Commits

#### Organização dos Commits

- Revise histórico de commits da branch
- Considere squash de commits relacionados se necessário
- Garanta mensagens de commit descritivas e consistentes

#### Regra de Ouro para Commits

**⚠️ IMPORTANTE**: Use `git add [arquivos específicos]` - NUNCA `git add .`

- Adicione apenas arquivos que você modificou intencionalmente
- Evite commits acidentais de arquivos de configuração local
- Revise `git status` antes de cada commit

#### Template de Mensagem de Commit

```
feat: implementa [funcionalidade específica]

- [Detalhe 1 do que foi implementado]
- [Detalhe 2 das mudanças realizadas]

Refs: [ID da issue/card]
```

### 3. Atualização do Sistema de Gestão

#### Status da Task

- Mova o card/issue para status **"In Review"** no Jira usando as configurações do `ai.properties.md`:
  ```typescript
  const jiraCloudId = 'jira_cloud_id do ai.properties.md';
  const jiraProjectKey = 'jira_project_key do ai.properties.md';

  // 1. Buscar task
  const searchResults = await mcp__atlassian__searchJiraIssuesUsingJql({
    cloudId: jiraCloudId,
    jql: `project = ${jiraProjectKey} AND summary ~ "${feature_slug}"`,
    fields: ['summary', 'status']
  });

  // 2. Obter transições disponíveis
  const transitions = await mcp__atlassian__getTransitionsForJiraIssue({
    cloudId: jiraCloudId,
    issueIdOrKey: searchResults.issues[0].key
  });

  // 3. Encontrar transição para "In Review" / "Em Revisão"
  const reviewTransition = transitions.find(
    (t) => t.name.includes('Review') || t.name.includes('Revisão')
  );

  // 4. Fazer transição
  if (reviewTransition) {
    await mcp__atlassian__transitionJiraIssue({
      cloudId: jiraCloudId,
      issueIdOrKey: searchResults.issues[0].key,
      transition: { id: reviewTransition.id }
    });
  }
  ```
- Adicione comentário com link do PR quando criado:
  ```typescript
  await mcp__atlassian__addCommentToJiraIssue({
    cloudId: jiraCloudId,
    issueIdOrKey: 'ISSUE_KEY',
    commentBody: '🔗 Pull Request criado: [PR_URL]'
  });
  ```
- Atualize estimativas se necessário

#### Documentação

- Confirme que `sessions/<feature_slug>/` está completo
- Verifique se `plan.md` está totalmente atualizado
- Adicione notas finais sobre a implementação

### 4. Criação do Pull Request

#### Informações Básicas

- **Título**: Claro e descritivo da funcionalidade
- **Base Branch**: Confirme que está mergando na branch correta
- **Labels**: Aplique labels apropriadas (feature, bug, enhancement)
- **Reviewers**: Sugira reviewers baseado na natureza da mudança

#### Template de Descrição do PR

```markdown
# [Título da Funcionalidade]

## 📋 Resumo

[Descrição breve do que foi implementado e por quê]

## 🎯 Objetivos

- [Objetivo 1]: [Status ✅]
- [Objetivo 2]: [Status ✅]

## 🔧 Principais Mudanças

### Arquivos Modificados

- `[arquivo1.ext]`: [Tipo de mudança]
- `[arquivo2.ext]`: [Funcionalidade adicionada]

### Novos Arquivos

- `[novo_arquivo.ext]`: [Propósito]

## 🧪 Testes

### Testes Adicionados

- [Teste 1]: [Cobertura]
- [Teste 2]: [Cenário testado]

### Cobertura

- **Antes**: [X%]
- **Depois**: [Y%]

## 🏗️ Arquitetura e Decisões Técnicas

### Abordagem Escolhida

[Explicação da solução implementada]

### Alternativas Consideradas

[Outras opções avaliadas e por que foram descartadas]

### Trade-offs Realizados

[Compromissos técnicos aceitos]

## 🚀 Deploy e Configuração

### Dependências

- [ ] Novas dependências adicionadas: [Lista]
- [ ] Configurações necessárias: [Variáveis de ambiente, etc.]
- [ ] Migrações de banco: [Se aplicável]

### Impactos

- **Performance**: [Impacto esperado]
- **Compatibilidade**: [Mudanças breaking se houver]
- **Infraestrutura**: [Recursos adicionais necessários]

## 🔗 Referências

- **Issue/Card**: [Link para issue original]
- **Especificação**: [Link para PRD/specs]
- **Documentação**: [Links para docs relacionadas]

## ✅ Checklist

- [x] Testes passando
- [x] Lint/formatação OK
- [x] Documentação atualizada
- [x] Performance validada
- [x] Segurança verificada
- [x] Compatibilidade testada

## 🧐 Review Guidelines

### Pontos de Atenção para Reviewers

- [Área específica que merece atenção especial]
- [Decisão arquitetural para validar]

### Testes Sugeridos

- [Cenário específico para testar manualmente]
- [Integração a verificar]
```

### 5. Monitoramento de Code Review Automatizado

#### Aguardar Feedback Automatizado

- **Primeiro check**: Aguarde 3 minutos após criação do PR
- **Segundo check**: Se não houver comentários, aguarde mais 3 minutos
- **Ferramentas típicas**: SonarQube, CodeClimate, GitHub Actions, etc.

#### Processamento de Comentários Automatizados

Para cada comentário recebido:

1. **Categorizar Severidade**:

   - 🚨 **Crítico**: Bugs, segurança, breaking changes
   - ⚠️ **Importante**: Performance, manutenibilidade
   - 💡 **Sugestão**: Melhorias de estilo, otimizações

2. **Análise e Resposta**:

   - **Críticos**: Corrija imediatamente
   - **Importantes**: Avalie necessidade e implemente
   - **Sugestões**: Considere valor vs esforço

3. **Apresentação ao Usuário**:

```markdown
## 🤖 Análise de Code Review Automatizado

### 🚨 Problemas Críticos (Total: X)

- **[Ferramenta]**: [Descrição do problema]
  - **Localização**: [Arquivo:linha]
  - **Recomendação**: [Ação proposta]

### ⚠️ Melhorias Importantes (Total: Y)

- **[Ferramenta]**: [Sugestão]
  - **Impacto**: [Justificativa para correção]

### 💡 Sugestões Opcionais (Total: Z)

- **[Ferramenta]**: [Melhoria sugerida]
  - **Avaliação**: [Recomendação de implementar ou não]

**Posso implementar as correções sugeridas?**
```

### 6. Implementação de Correções

#### Para Problemas Identificados

- Implemente correções necessárias
- Faça commit das mudanças com mensagem clara:

  ```
  fix: corrige problemas identificados no code review

  - [Correção 1]
  - [Correção 2]

  Co-authored-by: [Ferramenta de Code Review]
  ```

#### Push das Atualizações

- Faça push das correções para a mesma branch
- Monitore se ferramentas automatizadas validam as correções
- Confirme que testes ainda passam após mudanças

### 7. Notificação de Conclusão

Após completar todo o processo:

## Template de Task Completion

```markdown
# ✅ Task Completion Message

## 🎯 Tarefa Completada

### ✅ Validações Finais

- [x] Testes passando completamente
- [x] Build sem erros ou warnings
- [x] Code review automatizado processado
- [x] Correções implementadas e validadas

### 📋 Gestão de Projeto

- **Card/Issue**: [ID] movido para "In Review"
- **Status**: Pronto para revisão humana

### 🔗 Pull Request

- **Título**: [Título do PR]
- **URL**: [Link para o PR]
- **Reviewers**: [Lista de reviewers sugeridos]

## 🚀 Próximos Passos

O PR está pronto para:

1. Revisão técnica pelos peers
2. Aprovação final
3. Merge para branch principal

## 📊 Resumo da Implementação

- **Arquivos modificados**: [Número]
- **Linhas adicionadas**: [Número aproximado]
- **Cobertura de testes**: [Porcentagem final]
- **Tempo de desenvolvimento**: [Estimativa]
```

### 8. Handoff para Revisão Humana

#### Documentação Final

- Confirme que toda documentação está atualizada
- Valide que contexto está completo para reviewers
- Prepare para possíveis perguntas durante review

#### Disponibilidade para Esclarecimentos

- Mantenha-se disponível para responder perguntas
- Prepare-se para fazer ajustes baseados em feedback humano
- Documente decisões tomadas durante review se necessário

## Ferramentas de Integração

### Sistemas de Gestão de Tarefas

- **Task Manager**: Transição automática de status
- **GitHub Issues**: Linking automático com PR
- **Linear**: Sincronização de status
- **Local**: Atualização em `tasks/`

### Ferramentas de Code Review

- **GitHub**: Actions e bots automáticos
- **GitLab**: Pipelines de CI/CD
- **Bitbucket**: Pipelines e code insights
- **SonarQube**: Análise de qualidade

## Princípios do PR

1. **Transparência**: Documente decisões e trade-offs claramente
2. **Qualidade**: Nunca submeta código que falha em testes
3. **Responsabilidade**: Implemente feedback crítico obrigatoriamente
4. **Colaboração**: Facilite o trabalho dos reviewers
5. **Completude**: Inclua toda informação necessária para review

## Processo de Follow-up

Após submissão do PR:

1. **Monitor**: Acompanhe comentários e feedback
2. **Respond**: Responda perguntas rapidamente
3. **Iterate**: Implemente mudanças solicitadas
4. **Validate**: Confirme que mudanças atendem expectativas
5. **Celebrate**: Reconheça conclusão quando PR for aprovado e merged
