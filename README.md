# okrs-sharepoint

Páginas do SharePoint do Grupo DBVO (GitHub Pages, embutidas por iframe).

- `index.html` — painel de OKRs e páginas de empresa, time e empreendimento (`?pagina=`, `?empresa=`, `?time=`, `?empreendimento=`).
- `estudio/` — **Estúdio**: site com login para editar o conteúdo das páginas sem mexer em código. https://pdedbvo.github.io/okrs-sharepoint/estudio/ (`?demo` = testar sem gravar no banco).
- `fotos/` — fotos dos empreendimentos.

## De onde vem o conteúdo
O conteúdo editável fica no Firestore do projeto Firebase **sharepoint-studio** (coleção `conteudo`, leitura pública).
As páginas leem de lá (cópia no navegador por 10 min) e, se o banco falhar ou estiver vazio, usam o `CONFIG` do próprio `index.html`.
`?estudio=nao` no link ignora o Estúdio (para testes). Detalhes e regras de segurança: `estudio/ARQUITETURA-ESTUDIO.md`.
