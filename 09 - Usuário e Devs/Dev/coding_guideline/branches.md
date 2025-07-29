# Branches
## Estratégia de Branches

### Versão: 1.0
### Data: 07-2025

---

## 1. Estratégia Utilizada
- Fluxo principal: `master`, `homolog`, `develop`
- Branches de feature: `feature/nome-da-feature`
- Branches de bugfix: `bugfix/descricao-bug`
- Branches de hotfix: `hotfix/descricao-hotfix`

---

## 2. Nomenclatura
- Sempre usar nomes descritivos
- Separar palavras por hífen

---

## 3. Exemplo de Fluxo
1. Criar branch a partir de `develop`:
   - `git checkout develop && git pull`
   - `git checkout -b feature/login-social`
2. Ao finalizar, abrir PR para `develop`
3. Após testes, merge para `homolog` e depois para `master`

---

## 4. Dicas
- Evitar branches long-lived
- Atualizar branch com frequência
- Resolver conflitos antes do PR

---

## 5. Ferramentas
- GitHub Flow
- Proteção de branches no repositório

---

## 6. Referências
- [GitHub Flow](https://docs.github.com/pt/get-started/quickstart/github-flow)
