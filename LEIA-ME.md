# Backups do estado

Os arquivos desta pasta são **fotos de um momento**, não a fonte de verdade.

A fonte de verdade dos resultados é o **Firestore**:
projeto `copa-das-equipes---great-slam`, coleção `copa`, documento `estado`.

Assim que um resultado é lançado pelo app em modo admin, o Firestore é
atualizado e o JSON desta pasta fica desatualizado.

Antes de usar um destes arquivos para reconstruir o estado, confira o
Firestore ou o próprio app. Já houve um caso de dado antigo ser
reintroduzido por confiar no backup local.
