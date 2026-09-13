# Copa das Equipes – Great Slam 2026

Aplicativo de gestão da Copa das Equipes, competição por equipes entre
academias de tênis de Fortaleza, dentro do circuito GREAT SLAM.

**App publicado:** https://great-slam.github.io/Copa-das-Equipes/

## O que o app faz

- Ranking geral das academias em tempo real
- Resultados das partidas em card pronto para compartilhar
- Grupos por categoria
- Calendário e regulamento da competição
- Contatos da organização

Atletas e visitantes acessam em modo leitura. O organizador entra em modo
admin para lançar resultados, que aparecem para todos automaticamente.

## Estrutura do repositório

```
index.html          app publicado (arquivo único)
og-image.jpg        preview social do link
RESUMO.md           documentação técnica completa
fontes/             arquivos-fonte para manutenção
backup/             fotos do estado (o Firestore é a fonte de verdade)
```

## Manutenção

Leia o `RESUMO.md` antes de qualquer alteração. Ele documenta a estrutura
dos dados, as regras de pontuação, o funcionamento da sincronização com o
Firebase e os problemas já resolvidos.

---

Desenvolvido para o GREAT SLAM · Fortaleza/CE · 2026
