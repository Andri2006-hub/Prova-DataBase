# Prova-DataBaseCREATE SCHEMA academico;
CREATE SCHEMA seguranca;
CREATE SCHEMA academico;
CREATE SCHEMA seguranca;
DELETE FROM academico.aluno;
UPDATE academico.aluno
SET ativo = FALSE
WHERE id_matricula = 1;
CREATE ROLE professor_role;
CREATE ROLE coordenador_role;
GRANT ALL PRIVILEGES ON SCHEMA academico TO coordenador_role;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA academico TO coordenador_role;
GRANT SELECT ON academico.aluno TO professor_role;
GRANT SELECT ON academico.disciplina TO professor_role;
GRANT SELECT ON academico.docente TO professor_role;
GRANT SELECT ON academico.matricula TO professor_role;
REVOKE SELECT (email) ON academico.aluno FROM professor_role;
GRANT UPDATE (nota) ON academico.matricula TO professor_role;
GRANT UPDATE (nota) ON academico.matricula TO professor_role;
SELECT a.nome, d.nome AS disciplina, m.ciclo
FROM academico.matricula m
JOIN academico.aluno a ON a.id_matricula = m.id_matricula
JOIN academico.disciplina d ON d.cod_disciplina = m.cod_disciplina
WHERE m.ciclo = '2026/1';
SELECT d.nome, AVG(m.nota) AS media
FROM academico.matricula m
JOIN academico.disciplina d ON d.cod_disciplina = m.cod_disciplina
GROUP BY d.nome
HAVING AVG(m.nota) < 6;
SELECT doc.nome, d.nome AS disciplina
FROM academico.docente doc
LEFT JOIN academico.matricula m ON doc.id_docente = m.id_docente
LEFT JOIN academico.disciplina d ON d.cod_disciplina = m.cod_disciplina;
SELECT a.nome, m.nota
FROM academico.matricula m
JOIN academico.aluno a ON a.id_matricula = m.id_matricula
JOIN academico.disciplina d ON d.cod_disciplina = m.cod_disciplina
WHERE d.nome = 'Banco de Dados'
AND m.nota = (
    SELECT MAX(nota)
    FROM academico.matricula m2
    JOIN academico.disciplina d2 ON d2.cod_disciplina = m2.cod_disciplina
    WHERE d2.nome = 'Banco de Dados'
);
