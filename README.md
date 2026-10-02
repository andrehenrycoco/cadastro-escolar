# Sistema de Controle Escolar em COBOL

Sistema batch de informatização escolar escrito em COBOL. Mantém os cadastros de alunos, professores, matérias e séries, relaciona essas entidades entre si, lança notas, calcula médias e emite relatórios de aprovação e reprovação.

Projeto final do Curso de Formação de Programadores COBOL da Aprenda COBOL, desenvolvido a partir de uma especificação funcional fechada, com complexidade classificada como "C" pelo próprio documento.

## Status

Planejamento concluído, implementação em andamento. O desenho de arquivos, a divisão em módulos e as regras de negócio descritos abaixo são as decisões que guiam o desenvolvimento.

## Escopo

A especificação define quatro cadastros, três relacionamentos, o processamento de notas e dois relatórios.

**Cadastros.** Alunos, professores, matérias e séries, cadastrados individualmente. Cada um oferece as cinco operações previstas: inclusão, alteração, consulta, listagem e exclusão.

**Relacionamentos.** Aluno vinculado a série, série vinculada a matéria e matéria vinculada a professor.

**Notas.** Lançamento por aluno e matéria, cálculo da média e definição do status de aprovação.

**Relatórios.** Um de aprovados e outro de reprovados, ambos agrupados por matéria e ordenados pelo nome do aluno.

## Modelo de dados

A especificação deixa os layouts a cargo do programador. A escolha aqui foi arquivo indexado para os cadastros, que é o que permite consulta direta por chave sem varrer o arquivo inteiro, e sequencial para a saída dos relatórios.

| Arquivo | Organização | Chave | Conteúdo |
|---|---|---|---|
| ALUNOS | Indexado | Matrícula | Nome, data de nascimento, código da série, situação |
| PROFESSORES | Indexado | Código | Nome, titulação, situação |
| MATERIAS | Indexado | Código | Nome, carga horária, código do professor, situação |
| SERIES | Indexado | Código | Descrição, turno, situação |
| SERIEMAT | Indexado | Código da série e código da matéria | Vínculo entre série e matéria |
| NOTAS | Indexado | Matrícula e código da matéria | Quatro notas bimestrais, média e status |
| RELAPROV | Sequencial | Saída | Relatório de aprovados |
| RELREPRO | Sequencial | Saída | Relatório de reprovados |

O vínculo entre aluno e série fica na própria ficha do aluno, assim como o vínculo entre matéria e professor fica na ficha da matéria, porque ambos são relações de um para muitos. Série e matéria é relação de muitos para muitos, e por isso ganha arquivo próprio.

## Divisão em módulos

A especificação exige a nomenclatura `CFPPnnnD`, onde o bloco numérico corresponde ao código de usuário atribuído pelo curso. A divisão prevista é um programa principal que controla o menu e chama os demais via `CALL`.

| Módulo | Responsabilidade |
|---|---|
| CFPP101D | Menu principal e controle de fluxo |
| CFPP102D | Cadastro de alunos |
| CFPP103D | Cadastro de professores |
| CFPP104D | Cadastro de matérias |
| CFPP105D | Cadastro de séries |
| CFPP106D | Vínculo entre série e matéria |
| CFPP107D | Lançamento de notas e cálculo de média |
| CFPP108D | Relatório de alunos aprovados |
| CFPP109D | Relatório de alunos reprovados |

Os layouts ficam em copybooks separados, um por arquivo, para que a alteração de um campo não exija editar todos os programas que usam aquele registro.

## Regras de negócio

A especificação diz para criar as regras conforme o entendimento do programador. Estas são as adotadas.

**RN01.** A média de aprovação é 6,0. Igual ou acima aprova, abaixo reprova.

**RN02.** São quatro notas bimestrais por aluno e matéria, de 0 a 10. A média é aritmética simples.

**RN03.** Aluno com menos de quatro notas lançadas fica com status pendente e não entra em nenhum dos dois relatórios, porque a média ainda não é definitiva.

**RN04.** Inclusão com chave já existente é rejeitada com mensagem, sem sobrescrever o registro.

**RN05.** A exclusão é lógica, por campo de situação. O registro sai das listagens mas o histórico de notas continua íntegro.

**RN06.** Não se exclui série com aluno vinculado, nem matéria com nota lançada. A tentativa retorna o motivo da recusa.

**RN07.** Os relatórios são ordenados por matéria e, dentro de cada matéria, por nome do aluno em ordem alfabética.

## Estrutura de pastas

```
cobol-controle-escolar/
├── src/
│   ├── CFPP101D.cbl
│   ├── CFPP102D.cbl
│   └── ...
├── copybooks/
│   ├── CPALUNO.cpy
│   ├── CPPROF.cpy
│   ├── CPMATER.cpy
│   ├── CPSERIE.cpy
│   ├── CPSERMAT.cpy
│   └── CPNOTAS.cpy
├── data/
│   └── arquivos gerados em tempo de execução
├── docs/
│   └── CFP-ProjetoFinal-Especificacao.pdf
└── README.md
```

## Tecnologias

- COBOL, formato fixo
- GnuCOBOL como compilador
- Processamento batch
- Arquivos indexados e sequenciais, sem banco de dados externo

## Como rodar

Você precisa do [GnuCOBOL](https://gnucobol.sourceforge.io/) instalado.

```bash
git clone https://github.com/andrehenrycoco/cobol-controle-escolar.git
cd cobol-controle-escolar

# compila os módulos chamados
cobc -m -I copybooks src/CFPP10[2-9]D.cbl

# compila o programa principal
cobc -x -I copybooks src/CFPP101D.cbl -o controle-escolar

./controle-escolar
```

## Roadmap

**Fase 1.** Copybooks dos seis arquivos, programa de menu e cadastro de séries completo, servindo de modelo para os demais.

**Fase 2.** Cadastros de professores, matérias e alunos, reaproveitando a estrutura validada na fase anterior.

**Fase 3.** Vínculo entre série e matéria, e as validações de integridade da RN06.

**Fase 4.** Lançamento de notas, cálculo de média e definição do status de aprovação.

**Fase 5.** Os dois relatórios, com a ordenação da RN07.

**Fase 6.** Massa de dados de teste cobrindo aprovado, reprovado e pendente, para validar os relatórios nos três cenários.

## Especificação

O documento funcional que originou o projeto está em `docs/`. Versão 1.0, de 01/04/2021, emitida pela equipe Aprenda COBOL.

## Autor

André Henry Barboza Coco

Em transição da psicologia para desenvolvimento back-end. Curso Análise e Desenvolvimento de Sistemas na Universidade Cruzeiro do Sul, com conclusão prevista para 2027.

[LinkedIn](https://www.linkedin.com/in/andrecoco/) · andrehbcoco@gmail.com
