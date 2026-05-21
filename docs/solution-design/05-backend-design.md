# 05 — Design do Backend

> **Camada:** Core Design | **Depende de:** 03-architecture, 04-database-design

## Estrutura de Pacotes

```
br.gov.sifu
│
├── autenticacao/
│   ├── AutenticacaoController.java
│   ├── AutenticacaoService.java
│   ├── JwtService.java
│   ├── JwtFiltro.java
│   └── dto/
│       ├── RequisicaoLogin.java
│       ├── RespostaLogin.java
│       └── RequisicaoRedefinicaoSenha.java
│
├── unidadegestora/
│   ├── UnidadeGestoraController.java
│   ├── UnidadeGestoraService.java
│   ├── UnidadeGestoraRepository.java
│   ├── UnidadeGestora.java              ← entidade JPA
│   └── dto/
│
├── classificacao/
│   ├── acao/
│   │   ├── AcaoOrcamentariaController.java
│   │   ├── AcaoOrcamentariaService.java
│   │   ├── AcaoOrcamentariaRepository.java
│   │   └── AcaoOrcamentaria.java
│   ├── planointerno/
│   ├── naturezadespesa/
│   ├── fonterecurso/
│   └── ptres/
│
├── dotacao/
│   ├── DotacaoOrcamentariaController.java
│   ├── DotacaoOrcamentariaService.java
│   ├── DotacaoOrcamentariaRepository.java
│   ├── DotacaoOrcamentaria.java
│   └── dto/
│
├── notacredito/
│   ├── NotaCreditoController.java
│   ├── NotaCreditoService.java
│   ├── NotaCreditoRepository.java
│   ├── NotaCredito.java
│   └── dto/
│
├── fornecedor/
│
├── notaempenho/
│   ├── NotaEmpenhoController.java
│   ├── NotaEmpenhoService.java
│   ├── NotaEmpenhoRepository.java
│   ├── NotaEmpenho.java
│   └── dto/
│
├── liquidacao/
│   ├── LiquidacaoEmpenhoController.java
│   ├── LiquidacaoEmpenhoService.java
│   ├── LiquidacaoEmpenhoRepository.java
│   ├── LiquidacaoEmpenho.java
│   └── dto/
│
├── ordembancaria/
│
├── consulta/
│   ├── ConsultaController.java          ← execução orçamentária e dashboard
│   └── ConsultaService.java
│
├── usuario/
│   ├── UsuarioController.java
│   ├── UsuarioService.java
│   └── Usuario.java
│
├── token/
│   ├── TokenIntegracaoController.java
│   ├── TokenIntegracaoService.java
│   └── TokenIntegracao.java
│
├── auditoria/
│   ├── AuditoriaController.java
│   ├── AuditoriaService.java
│   ├── AuditoriaInterceptor.java        ← AOP aspect
│   └── Auditoria.java
│
└── comum/
    ├── excecao/
    │   ├── TratadorGlobalExcecoes.java  ← @ControllerAdvice
    │   ├── SaldoInsuficienteException.java
    │   ├── DocumentoNaoEncontradoException.java
    │   ├── TransicaoEstadoInvalidaException.java
    │   └── ...
    ├── paginacao/
    │   └── PaginacaoUtils.java
    ├── seguranca/
    │   ├── ConfiguracaoSeguranca.java
    │   └── ContextoSeguranca.java       ← acesso ao usuário logado
    └── sequencia/
        └── GeradorNumeracaoDocumento.java
```

## Camadas e Responsabilidades

### Controller
- Deserializa e valida a requisição HTTP (via Bean Validation `@Valid`)
- Chama o Service
- Serializa a resposta como DTO
- Não contém lógica de negócio

### Service
- Toda a lógica de negócio e validações de domínio
- Coordena operações entre repositórios
- Lança exceções de domínio (`SaldoInsuficienteException`, etc.)
- Anotado com `@Transactional` nas operações que alteram estado

### Repository
- Interfaces `JpaRepository` + queries JPQL/nativas para consultas complexas
- Named queries para consultas de execução orçamentária

### Entidade JPA
- Mapeamento direto com a tabela
- Validações de banco via `@Column(nullable = false)` etc.
- Sem lógica de negócio

## Tratamento de Erros

Todas as respostas de erro seguem o formato RFC 7807 (Problem Details):

```json
{
  "tipo": "https://sifu.gov.br/erros/saldo-insuficiente",
  "titulo": "Saldo insuficiente",
  "status": 422,
  "detalhe": "Saldo disponível da dotação 42 é R$ 1.000,00; valor solicitado: R$ 1.500,00",
  "instancia": "/api/v1/notas-empenho"
}
```

| Exceção | HTTP Status |
|---|---|
| Entidade não encontrada | 404 |
| Saldo insuficiente | 422 |
| Transição de estado inválida | 422 |
| Violação de regra de negócio | 422 |
| Validação de campo | 400 |
| Não autenticado | 401 |
| Token expirado/revogado | 401 |
| Conta bloqueada | 423 |

## Auditoria via AOP

O `AuditoriaInterceptor` usa Spring AOP para interceptar chamadas de service marcadas com `@Auditavel`:

```java
// Exemplo de uso
@Auditavel(operacao = "EMITIR_NE", entidade = "NotaEmpenho")
public NotaEmpenho emitir(RequisicaoNotaEmpenho requisicao) { ... }
```

O interceptor captura automaticamente:
- Usuário logado (via `ContextoSeguranca`)
- IP da requisição (via `HttpServletRequest`)
- Estado antes e depois (serializado em JSON)
- Timestamp

## Configuração Spring Security

```
Endpoints públicos:    POST /api/v1/auth/login
                       POST /api/v1/auth/recuperar-senha
                       GET  /swagger-ui/**
                       GET  /api-docs/**
                       GET  /actuator/health

Endpoints autenticados: todos os demais /api/v1/**
```

Dois tipos de token aceitos pelo `JwtFiltro`:
- **JWT de sessão**: emitido pelo login, expira em 8h, armazena `login` e `tipo=SESSAO`
- **Token de integração**: emitido via `/api/v1/tokens`, expira conforme configurado, armazena `login` e `tipo=INTEGRACAO`

## Paginação

Todas as listagens aceitam os parâmetros:

| Parâmetro | Padrão | Máximo |
|---|---|---|
| `pagina` | 0 | — |
| `tamanho` | 20 | 100 |
| `ordenarPor` | depende do endpoint | — |
| `direcao` | `ASC` | `ASC` ou `DESC` |

Resposta padrão de listagem:

```json
{
  "conteudo": [...],
  "pagina": 0,
  "tamanho": 20,
  "totalElementos": 143,
  "totalPaginas": 8,
  "ultima": false
}
```
