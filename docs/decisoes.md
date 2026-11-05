# Decisões do projeto

## Pergunta de decisão
Quais escolas entregam mais aprendizado do que o contexto socioeconômico delas prevê, e o que as distingue?

## Públicos e decisões apoiadas
1. Gestor de secretaria estadual: destinação de recursos, mapeamento de profissionais, acompanhamento de alunos
2. Analista de política pública: como replicar resultados; como adequar diferentes contextos; parcerias público-privadas, etc

## Fluxo de Git
- `main`: estável, recebe só PR de fim de marco
- `dev`: ligada ao workspace de desenvolvimento no Fabric
- `feat/*`, `docs/*`: branches curtas, PR para `dev`
- Commits no padrão conventional commits (feat, fix, docs, test, chore)

## Registro de decisões técnicas
| Data | Decisão | Por quê |
|---|---|---|
| 2026-11-04 | Repo público desde o início | Dados públicos; proteção de branch grátis só em repo público |