# Relatório Técnico de Engenharia de Software – Projeto Conecta Saúde

## Síntese da Análise Sociotécnica e Lições Aprendidas

O relatório destaca que o **Conecta Saúde** deve ser tratado como um **sistema sociotécnico**, onde software, pessoas, processos e hardware interagem. As principais falhas identificadas no protótipo atual são:

- **Acessibilidade**: o botão de acessibilidade é inacessível (fonte pequena, rodapé).
- **Feedback insuficiente**: falta de confirmação pós-agendamento e uso de *Lorem Ipsum* (que não representa a carga cognitiva real).
- **Erros latentes**: ausência de barreiras contra duplicidade de agendamentos e falta de tratamento para cenários de offline.

### Recomendações práticas imediatas

| O que fazer | Exemplo concreto |
|-------------|------------------|
| **Editar** | Melhorar a clareza da agenda: substituir “SEG, TER” por dias completos ou com dicas visuais, e adicionar navegação mensal. |
| **Adicionar** | Tela de confirmação de agendamento (“Deseja confirmar?”); tela de erro para horário indisponível ou falha de conexão (modo offline). |
| **Remover** | Reduzir telas intermediárias desnecessárias (ex: várias boas-vindas) para simplificar o fluxo do idoso. |

Essas alterações endereçam diretamente as **condições latentes para erro** mencionadas no Modelo do Queijo Suíço.

---

## Requisitos Funcionais e Não Funcionais – Como Implementá-los

### RF01 – Marcação de consultas
- **Java**: criar classe `Agendamento` com métodos `verificarDisponibilidade()` e `salvar()`.
- **Banco**: tabela `consultas` com campos `id_paciente`, `id_medico`, `data_hora`, `status`.

### RF02 – Triagem administrativa
- **Java**: lógica de validação de convênio e dados obrigatórios antes de confirmar agendamento.
- **Banco**: tabela `pacientes` com campos de convênio e documentos.

### RF03 – Histórico de consultas e exames
- **Java**: método `listarHistorico(int idPaciente)` que retorna lista de objetos `Consulta`.
- **Banco**: consultas SQL com `JOIN` entre tabelas `pacientes`, `consultas`, `exames`.

### RF04 – Ferramentas de acessibilidade
- **Front-end**: botões para aumentar fonte e alto contraste (CSS e JavaScript).
- **Back-end**: armazenar preferências do usuário na tabela `pacientes` (ex: `fonte_tamanho`, `contraste`).

### RF05 – Validação de agendamentos pelo funcionário
- **Java**: criar `Funcionario` com método `aprovarAgendamento(int idConsulta)` que altera status de “pendente” para “confirmado”.
- **Banco**: atualização do campo `status` na tabela `consultas`.

### RNF01 – Usabilidade para idosos
- **Front-end**: tipografia mínima de 16px, contraste mínimo 4.5:1, alvos de toque amplos.
- **Testes**: conduzir testes com usuários idosos (conforme item 3.4 do relatório).

### RNF02 – Suporte offline
- **Arquitetura**: cache local no navegador (LocalStorage/IndexedDB) para ações críticas (ex: consultas do dia) e sincronização quando houver conexão.
- **Java**: API com lógica de sincronização e detecção de conflitos.

### RNF03 – Interoperabilidade com sistemas legados
- **Back-end**: expor APIs REST (ou serviços) que possam ser consumidas por sistemas HIS, usando padrões como JSON ou XML.

### RNF04 – Segurança
- **Java**: autenticação com diferentes perfis (paciente, funcionário); uso de sessões ou tokens.
- **Banco**: senhas armazenadas com hash (ex: BCrypt).

### RNF05 – Confiabilidade (evitar duplicidade)
- **Java**: validações antes de inserir agendamento: checar se já existe consulta no mesmo horário para o mesmo médico e paciente.
- **Banco**: constraints de unicidade (ex: unique key composta por `id_medico`, `data_hora`).

---

## Estruturação Técnica: Java e Banco de Dados

### Classes Java (exemplos)

```java
// Paciente.java
public class Paciente {
    private int id;
    private String nome;
    private String cpf;
    private String rg;
    private Date dataNascimento;
    private String email;
    private String convenio;
    // getters/setters
}

// Medico.java
public class Medico {
    private int id;
    private String nome;
    private String email;
    private String especialidade;
}

// Consulta.java
public class Consulta {
    private int id;
    private Paciente paciente;
    private Medico medico;
    private LocalDateTime dataHora;
    private String status; // "PENDENTE", "CONFIRMADA", "CANCELADA"
}
