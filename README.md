Caça aos Erros —
Sistema Escolar de Cadastro

O objetivo foi identificar e corrigir erros intencionais em uma aplicação de gerenciamento escolar, garantindo o funcionamento correto dos endpoints de alunos, professores e matrículas.

Autor: Luccas Bentim


Tecnologias utilizadas:

Java 17,
Spring Boot,
Spring Web MVC,
Spring Data JPA + Hibernate,
Banco de dados H2 (em memória),
Maven,
Lombok,
Postman (para testes manuais).


Erros identificados e corrigidos:

1. Falta de transação no cadastro de professor
Arquivo: ProfessorController.java

Problema: O método de cadastro não estava dentro de uma transação.

Código incorreto:

@PostMapping
public void cadastrar(@RequestBody DadosCadastroProfessor dados) {
    repository.save(new Professor(dados));
}

Correção:

@PostMapping
@Transactional
public void cadastrar(@RequestBody DadosCadastroProfessor dados) {
    repository.save(new Professor(dados));
}

Motivo: A anotação @Transactional garante que a operação de persistência seja executada corretamente dentro de uma transação.


2. Entidade Aluno associada à tabela errada
Arquivo: Aluno.java

Problema: A entidade estava vinculada à tabela de professores.

Código incorreto:

@Table(name = "professores")

Correção:

@Table(name = "alunos")

Motivo: Cada entidade deve estar associada à sua própria tabela, evitando conflitos entre dados de alunos e professores.


3. Tipo incorreto no repositório de alunos
Arquivo: AlunoRepository.java

Problema: O repositório usava String como tipo de ID.

Código incorreto:

public interface AlunoRepository extends JpaRepository<Aluno, String> {
}

Correção:

public interface AlunoRepository extends JpaRepository<Aluno, Integer> {
}

Motivo: O campo id da entidade Aluno é do tipo Integer. O repositório deve usar o mesmo tipo.


4. Nome e e-mail invertidos na listagem de alunos
Arquivo: DadosListagemAluno.java

Problema: O DTO retornava nome e e-mail trocados.

Código incorreto:

this(
    aluno.getId(),
    aluno.getEmail(),
    aluno.getNome(),
    aluno.getRa(),
    aluno.getCurso()
);

Correção:

this(
    aluno.getId(),
    aluno.getNome(),
    aluno.getEmail(),
    aluno.getRa(),
    aluno.getCurso()
);

Motivo: O endpoint retornava o e-mail no campo nome e o nome no campo email.


5. Método HTTP incorreto na atualização de aluno
Arquivo: AlunoController.java

Problema: O método de atualização usava @PostMapping.

Código incorreto:

@PostMapping
@Transactional
public void atualizar(@RequestBody DadosAtualizacaoAluno dados) {
    var aluno = repository.getReferenceById(dados.id());
    aluno.atualizarInformacoes(dados);
}

Correção:

@PutMapping
@Transactional
public void atualizar(@RequestBody DadosAtualizacaoAluno dados) {
    var aluno = repository.getReferenceById(dados.id());
    aluno.atualizarInformacoes(dados);
}

Motivo: O cadastro deve usar POST, enquanto a atualização deve usar PUT.


6. Tipo incorreto no ID de exclusão de aluno
Arquivo: AlunoController.java  
Problema: O parâmetro do método de exclusão estava definido como String.

Código incorreto:

@DeleteMapping("/{id}")
@Transactional
public void excluir(@PathVariable String id) {
    repository.deleteById(id);
}

Correção:

@DeleteMapping("/{id}")
@Transactional
public void excluir(@PathVariable Integer id) {
    repository.deleteById(id);
}

Motivo: O repositório de alunos utiliza identificadores do tipo Integer.


7. Campo incorreto na atualização do professor
Arquivo: Professor.java  
Problema: O e-mail estava sendo atribuído ao campo nome.

Código incorreto:

if (dados.email() != null) {
    this.nome = dados.email();
}

Correção:

if (dados.email() != null) {
    this.email = dados.email();
}

Motivo: O erro alterava o nome do professor e mantinha o e-mail antigo.


8. Problemas no controller de matrículas
Arquivo: MatriculaController.java

Conversão incorreta do ID do aluno  
Código incorreto:

Aluno aluno = alunoRepository.getReferenceById(
    dados.alunoId().toString()
);

Correção:

Aluno aluno = alunoRepository.getReferenceById(
    dados.alunoId()
);

Motivo: O AlunoRepository utiliza identificadores do tipo Integer.

Nome incorreto no PathVariable  
Código incorreto:

@DeleteMapping("/{id}")
@Transactional
public void excluir(@PathVariable Integer ids) {
    repository.deleteById(ids);
}

Correção:

@DeleteMapping("/{id}")
@Transactional
public void excluir(@PathVariable Integer id) {
    repository.deleteById(id);
}

Motivo: O nome do parâmetro deve corresponder ao valor informado na rota {id}.
