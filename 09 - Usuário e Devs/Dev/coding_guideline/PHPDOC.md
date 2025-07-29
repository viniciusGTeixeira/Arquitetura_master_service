# PHPDOC
## Guia de Documentação PHPDoc

### Versão: 1.0
### Data: 07-2025

---

## 1. O que é PHPDoc?
- Padrão de documentação para código PHP
- Facilita geração automática de docs e análise de tipos

---

## 2. Sintaxe Básica
```php
/**
 * Descrição da função
 * @param string $param Descrição do parâmetro
 * @return int Descrição do retorno
 */
function exemplo($param) {
    // ...
}
```

---

## 3. Tags Principais
- `@param`: tipo e descrição dos parâmetros
- `@return`: tipo e descrição do retorno
- `@var`: tipo de variável
- `@throws`: exceções lançadas
- `@deprecated`: marca código obsoleto

---

## 4. Exemplo Completo
```php
/**
 * Soma dois números
 * @param int $a Primeiro número
 * @param int $b Segundo número
 * @return int Resultado da soma
 */
function soma($a, $b) {
    return $a + $b;
}
```

---

## 5. Ferramentas
- [phpDocumentor](https://www.phpdoc.org/)
- Integração com IDEs (PHPStorm, VSCode)

---

## 6. Referências
- [PHPDoc Manual](https://docs.phpdoc.org/latest/index.html)
