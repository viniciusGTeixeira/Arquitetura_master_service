# Padrões de Código
## Guia de Boas Práticas de Programação

### Versão: 1.0
### Data: 07-2025


## 1. Convenções Gerais
- Usar inglês para nomes de variáveis, funções e classes
- Seguir camelCase para variáveis e funções (`userName`, `getUserData`)
- PascalCase para classes e componentes (`UserService`, `LoginForm`)
- SCREAMING_SNAKE_CASE para constantes (`MAX_RETRIES`)
- Indentação de 2 espaços (JS/TS/Vue) ou 4 espaços (PHP)
- Linhas de até 120 caracteres


## 2. Organização de Código
- Um arquivo por componente/classe principal
- Separar lógica de negócio, apresentação e dados
- Utilizar pastas por domínio ou feature
- Imports ordenados: bibliotecas, módulos internos, estilos


## 3. Comentários e Documentação
- Comentar apenas o necessário (por quê, não o óbvio)
- Usar JSDoc/PHPDoc para funções, classes e métodos públicos
- TODOs e FIXMEs devem ser rastreados em issues


## 4. Exemplo de Estrutura
```typescript
// userService.ts
/**
 * Get user by ID
 * @param id string
 * @returns User
 */
export function getUserById(id: string): User {
  // ...
}
```

```php
// UserService.php
/**
 * Get user by ID
 * @param string $id
 * @return User
 */
public function getUserById(string $id): User {
    // ...
}
```


## 5. Boas Práticas
- Escrever código limpo e legível
- Evitar duplicidade de lógica
- Preferir funções puras e pequenas
- Validar entradas e tratar erros
- Testar e revisar antes de subir PR


## 6. Ferramentas de Qualidade
- ESLint, Prettier (JS/TS/Vue)
- PHP_CodeSniffer, PHPStan (PHP)
- Husky para hooks de commit


## 7. Referências
- [Guia oficial do Vue.js](https://vuejs.org/v2/style-guide/)
- [PSR-12 PHP Coding Style](https://www.php-fig.org/psr/psr-12/)
- [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/)
