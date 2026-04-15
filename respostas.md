1- A escolha de um SGBD relacional como PostgreSQL em vez de um modelo NoSQL se justifica principalmente quando o cenário exige **forte consistência, integridade dos dados e confiabilidade nas transações**.

Nos bancos relacionais, as propriedades **ACID** são garantidas:

* **Atomicidade**: cada transação é executada por completo ou não é executada. Isso evita estados inconsistentes (por exemplo, em uma transferência bancária, o valor não pode ser debitado sem ser creditado).
* **Consistência**: o banco sempre respeita regras definidas (constraints, chaves estrangeiras, etc.), garantindo que os dados permaneçam válidos antes e depois de cada operação.
* **Isolamento**: múltiplas transações simultâneas não interferem negativamente entre si, evitando problemas como leituras sujas ou inconsistentes.
* **Durabilidade**: uma vez confirmada, a transação não se perde, mesmo em casos de falha do sistema.

Além disso, SGBDs relacionais oferecem mecanismos robustos de **integridade referencial**, como:

* chaves primárias e estrangeiras
* restrições (constraints)
* normalização de dados

Esses recursos são essenciais em cenários onde os dados têm **relacionamentos complexos e regras rígidas**, como sistemas financeiros, ERPs ou sistemas acadêmicos.

Por outro lado, bancos NoSQL (como MongoDB) geralmente priorizam **escalabilidade e desempenho**, adotando modelos como BASE (Basically Available, Soft state, Eventually consistent). Isso implica que:

* a consistência pode ser eventual (não imediata)
* não há garantias tão fortes de integridade
* o controle de relacionamentos fica mais dependente da aplicação

Portanto, quando o cenário exige:

* transações seguras
* consistência imediata
* forte integridade dos dados


2- Em um ambiente profissional de Engenharia de Dados, usar **schemas organizados** (como `academico`, `seguranca`, `financeiro`) em um SGBD relacional como PostgreSQL é uma prática recomendada por motivos que vão além da simples organização — envolve **governança, segurança e escalabilidade do projeto**.

Primeiro, há a **organização lógica**. Separar tabelas por domínio facilita entender o banco: tudo que é do contexto acadêmico fica em `academico`, tudo que é autenticação e controle de acesso fica em `seguranca`. Em vez de um único “amontoado” no schema `public`, você tem uma estrutura clara, parecida com pacotes em código.

Depois vem a **segurança e controle de acesso**. Schemas permitem definir permissões específicas:

* um usuário pode ter acesso apenas ao schema `academico`
* outro pode acessar `seguranca`, mas sem permissão de escrita
  Isso reduz riscos e segue o princípio do menor privilégio, algo essencial em ambientes corporativos.

Outro ponto importante é evitar **conflitos de nomes**. Em projetos grandes, diferentes equipes podem precisar de tabelas com nomes iguais (`usuarios`, `logs`, etc.). Com schemas, isso não é um problema:

* `academico.usuarios`
* `seguranca.usuarios`
  Sem schemas, isso geraria colisões ou nomes artificiais.

Também há ganhos em **manutenção e versionamento**. Fica mais fácil:

* migrar ou versionar apenas um domínio específico
* aplicar scripts de alteração de forma segmentada
* isolar mudanças sem impactar todo o banco

Além disso, schemas ajudam na **integração com ferramentas e pipelines de dados**, permitindo separar ambientes lógicos (ex: `staging`, `raw`, `trusted`) dentro do mesmo banco, algo comum em engenharia de dados.

Por fim, usar apenas o schema `public` tende a:

* virar um “depósito desorganizado” com o tempo
* dificultar auditoria e governança
* aumentar o risco de erros humanos (como acessar ou alterar tabelas erradas)


3- Análise do Cenário: Concorrência na Atualização de Notas

Quando dois operadores da secretaria tentam alterar a nota do mesmo `ID_Matricula` ao mesmo tempo, ocorre uma situação clássica de **concorrência** em bancos de dados. Sem mecanismos adequados, isso poderia resultar em inconsistências, como perda de atualização (lost update) ou dados corrompidos.

# Papel do Isolamento (ACID)

A propriedade de **Isolamento** garante que transações concorrentes sejam executadas de forma segura, como se estivessem ocorrendo de maneira sequencial. Dependendo do nível de isolamento configurado (como *Read Committed*, *Repeatable Read* ou *Serializable*), o SGBD controla como e quando uma transação pode enxergar ou interferir nos dados de outra.

No caso descrito:

* Cada operador inicia uma transação para atualizar a nota.
* O SGBD assegura que uma transação não veja estados intermediários da outra.
* Em níveis mais altos de isolamento, evita-se que ambas leiam o mesmo valor inicial e sobrescrevam alterações, prevenindo inconsistências.

# Uso de Locks (Bloqueios)

Para garantir esse isolamento na prática, o SGBD utiliza **locks (bloqueios)**:

* Quando o primeiro operador inicia a atualização, o banco aplica um **lock exclusivo (write lock)** sobre o registro do `ID_Matricula`.
* Enquanto esse lock estiver ativo, o segundo operador:

  * fica bloqueado até a liberação do lock, ou
  * recebe um erro, dependendo da configuração e do tempo de espera (timeout).

Isso garante que:

1. Apenas uma transação modifique o dado por vez.
2. A segunda transação só execute após a conclusão da primeira (commit ou rollback).

# Resultado Final

Graças ao isolamento e aos locks:

* Não ocorre sobrescrita indevida de dados.
* O valor final da nota será consistente, refletindo uma ordem lógica de execução (como se uma operação tivesse ocorrido depois da outra).
* O banco evita estados inválidos ou corrupção de dados.

# Conclusão

A combinação de **Isolamento (ACID)** e (mecanismos de bloqueio) assegura que, mesmo com múltiplos acessos simultâneos, o banco de dados mantenha a integridade e consistência das informações. Esse controle é essencial em ambientes concorrentes, como sistemas acadêmicos, financeiros ou corporativos.

































































