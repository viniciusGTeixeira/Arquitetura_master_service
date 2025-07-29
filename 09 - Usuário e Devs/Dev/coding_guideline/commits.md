# Commits
## Guia de Mensagens de Commit

### Versão: 1.0
### Data: 07-2025

---

## 1. Padrão Utilizado
- Seguir o padrão [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/)
- Mensagens em português, sempre padronizadas

---

## 2. Estrutura da Mensagem
```
tipo(escopo): descrição breve

[corpo opcional]

[footer opcional]
```
- **tipo**: feat, fix, docs, style, refactor, test, chore, perf
- **escopo**: módulo, feature ou arquivo afetado
- **descrição**: clara e objetiva, até 72 caracteres

---

## 3. Exemplos
```
feat(auth): adicionar autenticação 2FA
fix(user): corrigir bug no cadastro de usuário
docs(readme): atualizar instruções de setup
style(login): ajustar espaçamento do botão
```

---

## 4. Dicas
- Commits pequenos e frequentes
- Descrever o "porquê" da mudança, se relevante
- Referenciar issues quando aplicável (ex: closes #123)
- Evitar mensagens genéricas como "update" ou "fixes"

---

## 5. Ferramentas
- Commitlint para validação automática

---

## 6. Referências
- [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/)
